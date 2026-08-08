# Portfolio Redesign — Design Spec

Date: 2026-08-07
Status: Approved by user, ready for implementation planning

## Goal

Full visual redesign of Johan Boshell's Astro portfolio site, moving from the current
hardcoded dark-purple styling to a token-based, modern-minimal design system that
supports both light and dark themes with a toggle. Alongside the visual work, fix
real bugs found during review and remove dead code.

## Decisions

These were confirmed with the user during brainstorming (recorded here so the
rationale isn't lost):

- **Scope**: full visual redesign, not just a cleanup pass or content refresh.
- **Style direction**: modern minimal — clean, content-first, restrained color,
  generous whitespace (as opposed to bold/vibrant or terminal/cyberpunk styles).
- **Theme**: light + dark with a user-facing toggle (not dark-only, not
  system-preference-only).
- **Projects data**: keep all 5 existing projects (Weather Query, Grocery Bud,
  Tierlist Maker, CSS Memorama, MineSweeper) as-is, restyle only — no content
  changes to the project list itself.
- **New section**: add a Skills/Tech Stack section, distinct from the per-project
  tag chips.
- **Project layout**: replace the current "sticky stack" scroll effect (cards
  pin and stack on top of each other while scrolling) with a standard responsive
  card grid. The stacking effect doesn't fit the minimal direction and adds
  scroll-logic complexity for little benefit; a grid is simpler to scan and
  maintain.

## Visual System

### Color tokens

Defined as CSS custom properties via Tailwind v4's `@theme` block in
`src/styles/global.css`, replacing every hardcoded hex value currently scattered
across components (`#10002B`, `#240046`, `#5A189A`, `#7B2CBF`, `#3C096C`, etc.).
Keeps the violet brand accent for continuity with the current site, but rebuilds
the neutral scale around a clean monochrome system instead of a single fixed
purple background.

| Token | Light | Dark |
|---|---|---|
| `--color-background` | `#FAFAFA` | `#0A0A0C` |
| `--color-foreground` | `#09090B` | `#F5F5F7` |
| `--color-card` | `#FFFFFF` | `#17171B` |
| `--color-card-foreground` | `#09090B` | `#F5F5F7` |
| `--color-muted` | `#E8ECF0` | `#1F1F24` |
| `--color-muted-foreground` | `#64748B` | `#A1A1AA` |
| `--color-border` | `#E4E4E7` | `#27272A` |
| `--color-accent` | `#7C3AED` | `#A78BFA` |
| `--color-accent-foreground` | `#FFFFFF` | `#09090B` |

All pairs must meet WCAG AA contrast (4.5:1 body text, 3:1 large text/UI) in
both themes — verify during implementation, not just by eyeballing the hex
values above.

### Typography

Keep **Onest Variable** (`@fontsource-variable/onest`, already installed) rather
than introducing a new font. It's already a clean geometric sans in the same
spirit as Inter, so there's no visual-quality reason to add a new font
dependency for a "modern minimal" look. Type scale: 12/14/16/18/24/32/48px.
Body line-height 1.5–1.625 (`leading-relaxed` or equivalent).

### Dark/light toggle mechanism

- Tailwind v4 class-based dark mode: `@custom-variant dark (&:where(.dark, .dark *));`
  in `global.css` (v4 defaults to media-query-based `dark:`, so this override is
  required to support a manual toggle).
- New `ThemeToggle.astro` component: an icon button (sun/moon) that flips a
  `.dark` class on `<html>` and persists the choice to `localStorage`.
- An inline, render-blocking script in `<head>` (in `Layout.astro`) applies the
  saved preference — or falls back to `prefers-color-scheme` if nothing is
  saved yet — before first paint, to avoid a flash of the wrong theme.
- Toggle respects `prefers-reduced-motion` for its own transition animation.

## Structure & Components

Page section order in `index.astro`: **About → Skills (new) → Projects → Contact**,
still wrapped in `Layout` + `SectionContainer` as today.

### New: `Skills.astro`

A distinct grid of tech/tool badges, separate from the per-project tag chips in
`Projects.astro`. Reuses existing icon components (JavaScript, HTML, CSS,
Tailwind, React, NextJS) and adds new icon components for the parts of the
actual stack not currently represented (Astro, TypeScript, Git). Larger than
project tags, with labels always visible (not hover-only).

### `Header.astro`

- Fixes a real responsive bug: the nav currently has no `gap` classes below the
  `lg` breakpoint (only `lg:gap-8 lg:gap-x-4`), so links crowd together on
  mobile/tablet widths.
- Adds a mobile hamburger menu that collapses the nav below ~640px, instead of
  relying on the pill nav shrinking indefinitely.
- Adds active-section styling (`aria-current` + visual state) as the user
  scrolls between `#about` / `#projects` / `#contact`.
- Adds `ThemeToggle` into the nav pill.
- Removes the dead commented-out base64 image blob at the top of the file
  (lines 1–2 today).

### `Projects.astro`

- Sticky-stack scroll effect replaced with a responsive card grid: 1 column
  below the `md` breakpoint (768px), 2 columns at `md` and above.
- Cards keep the existing `PROJECTS` / `TAGS` data shape — restyle only, no
  content changes.
- Restyled onto the token system: consistent border/shadow/radius scale,
  `bg-card`/`text-card-foreground`, hover states with 150–300ms transitions.
- Fixes invalid HTML: `<Button>` is currently nested inside `<a>` for the
  "View Code"/"Visit Website" links, which is invalid (interactive content
  inside interactive content) and confuses keyboard/focus behavior. `Button`
  becomes polymorphic — renders as `<a>` when given an `href` prop, `<button>`
  otherwise — so each control is a single, valid interactive element.
- Removes the unused `--index` inline CSS custom property (set per-card via
  `style={`--index:${idx + 1};`}` today but never consumed by any CSS rule —
  a leftover from the sticky-stack effect being removed).

### `AboutMe.astro`

- Restyled onto the token system.
- Fixes the CV download link: currently `/public/cv.pdf`, which 404s because
  Astro serves the contents of `public/` from the site root — corrected to
  `/cv.pdf`.
- Fixes the email entry in the `SOCIAL` array: currently `link: ""` (goes
  nowhere) — corrected to `mailto:johan.boshell@utp.edu.co`.

### `Contact.astro`

- The copy-to-clipboard control is currently a `<span onclick="...">` — not
  keyboard-focusable, no `role`, no keyboard activation, sub-minimum touch
  target semantics. Converted to a real `<button type="button">` with a visible
  focus ring.
- Fixes an actual functional bug: `handleClick()` reads `input.value` off a
  `<p id="emailPar">` element, which has no `.value` property (that's an
  `<input>`-only API) — clipboard copy silently does nothing today. Switched
  to reading `.textContent` instead.
- Restyled onto the token system.

### `Footer.astro`, `Badge.astro`, `Button.astro`, `SectionContainer.astro`

Restyled onto the token system (no hardcoded hex). Footer's copyright year
switches from the hardcoded `2025` to `new Date().getFullYear()`.

## Deletions

- `src/components/ProjectsOld.astro` — dead code, unused, superseded by
  `Projects.astro`, not imported anywhere in the codebase.
- The commented-out base64 image blob in `Header.astro`.
- The unused `--index` inline style in `Projects.astro` (see above).

## Explicitly out of scope

- `dist/` being tracked in git despite `.gitignore` excluding it — pre-existing,
  unrelated oddity already documented in `CLAUDE.md`; not touched by this work.
- Any build/tooling changes (no new npm dependencies — no icon library swap, no
  font swap, no test framework added).
- Adding an Experience/Education section (considered during brainstorming,
  user chose Skills/Tech Stack instead).
- Automated test coverage — this project has none today, and a static
  portfolio doesn't warrant adding a test framework for a styling/content pass.
  Verified manually instead (see Testing below).
- Internationalization / language toggle (out of scope; not requested for this
  pass).

## Testing / Verification Plan

Manual verification (per the UI/UX pro-max skill's pre-delivery checklist,
scoped to what applies to a static web site):

1. `npm run dev` and manually walk through both themes at 375px, 768px, 1024px,
   and 1440px widths.
2. Contrast check: body/heading text ≥4.5:1, large text/UI ≥3:1, in both
   themes.
3. Visible focus rings on every interactive element (nav links, theme toggle,
   hamburger menu, project card buttons/links, copy-email button) via keyboard
   tab order.
4. No horizontal scroll on mobile; hamburger menu opens/closes and is
   operable via keyboard.
5. Directly verify the three fixed bugs: CV link downloads/opens the PDF,
   mailto link opens a mail client, copy-email button actually places the
   email address on the clipboard.
6. `prefers-reduced-motion` disables/reduces the theme-toggle and hover/entry
   transitions.
7. `npm run build` succeeds with no errors.

No automated tests are added — see "Explicitly out of scope" above.
