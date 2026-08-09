# Projects & Skills Expansion — Design

## Overview

Expand the portfolio's Skills and Projects sections:

- **Skills**: add a years-of-experience value to every skill, reorder the whole grid by proficiency (most experienced first), and add 7 new tools reflecting the current stack (Nx, Terraform, Railway, Netlify, GitHub Actions, Neon, Prisma).
- **Projects**: add three new projects (UTP Móvil, El Impostor, Ice Cream Shop Manager) and reorder all eight projects from most to least complex. Two of the three new projects have no shareable source code or no live demo, which the current data model and template can't express — both need to change.

Every new user-facing string (descriptions, the two new UI labels) follows the existing language-toggle convention: authored in `src/i18n/translations.js` for both `en`/`es`, tagged with `data-i18n`/`data-i18n-attr` in the consuming component. Titles, company/proper names, and tag labels stay untranslated, matching the established rule.

## Skills section

**Data shape.** `SKILLS` entries in `src/components/Skills.astro` gain a `years: number` field:

```js
{ name: "Angular", icon: Angular, years: 3 },
```

Every entry starts at `years: 3` (an explicit placeholder — the user will hand-tune each value afterward). Array order **is** the proficiency order (there's no meaningful sort-by-years while every value is identical); reordering later, if years diverge from the placeholder, is a manual edit to the array, not a computed sort — keeps the component simple and avoids surprising re-ordering the user didn't ask for.

**Proposed order** (24 skills total: 17 existing + 7 new), most proficient first:

Angular, TypeScript, NestJS, Node.js, PostgreSQL, Prisma, JavaScript, Git, Nx, Terraform, Railway, Netlify, GitHub Actions, Neon, GCP, GitLab, Docker, MongoDB, HTML, CSS, Tailwind CSS, Astro, Next.js, React

**Display.** Each card adds a small muted years label next to the name:

```astro
<li class="flex items-center justify-between gap-3 rounded-xl border border-border bg-card px-4 py-3 text-card-foreground">
  <span class="flex items-center gap-3">
    <Icon class="size-6 shrink-0" aria-hidden="true" />
    <span class="font-medium">{name}</span>
  </span>
  <span class="text-xs text-muted-foreground shrink-0">{years}+ <span data-i18n="skills.years">years</span></span>
</li>
```

Only the word "years" is translated (`skills.years` → "años"); the number is plain interpolation, consistent with how tag names and titles elsewhere stay outside `data-i18n` spans.

**New icon components** (`src/components/icons/*.astro`, one per icon, matching the existing stroke-based/`currentColor` Tabler-style convention): `Nx.astro`, `Terraform.astro`, `Railway.astro`, `Netlify.astro`, `GitHubActions.astro`, `Neon.astro`, `Prisma.astro`. Sourcing priority: Tabler Icons first (style match with all existing icons); Simple Icons (CC0) as fallback for anything Tabler doesn't cover, adapted to stroke style where feasible. Exact source per icon is resolved at implementation time within this priority order.

## Projects data model changes

Today every `PROJECTS` entry hardcodes exactly two buttons (View Code from `github`, Visit Website from `link`), both effectively required. Two of the three new projects break that assumption: UTP Móvil has no shareable source at all, and Ice Cream Shop Manager has no live deployment. The model needs to support any combination.

**`src/components/Projects.astro` changes:**

- `github` and `link` become optional per entry.
- New optional `playStore` field.
- New optional `imageFit` field: `"cover"` (default, unchanged behavior) or `"contain"` — needed because the UTP Móvil screenshot is a portrait phone capture (589×1280) that would be destructively cropped inside the existing 16:9 `aspect-video` + `object-cover` card frame. `imageFit: "contain"` keeps the same card footprint (no grid-rhythm break) but letterboxes the image against the card's existing muted background instead of cropping it.
- Template renders each button conditionally:

```astro
{github && (
  <Button href={github} target="_blank" rel="noopener noreferrer" data-i18n="projects.viewCode">View Code</Button>
)}
{link && (
  <Button href={link} target="_blank" rel="noopener noreferrer" data-i18n="projects.visitWebsite">Visit Website</Button>
)}
{playStore && (
  <Button href={playStore} target="_blank" rel="noopener noreferrer" data-i18n="projects.viewPlayStore">View on Play Store</Button>
)}
```

- New `TAGS` entries needed (icons reused from the Skills set where already planned, imported into `Projects.astro` alongside the existing `Javascript`/`CSS`/`HTML`/`Tailwind`/`NextJS`/`React` icons): `ANGULAR`, `NESTJS`, `POSTGRESQL`, `PRISMA`, `RAILWAY`, `NEON`, `SASS`, `JAVA`, `SPRINGBOOT`. `Sass`, `Java`, and `SpringBoot` are new icon components (same Tabler-first/Simple-Icons-fallback sourcing as the Skills icons); `Angular`/`NestJS`/`PostgreSQL` already exist (used in `Skills.astro`) and just need importing here too; `Prisma`/`Railway`/`Neon` are the same new icon files the Skills section also needs — authored once, imported in both places.

## New translation keys (`src/i18n/translations.js`)

```js
// top-level, new "skills" object
skills: {
  years: "years", // es: "años"
},

// under existing "projects" object
projects: {
  viewPlayStore: "View on Play Store", // es: "Ver en Play Store"
  utpMobile: { description: /* see "Three new projects" below for full en/es text */ },
  impostor: { description: /* see "Three new projects" below for full en/es text */ },
  iceCreamWeb: { description: /* see "Three new projects" below for full en/es text */ },
}
```

## Three new projects

### UTP Móvil
- `key: "utpMobile"`, `title: "UTP Móvil"` (untranslated proper noun)
- `image: "/img/utp-app-movil.jpeg"`, `imageFit: "contain"`
- `playStore: "https://play.google.com/store/apps/details?id=co.edu.utp.utpMobile&hl=es_CO"`
- no `github`, no `link`
- `tags: [TAGS.POSTGRESQL]`
- Description (en): *"The official mobile app for Universidad Tecnológica de Pereira, live on Google Play and the App Store with 10,000+ downloads. I contributed the backend REST APIs, integrating multiple institutional microservices and complex PostgreSQL/Oracle queries across 30+ modules. Source is institutional and not publicly shareable."*
- Description (es): *"La aplicación móvil oficial de la Universidad Tecnológica de Pereira, disponible en Google Play y App Store con más de 10.000 descargas. Contribuí en las APIs REST del backend, integrando múltiples microservicios institucionales y consultas complejas en PostgreSQL/Oracle a través de más de 30 módulos. El código es institucional y no se puede compartir públicamente."*

### El Impostor
- `key: "impostor"`, `title: "El Impostor"` (untranslated — the app's own branded title, confirmed from the live page's `<title>`)
- `image`: a fresh screenshot captured from `https://impostorcol.netlify.app/` at implementation time (the setup/settings screen — landscape, so default `imageFit: "cover"` is fine, no special handling needed)
- `link: "https://impostorcol.netlify.app/"`
- `github: "https://github.com/BoshellJohan/imposter-game-app"`
- `tags: [TAGS.REACT, TAGS.SASS, TAGS.JAVA, TAGS.SPRINGBOOT]`
- Description (en): *"A party game for a group sharing one device — everyone gets the same secret word except the impostor, who has to bluff their way through. Built with React and Zustand for state management, backed by a Spring Boot + MongoDB API for the word bank, with a weighted-random selection algorithm that makes repeat impostors progressively less likely."*
- Description (es): *"Un juego de fiesta para jugar en un solo dispositivo — todos los jugadores reciben la misma palabra secreta, excepto el impostor, que debe disimular para no ser descubierto. Construido con React y Zustand para el manejo del estado, respaldado por una API en Spring Boot + MongoDB para el banco de palabras, con un algoritmo de selección ponderada que hace cada vez menos probable repetir como impostor."*

### Ice Cream Shop Manager
- `key: "iceCreamWeb"`, `title: "Ice Cream Shop Manager"` (display title; repo's internal name is "helados-app")
- `image: "/img/dashboard-ice-cream-web.png"` (landscape 1895×855, default `imageFit: "cover"`)
- `github: "https://github.com/BoshellJohan/ice-cream-web"`
- no `link` (no live deployment)
- `tags: [TAGS.ANGULAR, TAGS.NESTJS, TAGS.POSTGRESQL, TAGS.PRISMA, TAGS.RAILWAY, TAGS.NEON]`
- Description (en): *"An internal web app for a real ice cream startup, used daily from a tablet to manage orders, inventory, and product customization — flavors, toppings, and combo pricing. Built with Angular and a NestJS + Prisma + PostgreSQL backend, with role-based access, discount coupons, and sales/cash-reconciliation analytics."*
- Description (es): *"Una aplicación web interna para una startup real de helados, usada a diario desde una tablet para gestionar pedidos, inventario y personalización de productos — sabores, toppings y precios combinados. Construida con Angular y un backend en NestJS + Prisma + PostgreSQL, con acceso por roles, cupones de descuento y analíticas de ventas y conciliación de caja."*

## Final project order (all 8, most → least complex)

1. UTP Móvil
2. Ice Cream Shop Manager
3. El Impostor
4. Tierlist Maker
5. CSS Memorama
6. MineSweeper
7. Weather Query
8. Grocery Bud

(The five existing entries keep their current `key`/`title`/`description`/`link`/`github`/`image`/`tags` — only their position in the `PROJECTS` array changes.)

## Files touched

- `src/components/Skills.astro` — new `years` field + 7 new imports/entries + reorder + template update
- `src/components/Projects.astro` — optional fields, conditional buttons, `imageFit`, new `TAGS` entries, 3 new entries, full reorder
- `src/i18n/translations.js` — new `skills` object, new `projects.viewPlayStore` key, 3 new `projects.<key>.description` pairs (en/es each)
- `src/components/icons/Nx.astro`, `Terraform.astro`, `Railway.astro`, `Netlify.astro`, `GitHubActions.astro`, `Neon.astro`, `Prisma.astro`, `Sass.astro`, `Java.astro`, `SpringBoot.astro` — 10 new icon components
- `public/img/utp-app-movil.jpeg`, `public/img/dashboard-ice-cream-web.png` — already added by the user
- `public/img/<impostor-screenshot>.png` — captured during implementation

No changes to `Layout.astro`'s translation mechanism, the theme toggle, or any other section — this is additive content plus one small template/data-model extension to `Projects.astro`.

## Verification

- `npm run build` succeeds.
- Manual browser check in both languages: Skills grid shows all 24 entries in the new order with years; new project cards render with correct images (UTP Móvil letterboxed, not cropped), correct button sets (UTP Móvil: Play Store only; Ice Cream Shop Manager: View Code only; El Impostor: both), and correct tag icons; existing 5 projects unchanged apart from position; toggling language translates the 3 new descriptions and the new "View on Play Store" label; toggling theme still works normally against the new content.
