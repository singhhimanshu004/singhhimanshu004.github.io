---
title: "Personal Portfolio & Resume Site"
description: "A fast, accessible portfolio and blog built with Astro and deployed to GitHub Pages via GitHub Actions."
problem: "I wanted a clean, fast, no-tracking home on the web to host my resume, projects, and writing — something I fully own and can update with a simple git push."
role: "I designed and built the site end to end: information architecture, design system, content collections for the blog and projects, and a CI/CD pipeline that builds and deploys on every push."
tech:
  - "Astro"
  - "TypeScript"
  - "GitHub Pages"
  - "GitHub Actions"
  - "Accessibility"
impact: "A statically generated site that loads fast, ships almost no JavaScript, and redeploys automatically on every commit. The site you are reading right now."
repoUrl: "https://github.com/singhhimanshu004/singhhimanshu004.github.io"
liveUrl: "https://singhhimanshu004.github.io"
featured: true
order: 2
---

## What it is

This is the site you are looking at. It hosts my resume, project case studies, and a blog, and it is built to be fast, accessible, and easy to maintain.

## How it is built

The site is built with [Astro](https://astro.build/), which renders everything to static HTML and ships minimal JavaScript. Blog posts and projects are managed as Markdown content collections with typed frontmatter, so adding a post or a project is just writing a Markdown file.

Design is handled through a small set of CSS design tokens for color, spacing, and typography, with a light and dark theme. The goal throughout was a calm, readable layout that works well on any device and respects accessibility basics like semantic structure and keyboard navigation.

## How it ships

The site deploys to GitHub Pages through a GitHub Actions workflow. Every push to the main branch triggers a build and a deploy — there is no manual publishing step. The resume PDF is served directly from the site, so the downloadable version and the on-page version stay in sync.
