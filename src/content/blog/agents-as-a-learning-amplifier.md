---
title: "Many Hands, One Judgment — Running a Team of AI Agents on a Learning Project"
description: "How I use a multi-agent CLI as a learning amplifier — not a code vending machine — on a build-from-scratch project, and why README-first keeps the human accountable for understanding."
pubDate: 2026-06-06
tags:
  - "Agentic AI"
  - "AI-Assisted Development"
  - "Learning in Public"
  - "Developer Tools"
draft: false
---

The fastest way to *not* learn something is to ask an AI to write it for you. You get working code and an empty head. So when I started a project whose entire point is *understanding* — building foundational tools like a [Huffman compressor](/blog/building-a-huffman-compressor/) and a [JSON parser](/blog/writing-a-json-parser-by-hand/) from scratch — I had to be deliberate about where AI helps and where it quietly robs you of the lesson.

This post is about that line: how I use a team of AI agents as a learning amplifier on my [Coding Challenges Deep Dive](https://github.com/singhhimanshu004/coding-challenges-deepdive), and the rule that keeps it honest.

## A conversation vs. a team

The default AI coding workflow is a single thread: you ask one assistant to explain a concept, draft some code, fix an error, repeat. It's useful, but it has one voice and one bias. Ask it for a Huffman tree and it gives you *a* Huffman tree — it won't spontaneously argue with itself about whether your encode and decode paths build the *same* tree.

An agentic workflow splits that one voice into roles with different responsibilities:

- A **lead** keeps the implementation aligned with the *learning goal*, not just "does it run."
- A **domain specialist** cares about the actual mechanics — the wire format, the bit order, the grammar.
- A **tester** thinks in edge cases: empty input, a file of one repeated byte, a trailing comma, a malformed surrogate pair.
- A **reviewer** reads the README and asks where the explanation went hand-wavy.

None of these replace engineering judgment. They create *structured friction*. Instead of one broad question, the work gets pushed on from several angles at once — and the angles you'd personally forget are exactly the ones a dedicated role won't.

## The most valuable use isn't code — it's preparation

The highest-leverage moment for agents on this project is *before* I write a line of implementation.

Take the Huffman challenge. Before coding, I need to actually understand: prefix codes, why a min-heap gives you the two least-frequent nodes cheaply, MSB-first bit packing, and the subtle requirement that encode and decode reconstruct an *identical* tree. (That last one hides a real bug: Go iterates maps in randomized order, so without a deterministic tie-break, the two sides can build different valid trees and the round-trip silently fails.) Agents are genuinely useful here as research partners — turning a big topic into a learning plan, naming the edge cases, pushing back when my mental model is vague.

The same was true for the JSON parser: surrogate pairs, leading-zero rejection, LL(1) lookahead, trailing-comma handling. The agents didn't *teach* me those by writing the parser. They helped me build the *map* of what I needed to understand before I earned the right to write the code.

## README-first: the rule that keeps the human accountable

Here's the discipline that makes the whole thing work, and it predates the agents: **the README comes before the code.**

Before implementing a challenge, I write a teaching README — what we're building, the core concepts, how the real tool behaves, the architecture, a step-by-step plan, the tests, and the takeaways. Only then do I write code. The code proves I understood the *mechanics*; the README proves I can *explain* the idea well enough for someone else to learn from it.

> It's entirely possible to make a program pass a few examples while understanding almost nothing. Writing the explanation first makes that impossible to fake — if I can't write the concept down clearly, I don't understand it yet, and no amount of green tests changes that.

This is also the perfect job for agents, but in a *reviewer* posture, not an author one. A domain agent can check whether my description of the DNS wire format is actually correct. A reviewer agent can flag that my README explains the happy path but never the failure modes. They strengthen the artifact I'm accountable for — they don't produce it for me.

## Where the human stays responsible

The limits are real, and I treat them as non-negotiable:

- **Agents are confident and sometimes wrong.** A fluent, plausible explanation still has to be checked against the spec, the source material, or the behavior of the real tool. For the JSON parser, the ground truth was `json.loads` — every "valid" and "invalid" claim got tested against the standard library, not against an agent's say-so.
- **Agents over-engineer small problems.** Part of my job is keeping the first version *small*.
- **Agents make it tempting to accept an answer before you've internalized it.** That's the most dangerous one, because it feels like progress.

So the human loop stays fixed: I choose the learning goal, read the source challenge, write or revise the README, decide the approach, run the tests, and make the final call. **If I can't explain a section in my own words, it isn't done. If I can't justify a design decision, it isn't ready.**

## A workflow, not magic

I don't think agentic development deserves either the hype or the dismissal it usually gets. It's a workflow pattern: give agents clear roles, constrain them with real source material, ask them to produce *reviewable* artifacts, and keep one human accountable for understanding the result.

Used that way, a multi-agent CLI becomes a learning amplifier — it helps me move through a large, deliberately hard curriculum with more structure and sharper questions, without quietly removing the part where I actually learn. For a project whose whole purpose is to understand the tools we use every day and share that understanding back, that distinction is the entire game.

The project, including the build-from-scratch tools and their teaching READMEs, is public at [github.com/singhhimanshu004](https://github.com/singhhimanshu004/coding-challenges-deepdive).
