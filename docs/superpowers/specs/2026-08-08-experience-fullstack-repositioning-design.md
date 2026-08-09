# Full-Stack Repositioning + Experience Section — Design Spec

Date: 2026-08-08
Status: Approved by user, ready for implementation planning

## Goal

Reposition the portfolio from a frontend-only "Web Developer" framing to reflect
Johan's actual professional profile (Full Stack Developer, per his CV), and add
a new Experience section presenting his work history as a timeline. Also fix a
pre-existing bug where the two CV files' language labels are swapped.

Adding a new project to the Projects section is a related but separate,
smaller follow-up — the user will share that project's details afterward. It
is not blocked by this spec and doesn't need its own design doc (it's a
same-shape data addition to the existing `PROJECTS` array, per the pattern
already established in `Projects.astro`).

## Decisions

Confirmed with the user during brainstorming:

- **Positioning**: reposition the site as Full Stack (not just add a passing
  mention) — update both the About Me bio and the Skills section to reflect
  the CV's actual breadth (Angular, Node.js, NestJS, PostgreSQL, GCP, etc.),
  not just the frontend/React/Astro/Tailwind framing the site had before.
- **CV files**: `cv_english.pdf` and `cv_spanish.pdf` currently have their
  content swapped (verified by reading both PDFs directly) — fix by renaming
  so each filename matches its actual language content.
- **Section order**: About → Skills → **Experience** (new) → Projects →
  Contact.
- **Experience visual style**: a vertical timeline, not cards — thin line on
  the left with accent-colored dots per entry, each split into a label column
  (role in accent color, company, dates + location) and a content column
  (condensed highlight bullets). Modeled on the timeline pattern at
  https://porfolio.dev/'s "Experiencia laboral" section (reference screenshot
  reviewed during brainstorming), adapted to use bullet highlights instead of
  a single paragraph since the CV content is naturally bullet-driven
  (quantified achievements read better as bullets than mashed into prose).
- **Experience content curation**: condense each role's CV bullets from
  6–7 down to the 3–4 most impactful/differentiated ones, rather than
  reproducing the full CV verbatim. A portfolio page benefits from being
  punchier than a resume.
- **Skills curation**: rather than including every CV line item (some — JWT,
  CI/CD, Terraform, Prisma, MySQL, Oracle — are either too granular or
  redundant with broader entries already represented), add a focused set of
  8 new skills to the existing 9, rather than an exhaustive ~20-item list.

## Content

### About Me

New subheadline (in `index.astro`'s `<h1>`, unchanged) stays "Hey, I'm Johan
Boshell" — only `AboutMe.astro`'s content changes.

New sub-headline text (was "Web Developer from Pereira, Colombia"):
> "Full Stack Developer from Pereira, Colombia"

New bio paragraph (was the React/JavaScript/Astro/Tailwind-focused one),
replacing the highlighted inline tech spans with ones grounded in the CV
profile, keeping the existing closing sentence:
> "I design and build web applications across the full stack — from
> responsive interfaces to the APIs and services behind them. My day-to-day
> toolkit spans **Angular**, **TypeScript**, **React**, **Node.js** and
> **NestJS**, working with REST APIs, service-oriented architectures, and
> relational databases like PostgreSQL. I have hands-on experience
> modernizing legacy systems, migrating Firebase-based apps to custom backend
> architectures, and keeping cloud deployments running smoothly. I love
> learning, solving problems, and building products that make a difference."

(Bolded terms above are the inline `text-accent font-medium` highlighted spans,
following the existing markup pattern in `AboutMe.astro`.)

### Skills (add to existing 9)

Existing 9 (kept, unchanged): JavaScript, TypeScript, React, Next.js, Astro,
Tailwind CSS, HTML, CSS, Git.

New 8: **Angular, Node.js, NestJS, PostgreSQL, MongoDB, Docker, GCP, GitLab.**

Each needs a new hand-drawn stroke-icon component (24×24 viewBox,
`stroke="currentColor"`, `{...Astro.props}` spread — matching every existing
icon in `src/components/icons/`), since the project has no icon library
dependency and this stays consistent with that established pattern. These
are simplified/generic representations, not official trademarked logos —
same disclosed approach already used for the Astro/TypeScript/Git icons added
in the earlier redesign.

### Experience (new section)

Two entries, sourced from the English CV (`cv_spanish.pdf` today, which will
become `cv_english.pdf` after the filename fix — see below), condensed per
the curation decision above:

**Full Stack Developer — Prestémonos — Dec 2023 – Present — Dosquebradas,
Colombia**
- Led the evolution of a Firebase-based architecture toward a custom Node.js
  backend, reducing backend coupling by ~30% and improving maintainability
  across 6 core modules.
- Designed and deployed a marketing-automation microservice on Google Cloud
  Platform (GCP), contributing to a 23% increase in requested credits.
- Modernized 2 legacy Angular 6 applications to current Angular versions,
  reducing technical debt and improving frontend tooling compatibility.
- Designed RESTful APIs supporting 40+ endpoints while maintaining ~99%
  service availability.

**Backend Developer — Universidad Tecnológica de Pereira (UTP) — Aug 2025 –
Present — Pereira, Colombia**
- Contributed to the UTP Mobile API's backend, supporting an app with
  10,000+ downloads on Google Play, also distributed on the App Store.
- Designed reusable REST APIs and decoupled service modules consumed by 30+
  institutional applications.
- Developed and maintained PostgreSQL/Oracle database integrations across
  30+ institutional modules.
- Contributed to legacy modernization and delivered 20+ features/releases
  via Scrum and GitLab-based merge-request workflows.

### CV filename fix

`cv_english.pdf` (currently contains the Spanish-language CV content) and
`cv_spanish.pdf` (currently contains the English-language CV content) are
swapped. Fix by renaming so each file's name matches its actual content
language. The `AboutMe.astro` `SOCIAL` array's `CV` entry link (currently
`/cv_spanish.pdf`, i.e. pointing at the English PDF under the wrong name)
must be updated to point at the correctly-renamed English CV file, since
the site is in English.

## Structure & Components

### New: `Experience.astro`

A new component following the same "data array at the top, mapped in the
template" pattern as `Skills.astro`/`Projects.astro`/`AboutMe.astro`. An
`EXPERIENCE` array of `{ role, company, dates, location, highlights: string[] }`
entries, rendered as an ordered list (`<ol>`) with:
- A vertical line (`border-l-2 border-border`) running down the left edge.
- Each entry (`<li>`) has an absolutely-positioned accent-colored dot marking
  its position on the line.
- A two-column layout at `md:` and up (label column: role/company/dates,
  fixed/shrink width; content column: highlight bullets) that stacks to a
  single column on mobile.
- Highlight bullets use `text-muted-foreground`, token-based throughout (no
  raw hex), consistent with every other component from the prior redesign.

### `index.astro`

- Add `import Experience from "../components/Experience.astro"` and a new
  `<SectionContainer id="experience"><h2 class="text-4xl font-semibold mb-6">Experience</h2><Experience /></SectionContainer>`
  block, positioned between the Skills and Projects `SectionContainer`
  blocks (matching the approved section order).
- No heading icon for Experience — matches the existing inconsistency where
  About/Skills headings also have no icon (only Projects/Contact do); not
  worth resolving in this pass since it would require designing yet another
  new icon for a purely cosmetic gap.

### `Header.astro`

- `NAV_LINKS` gains `{ href: '#experience', label: 'Experience' }`, inserted
  between the Skills and Projects entries (matching the approved page order).
- The scrollspy's `sectionIds` array gains `'experience'` in the same
  position. No other change to the scrollspy logic needed — the existing
  `updateActiveSection()` algorithm (rebuilt during the prior redesign's
  final review fix wave) is generic over the number/order of sections and
  requires no special-casing for the new entry.

### `Skills.astro`

- Add 8 new icon imports and 8 new entries to the existing `SKILLS` array,
  in the order listed above. No structural/markup change — the existing
  grid (`grid-cols-2 sm:grid-cols-3`) already reflows automatically for the
  larger item count.

### `AboutMe.astro`

- Replace the sub-headline `<p>` text and the bio `<p>` (including its
  inline highlighted tech spans) per the Content section above. No
  structural change to the component; the `SOCIAL` array (LinkedIn/GitHub/CV/
  email chips) is unaffected by this spec except for the CV link's target
  filename (see CV filename fix above).

### New icon components (8)

`src/components/icons/Angular.astro`, `NodeJS.astro`, `NestJS.astro`,
`PostgreSQL.astro`, `MongoDB.astro`, `Docker.astro`, `GCP.astro`,
`GitLab.astro` — each following the exact established pattern: 24×24
viewBox, `fill="none"`, `stroke="currentColor"`, `stroke-linecap="round"`,
`stroke-linejoin="round"`, `stroke-width="2"`, `{...Astro.props}` spread as
the first attribute.

### CV files

`public/cv_english.pdf` and `public/cv_spanish.pdf` are renamed (content
swapped back to match filename) via a two-step rename (through a temporary
name, since a direct swap would collide) to fix the language mismatch.

## Explicitly out of scope

- The new project the user will share separately — tracked as a follow-up,
  not blocked by or bundled into this spec.
- Any further CV content (education, languages) — not requested for this
  pass; the existing site has no Education/Languages section and none was
  asked for.
- Resolving the pre-existing About/Skills-vs-Projects/Contact heading-icon
  inconsistency — noted above, deliberately left as-is.
- Any visual/token/color-system changes — this pass is content and one new
  section, not a design-system change; all new markup uses the existing
  token classes established in the prior redesign.

## Testing / Verification Plan

Consistent with the project's existing approach (no automated test
framework; verification via `npm run build` plus manual/browser checks):

1. `npm run build` succeeds.
2. Manual check in both light and dark themes: new Experience section
   renders correctly (timeline line/dots visible and using token colors,
   not raw hex), at both mobile (375px, single-column stacking) and desktop
   (1440px, two-column) widths.
3. Nav: "Experience" link appears in the correct position (desktop nav,
   mobile menu), scrolls to the right section, and the scrollspy correctly
   highlights it when that section is in view (regression-check the
   existing About/Skills/Projects/Contact highlighting still works with the
   new section inserted).
4. Skills grid: all 17 items render with visible, correctly-sized icons in
   both themes; new icons follow the same visual language as existing ones
   (no glaring stroke-width/style mismatch).
5. CV link: after the rename, clicking "CV" downloads/opens the English PDF
   (confirm by content, not just filename — e.g. spot-check the visible
   page-1 text) and resolves 200, not 404.
6. About Me: new bio renders with the tech-name spans correctly colored
   (`text-accent`), matching the existing visual pattern.
