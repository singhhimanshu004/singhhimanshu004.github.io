---
title: "Bits, Not Keys — Building a Bloom Filter Spell-Checker"
description: "A from-scratch Bloom filter in Go: the sizing math, the Kirsch–Mitzenmacher trick that fakes k hashes from two, why it never gives a false negative, and a self-describing file format so you build the dictionary once."
pubDate: 2026-06-10
tags:
  - "Data Structures"
  - "Probabilistic"
  - "Go"
  - "Algorithms"
heroImage: "/blog/bloom-filter.svg"
heroAlt: "A Bloom filter: the key 'cat' is run through three hash functions that set three bits in a bit array; a lookup with any unset bit is a definite miss."
heroCaption: "A Bloom filter stores only bits. k hashes set k bits per key; any unset bit on lookup is a guaranteed 'not present'."
draft: false
---

How do you check a word against a 100,000-word dictionary using almost no memory, in constant time, and accept that you'll *occasionally* be wrong on purpose?

That sentence is a Bloom filter, and it's one of the most elegant data structures I know. I built one in Go as a spell-checker — the classic application — for my [Coding Challenges Deep Dive](https://github.com/singhhimanshu004/coding-challenges-deepdive). It's small, but it forces you to confront three genuinely interesting ideas: probabilistic correctness, hashing economics, and bit-level storage. Here's the whole thing.

## The one-sided guarantee

A Bloom filter answers "have I seen this key?" with two possible truths:

- **"Definitely not."** If it says no, the key was *never* added. No false negatives, ever.
- **"Probably yes."** If it says yes, the key is *probably* present — but it might be a false positive, at a rate you choose up front.

That asymmetry is the whole point, and it's perfect for a spell-checker: a word the filter rejects is *definitely* misspelled; a word it accepts is *probably* fine. You trade a small, tunable false-positive rate for indexing the entire dictionary in a fraction of the memory a hash set would need — because **the filter never stores the words themselves**. It stores bits.

```go
// Contains reports whether a key is *probably* present. A false result means
// "definitely not present" (never a false negative).
func (f *Filter) Contains(key []byte) bool {
	for _, idx := range hashes(key, f.k, f.m) {
		if !f.bits.Test(idx) {
			return false   // one unset bit ⇒ definitely never added
		}
	}
	return true
}
```

## Sizing it: how many bits, how many hashes?

A filter has two parameters: `m`, the number of bits in the array, and `k`, the number of hash functions. You don't guess them — you derive them from how many items you expect (`n`) and the false-positive rate you'll tolerate (`p`). These are the classic Bloom results:

```go
m = -(n * ln p) / (ln 2)^2      // optimal number of bits
k = (m / n) * ln 2              // optimal number of hash functions
```

The intuition is worth internalizing:

- **Smaller `p` demands more bits.** Accuracy costs space — `m` grows as `p` shrinks. There's no free lunch.
- **`k` is tuned to keep the array about half full at capacity.** That's where the false-positive probability bottoms out. Too few hashes and you underuse the array; too many and you flood it so every lookup collides on already-set bits.

For a 100k-word dictionary at a 1% false-positive rate, that works out to roughly 1.2 MB of bits and 7 hash functions — versus storing 100k actual strings.

## The trick: k hashes for the price of two

Here's the part that made me grin. A Bloom filter needs `k` *independent* hash functions. Writing and running seven different hash algorithms would be wasteful. You don't have to.

The **Kirsch–Mitzenmacher** technique proves you can synthesize as many hashes as you want from just two:

```
g_i(x) = h1(x) + i*h2(x)   for i = 0, 1, ..., k-1
```

For Bloom-filter purposes, that family behaves statistically like `k` independent hashes — with no measurable increase in false-positive rate. So I compute *one* 64-bit FNV-1a digest and split it into two 32-bit halves:

```go
func baseHashes(data []byte) (h1, h2 uint64) {
	h := fnv.New64a()
	h.Write(data)
	sum := h.Sum64()

	h1 = sum >> 32          // high 32 bits
	h2 = sum & 0xffffffff   // low 32 bits
	// If h2 were 0, every derived hash would collapse to h1 (i*0 == 0) and the
	// filter would effectively use a single bit position. Force it non-zero.
	if h2 == 0 {
		h2 = 1
	}
	return h1, h2
}

func hashes(data []byte, k, m uint64) []uint64 {
	h1, h2 := baseHashes(data)
	out := make([]uint64, k)
	for i := uint64(0); i < k; i++ {
		out[i] = (h1 + i*h2) % m   // g_i(x), folded into [0, m)
	}
	return out
}
```

Two details that are easy to get wrong: the `h2 == 0` guard (without it, every derived index collapses to the same bit), and the choice of FNV-1a — it's tiny, dependency-free, fast on short strings, and disperses bits well. It is *not* cryptographic, and that's fine: we need good distribution, not adversarial collision resistance.

## Storing bits when the smallest unit is a byte

Memory frugality is the *entire point* of a Bloom filter, so I can't waste a byte per bit. I pack them: bit `i` lives in byte `i/8`, at offset `i%8`.

```go
func (b *BitSet) Set(i uint64) {
	i %= b.n
	b.bits[i/8] |= 1 << (i % 8)
}

func (b *BitSet) Test(i uint64) bool {
	i %= b.n
	return b.bits[i/8]&(1<<(i%8)) != 0
}
```

Using one byte per bit instead would waste 8× the memory — and defeat the purpose. To estimate the *realised* false-positive rate later, I also need a population count (how many bits are actually set), which is the classic Brian Kernighan loop — `x &= x-1` clears the lowest set bit each pass:

```go
func popcount(x byte) int {
	count := 0
	for x != 0 {
		x &= x - 1
		count++
	}
	return count
}
```

With that, the filter can report its *current* false-positive probability from how full it actually is — `p ≈ (fraction of bits set)^k` — which is a nice sanity check against the target you configured.

## Build once, query forever

Hashing a 100k-word dictionary isn't free. You don't want to pay that cost on every spell-check run, so the filter serializes to a small, self-describing binary file:

```
offset  size  field
------  ----  ---------------------------------------------
0       4     magic   "BLM1"   — format + version
4       1     version (1)
5       8     m       uint64   — number of bits
13      8     k       uint64   — number of hash functions
21      8     nbytes  uint64   — length of the packed payload
29      nbytes         — the raw bit array
```

Storing `m` and `k` in the header isn't optional — it's a correctness requirement. The bit indices a key maps to depend *entirely* on `m` and `k`. A reader that guessed different values would compute different indices and every lookup would be wrong. Saving them makes the file self-describing: loading needs nothing but the file itself, and a `"BLM1"` magic guards against feeding it garbage.

## Why this one's worth building

The Bloom filter is a master class in engineering trade-offs done deliberately: accuracy *for* space, one-sided error *for* constant-time lookups, two real hashes *for* k synthetic ones. None of it is accidental — every parameter falls out of a formula you can derive and defend. Build it once and you'll start spotting where it fits in the real world: databases skipping disk reads for absent keys, CDNs avoiding one-hit-wonder caching, crawlers deduplicating URLs at scale.

Full source, tests, and a teaching README: [bloom-filter-spell-checker](https://github.com/singhhimanshu004/coding-challenges-deepdive/tree/master/phase-01-foundations/bloom-filter-spell-checker).

**Further reading:** Burton Bloom's 1970 paper *Space/Time Trade-offs in Hash Coding with Allowable Errors*; Kirsch & Mitzenmacher, *Less Hashing, Same Performance* (2006), for the two-hash proof.
