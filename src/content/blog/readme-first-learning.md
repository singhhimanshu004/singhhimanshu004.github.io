---
title: "README-First: Teach the Concept Before You Write the Code"
description: "A practical README-first learning method for developers who want to understand tools deeply enough to explain and build them."
pubDate: 2026-06-09
tags:
  - "Learning in Public"
  - "Technical Writing"
  - "Software Engineering"
  - "Best Practices"
draft: false
---

Most developers have experienced the gap between making code work and understanding why it works. A program can pass a few test cases while the underlying concept is still unclear. That gap becomes obvious when someone asks a simple question: can you explain it?

README-first learning is my way of closing that gap. Before writing implementation code, I write a teaching README that explains the tool, the concepts, the real-world behavior, the design, the build plan, the tests, and the takeaways. I am using this method in my Coding Challenges Deep Dive project, based on [John Crickett's Coding Challenges](https://codingchallenges.fyi/), where the goal is to build 64 tools from scratch and share the learning clearly.

The rule is simple: the README comes before the code.

## Why write before coding

Writing first changes the shape of the work. It forces me to define the problem before solving it. That sounds obvious, but it is easy to skip when the first implementation idea appears quickly.

Take a familiar tool like `grep`. A code-first approach might start with reading a file and matching lines with a regular expression. That is a useful start, but it does not capture the full learning opportunity. A README-first approach asks more questions before the first function is written. What problem does `grep` solve? How does it behave with stdin? What do flags like recursive search or context lines imply for the architecture? What should happen on no match, invalid input, or multiple files? How can we test our implementation against the real tool?

Those questions lead to better code because they create a clearer target. They also lead to better learning because they reveal what I do not yet understand.

## The README structure

For each challenge, I use the same structure.

First, I write what we are building. This section should be plain and concrete. If the project is a JSON parser, it should say that we are turning JSON text into structured values and validating the grammar along the way.

Second, I write the core concepts. This is where the learning lives: tokenization, parsing, buffered I/O, sockets, hashing, compression, concurrency, or whatever the challenge requires. The goal is not to write a textbook. The goal is to identify the ideas that make the implementation possible.

Third, I explain how it works in the real world. Many tools have behavior that is easy to miss in toy examples. `wc` has flags and stdin behavior. DNS has packet formats, retries, and timeouts. Git has objects, refs, and an index. This section connects the exercise to the actual tool or protocol.

Fourth, I outline the architecture. This is the bridge from concept to code. What are the main components? What data structures matter? Where does input enter, how is it transformed, and where does output leave?

Fifth, I write the step-by-step implementation plan. This keeps the first version small. A good plan starts with the simplest useful behavior, then adds features deliberately.

Sixth, I describe testing. Whenever possible, I want to compare behavior against the real tool, protocol expectations, sample fixtures, or documented edge cases.

Finally, I write key takeaways and further reading. These sections make the README useful after the code is finished. They also help another developer decide what to study next.

## What this does for understanding

README-first learning makes uncertainty visible. If I cannot explain a concept clearly, I probably do not understand it well enough to implement it confidently. If the testing section is vague, the implementation will probably be hard to trust. If the architecture section has too many moving parts, the first version is probably too large.

It also creates a better feedback loop. The README is not a static document written after the work is done. It is a design artifact before coding, a checklist during coding, and a teaching artifact after coding. As the implementation teaches me something new, I can refine the explanation.

This is especially useful for learning in public. Code alone is helpful, but code plus explanation is much more reusable. Another developer can read the README, understand the path, and then decide whether to inspect the implementation, try the challenge, or adapt the method.

## How to try it

Pick one project that is small enough to finish but real enough to teach something. Before writing code, create a README with the sections above. Keep the first draft honest. Use placeholders only where you truly need to research more, and treat those placeholders as work, not decoration.

Then implement in small steps. After each step, return to the README. Did the architecture hold up? Did the tests catch the behavior that matters? Did the key takeaway change? If so, update the document.

The goal is not perfect documentation. The goal is disciplined learning. Teaching the concept before writing the code makes the work slower at the beginning, but it makes the understanding stronger at the end. For me, that trade-off is worth it.
