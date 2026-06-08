---
title: "Agentic Development in Practice: Learning With a Team of AI Agents"
description: "A practical look at using Squad CLI as an AI-assisted learning partner while building real tools from scratch."
pubDate: 2026-06-09
tags:
  - "Agentic AI"
  - "AI-Assisted Development"
  - "Squad CLI"
  - "Developer Tools"
draft: false
---

AI-assisted development is most useful to me when it makes learning more active, not more passive. I do not want an assistant to replace the thinking required to understand a parser, a protocol, or a server. I want a system that can help me ask better questions, compare design choices, test my assumptions, and keep the work organized.

That is why I am using Squad CLI in my Coding Challenges Deep Dive project, which is based on [John Crickett's Coding Challenges](https://codingchallenges.fyi/). The project is currently being set up and developed in public. The goal is to build 64 tools from scratch and write clear teaching material for each one. Squad CLI helps by coordinating a small team of specialized AI agents around the work.

## What I mean by an AI agent team

In a normal AI coding workflow, I might ask one assistant to explain a concept, draft code, or debug an error. That can be helpful, but it often becomes a single-threaded conversation. An agentic workflow gives different responsibilities to different agents.

For example, one agent can act like a technical lead and keep the implementation aligned with the learning goal. Another can focus on the domain, such as parsing, networking, or command-line design. A tester can think about edge cases, expected behavior, and how to compare the implementation against a real tool. A reviewer can look for gaps in the README or places where the explanation is too vague.

This does not make the agents a replacement for engineering judgment. It makes the workflow more structured. Instead of asking one broad question, I can break the work into roles and let each role challenge a different part of the solution.

## How I use it for learning

The most important use case is not code generation. It is preparation.

Before writing code for a challenge, I need to understand the real behavior of the tool or protocol. If the challenge is `wc`, that means bytes, lines, words, flags, stdin behavior, and differences between simple examples and real terminal usage. If the challenge is a DNS resolver, that means packet structure, UDP, timeouts, name compression, and recursive resolution. If the challenge is Git, that means objects, refs, the index, packfiles, and how a simple command maps to stored data.

In that stage, agents are useful as research partners. They can help turn a large problem into a learning plan. They can suggest what to read, what terms to define, what edge cases to consider, and how to structure the README. They can also push back when the explanation is hand-wavy.

After the README is drafted, the agents become more implementation-focused. The developer agent can propose a small first version. The tester can ask how we will verify it. The reviewer can compare the code against the teaching goals. This keeps the challenge from becoming only a coding exercise.

## Where it helps

Agentic development helps most when the problem has multiple dimensions. These challenges are not just about producing files. They involve real-world behavior, user-facing command-line ergonomics, architecture, testability, and documentation.

Having separate agents makes those dimensions harder to ignore. A domain agent might care about whether the DNS wire format is explained correctly. A tester might care about timeouts and malformed input. A lead might care about whether the implementation is still small enough for the current step. A writing-focused review might notice that the README explains the happy path but not the failure modes.

That kind of structured friction is valuable. It slows me down in the right places.

## Where the human stays responsible

There are limits. AI agents can be confident and wrong. They can produce plausible explanations that need verification against documentation, source material, or the behavior of real tools. They can over-engineer a small challenge. They can also make it tempting to accept an answer before I have internalized it.

For this project, the human loop is non-negotiable. I still choose the learning goal, read the source challenge, write or revise the README, decide the implementation approach, run the tests, and make the final judgment. If I cannot explain a section in my own words, the section is not done. If I cannot justify a design decision, the decision is not ready.

That is the balance I want: AI agents as collaborators, not shortcuts.

## A practical middle ground

I do not think agentic development needs to be treated as magic. It is a workflow pattern. Give agents clear roles, constrain them with real source material, ask them to produce reviewable artifacts, and keep the human accountable for understanding.

Used that way, Squad CLI becomes a learning amplifier. It helps me move through a large curriculum with more structure, but it does not remove the responsibility to learn the material deeply. For a project built around teaching and giving back, that distinction matters.
