---
title: "Coding Challenges Deep Dive"
description: "Building 64 real-world tools from scratch — from wc and grep to a Redis server, Git, and a Lisp interpreter — with a README-first teaching approach."
problem: "Most tutorials teach you to use tools, not to understand how they work. I wanted to rebuild the everyday tools I rely on, from first principles, and teach each one clearly enough that someone else could learn from it."
role: "I designed a progressive, dependency-ordered curriculum of 64 challenges across 8 phases, and I build each tool from scratch — picking the language that best fits the problem (Go, Python, Java, or TypeScript)."
tech:
  - "Go"
  - "Python"
  - "Java"
  - "TypeScript"
  - "Systems Programming"
  - "Learning in Public"
impact: "An open, in-progress learning resource for the developer community. Each challenge ships a teaching README written before the code, so the repository doubles as a set of explainers, not just solutions."
repoUrl: "https://github.com/singhhimanshu004/coding-challenges-deepdive"
featured: true
order: 1
---

## What it is

Coding Challenges Deep Dive is my attempt to deeply understand the tools I use every day by rebuilding them from scratch. It follows John Crickett's excellent [Coding Challenges](https://codingchallenges.fyi/), and organizes 64 challenges into eight progressive phases — starting with parsing bytes and finishing with servers, version control, and language interpreters.

## The approach: README-first

The core rule of the project is that I write a teaching `README.md` *before* I write any code. Each README walks through what we're building, the core concepts, how the real tool works, the architecture, a step-by-step implementation, and key takeaways. The code proves I understood the problem; the README proves I can teach it.

This is deliberately about giving back. A solution you can't explain helps no one. A clear write-up helps the next person who is curious about how `grep`, a JSON parser, or a load balancer actually works.

## Where it stands

The curriculum and structure are in place, and early challenges are implemented — including a JSON parser in Python (with tests) and a Huffman compression tool in Go. The remaining phases are scaffolded and being worked through in public, one challenge at a time.

The repository is open source, so anyone can read the explainers, follow along, or build their own version alongside mine.
