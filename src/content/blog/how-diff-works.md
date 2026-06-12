---
title: "Computing the Difference — How diff Finds the Smallest Set of Changes"
description: "Reimplementing Unix diff from scratch: the longest-common-subsequence table that powers it, the backtrack that turns it into an edit script, and the GNU-compatible details — unified hunks, context windows, and the bias that lists deletions first."
pubDate: 2026-06-12
tags:
  - "Algorithms"
  - "Dynamic Programming"
  - "Go"
  - "Unix"
heroImage: "/blog/diff-lcs.svg"
heroAlt: "An LCS dynamic-programming table for two short word lists, with a dashed backtrack path from the bottom-right corner producing an edit script of context, delete, and insert lines."
heroCaption: "diff fills an LCS table, then walks it backwards. The diagonal steps are matches; the steps you don't take become the + and - lines."
draft: false
---

Every time you read a code review, run `git diff`, or apply a patch, you're looking at the output of a sixty-year-old idea: the **longest common subsequence**. `diff` doesn't compare files line by line — it finds the largest set of lines the two files *share, in order*, and then everything that isn't in that set is, by definition, either an insertion or a deletion.

I rebuilt `diff` from scratch in Go as part of my [Coding Challenges Deep Dive](https://github.com/singhhimanshu004/coding-challenges-deepdive) — not the `git diff` you use daily, but a faithful clone that produces normal, unified (`-u`), and context (`-c`) output that real tools accept. The algorithm is short. The thing that took the real work was getting the output *byte-identical to GNU diff*, and that's where the interesting decisions live.

## The core: longest common subsequence

A *subsequence* keeps order but allows gaps. For `[A, B, C, D]` and `[A, C, D, E]`, the LCS is `[A, C, D]` — length 3. Once you know that, the diff writes itself: the LCS lines are unchanged, the leftover lines in file A were deleted, and the leftover lines in file B were inserted.

You compute the LCS *length* for every prefix pair with a two-line recurrence, filling an `(n+1) × (m+1)` table:

```go
func lcsTable(a, b []string) [][]int {
    n, m := len(a), len(b)
    table := make([][]int, n+1)
    for i := range table {
        table[i] = make([]int, m+1)   // Go zeroes this for us
    }
    for i := 1; i <= n; i++ {
        for j := 1; j <= m; j++ {
            if a[i-1] == b[j-1] {
                table[i][j] = table[i-1][j-1] + 1   // lines match: extend the diagonal
            } else {
                table[i][j] = max2(table[i-1][j], table[i][j-1])  // take the better neighbour
            }
        }
    }
    return table
}
```

`table[i][j]` is the LCS length of the first `i` lines of A and the first `j` lines of B. Row 0 and column 0 stay zero — an empty file shares nothing — which is exactly why the inputs are indexed with `i-1` / `j-1`. The whole algorithm is those three lines in the inner loop. It runs in `O(n·m)` time and space, which is fine for source files and the reason real `diff` switches to the smarter Myers algorithm only for very large inputs.

## Backtracking: from a table of numbers to a list of edits

The table tells you *how long* the LCS is. To produce a diff you have to recover *which* lines it contains, and you do that by walking the table **backwards** from the bottom-right corner — the mirror image of how you filled it:

```go
for i > 0 || j > 0 {
    switch {
    case i > 0 && j > 0 && a[i-1] == b[j-1]:
        rev = append(rev, edit{kind: opEqual, text: a[i-1]})  // on the LCS: step diagonally
        i--; j--
    case j > 0 && (i == 0 || table[i][j-1] >= table[i-1][j]):
        rev = append(rev, edit{kind: opInsert, text: b[j-1]})  // came from the left
        j--
    default:
        rev = append(rev, edit{kind: opDelete, text: a[i-1]})  // came from above
        i--
    }
}
```

Three cases: matching lines step diagonally (they're context), a leftward move is an insertion, an upward move is a deletion. Because we walk backwards, edits come out reversed and we flip the slice at the end.

That innocuous `>=` in the second case is not arbitrary. **When the two neighbours tie, this code prefers the upward move (a deletion) over the leftward one.** That single choice is what makes a changed region list its `-` lines *before* its `+` lines — exactly the convention GNU diff uses. Flip it to `>` and your output is still a *correct* diff, but it no longer matches the tool everyone's eyes and patch programs are calibrated against. Reproducing a standard tool is a lesson in how many of its behaviors are conventions, not consequences.

## Grouping into hunks: the part nobody mentions

A correct edit script isn't readable output. Unified diff (`@@ -3,6 +3,7 @@`) shows changes *with a few lines of surrounding context*, and it merges nearby changes into a single hunk so you don't get a wall of tiny fragments. That grouping logic is its own small algorithm:

```go
func groupHunks(edits []edit, ctx int) []hunkRange {
    var changes []int
    for idx, e := range edits {
        if e.kind != opEqual {
            changes = append(changes, idx)
        }
    }
    // start the first hunk: ctx lines before/after the first change
    hs := clampLow(changes[0] - ctx)
    he := clampHigh(changes[0]+ctx+1, len(edits))
    for _, c := range changes[1:] {
        cs := clampLow(c - ctx)
        if cs <= he {
            he = clampHigh(c+ctx+1, len(edits))   // windows touch: extend this hunk
        } else {
            hunks = append(hunks, hunkRange{hs, he})  // gap: start a new hunk
            hs, he = cs, clampHigh(c+ctx+1, len(edits))
        }
    }
    // ...append the final hunk
}
```

Each change pulls in `ctx` lines (default 3) on each side. If two changes' context windows touch or overlap, they collapse into one hunk; otherwise a fresh hunk starts. This is precisely what GNU diff does, and getting it wrong is how you end up with output that looks *almost* right but won't apply cleanly.

## The off-by-one that is actually a convention

The hunk header `@@ -3,6 +3,7 @@` encodes a start line and a length for each side. The subtlety: a hunk that's a *pure* insertion has zero lines on the A side, and a pure deletion has zero on the B side. GNU diff reports those zero-length sides with the line number *before* the change, written as `-l,0`:

```go
// A zero-length side is reported with the line number BEFORE the change,
// matching GNU diff's "-l,0" form.
if length == 0 {
    if aSide {
        start = seg[0].aIndex
    } else {
        start = seg[0].bIndex
    }
}
```

There's no algorithmic reason for `-l,0` versus `-l+1,0`; it's a convention baked into `patch` and every tool that parses unified diffs. Cloning a Unix tool teaches you that "correct" and "compatible" are different bars, and the gap between them is full of these tiny, load-bearing decisions.

## Why diff is the perfect dynamic-programming project

Most explanations of LCS stop at "fill the table, read off the number." But the number is the boring part. The *useful* part is the backtrack that turns the table into an edit script, the hunk grouping that makes it readable, and the dozen formatting conventions that make it interoperable. Building the whole pipeline — and then diffing my diff's output against GNU diff until they matched character for character — is what finally made dynamic programming feel less like a textbook trick and more like a tool I own.

Full source — `lcs.go`, `editscript.go`, `format.go`, plus a test suite that checks output against fixtures: [diff](https://github.com/singhhimanshu004/coding-challenges-deepdive/tree/master/phase-02-core-unix/diff).

**Further reading:** Eugene Myers' 1986 paper *An O(ND) Difference Algorithm and Its Variations* (what `git diff` actually uses); the GNU `diffutils` manual for the output formats; and my [Huffman](/blog/building-a-huffman-compressor/) post for another from-scratch compression/encoding deep dive.
