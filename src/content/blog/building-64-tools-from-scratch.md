---
title: "Building 64 Tools From Scratch (and Sharing the Whole Journey)"
description: "Why I started a README-first learning project around John Crickett's Coding Challenges, and how I plan to share the work back with developers."
pubDate: 2026-06-09
tags:
  - "Learning in Public"
  - "Coding Challenges"
  - "Software Engineering"
  - "Career"
draft: false
---

I have always learned programming best when I build something concrete. Reading about a protocol, a parser, or a command-line tool is useful, but the ideas become much clearer when I have to make the thing work myself. That is the reason I started the Coding Challenges Deep Dive: a long-form learning project around [John Crickett's Coding Challenges](https://codingchallenges.fyi/), where the goal is to build real tools from scratch and document the journey carefully enough that other developers can follow along.

The project is still in the setup and early in-progress stage, so I do not want to present it as a finished library or a completed curriculum. It is a learning-in-public effort. My goal is to take each challenge seriously, write down what I learn, and turn the work into material that is useful beyond my own practice.

## Why these challenges

Coding challenges are easy to treat as interview drills, but the challenges in this project are different. They are small enough to start, but real enough to expose how software works underneath the abstractions we use every day.

The curriculum covers 64 challenges across eight progressive phases. It starts with foundations like parsing, encoding, and data structures. From there it moves through Unix text tools, advanced command-line programs, networking, servers and infrastructure, applications, developer tools, and finally games and interpreters.

That progression matters. Building `wc`, `cut`, `grep`, and `sed` teaches streaming I/O, files, flags, and the Unix philosophy. Building a JSON parser teaches how raw text becomes structured data. A DNS resolver forces you to understand binary protocols and network byte order. A load balancer makes server behavior and health checks more concrete. A Redis server, Git implementation, or Lisp interpreter is not just a project; it is a way to understand tools that many of us use without thinking about their internals.

## The shape of the project

The plan is to build tools such as `wc`, `cut`, `grep`, `sed`, a JSON parser, a DNS resolver, a load balancer, a Redis server, Git, a Lisp interpreter, and more. The project is intentionally multi-language. Go, Python, Java, and TypeScript are all available depending on what each challenge is trying to teach.

That choice is part of the learning. Go is a natural fit for command-line tools, networking, and servers because of its I/O model and simple deployment story. Python is useful for parsing, algorithms, and experimentation. TypeScript fits web applications and browser-based work. Java remains a good option for comparing server-side concurrency and type-system trade-offs.

The point is not to prove that one language is best. The point is to choose a language that makes the core lesson visible.

## README-first, not code-first

The most important rule in the project is README-first. Before writing the implementation, I write a teaching README for the challenge.

Each README follows the same structure: what we are building, the core concepts, how the tool works in the real world, the architecture, a step-by-step implementation plan, testing, key takeaways, and further reading. The code proves that I understood the mechanics. The README proves that I can explain the idea clearly enough for someone else to learn from it.

That distinction is important. It is possible to make a program pass a few examples while still having only a shallow understanding of the problem. Teaching the concept first forces me to slow down. I have to define the behavior, identify the edge cases, understand the real tool or protocol, and make the architecture explicit before I start filling in functions.

## Learning in public

I am sharing this because I want the project to be useful to the developer community. Many developers want to go deeper into systems, networking, command-line tools, and developer tooling, but the path can feel scattered. My hope is that this work becomes a practical trail: not just source code, but explanations, references, implementation notes, and honest reflections on where the hard parts are.

I will keep the work grounded. I will not claim completion before the work is done, and I will not turn the project into a collection of inflated metrics. The value should come from clarity, consistency, and usefulness.

As the project grows, I plan to share updates through my portfolio and GitHub at [github.com/singhhimanshu004](https://github.com/singhhimanshu004). If even one explanation helps another developer understand a tool they use every day, the effort is worth it.
