# Architecture and Information Architecture Spec

## Context and goal

Build a fast Astro portfolio + blog for Himanshu Singh, deployed to GitHub Pages. The site must help recruiters understand fit in about 30 seconds and support a job switch in 2-3 months. Prioritize clarity, credible projects, resume access, and contact paths over feature breadth.

## Site map

| Page | Route | Purpose | Primary action |
| --- | --- | --- | --- |
| Home | `/` | Recruiter landing page: who Himanshu is, target role, strongest proof, and next steps. | Open resume PDF, view projects, contact. |
| About | `/about/` | Short professional story, strengths, working style, and career direction. | Continue to resume or projects. |
| Resume | `/resume/` | Skimmable web resume plus downloadable PDF. | Download PDF or contact. |
| Projects | `/projects/` | Proof of ability through selected, outcome-oriented work. | Open featured project detail/repo/live demo. |
| Project detail | `/projects/[slug]/` | Explain problem, Himanshu's role, tech decisions, impact, and links. | Visit repo/live demo or contact. |
| Blog | `/blog/` | Demonstrate technical thinking and learning in public. | Read recent posts by topic. |
| Blog post | `/blog/[slug]/` | Long-form technical writing. | Navigate to related posts or contact. |
| Contact | `/contact/` | Give low-friction ways to reach Himanshu. | Email, LinkedIn, GitHub. |

## Home information architecture

Above the fold must answer four recruiter questions without scrolling:

1. **Who:** Himanshu Singh.
2. **Role:** target professional identity, e.g. "Software Engineer / Full-stack Developer" once confirmed.
3. **What he wants:** a concise line such as "Open to software engineering roles starting in the next 2-3 months."
4. **Why believe him:** one proof line summarizing strongest domains/projects.

Required hero elements:

- Name: `Himanshu Singh`.
- Role headline: one clear target role, not a list of buzzwords.
- Availability/intent: what role/location/work mode he is looking for.
- Short value proposition: one sentence focused on outcomes.
- CTAs, in priority order:
  1. `Download Resume` -> resume PDF in `/public/`.
  2. `View Projects` -> `/projects/`.
  3. `Contact Me` -> `/contact/`.

Below the fold sequence:

1. Featured projects: 2-3 strongest projects only.
2. Resume snapshot: skills, experience/education highlights, PDF link.
3. Recent writing: 2-3 latest non-draft posts.
4. Contact strip: email/LinkedIn/GitHub.

## Content model

Use Astro content collections under `src/content/`. Keep schemas strict so Linus can type pages/components from collection data.

### `projects` collection

Recommended file shape: `src/content/projects/{slug}.md` or `.mdx`.

Exact frontmatter fields:

```yaml
title: "Project title"
description: "One-sentence recruiter-friendly summary."
problem: "The user/business/technical problem this project solved."
role: "Himanshu's specific contribution and ownership."
tech:
  - "Astro"
  - "TypeScript"
impact: "Measurable or concrete outcome; use qualitative impact until metrics exist."
repoUrl: "https://github.com/..."
liveUrl: "https://..."
featured: true
order: 1
```

Field decisions:

- `title`: string, required.
- `description`: string, required, used on cards and meta descriptions.
- `problem`: string, required.
- `role`: string, required.
- `tech`: string array, required.
- `impact`: string, required.
- `repoUrl`: URL string, optional if private/unavailable.
- `liveUrl`: URL string, optional if no live demo.
- `featured`: boolean, required; controls Home placement.
- `order`: number, required; lower numbers sort first.

### `blog` collection

Recommended file shape: `src/content/blog/{slug}.md` or `.mdx`.

Exact frontmatter fields:

```yaml
title: "Post title"
description: "One-sentence summary for listings and SEO."
pubDate: 2026-06-08
tags:
  - "Astro"
  - "JavaScript"
draft: false
```

Field decisions:

- `title`: string, required.
- `description`: string, required.
- `pubDate`: date, required.
- `tags`: string array, required; allow empty array only for drafts.
- `draft`: boolean, required; draft posts never appear in production lists/RSS.

## Component breakdown for Linus

Linus should build the implementation around small Astro components and layout shells:

- `BaseLayout`: HTML shell, metadata, global styles, theme class wiring.
- `PageLayout`: standard page width, heading area, content slot.
- `PostLayout`: blog post wrapper, dates, tags, prose container.
- `ProjectLayout`: project detail wrapper with problem/role/tech/impact/link sections.
- `Nav`: primary navigation for Home, About, Resume, Projects, Blog, Contact.
- `Footer`: contact links, copyright, GitHub/LinkedIn.
- `ThemeToggle`: minimal light/dark toggle; progressive enhancement only.
- `ProjectCard`: title, description, tech chips, impact, repo/live links.
- `PostCard`: title, description, pubDate, tags.
- `Hero`: Home above-the-fold content and CTAs.
- `SectionHeader`: reusable section title + short intro.
- `ContactLinks`: email, GitHub, LinkedIn, optional location/work mode.

Do not over-componentize one-off content blocks until repetition appears.

## Prioritized build sequence

1. **Ship the recruiter path first:** Home hero, Resume page/PDF link, Projects list with 2-3 credible project cards, Contact page.
2. **Add project depth:** project detail pages with problem, role, tech, impact, repo/live links.
3. **Add credibility polish:** About page, resume snapshot on Home, SEO metadata, social links, GitHub Pages base path check.
4. **Add blog foundation:** content collection, Blog index, Post pages, draft filtering.
5. **Add blog extras only after core pages work:** tags, RSS, related posts, pagination if needed.
6. **Final 30-second pass:** reorder Home content by strongest proof, remove weak projects, ensure resume/contact are visible from every page.

## Direction decisions

- Optimize the first release for recruiter skimming, not a complete personal website.
- Treat Projects as the strongest proof section; Home should feature only the best 2-3.
- Keep JavaScript optional except theme toggle; Astro should ship mostly static HTML.
- Use content collections for both blog and projects so cards/detail pages stay typed and consistent.
- Make `order` the explicit manual sorting control for projects; do not rely on file names or dates.

## Open questions for Himanshu

1. What GitHub username should be used for profile/repo links?
2. What LinkedIn URL and preferred contact email should be public?
3. What exact target role should the hero state: Software Engineer, Frontend Engineer, Full-stack Developer, or another title?
4. What location/work mode should the site mention: remote, hybrid, relocation, specific cities?
5. Which 2-3 real projects are strongest and should be featured first?
6. Do those projects have live demos, public repos, screenshots, or measurable outcomes?
7. Is there an existing resume PDF, and what should its public filename be?
8. Will the site use the default GitHub Pages URL or a custom domain?
9. Should blog posts be technical tutorials, project write-ups, career notes, or a mix?
10. Are there any employers, clients, or private projects that must not be mentioned?
