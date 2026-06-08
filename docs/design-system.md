# Design System

## Direction

Clean, minimal, modern portfolio — the site should feel calm, confident, and easy to scan. Use generous whitespace, strong typography, restrained surfaces, and one purposeful accent color. Avoid noisy gradients, decorative cards for their own sake, skill bars, excessive animation, and competing colors.

The desired recruiter impression in 30 seconds: **credible engineer, clear communicator, polished product taste**.

## Mood

- **Quiet confidence:** neutral canvas, crisp type, high contrast.
- **Editorial portfolio:** content hierarchy matters more than decoration.
- **Product-minded:** polished but not flashy; every visual choice should improve readability.
- **Fast and accessible:** readable in light/dark mode, no heavy visual dependencies.

## Color Palette

Use one accent color: **Signal Blue**. It is professional, readable, and familiar in developer/recruiting contexts.

### Light Mode

| Token | Hex | Usage |
| --- | --- | --- |
| `--color-bg` | `#FAFAF9` | Page background, warm off-white |
| `--color-surface` | `#FFFFFF` | Cards, raised sections |
| `--color-surface-muted` | `#F4F4F5` | Subtle section bands |
| `--color-text` | `#18181B` | Primary text |
| `--color-text-muted` | `#52525B` | Secondary copy |
| `--color-text-subtle` | `#71717A` | Metadata, captions |
| `--color-border` | `#E4E4E7` | Dividers, card outlines |
| `--color-accent` | `#2563EB` | Links, CTAs, active states |
| `--color-accent-hover` | `#1D4ED8` | Accent hover |
| `--color-accent-soft` | `#DBEAFE` | Subtle accent backgrounds |

### Dark Mode

| Token | Hex | Usage |
| --- | --- | --- |
| `--color-bg` | `#09090B` | Page background |
| `--color-surface` | `#18181B` | Cards, raised sections |
| `--color-surface-muted` | `#27272A` | Subtle section bands |
| `--color-text` | `#F4F4F5` | Primary text |
| `--color-text-muted` | `#D4D4D8` | Secondary copy |
| `--color-text-subtle` | `#A1A1AA` | Metadata, captions |
| `--color-border` | `#3F3F46` | Dividers, card outlines |
| `--color-accent` | `#60A5FA` | Links, CTAs, active states |
| `--color-accent-hover` | `#93C5FD` | Accent hover |
| `--color-accent-soft` | `#172554` | Subtle accent backgrounds |

## Typography

Prefer system fonts for GitHub Pages performance. The stack should include Inter first so Linus can optionally load/self-host it later without changing tokens, but the design works with native system fonts.

- **Sans:** `Inter`, `ui-sans-serif`, `system-ui`, `-apple-system`, `BlinkMacSystemFont`, `Segoe UI`, `sans-serif`
- **Mono:** `ui-monospace`, `SFMono-Regular`, `Menlo`, `Monaco`, `Consolas`, monospace

### Type Scale

| Token | Size | Line Height | Weight | Use |
| --- | ---: | ---: | ---: | --- |
| `--text-xs` | `0.75rem` / 12px | 1.45 | 500 | Tags, metadata |
| `--text-sm` | `0.875rem` / 14px | 1.55 | 400–500 | Supporting copy |
| `--text-base` | `1rem` / 16px | 1.7 | 400 | Body copy |
| `--text-lg` | `1.125rem` / 18px | 1.65 | 400–500 | Lead text |
| `--text-xl` | `1.25rem` / 20px | 1.45 | 600 | Section intros |
| `--text-2xl` | `1.5rem` / 24px | 1.35 | 650 | Card/section headings |
| `--text-3xl` | `1.875rem` / 30px | 1.2 | 700 | Page headings |
| `--text-4xl` | `2.25rem` / 36px | 1.12 | 750 | Hero heading mobile |
| `--text-5xl` | `3rem` / 48px | 1.05 | 750 | Hero heading desktop |

Guidance:

- Keep body text at 16px minimum.
- Use tight tracking only for large headings.
- Use weight contrast before color contrast; most pages need only 3–4 type sizes.
- Keep paragraphs at `65ch` max for comfort.

## Spacing Scale

Use a compact 4px-based scale with larger jumps for section rhythm.

| Token | Value | Use |
| --- | ---: | --- |
| `--space-1` | `0.25rem` / 4px | Fine gaps |
| `--space-2` | `0.5rem` / 8px | Inline gaps |
| `--space-3` | `0.75rem` / 12px | Compact stack |
| `--space-4` | `1rem` / 16px | Default gap |
| `--space-5` | `1.25rem` / 20px | Card internals |
| `--space-6` | `1.5rem` / 24px | Section/card spacing |
| `--space-8` | `2rem` / 32px | Large gaps |
| `--space-10` | `2.5rem` / 40px | Section rhythm |
| `--space-12` | `3rem` / 48px | Major separation |
| `--space-16` | `4rem` / 64px | Page sections |
| `--space-20` | `5rem` / 80px | Hero/major sections |
| `--space-24` | `6rem` / 96px | Top-level rhythm |

## Radius

- `--radius-sm`: `0.375rem` / 6px — tags, small controls
- `--radius-md`: `0.75rem` / 12px — buttons, inputs
- `--radius-lg`: `1rem` / 16px — cards
- `--radius-xl`: `1.5rem` / 24px — feature panels
- `--radius-full`: `999px` — pills/avatar masks

Use radius sparingly. A recruiter portfolio should feel refined, not bubbly.

## Shadows

Prefer borders and whitespace over heavy shadows.

- `--shadow-sm`: subtle lift for interactive surfaces.
- `--shadow-md`: cards or sticky navigation only when necessary.
- `--shadow-lg`: rare hero/feature emphasis.

Dark mode should use softer shadows and stronger borders because shadows are less visible.

## Layout Grid

- **Page max width:** `1120px`
- **Reading max width:** `720px` or `65ch`
- **Wide max width:** `1280px` for rare visual sections
- **Content padding:** `24px` mobile, `32px` tablet, `40px` desktop
- **Grid:** 12 columns desktop, 6 tablet, 1 mobile
- **Column gap:** `24px` mobile/tablet, `32px` desktop

Recommended structure:

- Hero: concise intro, role signal, current focus, 1 primary CTA + 1 secondary link.
- Projects: show fewer, stronger projects with outcomes and stack chips.
- Blog: titles must be scannable; metadata secondary.
- Resume/About: use short sections and bullets, not dense paragraphs.

## Skimmability Rules

For the 30-second recruiter scan:

1. Lead with role clarity: who Himanshu is, what he builds, what he wants next.
2. Make the strongest proof visible above the fold: projects, impact, technologies.
3. Use short blocks: headings, bullets, and one-sentence summaries.
4. Keep metadata visually quiet; titles and outcomes should dominate.
5. Use accent only for action and orientation: links, CTAs, active nav, focus rings.
6. Maintain strong contrast in both themes.
7. Avoid percentage skill bars; prefer grouped skills and evidence from projects.

## Motion

Keep motion minimal and respectful:

- Duration: `150ms` for hover/focus, `220ms` for small reveals.
- Easing: `cubic-bezier(0.2, 0, 0, 1)`.
- Honor `prefers-reduced-motion` when components are implemented.

## Implementation Notes for Linus

- Import `src/styles/tokens.css` before component/page styles.
- Build components from semantic tokens, not raw hex values.
- Support automatic themes via `prefers-color-scheme` and manual override via `[data-theme="light"]` / `[data-theme="dark"]` on `html` or `body`.
- Do not introduce additional brand colors without a team decision.
