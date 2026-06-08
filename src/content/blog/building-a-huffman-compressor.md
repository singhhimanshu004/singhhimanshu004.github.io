---
title: "Short Codes for Common Bytes — Building a Huffman Compressor That Round-Trips"
description: "A from-scratch Huffman compressor in Go: a min-heap that builds the tree, prefix codes that need no separators, MSB-first bit I/O, and the one-line tie-break bug that quietly breaks decoding."
pubDate: 2026-06-09
tags:
  - "Compression"
  - "Algorithms"
  - "Go"
  - "Systems Programming"
draft: false
---

Compression feels like magic until you write one. Then it becomes a very concrete question: how do you store the letter `e` in fewer than eight bits, store a rare `~` in more, and still know where one symbol ends and the next begins — with no commas, no lengths, nothing but a stream of bits?

That question is the whole of Huffman coding, and it is the first real tool in my [Coding Challenges Deep Dive](https://github.com/singhhimanshu004/coding-challenges-deepdive). I built it in Go because the challenge is fundamentally about bytes and I/O, and Go's `io.Reader`/`io.Writer` model keeps that honest. This post walks the design end to end — including the one-line bug that compiles, passes a casual test, and then silently corrupts your data.

## The core idea, stated precisely

Assign **short** bit-codes to **frequent** symbols and **long** codes to rare ones. The catch is decoding: if `e` is `01` and another symbol is `011`, a decoder reading `011` can't tell whether it just saw `e` followed by something, or the longer code.

Huffman's answer is a **prefix code** — no code is a prefix of any other. Build the codes by walking a binary tree where every symbol is a *leaf*: the path from the root (left = `0`, right = `1`) *is* the code. Because symbols only ever sit at leaves, no code can be a prefix of another, and the bit-stream decodes unambiguously. The greedy "always merge the two least-frequent nodes" construction is provably optimal for symbol-by-symbol coding.

The pipeline is four stages:

```
bytes ──▶ count frequencies ──▶ build tree (min-heap) ──▶ derive codes ──▶ pack bits
```

## Building the tree with a min-heap

We repeatedly need the two lowest-frequency nodes. That is exactly what a min-heap is for: `O(log n)` extract-min and insert, so the whole build is `O(n log n)` over the distinct symbols instead of re-scanning a list on every merge.

```go
func buildTree(freqs map[byte]uint64) *node {
	if len(freqs) == 0 {
		return nil
	}
	pq := &priorityQueue{}
	for sym, f := range freqs {
		pq.items = append(pq.items, &node{symbol: sym, freq: f, leaf: true, order: int(sym)})
	}
	order := 256
	initHeap(pq)

	for pq.Len() > 1 {
		a := popNode(pq)
		b := popNode(pq)
		parent := &node{freq: a.freq + b.freq, left: a, right: b, order: order}
		order++
		pushNode(pq, parent)
	}
	return popNode(pq)
}
```

Then a recursive walk turns the tree into a `symbol → bit-string` table, accumulating `0` for every left edge and `1` for every right edge:

```go
walk = func(n *node, prefix string) {
	if n.leaf {
		codes[n.symbol] = prefix
		return
	}
	walk(n.left, prefix+"0")
	walk(n.right, prefix+"1")
}
```

## The bug that doesn't look like a bug

Here is the subtle part, and it is the reason this challenge is worth doing slowly.

To **decompress**, I don't store the codes in the file. I store the *frequency table* and rebuild the identical tree on the way out. That only works if encode and decode build the **exact same tree**. And there is a trap: when two nodes have equal frequency, which one comes out of the heap first?

In Go, `for sym, f := range freqs` iterates a map in **randomized** order. Heapify two equal-frequency nodes that arrived in different orders, and you get two *different* — but individually valid — trees. Encode with one, decode with the other, and the round-trip fails. The compiler is happy. A test on `"hello"` might even pass. Then a real file comes through and the bytes come out scrambled.

The fix is a deterministic tie-break. Every node gets a stable `order` id (leaves keyed on their byte value `0..255`, internal nodes numbered from 256 in creation order), and the heap breaks frequency ties on it:

```go
func (pq *priorityQueue) Less(i, j int) bool {
	a, b := pq.items[i], pq.items[j]
	if a.freq != b.freq {
		return a.freq < b.freq
	}
	// Deterministic tie-break: lower insertion order wins, so encode and
	// decode build an identical tree and the round-trip is exact.
	return a.order < b.order
}
```

> The lesson generalizes well beyond compression: any time two sides must independently reconstruct the same structure, *non-determinism is a correctness bug*, not a style preference. Map iteration order, floating-point summation order, goroutine scheduling — they all bite here.

## Packing bits into bytes

Codes are variable-length runs of bits, but the OS only deals in whole bytes. So you need a small adapter that accumulates bits and flushes a byte once eight are queued. I write **most-significant-bit first**, so the first bit written becomes the top of the byte — which mirrors how the reader walks the tree left to right.

```go
func (bw *BitWriter) WriteBit(bit uint) error {
	bw.cur <<= 1          // make room in the next-lower slot
	if bit != 0 {
		bw.cur |= 1
	}
	bw.nbits++
	if bw.nbits == 8 {     // a full byte is ready
		if err := bw.w.WriteByte(bw.cur); err != nil {
			return err
		}
		bw.cur, bw.nbits = 0, 0
	}
	return nil
}
```

The last byte is almost never exactly full, so `Flush` left-justifies the leftover bits and zero-pads the rest. That padding is *harmless* only because of a deliberate header choice (next section): the decoder knows the exact symbol count and stops before it can mistake padding for data.

This "pack bits into bytes" routine is not throwaway code — the same skill reappears in `tar`, `xxd`, and binary network protocols like DNS, which is exactly why it's worth building once, carefully, by hand.

## A self-describing file format

A compressed file nobody can decode is just expensive noise. So the output is a small container with a header:

```
+---------+--------------+-------------+-----------------------------+-------------+
| "HUF1"  | totalSymbols | numDistinct | table: [sym][uvarint freq]  | packed bits |
| 4 bytes |   uint64     |   uint16    |    numDistinct entries      |    body     |
+---------+--------------+-------------+-----------------------------+-------------+
```

Two decisions worth calling out:

- **`totalSymbols` is what makes zero-padding safe.** Decoding walks the tree bit by bit and stops the instant it has emitted exactly that many bytes — so the trailing pad bits are never decoded.
- **I store frequencies, not codes.** The more compact alternative is *canonical Huffman*: store only code *lengths* (one byte per symbol) and reconstruct codes by a fixed rule. I chose the frequency table because it's the most direct, easiest-to-verify teaching artifact, and the few-hundred-byte overhead is negligible on any non-trivial input. That trade-off is a conscious one, documented in the README rather than hidden.

A magic string `"HUF1"` guards the front. If I change the layout later, I bump the version so old files are *rejected* instead of silently misread — and `Decompress` returns a typed `ErrBadFormat` the moment the magic doesn't match.

## Two edge cases that break naive code

- **Empty input.** Zero symbols means no tree (`buildTree` returns `nil`) and an empty body. The header alone is enough to reconstruct an empty file.
- **A file of one repeated byte.** A single distinct symbol produces a tree of *one leaf* — depth zero, so the natural code is the empty string, which encodes nothing. The convention is to give that lone symbol the 1-bit code `"0"`, and on decode, skip the tree walk entirely and just replay the symbol `totalSymbols` times.

These aren't hypothetical. They're the inputs that turn a "works on my paragraph of text" toy into something that round-trips *everything*.

## Proving it actually works

Correctness here means one thing: `Decompress(Compress(x)) == x` for every `x`. The test suite encodes that directly, including a randomized fuzz pass and a property check that the generated codes really are prefix-free:

```
TestRoundTripVariedInputs
TestRandomFuzzRoundTrip
TestBuildCodesArePrefixFree
TestCompressionActuallyShrinksSkewedData
TestDecompressRejectsBadMagic
```

The fuzz test is the one I trust most. Hand-picked examples confirm the cases you *thought* of; random inputs find the tie-break bug, the off-by-one in the last byte, and the single-symbol corner you forgot. If you build one of these, make the round-trip a property test before you call it done.

## What building it taught me

Huffman is small enough to finish in an afternoon and deep enough to expose ideas you'll reuse for years: greedy optimality, prefix codes, bit-level I/O, and self-describing binary formats. But the real lesson was the tie-break — a reminder that "valid" and "deterministic" are different guarantees, and that for any encode/decode pair, you need both.

The full implementation, tests, and a longer teaching README are in the repo: [huffman-compression](https://github.com/singhhimanshu004/coding-challenges-deepdive/tree/master/phase-01-foundations/huffman-compression). It's part of a larger effort to build foundational tools from scratch and write down what each one teaches.

**Further reading:** David Huffman's original 1952 paper *A Method for the Construction of Minimum-Redundancy Codes*; the DEFLATE spec (RFC 1951) for how canonical Huffman and LZ77 combine in real-world gzip/zlib.
