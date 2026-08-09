# Full-Stack Repositioning + Experience Section Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Reposition the portfolio as Full Stack (About Me + Skills), add a new Experience timeline section sourced from the CV, and fix the swapped CV filenames.

**Architecture:** Content and one new component, built entirely on the existing token-based design system (no new tokens, no new dependencies). New `Experience.astro` follows the same "data array + map" pattern as `Skills.astro`/`Projects.astro`. 8 new hand-drawn stroke icons follow the exact pattern established by the existing icon set.

**Tech Stack:** Astro 5, Tailwind CSS v4. No new dependencies.

## Global Constraints

- No new npm dependencies.
- Every color in every touched/new file must use the existing token classes (`bg-background`, `text-foreground`, `bg-card`, `text-card-foreground`, `bg-muted`, `text-muted-foreground`, `border-border`, `bg-accent`, `text-accent-foreground`) — no raw hex.
- No automated test framework is added. Verification per task is `npm run build` succeeding plus an explicit manual browser check.
- New icons follow the established pattern exactly: 24×24 viewBox, `fill="none"`, `stroke="currentColor"`, `stroke-linecap="round"`, `stroke-linejoin="round"`, `stroke-width="2"`, `{...Astro.props}` spread as the first attribute. They are simplified/generic representations, not official trademarked logos — consistent with the existing Astro/TypeScript/Git icons.
- This repo has pre-existing tracked build-artifact files (`.astro/*`, `dist/*`) that regenerate on `npm run build`/`npm run dev` and show as modified in `git status`. These are never part of any task's commit — always `git checkout -- .astro dist` to revert them before committing, and never stage them.
- The new project the user will share separately is explicitly out of scope for this plan.

---

## File Structure

| File | Action | Responsibility |
|---|---|---|
| `src/components/icons/Angular.astro` | Create | Skill icon (Angular) |
| `src/components/icons/NodeJS.astro` | Create | Skill icon (Node.js) |
| `src/components/icons/NestJS.astro` | Create | Skill icon (NestJS) |
| `src/components/icons/PostgreSQL.astro` | Create | Skill icon (PostgreSQL) |
| `src/components/icons/MongoDB.astro` | Create | Skill icon (MongoDB) |
| `src/components/icons/Docker.astro` | Create | Skill icon (Docker) |
| `src/components/icons/GCP.astro` | Create | Skill icon (Google Cloud Platform) |
| `src/components/icons/GitLab.astro` | Create | Skill icon (GitLab) |
| `public/cv_english.pdf`, `public/cv_spanish.pdf` | Rename (swap) | Fix swapped CV file content/filename mismatch |
| `src/components/AboutMe.astro` | Modify | Full-stack bio copy; CV link points at the corrected English PDF |
| `src/components/Skills.astro` | Modify | Adds 8 new skill entries using the icons above |
| `src/components/Experience.astro` | Create | New Experience timeline section content |
| `src/pages/index.astro` | Modify | Adds `Experience` section (`id="experience"`) between Skills and Projects |
| `src/components/Header.astro` | Modify | Adds `#experience` to `NAV_LINKS` and the scrollspy's `sectionIds`, in the same position |

---

### Task 1: New skill icon components

**Files:**
- Create: `src/components/icons/Angular.astro`
- Create: `src/components/icons/NodeJS.astro`
- Create: `src/components/icons/NestJS.astro`
- Create: `src/components/icons/PostgreSQL.astro`
- Create: `src/components/icons/MongoDB.astro`
- Create: `src/components/icons/Docker.astro`
- Create: `src/components/icons/GCP.astro`
- Create: `src/components/icons/GitLab.astro`

**Interfaces:**
- Produces: 8 new icon components, each accepting the same prop shape as existing icons (`{...Astro.props}` spread, so callers can pass `class`, `aria-hidden`, etc.).
- Consumes: none (leaf components).
- Consumed by: `Skills.astro` (Task 4).

- [ ] **Step 1: Create `src/components/icons/Angular.astro`**

```astro
<svg
    {...Astro.props}
    xmlns="http://www.w3.org/2000/svg"
    viewBox="0 0 24 24"
    fill="none"
    stroke="currentColor"
    stroke-linecap="round"
    stroke-linejoin="round"
    width="24"
    height="24"
    stroke-width="2"
>
    <path d="M12 3l8 3.5l-1.3 10.5l-6.7 4l-6.7 -4l-1.3 -10.5z"></path>
    <path d="M8.5 15l3.5 -8l3.5 8"></path>
    <path d="M10 12h4"></path>
</svg>
```

- [ ] **Step 2: Create `src/components/icons/NodeJS.astro`**

```astro
<svg
    {...Astro.props}
    xmlns="http://www.w3.org/2000/svg"
    viewBox="0 0 24 24"
    fill="none"
    stroke="currentColor"
    stroke-linecap="round"
    stroke-linejoin="round"
    width="24"
    height="24"
    stroke-width="2"
>
    <path d="M12 3l7.79 4.5v9l-7.79 4.5l-7.79 -4.5v-9z"></path>
</svg>
```

- [ ] **Step 3: Create `src/components/icons/NestJS.astro`**

```astro
<svg
    {...Astro.props}
    xmlns="http://www.w3.org/2000/svg"
    viewBox="0 0 24 24"
    fill="none"
    stroke="currentColor"
    stroke-linecap="round"
    stroke-linejoin="round"
    width="24"
    height="24"
    stroke-width="2"
>
    <path d="M4 4m0 2a2 2 0 0 1 2 -2h12a2 2 0 0 1 2 2v12a2 2 0 0 1 -2 2h-12a2 2 0 0 1 -2 -2z"></path>
    <path d="M9 16v-8l6 8v-8" stroke-width="1.6"></path>
</svg>
```

- [ ] **Step 4: Create `src/components/icons/PostgreSQL.astro`**

```astro
<svg
    {...Astro.props}
    xmlns="http://www.w3.org/2000/svg"
    viewBox="0 0 24 24"
    fill="none"
    stroke="currentColor"
    stroke-linecap="round"
    stroke-linejoin="round"
    width="24"
    height="24"
    stroke-width="2"
>
    <path d="M4 6c0 -1.66 3.58 -3 8 -3s8 1.34 8 3s-3.58 3 -8 3s-8 -1.34 -8 -3z"></path>
    <path d="M4 6v12c0 1.66 3.58 3 8 3s8 -1.34 8 -3v-12"></path>
    <path d="M4 12c0 1.66 3.58 3 8 3s8 -1.34 8 -3"></path>
</svg>
```

- [ ] **Step 5: Create `src/components/icons/MongoDB.astro`**

```astro
<svg
    {...Astro.props}
    xmlns="http://www.w3.org/2000/svg"
    viewBox="0 0 24 24"
    fill="none"
    stroke="currentColor"
    stroke-linecap="round"
    stroke-linejoin="round"
    width="24"
    height="24"
    stroke-width="2"
>
    <path d="M12 2c3 3 5 6.5 5 10a5 5 0 0 1 -10 0c0 -3.5 2 -7 5 -10z"></path>
    <path d="M12 13v9"></path>
</svg>
```

- [ ] **Step 6: Create `src/components/icons/Docker.astro`**

```astro
<svg
    {...Astro.props}
    xmlns="http://www.w3.org/2000/svg"
    viewBox="0 0 24 24"
    fill="none"
    stroke="currentColor"
    stroke-linecap="round"
    stroke-linejoin="round"
    width="24"
    height="24"
    stroke-width="2"
>
    <path d="M4 13h4v4h-4z"></path>
    <path d="M9 13h4v4h-4z"></path>
    <path d="M14 13h4v4h-4z"></path>
    <path d="M9 8h4v4h-4z"></path>
    <path d="M14 8h4v4h-4z"></path>
    <path d="M3 17c1 2.5 4 4 9 4c5.5 0 9 -2.5 10 -6"></path>
</svg>
```

- [ ] **Step 7: Create `src/components/icons/GCP.astro`**

```astro
<svg
    {...Astro.props}
    xmlns="http://www.w3.org/2000/svg"
    viewBox="0 0 24 24"
    fill="none"
    stroke="currentColor"
    stroke-linecap="round"
    stroke-linejoin="round"
    width="24"
    height="24"
    stroke-width="2"
>
    <path d="M6 19a4 4 0 0 1 0 -8a5 5 0 0 1 9.8 -1.5a4.5 4.5 0 0 1 -0.8 9.5z"></path>
</svg>
```

- [ ] **Step 8: Create `src/components/icons/GitLab.astro`**

```astro
<svg
    {...Astro.props}
    xmlns="http://www.w3.org/2000/svg"
    viewBox="0 0 24 24"
    fill="none"
    stroke="currentColor"
    stroke-linecap="round"
    stroke-linejoin="round"
    width="24"
    height="24"
    stroke-width="2"
>
    <path d="M12 21l-7 -5l-1.5 -9l4.5 5z"></path>
    <path d="M12 21l7 -5l1.5 -9l-4.5 5z"></path>
    <path d="M8 12l4 -8l4 8"></path>
</svg>
```

- [ ] **Step 9: Verify the build succeeds**

Run: `npm run build`
Expected: Build completes with no errors. (These icons aren't imported anywhere yet, so this only validates the `.astro` syntax.)

- [ ] **Step 10: Commit**

```bash
git add src/components/icons/Angular.astro src/components/icons/NodeJS.astro src/components/icons/NestJS.astro src/components/icons/PostgreSQL.astro src/components/icons/MongoDB.astro src/components/icons/Docker.astro src/components/icons/GCP.astro src/components/icons/GitLab.astro
git commit -m "feat: add Angular, Node.js, NestJS, PostgreSQL, MongoDB, Docker, GCP, GitLab skill icons"
```

---

### Task 2: Fix swapped CV filenames

**Files:**
- Rename: `public/cv_english.pdf` → `public/cv_spanish.pdf` (content-wise: the file currently named `cv_english.pdf` contains Spanish text; the file currently named `cv_spanish.pdf` contains English text — this task swaps the names so each matches its actual content)

**Interfaces:**
- Consumed by: `AboutMe.astro`'s CV link (Task 3), which will point at `/cv_english.pdf` expecting English content.

- [ ] **Step 1: Swap the two files through a temporary name**

A direct two-way rename isn't possible in one step without a collision, so use a temporary intermediate name:

```bash
git mv public/cv_english.pdf public/cv_temp.pdf
git mv public/cv_spanish.pdf public/cv_english.pdf
git mv public/cv_temp.pdf public/cv_spanish.pdf
```

- [ ] **Step 2: Verify the content now matches the filenames**

Read the first page of `public/cv_english.pdf` and confirm it is in English (starts with "Johan Boshell Longas" / "PROFILE" section, not "PERFIL"). Read the first page of `public/cv_spanish.pdf` and confirm it is in Spanish ("PERFIL" section, not "PROFILE").

- [ ] **Step 3: Verify the build succeeds**

Run: `npm run build`
Expected: Build completes with no errors. (Nothing references these files by path yet in a way the build checks — this just confirms nothing else broke.)

- [ ] **Step 4: Commit**

```bash
git commit -m "fix: swap CV filenames — cv_english.pdf and cv_spanish.pdf had swapped content"
```

(The `git mv` calls in Step 1 already stage the rename; this commit records it. If `git status` shows anything under `.astro/` or `dist/` at this point, run `git checkout -- .astro dist` first so the commit contains only the two renamed files.)

---

### Task 3: `AboutMe` full-stack repositioning

**Files:**
- Modify: `src/components/AboutMe.astro`

**Interfaces:**
- Consumes: the renamed `public/cv_english.pdf` from Task 2 (this task's CV link must point at `/cv_english.pdf`, not the old `/cv_spanish.pdf` path — this is a genuine dependency on Task 2 being done first).
- No structural change to the `SOCIAL` array's shape — only the CV entry's `link` value and the two `<p>` text blocks change.

- [ ] **Step 1: Replace `src/components/AboutMe.astro`**

```astro
---
import LinkedIn from './icons/LinkedIn.astro'
import Github from './icons/Github.astro'
import Mail from './icons/Mail.astro'


const SOCIAL = [
    {
        name: "LinkedIn",
        link: "https://www.linkedin.com/in/johan-boshell-longas-076593283/",
        icon: LinkedIn,
        class: "bg-card text-card-foreground border border-border hover:bg-muted"
    },
    {
        name: "Github",
        link: "https://github.com/BoshellJohan",
        icon: Github,
        class: "bg-card text-card-foreground border border-border hover:bg-muted"
    },
    {
        name: "CV",
        link: "/cv_english.pdf",
        icon: null,
        class: "bg-accent text-accent-foreground hover:opacity-90"
    },

    {
        name: "boshelljohan@gmail.com",
        link: "mailto:boshelljohan@gmail.com",
        icon: Mail,
        class: "bg-card text-card-foreground border border-border hover:bg-muted"
    }
]

---

<article class="mx-auto flex flex-col items-center gap-8 bg-card text-card-foreground border border-border rounded-lg px-8 py-10 mb-16 shadow-sm">
    <!-- Imagen de perfil -->
    <div class="flex flex-col md:flex-row items-center gap-8">
        <img
          src="/img/profile.jpg"
          alt="Photo of Johan"
          class="w-40 h-40 rounded-full object-cover border-4 border-border shadow-md"
          width="160"
          height="160"
        />
        <!-- Profile text -->
        <div class="text-left">
          <p class="text-2xl font-semibold mb-4">Full Stack Developer from Pereira, Colombia</p>
          <p class="text-base leading-relaxed max-w-xl text-muted-foreground">
            I design and build web applications across the full stack — from responsive interfaces to the APIs and services behind them. My day-to-day toolkit spans <span class="text-accent font-medium">Angular</span>, <span class="text-accent font-medium">TypeScript</span>, <span class="text-accent font-medium">React</span>, <span class="text-accent font-medium">Node.js</span> and <span class="text-accent font-medium">NestJS</span>, working with REST APIs, service-oriented architectures, and relational databases like PostgreSQL.
            I have hands-on experience modernizing legacy systems, migrating Firebase-based apps to custom backend architectures, and keeping cloud deployments running smoothly. I love learning, solving problems, and building products that make a difference.
          </p>
        </div>
      </div>


    <!-- Tecnologías -->
    <ul class="flex flex-wrap gap-2 self-start">
        {SOCIAL.map((tag) => (
            <li>
                <span class={`px-3.5 py-2 rounded-lg items-center flex gap-x-2 text-sm font-semibold transition-colors ${tag.class}`}>
                    {tag.icon && <tag.icon class="size-4" aria-hidden="true" />}

                    {tag.name != "CV" && <a href={tag.link} target="_blank" rel="noopener noreferrer">{tag.name}</a>}
                    {tag.name == "CV" && <a href={tag.link} target="_blank" download="CV-JohanBoshell">{tag.name}</a>}
                </span>
            </li>
        ))}
    </ul>
</article>
```

- [ ] **Step 2: Verify the build succeeds**

Run: `npm run build`
Expected: Build completes with no errors.

- [ ] **Step 3: Manual verification**

Run `npm run dev` and confirm:
- The About section shows "Full Stack Developer from Pereira, Colombia" and the new bio paragraph with 5 accent-colored tech spans (Angular, TypeScript, React, Node.js, NestJS).
- Clicking "CV" downloads/opens a PDF whose visible content is in English (not a 404, and not the Spanish version).

- [ ] **Step 4: Commit**

```bash
git add src/components/AboutMe.astro
git commit -m "feat: reposition About Me as Full Stack Developer; fix CV link to renamed English PDF"
```

---

### Task 4: `Skills` full-stack expansion

**Files:**
- Modify: `src/components/Skills.astro`

**Interfaces:**
- Consumes: the 8 new icon components from Task 1.
- No structural/markup change — same grid, same `{SKILLS.map(...)}` pattern, just a longer array.

- [ ] **Step 1: Replace `src/components/Skills.astro`**

```astro
---
import AstroIcon from "./icons/Astro.astro";
import TypeScript from "./icons/TypeScript.astro";
import Git from "./icons/Git.astro";
import Javascript from "./icons/Javascript.astro";
import CSS from "./icons/CSS.astro";
import HTML from "./icons/HTML.astro";
import Tailwind from "./icons/Tailwind.astro";
import NextJS from "./icons/NextJS.astro";
import React from "./icons/React.astro";
import Angular from "./icons/Angular.astro";
import NodeJS from "./icons/NodeJS.astro";
import NestJS from "./icons/NestJS.astro";
import PostgreSQL from "./icons/PostgreSQL.astro";
import MongoDB from "./icons/MongoDB.astro";
import Docker from "./icons/Docker.astro";
import GCP from "./icons/GCP.astro";
import GitLab from "./icons/GitLab.astro";

const SKILLS = [
  { name: "JavaScript", icon: Javascript },
  { name: "TypeScript", icon: TypeScript },
  { name: "React", icon: React },
  { name: "Next.js", icon: NextJS },
  { name: "Astro", icon: AstroIcon },
  { name: "Tailwind CSS", icon: Tailwind },
  { name: "HTML", icon: HTML },
  { name: "CSS", icon: CSS },
  { name: "Git", icon: Git },
  { name: "Angular", icon: Angular },
  { name: "Node.js", icon: NodeJS },
  { name: "NestJS", icon: NestJS },
  { name: "PostgreSQL", icon: PostgreSQL },
  { name: "MongoDB", icon: MongoDB },
  { name: "Docker", icon: Docker },
  { name: "GCP", icon: GCP },
  { name: "GitLab", icon: GitLab },
];
---

<ul class="grid grid-cols-2 sm:grid-cols-3 gap-3">
  {SKILLS.map(({ name, icon: Icon }) => (
    <li class="flex items-center gap-3 rounded-xl border border-border bg-card px-4 py-3 text-card-foreground">
      <Icon class="size-6 shrink-0" aria-hidden="true" />
      <span class="font-medium">{name}</span>
    </li>
  ))}
</ul>
```

- [ ] **Step 2: Verify the build succeeds**

Run: `npm run build`
Expected: Build completes with no errors.

- [ ] **Step 3: Manual verification**

Run `npm run dev` and confirm: the Skills grid now shows 17 items, all 8 new ones render a visible icon (not a blank/broken image) in both light and dark themes, at both mobile (2 columns) and desktop (3 columns) widths.

- [ ] **Step 4: Commit**

```bash
git add src/components/Skills.astro
git commit -m "feat: expand Skills with Angular, Node.js, NestJS, PostgreSQL, MongoDB, Docker, GCP, GitLab"
```

---

### Task 5: `Experience` section (new)

**Files:**
- Create: `src/components/Experience.astro`

**Interfaces:**
- No props. Produces `<Experience />`, consumed by `index.astro` (Task 6) inside a `<SectionContainer id="experience">`.
- No dependency on Tasks 1–4 — this can be implemented independently, though it's sequenced after them in this plan.

- [ ] **Step 1: Create `src/components/Experience.astro`**

```astro
---
const EXPERIENCE = [
  {
    role: "Full Stack Developer",
    company: "Prestémonos",
    dates: "Dec 2023 – Present",
    location: "Dosquebradas, Colombia",
    highlights: [
      "Led the evolution of a Firebase-based architecture toward a custom Node.js backend, reducing backend coupling by ~30% and improving maintainability across 6 core modules.",
      "Designed and deployed a marketing-automation microservice on Google Cloud Platform (GCP), contributing to a 23% increase in requested credits.",
      "Modernized 2 legacy Angular 6 applications to current Angular versions, reducing technical debt and improving frontend tooling compatibility.",
      "Designed RESTful APIs supporting 40+ endpoints while maintaining ~99% service availability.",
    ],
  },
  {
    role: "Backend Developer",
    company: "Universidad Tecnológica de Pereira (UTP)",
    dates: "Aug 2025 – Present",
    location: "Pereira, Colombia",
    highlights: [
      "Contributed to the UTP Mobile API's backend, supporting an app with 10,000+ downloads on Google Play, also distributed on the App Store.",
      "Designed reusable REST APIs and decoupled service modules consumed by 30+ institutional applications.",
      "Developed and maintained PostgreSQL/Oracle database integrations across 30+ institutional modules.",
      "Contributed to legacy modernization and delivered 20+ features/releases via Scrum and GitLab-based merge-request workflows.",
    ],
  },
];
---

<ol class="relative border-l border-border ml-3">
  {EXPERIENCE.map((job) => (
    <li class="mb-10 ml-6 last:mb-0">
      <span class="absolute -left-[7px] mt-1.5 size-3 rounded-full border-2 border-background bg-accent"></span>
      <div class="flex flex-col gap-2 md:flex-row md:gap-8">
        <div class="md:w-64 md:shrink-0">
          <p class="text-lg font-bold text-accent">{job.role}</p>
          <p class="font-semibold">{job.company}</p>
          <p class="text-sm text-muted-foreground">{job.dates}</p>
          <p class="text-sm text-muted-foreground">{job.location}</p>
        </div>
        <ul class="list-inside list-disc space-y-1.5 text-sm leading-relaxed text-muted-foreground">
          {job.highlights.map((highlight) => (
            <li>{highlight}</li>
          ))}
        </ul>
      </div>
    </li>
  ))}
</ol>
```

Note on the dot alignment (`-left-[7px]` on the `size-3` dot against the `border-l` line): this positioning relies on the dot's `<span>` being `absolute` with no explicit `top`, so it falls back to its in-flow ("static-position") vertical placement per each `<li>` — this is the standard technique for this exact timeline pattern and is why each dot lines up with its own entry rather than stacking at the top of the `<ol>`. The `-left-[7px]` horizontal value is a reasonable starting point for a 1px line and a 12px (`size-3`) dot; if it isn't visually centered on the line when you check it in Step 3, adjust the arbitrary value (e.g. `-left-[6px]` or `-left-[8px]`) until it is — this is a cosmetic pixel-alignment detail, not a correctness issue.

- [ ] **Step 2: Verify the build succeeds**

Run: `npm run build`
Expected: Build completes with no errors. (Not wired into `index.astro` yet — Task 6 does that — so this only validates the component's own syntax.)

- [ ] **Step 3: Commit**

```bash
git add src/components/Experience.astro
git commit -m "feat: add Experience timeline component"
```

---

### Task 6: Wire `Experience` into the page and navigation

**Files:**
- Modify: `src/pages/index.astro`
- Modify: `src/components/Header.astro`

**Interfaces:**
- Consumes: `Experience` from Task 5.
- Produces: a live `id="experience"` section reachable from the nav and correctly tracked by the existing scrollspy in `Header.astro` (no scrollspy logic changes needed beyond adding the id to the array — `updateActiveSection()` is generic over the section list).

- [ ] **Step 1: Add the Experience section to `src/pages/index.astro`**

Replace the full contents of `src/pages/index.astro` with:

```astro
---
import Badge from "../components/Badge.astro";
import SectionContainer from "../components/SectionContainer.astro";
import Layout from "../layouts/Layout.astro";
import Projects from "../components/Projects.astro";
import Skills from "../components/Skills.astro";
import Experience from "../components/Experience.astro";
import CodeIcon from "../components/icons/CodeIcon.astro";
import AboutMe from "../components/AboutMe.astro";
import Contact from "../components/Contact.astro";
import Mail from "../components/icons/Mail.astro";
import "../styles/global.css";
---

<Layout
	title="Portfolio de Johan Boshell - Desarrollador y Programador Web"
	description="Contrata a Johan para crear tú aplicación web. Desarrollador web"
>
	<SectionContainer id="about">
		<div class="flex flex-wrap items-center gap-4 mb-4 font-semibold">
			<h1 class="text-4xl">
				Hey, I'm <span class="text-accent">Johan</span> Boshell
			</h1>
			<Badge>Available for hire</Badge>
		</div>
		<AboutMe />
	</SectionContainer>

	<SectionContainer id="skills">
		<h2 class="text-4xl font-semibold mb-6">Skills</h2>
		<Skills />
	</SectionContainer>

	<SectionContainer id="experience">
		<h2 class="text-4xl font-semibold mb-6">Experience</h2>
		<Experience />
	</SectionContainer>

	<SectionContainer id="projects">
		<h2 class="text-4xl font-semibold mb-6 flex gap-x-3 items-center">
			<CodeIcon class="size-10" aria-hidden="true" />
			Projects
		</h2>
		<Projects />
	</SectionContainer>

	<SectionContainer id="contact">
		<h2 class="text-4xl font-semibold mb-6 flex gap-x-3 items-center">
			<Mail class="size-10" aria-hidden="true" />
			Contact
		</h2>
		<Contact />
	</SectionContainer>
</Layout>
```

- [ ] **Step 2: Add `#experience` to `Header.astro`'s nav links and scrollspy**

In `src/components/Header.astro`, find the `NAV_LINKS` array:

```js
const NAV_LINKS = [
    { href: '#about', label: 'About' },
    { href: '#skills', label: 'Skills' },
    { href: '#projects', label: 'Projects' },
    { href: '#contact', label: 'Contact' },
]
```

Replace it with:

```js
const NAV_LINKS = [
    { href: '#about', label: 'About' },
    { href: '#skills', label: 'Skills' },
    { href: '#experience', label: 'Experience' },
    { href: '#projects', label: 'Projects' },
    { href: '#contact', label: 'Contact' },
]
```

Then find the scrollspy's section-id list inside the `<script>`:

```js
var sectionIds = ['about', 'skills', 'projects', 'contact'];
```

Replace it with:

```js
var sectionIds = ['about', 'skills', 'experience', 'projects', 'contact'];
```

Do not change anything else in either file — no other part of the scrollspy logic (`updateActiveSection`, `isAtBottom`, the resize/scroll listeners) needs to change; it already works generically over whatever is in `sectionIds`.

- [ ] **Step 3: Verify the build succeeds**

Run: `npm run build`
Expected: Build completes with no errors.

- [ ] **Step 4: Manual verification**

Run `npm run dev` and confirm, at both mobile (375px) and desktop (1440px):
- "Experience" appears in the nav (desktop inline nav and mobile menu) between "Skills" and "Projects", and clicking it scrolls to the new section with its heading visible below the fixed header (not hidden behind it).
- The new Experience section renders the two-entry timeline: a vertical line with two accent-colored dots, each entry showing role/company/dates/location on the left and highlight bullets on the right (stacked below on mobile), in both light and dark themes.
- Scrolling through the whole page in order (About → Skills → Experience → Projects → Contact) correctly updates which nav link is highlighted as active at each section, including Experience — this is a regression check on the existing scrollspy now that a 5th section exists.
- Scrolling to the very bottom of the page still correctly highlights "Contact" (regression check on the `isAtBottom` fallback with the new section in the list).

- [ ] **Step 5: Commit**

```bash
git add src/pages/index.astro src/components/Header.astro
git commit -m "feat: wire Experience section into the page and nav/scrollspy"
```

---

### Task 7: Full integration pass and manual QA

**Files:** none (verification-only task)

**Interfaces:** none — this task exercises the whole page as assembled by Tasks 1–6.

- [ ] **Step 1: Build check**

Run: `npm run build`
Expected: Build completes with no errors.

- [ ] **Step 2: Full-page theme and responsive QA**

Run `npm run dev` (or `npm run preview` after a build) and check, at 375px, 768px, 1024px, and 1440px, in both light and dark themes:
- About Me: new "Full Stack Developer" subheadline and bio render correctly, CV link opens the correct-language (English) PDF.
- Skills: all 17 items render with visible icons.
- Experience: timeline renders correctly with no layout breakage, dot-to-line alignment looks correct (not obviously offset).
- Nav: "Experience" is in the right position everywhere (desktop nav, mobile menu) and the scrollspy correctly tracks all 5 sections including at the very top and very bottom of the page.
- No horizontal scroll at any width.

- [ ] **Step 3: CV content spot-check**

Open both `/cv_english.pdf` and `/cv_spanish.pdf` directly and confirm each one's visible text content actually matches its filename's language (this directly verifies Task 2's fix held through all subsequent commits).

- [ ] **Step 4: Final commit (if any QA fixes were needed)**

If Step 2 or 3 turned up any issues, fix them in the relevant file(s) and commit:

```bash
git add -A
git commit -m "fix: address issues found in full-stack repositioning QA"
```

If no issues were found, no commit is needed for this task.
