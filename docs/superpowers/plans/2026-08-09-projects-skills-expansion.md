# Projects & Skills Expansion Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add years-of-experience to the Skills grid (reordered by proficiency, 7 new tools), and add three new Projects entries (UTP Móvil, El Impostor, Ice Cream Shop Manager) with a full 8-project reorder by complexity.

**Architecture:** Additive content plus one small data-model/template extension to `Projects.astro` (optional `github`/`link`, new optional `playStore` and `imageFit` fields, conditional button rendering) so it can represent projects without shareable source or without a live demo. All new user-facing copy goes through the existing `translations.js` + `data-i18n`/`data-i18n-attr` convention. 10 new icon components follow the existing one-file-per-icon pattern.

**Tech Stack:** Astro 5, Tailwind CSS 4 (no config file), the existing `src/i18n/translations.js` + `applyTranslations()` mechanism (unchanged by this plan).

## Global Constraints

- No new npm dependencies.
- No automated test framework exists in this repo — verification is `npm run build` succeeding plus manual browser checks (per `CLAUDE.md`).
- `dist/`/`.astro/` build artifacts must never be staged/committed.
- New icon components (`src/components/icons/*.astro`) must match the existing wrapper convention: `<svg {...Astro.props} xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" ...>` accepting a `class` prop via spread, one icon per file, PascalCase filename matching the component's default export name.
- Icon sourcing priority: Tabler Icons first (stroke-based, `fill="none" stroke="currentColor" stroke-width="2"`, matching every existing icon in this repo) for style consistency; Simple Icons (CC0-licensed, filled `fill="currentColor"` single-path style) as fallback for any tool Tabler doesn't cover. Where a fallback is used, its filled style is a deliberate, accepted deviation — do not attempt to hand-redraw a brand mark in stroke style, that would misrepresent the actual logo.
- Titles, company/proper names, and tag labels stay untranslated (existing rule, established in the language-toggle work) — only descriptions and the new `projects.viewPlayStore` UI label get `data-i18n` tags.
- Tag chip styling for every new `TAGS` entry is `"bg-muted text-muted-foreground"`, matching every existing tag (no per-tag color variation exists in this codebase).

---

### Task 1: New icon components

**Files:**
- Create: `src/components/icons/Nx.astro`
- Create: `src/components/icons/Terraform.astro`
- Create: `src/components/icons/Railway.astro`
- Create: `src/components/icons/Netlify.astro`
- Create: `src/components/icons/GitHubActions.astro`
- Create: `src/components/icons/Neon.astro`
- Create: `src/components/icons/Prisma.astro`
- Create: `src/components/icons/Sass.astro`
- Create: `src/components/icons/Java.astro`
- Create: `src/components/icons/SpringBoot.astro`

**Interfaces:**
- Produces: 10 Astro components, each with no props of their own beyond the spread `Astro.props` (so callers can pass `class`/`aria-hidden` exactly like every existing icon, e.g. `<Nx class="size-6 shrink-0" aria-hidden="true" />`). Tasks 4 and 5 import these by exact filename.

- [ ] **Step 1: Look at the existing pattern**

Read `src/components/icons/NestJS.astro` and `src/components/icons/GitLab.astro` — both are Tabler Icons "brand-*" icons already in this repo, wrapped like this:

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
    <path d="..."></path>
</svg>
```

- [ ] **Step 2: Source each icon, in this order of preference**

For each of the 10 tools (Nx, Terraform, Railway, Netlify, GitHub Actions, Neon, Prisma, Sass, Java, Spring Boot):

1. **Try Tabler Icons first.** Tabler's brand icon set uses the filename pattern `brand-<slug>.svg` (e.g. `brand-terraform.svg`, `brand-netlify.svg`, `brand-sass.svg`, `brand-github.svg`). Check `https://raw.githubusercontent.com/tabler/tabler-icons/main/icons/outline/brand-<slug>.svg` (try the tool's obvious lowercase slug — `terraform`, `netlify`, `sass`, `githubactions` or `github-actions`, `java`, `springboot` or `spring-boot`, `nx`, `neon`, `prisma`). If it 404s, try one or two obvious slug variants before concluding it doesn't exist in Tabler.
2. **If Tabler doesn't have it, use Simple Icons.** Check `https://raw.githubusercontent.com/simple-icons/simple-icons/develop/icons/<slug>.svg` (slugs are lowercase, no spaces — e.g. `nx.svg`, `terraform.svg`, `railway.svg`, `netlify.svg`, `githubactions.svg`, `prisma.svg`, `sass.svg`, `openjdk.svg` for Java, `springboot.svg` or `spring.svg`). Simple Icons' Neon slug may collide with unrelated tools named "Neon" — verify the fetched SVG actually represents Neon (the serverless Postgres platform, not the design tool or the game) before using it; if ambiguous or unavailable, it's acceptable to author a simple, generic "database/lightning-bolt" glyph by hand as a last resort, but only after confirming no accurate source exists.
3. **Extract the path data.** For a Tabler hit, copy the inner `<path>` element(s) verbatim into the wrapper template from Step 1 (same `viewBox`, `stroke`, etc. — only the filename and inner paths change). For a Simple Icons hit, wrap the raw `<path fill="currentColor" d="...">` in a simpler template:

```astro
<svg
    {...Astro.props}
    xmlns="http://www.w3.org/2000/svg"
    viewBox="0 0 24 24"
    fill="currentColor"
>
    <path d="..."></path>
</svg>
```

(Simple Icons SVGs are natively `viewBox="0 0 24 24"` already; if a fetched icon uses a different viewBox, keep its own viewBox rather than distorting the path.)

- [ ] **Step 3: Write all 10 files**

Create each of the 10 files listed above using the sourced path data in the appropriate template (stroke-style for Tabler hits, filled-style for Simple Icons hits). Component name matches filename (e.g. `Nx.astro`'s content is just the `<svg>` markup — Astro infers the component name from the filename, no explicit export needed, matching every existing icon file).

- [ ] **Step 4: Visually verify each icon**

Run `npm run dev` and temporarily render all 10 new icons in a scratch spot (e.g. temporarily add them to the top of `src/pages/index.astro` inside a throwaway `<div>`, or use a browser tool to check each `.astro` file's markup visually renders a recognizable mark, not a blank/broken shape) — then remove the scratch markup before moving on. Confirm no icon is a blank square, a giant/tiny shape outside its viewBox, or visually wrong for its brand. Discard the scratch edit; it must not be committed.

- [ ] **Step 5: Verify the build succeeds**

Run: `npm run build`
Expected: Build completes with no errors.

- [ ] **Step 6: Commit**

```bash
git add src/components/icons/Nx.astro src/components/icons/Terraform.astro src/components/icons/Railway.astro src/components/icons/Netlify.astro src/components/icons/GitHubActions.astro src/components/icons/Neon.astro src/components/icons/Prisma.astro src/components/icons/Sass.astro src/components/icons/Java.astro src/components/icons/SpringBoot.astro
git commit -m "feat: add icon components for Nx, Terraform, Railway, Netlify, GitHub Actions, Neon, Prisma, Sass, Java, Spring Boot"
```

---

### Task 2: Translation data — Skills years label, Play Store button label, 3 new project descriptions

**Files:**
- Modify: `src/i18n/translations.js`

**Interfaces:**
- Produces: `translations.en.skills.years` / `translations.es.skills.years`; `translations.en.projects.viewPlayStore` / `translations.es.projects.viewPlayStore`; `translations.en.projects.utpMobile.description` / `.impostor.description` / `.iceCreamWeb.description` (and the matching `es` values). Tasks 4 and 5 consume these exact dot-paths.

- [ ] **Step 1: Add the new `skills` object**

In `src/i18n/translations.js`, inside the `en: { ... }` top-level object, add a new `skills` key. Place it anywhere among the other top-level keys (e.g. right after `header`):

```js
    skills: {
      years: "years",
    },
```

In the `es: { ... }` object, at the matching position:

```js
    skills: {
      years: "años",
    },
```

- [ ] **Step 2: Add `viewPlayStore` to the existing `projects` object (both languages)**

In `en.projects`, add alongside the existing `viewCode`/`visitWebsite`:

```js
      viewPlayStore: "View on Play Store",
```

In `es.projects`:

```js
      viewPlayStore: "Ver en Play Store",
```

- [ ] **Step 3: Add the 3 new project description entries to `en.projects`**

Add these three entries into `en.projects`, alongside the existing `weatherQuery`/`groceryBud`/etc. entries:

```js
      utpMobile: {
        description: "The official mobile app for Universidad Tecnológica de Pereira, live on Google Play and the App Store with 10,000+ downloads. I contributed the backend REST APIs, integrating multiple institutional microservices and complex PostgreSQL/Oracle queries across 30+ modules. Source is institutional and not publicly shareable.",
      },
      impostor: {
        description: "A party game for a group sharing one device — everyone gets the same secret word except the impostor, who has to bluff their way through. Built with React and Zustand for state management, backed by a Spring Boot + MongoDB API for the word bank, with a weighted-random selection algorithm that makes repeat impostors progressively less likely.",
      },
      iceCreamWeb: {
        description: "An internal web app for a real ice cream startup, used daily from a tablet to manage orders, inventory, and product customization — flavors, toppings, and combo pricing. Built with Angular and a NestJS + Prisma + PostgreSQL backend, with role-based access, discount coupons, and sales/cash-reconciliation analytics.",
      },
```

- [ ] **Step 4: Add the matching 3 entries to `es.projects`**

```js
      utpMobile: {
        description: "La aplicación móvil oficial de la Universidad Tecnológica de Pereira, disponible en Google Play y App Store con más de 10.000 descargas. Contribuí en las APIs REST del backend, integrando múltiples microservicios institucionales y consultas complejas en PostgreSQL/Oracle a través de más de 30 módulos. El código es institucional y no se puede compartir públicamente.",
      },
      impostor: {
        description: "Un juego de fiesta para jugar en un solo dispositivo — todos los jugadores reciben la misma palabra secreta, excepto el impostor, que debe disimular para no ser descubierto. Construido con React y Zustand para el manejo del estado, respaldado por una API en Spring Boot + MongoDB para el banco de palabras, con un algoritmo de selección ponderada que hace cada vez menos probable repetir como impostor.",
      },
      iceCreamWeb: {
        description: "Una aplicación web interna para una startup real de helados, usada a diario desde una tablet para gestionar pedidos, inventario y personalización de productos — sabores, toppings y precios combinados. Construida con Angular y un backend en NestJS + Prisma + PostgreSQL, con acceso por roles, cupones de descuento y analíticas de ventas y conciliación de caja.",
      },
```

- [ ] **Step 5: Verify the build succeeds**

Run: `npm run build`
Expected: Build completes with no errors (this file isn't consumed by anything new yet, so this just checks for a JS syntax error in the edit).

- [ ] **Step 6: Commit**

```bash
git add src/i18n/translations.js
git commit -m "feat: add translation keys for skills years label, Play Store button, and 3 new project descriptions"
```

---

### Task 3: Capture the El Impostor live screenshot

**Files:**
- Create: `public/img/impostor-settings.png`

**Interfaces:**
- Produces: `public/img/impostor-settings.png`, referenced by exact path in Task 5's `impostor` project entry.

- [ ] **Step 1: Capture the screenshot**

Using a browser automation tool, navigate to `https://impostorcol.netlify.app/` and wait for the page to fully load (the backend runs on a free tier and can take 30–60 seconds to wake up on first load — wait for the "Jugadores"/"Categorías" setup screen to actually render before capturing, not just for network idle). Take a viewport screenshot and save it to `public/img/impostor-settings.png`. This is the game's setup/settings screen — a landscape layout, so it needs no special handling in Task 5 (default `imageFit: "cover"` applies).

- [ ] **Step 2: Verify the file**

Confirm `public/img/impostor-settings.png` exists and, opened directly, shows a legible landscape screenshot of the game's setup screen (player list, categories, options) — not a blank/loading/error page.

- [ ] **Step 3: Commit**

```bash
git add public/img/impostor-settings.png
git commit -m "feat: add El Impostor live screenshot"
```

---

### Task 4: Skills.astro — years field, 7 new tools, reorder, display

**Files:**
- Modify: `src/components/Skills.astro`

**Interfaces:**
- Consumes: the 7 new icon components from Task 1 (`Nx`, `Terraform`, `Railway`, `Netlify`, `GitHubActions`, `Neon`, `Prisma`); `translations.en.skills.years` / `translations.es.skills.years` from Task 2 (via the `data-i18n="skills.years"` tag — no direct JS import needed here, translation happens client-side).

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
import Nx from "./icons/Nx.astro";
import Terraform from "./icons/Terraform.astro";
import Railway from "./icons/Railway.astro";
import Netlify from "./icons/Netlify.astro";
import GitHubActions from "./icons/GitHubActions.astro";
import Neon from "./icons/Neon.astro";
import Prisma from "./icons/Prisma.astro";

const SKILLS = [
  { name: "Angular", icon: Angular, years: 3 },
  { name: "TypeScript", icon: TypeScript, years: 3 },
  { name: "NestJS", icon: NestJS, years: 3 },
  { name: "Node.js", icon: NodeJS, years: 3 },
  { name: "PostgreSQL", icon: PostgreSQL, years: 3 },
  { name: "Prisma", icon: Prisma, years: 3 },
  { name: "JavaScript", icon: Javascript, years: 3 },
  { name: "Git", icon: Git, years: 3 },
  { name: "Nx", icon: Nx, years: 3 },
  { name: "Terraform", icon: Terraform, years: 3 },
  { name: "Railway", icon: Railway, years: 3 },
  { name: "Netlify", icon: Netlify, years: 3 },
  { name: "GitHub Actions", icon: GitHubActions, years: 3 },
  { name: "Neon", icon: Neon, years: 3 },
  { name: "GCP", icon: GCP, years: 3 },
  { name: "GitLab", icon: GitLab, years: 3 },
  { name: "Docker", icon: Docker, years: 3 },
  { name: "MongoDB", icon: MongoDB, years: 3 },
  { name: "HTML", icon: HTML, years: 3 },
  { name: "CSS", icon: CSS, years: 3 },
  { name: "Tailwind CSS", icon: Tailwind, years: 3 },
  { name: "Astro", icon: AstroIcon, years: 3 },
  { name: "Next.js", icon: NextJS, years: 3 },
  { name: "React", icon: React, years: 3 },
];
---

<ul class="grid grid-cols-2 sm:grid-cols-3 gap-3">
  {SKILLS.map(({ name, icon: Icon, years }) => (
    <li class="flex items-center justify-between gap-3 rounded-xl border border-border bg-card px-4 py-3 text-card-foreground">
      <span class="flex items-center gap-3">
        <Icon class="size-6 shrink-0" aria-hidden="true" />
        <span class="font-medium">{name}</span>
      </span>
      <span class="text-xs text-muted-foreground shrink-0">{years}+ <span data-i18n="skills.years">years</span></span>
    </li>
  ))}
</ul>
```

Every skill's `years` is the literal number `3` — an explicit placeholder the user will hand-tune per skill afterward. Do not try to infer "more accurate" per-skill values; use `3` for all 24 exactly as shown.

- [ ] **Step 2: Verify the build succeeds**

Run: `npm run build`
Expected: Build completes with no errors.

- [ ] **Step 3: Manual verification**

Run `npm run dev`. Confirm: the Skills grid shows 24 entries in the order listed above; each card shows its icon, name, and "3+ years" in muted text on the right; none of the 7 new icons render as blank/broken. Toggle to Spanish and confirm "years" becomes "años" (e.g. "3+ años") on every card, while the number and skill names stay unchanged. Toggle back to English and confirm restoration.

- [ ] **Step 4: Commit**

```bash
git add src/components/Skills.astro
git commit -m "feat: add years-of-experience display, 7 new tools, reorder Skills by proficiency"
```

---

### Task 5: Projects.astro — optional link fields, 3 new projects, full reorder

**Files:**
- Modify: `src/components/Projects.astro`

**Interfaces:**
- Consumes: `Angular`, `NestJS`, `PostgreSQL`, `Prisma`, `Railway`, `Neon`, `Sass`, `Java`, `SpringBoot` icon components (the first three already exist; the rest come from Task 1); `translations.en.projects.viewPlayStore`, `.utpMobile.description`, `.impostor.description`, `.iceCreamWeb.description` from Task 2; `public/img/impostor-settings.png` from Task 3; `public/img/utp-app-movil.jpeg` and `public/img/dashboard-ice-cream-web.png` (already present in the repo, added by the user before this plan).

- [ ] **Step 1: Replace `src/components/Projects.astro`**

```astro
---
import Javascript from "./icons/Javascript.astro";
import CSS from "./icons/CSS.astro";
import HTML from "./icons/HTML.astro";
import Tailwind from "./icons/Tailwind.astro";
import NextJS from "./icons/NextJS.astro";
import React from "./icons/React.astro";
import Angular from "./icons/Angular.astro";
import NestJS from "./icons/NestJS.astro";
import PostgreSQL from "./icons/PostgreSQL.astro";
import Prisma from "./icons/Prisma.astro";
import Railway from "./icons/Railway.astro";
import Neon from "./icons/Neon.astro";
import Sass from "./icons/Sass.astro";
import Java from "./icons/Java.astro";
import SpringBoot from "./icons/SpringBoot.astro";
import { translations } from "../i18n/translations.js";

import Button from "./Button.astro";

const TAGS = {
  JAVASCRIPT: {
    name: "JavaScript",
    class: "bg-muted text-muted-foreground",
    icon: Javascript,
  },
  CSS: {
    name: "CSS",
    class: "bg-muted text-muted-foreground",
    icon: CSS,
  },
  TAILWIND: {
    name: "Tailwind",
    class: "bg-muted text-muted-foreground",
    icon: Tailwind,
  },
  NEXTJS: {
    name: "NextJS",
    class: "bg-muted text-muted-foreground",
    icon: NextJS,
  },
  HTML: {
    name: "HTML",
    class: "bg-muted text-muted-foreground",
    icon: HTML,
  },
  REACT: {
    name: "React",
    class: "bg-muted text-muted-foreground",
    icon: React,
  },
  ANGULAR: {
    name: "Angular",
    class: "bg-muted text-muted-foreground",
    icon: Angular,
  },
  NESTJS: {
    name: "NestJS",
    class: "bg-muted text-muted-foreground",
    icon: NestJS,
  },
  POSTGRESQL: {
    name: "PostgreSQL",
    class: "bg-muted text-muted-foreground",
    icon: PostgreSQL,
  },
  PRISMA: {
    name: "Prisma",
    class: "bg-muted text-muted-foreground",
    icon: Prisma,
  },
  RAILWAY: {
    name: "Railway",
    class: "bg-muted text-muted-foreground",
    icon: Railway,
  },
  NEON: {
    name: "Neon",
    class: "bg-muted text-muted-foreground",
    icon: Neon,
  },
  SASS: {
    name: "Sass",
    class: "bg-muted text-muted-foreground",
    icon: Sass,
  },
  JAVA: {
    name: "Java",
    class: "bg-muted text-muted-foreground",
    icon: Java,
  },
  SPRINGBOOT: {
    name: "Spring Boot",
    class: "bg-muted text-muted-foreground",
    icon: SpringBoot,
  },
};

const en = translations.en.projects;

const PROJECTS = [
  {
    key: "utpMobile",
    title: "UTP Móvil",
    description: en.utpMobile.description,
    playStore: "https://play.google.com/store/apps/details?id=co.edu.utp.utpMobile&hl=es_CO",
    image: "/img/utp-app-movil.jpeg",
    imageFit: "contain",
    tags: [TAGS.POSTGRESQL],
  },
  {
    key: "iceCreamWeb",
    title: "Ice Cream Shop Manager",
    description: en.iceCreamWeb.description,
    github: "https://github.com/BoshellJohan/ice-cream-web",
    image: "/img/dashboard-ice-cream-web.png",
    tags: [TAGS.ANGULAR, TAGS.NESTJS, TAGS.POSTGRESQL, TAGS.PRISMA, TAGS.RAILWAY, TAGS.NEON],
  },
  {
    key: "impostor",
    title: "El Impostor",
    description: en.impostor.description,
    link: "https://impostorcol.netlify.app/",
    github: "https://github.com/BoshellJohan/imposter-game-app",
    image: "/img/impostor-settings.png",
    tags: [TAGS.REACT, TAGS.SASS, TAGS.JAVA, TAGS.SPRINGBOOT],
  },
  {
    key: "tierlistMaker",
    title: "Tierlist Maker",
    description: en.tierlistMaker.description,
    link: "https://tierlistmaker-boshelljohan.netlify.app/",
    github: "https://github.com/BoshellJohan",
    image: "/img/tierlist-maker.png",
    tags: [TAGS.JAVASCRIPT, TAGS.HTML, TAGS.CSS],
  },
  {
    key: "cssMemorama",
    title: "CSS Memorama",
    description: en.cssMemorama.description,
    link: "https://memorama-boshelljohan.netlify.app/",
    github: "https://github.com/BoshellJohan/Memograma-Final-Version",
    image: "/img/Memorama.png",
    tags: [TAGS.HTML, TAGS.CSS],
  },
  {
    key: "minesweeper",
    title: "MineSweeper",
    description: en.minesweeper.description,
    link: "https://minesweeper-boshell.netlify.app/",
    github: "https://github.com/BoshellJohan/Minesweeper",
    image: "/img/Minesweeper.png",
    tags: [TAGS.JAVASCRIPT, TAGS.HTML, TAGS.CSS],
  },
  {
    key: "weatherQuery",
    title: "Weather Query",
    description: en.weatherQuery.description,
    link: "https://weatherapi-boshell.netlify.app/",
    github: "https://github.com/BoshellJohan/Weather_API",
    image: "/img/weatherQuery.png",
    tags: [TAGS.NEXTJS, TAGS.TAILWIND],
  },
  {
    key: "groceryBud",
    title: "Grocery Bud",
    description: en.groceryBud.description,
    link: "https://grocerybud-boshelljohan.netlify.app/",
    github: "https://github.com/BoshellJohan/GroceryBud",
    image: "/img/GroceryBud.png",
    tags: [TAGS.REACT],
  },
];
---

<ul class="w-full grid grid-cols-1 md:grid-cols-2 gap-6">
  {
    PROJECTS.map(({ key, image, title, description, tags, link, github, playStore, imageFit = "cover" }) => (
      <li>
        <article class="flex flex-col h-full bg-card text-card-foreground border border-border rounded-xl p-6 shadow-sm hover:shadow-md transition-shadow">
          <div class="rounded-lg overflow-hidden border border-border mb-4 aspect-video bg-muted">
            <img
              class={`w-full h-full ${imageFit === "contain" ? "object-contain" : "object-cover"}`}
              src={image}
              alt={title}
              loading="lazy"
              width="640"
              height="360"
            />
          </div>

          <h3 class="text-xl font-bold mb-2">{title}</h3>
          <p class="text-muted-foreground leading-relaxed mb-4" data-i18n={`projects.${key}.description`}>{description}</p>

          <ul class="flex flex-wrap gap-2 mb-6">
            {tags.map((tag) => (
              <li>
                <span
                  class={`px-3 py-1.5 rounded-lg flex gap-2 text-xs items-center ${tag.class}`}
                >
                  <tag.icon class="size-4" aria-hidden="true" />
                  <span class="font-semibold">{tag.name}</span>
                </span>
              </li>
            ))}
          </ul>

          <div class="flex gap-3 flex-wrap mt-auto">
            {github && (
              <Button href={github} target="_blank" rel="noopener noreferrer" data-i18n="projects.viewCode">
                View Code
              </Button>
            )}
            {link && (
              <Button href={link} target="_blank" rel="noopener noreferrer" data-i18n="projects.visitWebsite">
                Visit Website
              </Button>
            )}
            {playStore && (
              <Button href={playStore} target="_blank" rel="noopener noreferrer" data-i18n="projects.viewPlayStore">
                View on Play Store
              </Button>
            )}
          </div>
        </article>
      </li>
    ))
  }
</ul>
```

Note the button row gained `flex-wrap` (was `flex gap-3 mt-auto`, now `flex gap-3 flex-wrap mt-auto`) — a small defensive addition so a card with three buttons (none of the current 8 projects has three, but the model now supports it) wraps gracefully on narrow cards instead of overflowing.

- [ ] **Step 2: Verify the build succeeds**

Run: `npm run build`
Expected: Build completes with no errors.

- [ ] **Step 3: Manual verification**

Run `npm run dev`. Confirm, in this order:
- All 8 project cards render in the order: UTP Móvil, Ice Cream Shop Manager, El Impostor, Tierlist Maker, CSS Memorama, MineSweeper, Weather Query, Grocery Bud.
- UTP Móvil's image shows the full portrait screenshot letterboxed (not cropped) inside the card, and only a "View on Play Store" button appears (no View Code, no Visit Website).
- Ice Cream Shop Manager's image fills the card normally (landscape, cropped-to-fill is fine), and only a "View Code" button appears (no Visit Website, no Play Store).
- El Impostor's image fills the card normally, and both "View Code" and "Visit Website" buttons appear, no Play Store button.
- The 5 pre-existing projects are visually unchanged apart from their new position in the grid.
- All new tag chips (PostgreSQL, Angular, NestJS, Prisma, Railway, Neon, React, Sass, Java, Spring Boot) render a recognizable icon, not a blank shape.
- Toggle to Spanish: the 3 new descriptions translate, "View on Play Store" becomes "Ver en Play Store", the "View Code"/"Visit Website" labels on the new cards translate exactly like they already do on the 5 existing cards. Toggle back to English and confirm restoration.
- Toggle dark/light theme with the new cards visible — no visual regressions.

- [ ] **Step 4: Commit**

```bash
git add src/components/Projects.astro
git commit -m "feat: add UTP Móvil, El Impostor, and Ice Cream Shop Manager projects; reorder by complexity"
```

---

### Task 6: Full integration QA pass

**Files:** none (verification-only task)

**Interfaces:** none — this task exercises the whole page as assembled by Tasks 1–5.

- [ ] **Step 1: Build check**

Run: `npm run build`
Expected: Build completes with no errors.

- [ ] **Step 2: Full-page visual QA**

Run `npm run dev` (or `npm run preview` after a build). Load the page fresh (no stored `lang`/`theme`), and check:
- Skills section: 24 entries, correct order, each showing "3+ years" (English) / "3+ años" (Spanish after toggling).
- Projects section: 8 cards in the new order, correct buttons per project (UTP Móvil: Play Store only; Ice Cream Shop Manager: View Code only; El Impostor: both View Code and Visit Website; the 5 existing projects unchanged), correct images (UTP Móvil not cropped), correct tags.
- Language toggle affects all of the above correctly and instantly, with no page reload, and toggling back to English restores everything exactly.
- Theme toggle still works correctly with the new content visible in both themes.
- At 375px width: Skills grid and Projects grid both remain usable (no horizontal overflow, cards/buttons don't overlap or get cut off) — check specifically the 3-button case doesn't exist yet, but the wrapped 1- and 2-button cases (UTP Móvil, Ice Cream Shop Manager) look correct.
- Scrollspy, anchor-nav, and the rest of the existing site (About, Experience, Contact, Footer) are unaffected — quick spot check, not a full re-QA (out of scope for this plan; nothing in Tasks 1–5 touches those files).

- [ ] **Step 3: Final commit (if any QA fixes were needed)**

If any of the above turned up issues, fix them in the relevant file(s) and commit:

```bash
git add -A
git commit -m "fix: address issues found in projects/skills expansion QA"
```

If no issues were found, no commit is needed for this task.
