# English/Spanish Language Toggle — Design Spec

Date: 2026-08-08
Status: Approved by user, ready for implementation planning

## Goal

Add a site-wide English/Spanish language toggle, matching the instant,
no-reload UX of the existing dark/light theme toggle. The CV download link
must switch between the two CV PDFs (`cv_english.pdf`/`cv_spanish.pdf`)
along with the language. All visible body content is translated — nav,
headings, About Me, Experience, Projects descriptions, Contact, Footer, and
the page title/meta description. Technology names (React, PostgreSQL,
Angular, etc.), company names (Prestémonos, Universidad Tecnológica de
Pereira), and project titles (Weather Query, Grocery Bud, etc.) are treated
as proper nouns and stay unchanged in both languages.

## Decisions

Confirmed with the user during brainstorming:

- **Mechanism**: instant client-side toggle, no page reload — same pattern
  as the dark/light theme toggle. Choice persists via `localStorage`.
- **Default language**: English, for first-time visitors with no stored
  preference.
- **Translation scope**: everything — nav, headings, About Me bio,
  Experience highlights, Projects descriptions, Contact/Footer text, and
  page title/meta. Tech names, company names, and project titles are not
  translated (treated as proper nouns).
- **Data architecture**: centralized translation data (`src/i18n/translations.js`,
  one `{ en: {...}, es: {...} }` object keyed by dot-path), matching the
  existing `PROJECTS`/`SKILLS`/`EXPERIENCE` data-array pattern already used
  throughout the site — not a dual-language-markup/CSS-toggle approach.
  Elements are tagged one of two ways: `data-i18n="key"` (text content, via
  `innerHTML` so rich content like the About Me bio's embedded colored
  tech-name spans works), or `data-i18n-attr="attribute:key"` (sets an
  arbitrary attribute instead of innerHTML — covers the CV link's `href`,
  and see the dynamic-aria-label note below).
- **Dynamic aria-labels need an extra layer.** `ThemeToggle`'s
  "Switch to dark/light theme" label and the mobile hamburger's
  "Open/Close menu" label are not static — each is already set by existing
  JS based on current state (which theme is active; whether the menu is
  open). A plain `data-i18n-attr` tag on `aria-label` would get overwritten
  the next time that existing state-change code runs. Fix: give each button
  static `data-label-*` attributes holding the current-language phrase for
  each state (e.g. `data-label-dark`/`data-label-light`,
  `data-label-open`/`data-label-closed`), translated via `data-i18n-attr`
  same as any other attribute, and change the existing state-change
  functions (`ThemeToggle`'s `labelFor`, `Header`'s `openMenu`/`closeMenu`)
  to read the phrase from those data attributes instead of a hardcoded
  string literal. This is a small, targeted change to already-shipped code
  from the prior redesign — a direct, necessary consequence of translating
  "everything" including these labels. One accepted edge case: if the user
  toggles language while the mobile menu happens to be open, or right as
  the theme toggle is focused, the *currently displayed* label only
  refreshes on the *next* state change (next open/close, next theme
  toggle), not instantaneously — not worth the extra complexity to close
  for a personal portfolio.
- **Single source of truth**: `Experience.astro` and `Projects.astro`'s
  bilingual content (role/dates/highlights; descriptions) is sourced from
  `translations.js`'s `en` values for their default static render, rather
  than duplicating English text both there and in the translations file.
  This is a small refactor of those two components from the prior redesign,
  a direct consequence of centralizing translation data as the single
  source of truth.
- **Accepted limitation**: because the actual text swap requires the DOM to
  exist (unlike the theme toggle, which only flips a CSS class), a
  returning visitor who previously chose Spanish may see a brief
  (sub-frame, typically imperceptible) flash of English text before the
  bundled script applies the stored preference. Not worth a fully
  SSR-aware fix for a personal portfolio.
- **Toggle UI**: a text-badge button (not a flag icon — flags are
  ambiguous for a language spoken across many countries) showing the
  *other* language's code: "ES" while the site is in English, "EN" while
  in Spanish. Placed next to `ThemeToggle` in the header.

## Content

Full translation table. English strings are the site's current copy
(post the prior redesign's final-review fixes); Spanish is the approved
translation, sourced from the CV where the content originated there
(Experience), translated fresh otherwise.

### Nav labels (reused across desktop nav, mobile menu, and Footer where applicable)

| Key | English | Spanish |
|---|---|---|
| `nav.home` | Home | Inicio |
| `nav.about` | About | Sobre mí |
| `nav.skills` | Skills | Habilidades |
| `nav.experience` | Experience | Experiencia |
| `nav.projects` | Projects | Proyectos |
| `nav.contact` | Contact | Contacto |

### Header ARIA / misc

| Key | English | Spanish |
|---|---|---|
| `header.openMenu` | Open menu | Abrir menú |
| `header.closeMenu` | Close menu | Cerrar menú |
| `theme.switchToDark` | Switch to dark theme | Cambiar a tema oscuro |
| `theme.switchToLight` | Switch to light theme | Cambiar a tema claro |
| `lang.switchToSpanish` | Switch to Spanish | Cambiar a español |
| `lang.switchToEnglish` | Switch to English | Cambiar a inglés |

### About

| Key | English | Spanish |
|---|---|---|
| `about.heading` | Hey, I'm Johan Boshell | Hola, soy Johan Boshell |
| `about.badge` | Available for hire | Disponible para contratar |
| `about.subheadline` | Full Stack Developer from Pereira, Colombia | Desarrollador Full Stack de Pereira, Colombia |
| `about.bio` | (see below) | (see below) |
| `about.cvHref` | `/cv_english.pdf` | `/cv_spanish.pdf` |

`about.bio` (rendered via `innerHTML`, includes the existing 5 accent-colored
tech-name spans):

- EN: `I design and build web applications across the full stack — from responsive interfaces to the APIs and services behind them. My day-to-day toolkit spans <span class="text-accent font-medium">Angular</span>, <span class="text-accent font-medium">TypeScript</span>, <span class="text-accent font-medium">React</span>, <span class="text-accent font-medium">Node.js</span> and <span class="text-accent font-medium">NestJS</span>, working with REST APIs, service-oriented architectures, and relational databases like PostgreSQL. I have hands-on experience modernizing legacy systems, migrating Firebase-based apps to custom backend architectures, and keeping cloud deployments running smoothly. I love learning, solving problems, and building products that make a difference.`
- ES: `Diseño y desarrollo aplicaciones web a través de todo el stack — desde interfaces responsivas hasta las APIs y servicios detrás de ellas. Mi día a día incluye <span class="text-accent font-medium">Angular</span>, <span class="text-accent font-medium">TypeScript</span>, <span class="text-accent font-medium">React</span>, <span class="text-accent font-medium">Node.js</span> y <span class="text-accent font-medium">NestJS</span>, trabajando con APIs REST, arquitecturas orientadas a servicios y bases de datos relacionales como PostgreSQL. Tengo experiencia práctica modernizando sistemas legacy, migrando aplicaciones basadas en Firebase hacia arquitecturas de backend propias, y manteniendo despliegues en la nube funcionando sin problemas. Me encanta aprender, resolver problemas y construir productos que marquen la diferencia.`

### Experience

| Key | English | Spanish |
|---|---|---|
| `experience.heading` | Experience | Experiencia |
| `experience.prestemonos.role` | Full Stack Developer | Desarrollador Full Stack |
| `experience.prestemonos.dates` | Dec 2023 – Present | Dic 2023 – Actualidad |
| `experience.utp.role` | Backend Developer | Desarrollador Backend |
| `experience.utp.dates` | Aug 2025 – Present | Ago 2025 – Actualidad |

`experience.prestemonos.highlights` (array, order preserved):
1. EN: `Led the evolution of a Firebase-based architecture toward a custom Node.js backend, reducing backend coupling by ~30% and improving maintainability across 6 core modules.`
   ES: `Lideré la evolución de una arquitectura basada en Firebase hacia un backend propio en Node.js, reduciendo el acoplamiento del backend en ~30% y mejorando la mantenibilidad en 6 módulos principales.`
2. EN: `Designed and deployed a marketing-automation microservice on Google Cloud Platform (GCP), contributing to a 23% increase in requested credits.`
   ES: `Diseñé e implementé un microservicio de automatización de marketing en Google Cloud Platform (GCP), contribuyendo a un incremento del 23% en las solicitudes de crédito.`
3. EN: `Modernized 2 legacy Angular 6 applications to current Angular versions, reducing technical debt and improving frontend tooling compatibility.`
   ES: `Moderné 2 aplicaciones legacy de Angular 6 a versiones actuales de Angular, reduciendo la deuda técnica y mejorando la compatibilidad con las herramientas frontend actuales.`
4. EN: `Designed RESTful APIs supporting 40+ endpoints, and managed cloud deployments maintaining ~99% service availability.`
   ES: `Diseñé APIs RESTful soportando más de 40 endpoints, y gestioné despliegues en la nube manteniendo ~99% de disponibilidad del servicio.`

`experience.utp.highlights` (array, order preserved):
1. EN: `Contributed to the UTP Mobile API's backend, supporting an app with 10,000+ downloads on Google Play, also distributed on the App Store.`
   ES: `Contribuí al backend de la API Mobile de la UTP, dando soporte a una aplicación con más de 10.000 descargas en Google Play, también disponible en App Store.`
2. EN: `Designed reusable REST APIs and decoupled service modules consumed by multiple institutional applications.`
   ES: `Diseñé APIs REST reutilizables y módulos de servicios desacoplados consumidos por múltiples aplicaciones institucionales.`
3. EN: `Developed and maintained PostgreSQL/Oracle database integrations across 30+ institutional modules.`
   ES: `Desarrollé y mantuve integraciones de bases de datos PostgreSQL/Oracle en más de 30 módulos institucionales.`
4. EN: `Contributed to legacy modernization and delivered 20+ features/releases via Scrum and GitLab-based merge-request workflows.`
   ES: `Contribuí a la modernización de sistemas legacy y entregué más de 20 funcionalidades/releases mediante Scrum y flujos de merge requests en GitLab.`

`experience.prestemonos.company` and `experience.utp.company` (Prestémonos /
Universidad Tecnológica de Pereira (UTP)) and both roles' `location`
(Dosquebradas, Colombia / Pereira, Colombia) are **not** translated — proper
nouns, per the spec decision above — so these fields don't get `data-i18n`
tags at all, they stay as plain static text.

### Projects

| Key | English | Spanish |
|---|---|---|
| `projects.heading` | Projects | Proyectos |
| `projects.viewCode` | View Code | Ver Código |
| `projects.visitWebsite` | Visit Website | Visitar Sitio |
| `projects.weatherQuery.description` | Enter a city and get its weather! This app uses the OpenWeather API to show a quick summary of the weather forecast. | ¡Ingresa una ciudad y obtén su clima! Esta aplicación usa la API de OpenWeather para mostrar un resumen rápido del pronóstico del tiempo. |
| `projects.groceryBud.description` | A minimal React app to manage your grocery list. Add, edit, or delete items with ease — all saved in your browser using local storage. | Una aplicación minimalista en React para gestionar tu lista de compras. Agrega, edita o elimina artículos fácilmente — todo guardado en tu navegador usando local storage. |
| `projects.tierlistMaker.description` | An interactive Tier List Maker built with HTML, CSS, and JavaScript that lets users upload images, organize them into ranked tiers, and export the final tier list as an image. Ideal for ranking characters, games, foods, or anything visual—completely offline and fully customizable | Un creador de Tier Lists interactivo construido con HTML, CSS y JavaScript que permite a los usuarios subir imágenes, organizarlas en niveles y exportar la lista final como imagen. Ideal para clasificar personajes, juegos, comidas o cualquier cosa visual—completamente offline y totalmente personalizable |
| `projects.cssMemorama.description` | A memory matching game built entirely with HTML and CSS — no JavaScript. Uses pseudoclasses and CSS selectors to manage card flipping and matching logic. | Un juego de memoria construido completamente con HTML y CSS — sin JavaScript. Usa pseudoclases y selectores CSS para gestionar el volteo de cartas y la lógica de coincidencias. |
| `projects.minesweeper.description` | A classic Minesweeper game built using vanilla JavaScript, HTML, and CSS — no frameworks, just raw logic and DOM manipulation. | Un juego clásico de Buscaminas construido con JavaScript puro, HTML y CSS — sin frameworks, solo lógica pura y manipulación del DOM. |

Project titles, links, images, and tags are unchanged — no `data-i18n` on
those fields.

### Contact

| Key | English | Spanish |
|---|---|---|
| `contact.heading` | Contact | Contacto |
| `contact.copyAriaLabel` | Copy email address to clipboard | Copiar dirección de correo al portapapeles |
| `contact.copied` | Copied! | ¡Copiado! |

The email address itself (`boshelljohan@gmail.com`) is not translated.

### Footer

| Key | English | Spanish |
|---|---|---|
| `footer.copyright` | `© {year} <a href="#" class="hover:underline">Johan Boshell™</a>. All Rights Reserved.` | `© {year} <a href="#" class="hover:underline">Johan Boshell™</a>. Todos los derechos reservados.` |

`{year}` is substituted at apply-time with the value already computed by
`Footer.astro`'s existing `new Date().getFullYear()`. Footer's "About"/
"Contact" links reuse `nav.about`/`nav.contact`.

### Page title / meta description

| Key | English | Spanish |
|---|---|---|
| `meta.title` | Johan Boshell - Full Stack Developer | Johan Boshell - Desarrollador Full Stack |
| `meta.description` | Full Stack Developer from Pereira, Colombia. Angular, TypeScript, React, Node.js, NestJS, PostgreSQL. | Desarrollador Full Stack de Pereira, Colombia. Angular, TypeScript, React, Node.js, NestJS, PostgreSQL. |

## Structure & Components

### New: `src/i18n/translations.js`

Plain JS module exporting `{ en: {...}, es: {...} }` matching the dot-path
keys above (nested objects, e.g. `en.nav.home`, `en.experience.prestemonos.highlights[0]`).
This is the single source of truth for every bilingual string in the site.

### New: `src/components/LanguageToggle.astro`

Modeled directly on `ThemeToggle.astro`'s structure: a `<button id="lang-toggle">`
showing the target language's code as text ("ES" while in English, "EN"
while in Spanish), with an inline script that reads/writes
`document.documentElement.lang` and `localStorage['lang']`, and calls the
shared `applyTranslations()` function (see Layout.astro below) after
toggling. Its own `aria-label` follows the same dynamic-label pattern as
`ThemeToggle`/the hamburger button: static `data-label-es="Switch to Spanish"`/
`data-label-en="Switch to English"` attributes, each tagged with
`data-i18n-attr` pointing at `lang.switchToSpanish`/`lang.switchToEnglish`,
read by the toggle's own label-update logic instead of a hardcoded string.
Rendered next to `ThemeToggle` in `Header.astro`.

### `src/layouts/Layout.astro`

Adds a bundled `<script type="module">` (not `is:inline`, since it needs to
`import` `translations.js` and doesn't share the theme toggle's
before-first-paint timing requirement) that:
- Defines `applyTranslations(lang)`: walks every `[data-i18n]` element and
  sets `innerHTML` from `translations[lang][key]` (resolving the dot-path);
  walks every `[data-i18n-attr]` element, parses its `"attribute:key"`
  value, and sets that attribute from `translations[lang][key]`; updates
  `document.title` and the `<meta name="description">` content;
  substitutes `{year}` in `footer.copyright` before applying it.
- On module load: reads `localStorage.getItem('lang') || 'en'`, sets
  `document.documentElement.lang` to it, and calls `applyTranslations()`
  once immediately (this is the one-time flash-of-English-before-Spanish
  moment described as an accepted limitation above).
- Exposes `applyTranslations` on `window` (or via a small shared module) so
  `LanguageToggle.astro`'s own script can call it after updating the stored
  preference.

### `src/components/Header.astro`

- Adds `LanguageToggle` next to `ThemeToggle`.
- Tags both the desktop and mobile nav `<a>` elements with
  `data-i18n="nav.home"` / `data-i18n="nav.about"` / etc.
- The hamburger button gets two new static attributes,
  `data-label-open="Open menu"` and `data-label-closed="Close menu"`,
  each tagged `data-i18n-attr="data-label-open:header.openMenu"` /
  `data-i18n-attr="data-label-closed:header.closeMenu"` so they translate.
  The existing `openMenu()`/`closeMenu()` functions change from
  `menuToggle.setAttribute('aria-label', 'Open menu')` (hardcoded string)
  to `menuToggle.setAttribute('aria-label', menuToggle.getAttribute('data-label-open'))`
  (reads the current-language phrase) — same change mirrored for the
  "Close menu" branch.

### `src/components/ThemeToggle.astro`

Same pattern as the hamburger button: adds
`data-label-dark="Switch to dark theme"` / `data-label-light="Switch to light theme"`
attributes to the button, each tagged with `data-i18n-attr` pointing at
`theme.switchToDark` / `theme.switchToLight`. The existing `labelFor(theme)`
function changes from returning a hardcoded string to reading
`toggle.getAttribute(theme === 'dark' ? 'data-label-light' : 'data-label-dark')`
(the label describes the *action*, i.e. what you'll switch *to* — this
logic itself doesn't change, only where the phrase comes from).

### `src/pages/index.astro`

Tags the `<h1>` (`about.heading`), the "Available for hire" `<Badge>`
(`about.badge`), and the four section `<h2>` headings
(`skills.heading`... — reusing `nav.skills` etc. one-for-one, since e.g.
"Skills" as a nav label and "Skills" as a section heading are the identical
string in both languages here, so no separate `skills.heading` key is
needed; same for Projects/Contact/Experience headings via `nav.projects`
etc.). Uses `nav.skills`/`nav.experience`/`nav.projects`/`nav.contact` for
the four `<h2>`s directly.

### `src/components/AboutMe.astro`

Tags the subheadline (`about.subheadline`) and bio paragraph (`about.bio`,
via `innerHTML` since it contains the tech-name spans) with `data-i18n`.
Tags the CV `<a>`'s `href` with `data-i18n-attr="href:about.cvHref"`.

### `src/components/Experience.astro`

Refactored to import `en` from `translations.js` for its default static
render (role, dates, highlights) instead of a separate local `EXPERIENCE`
array — eliminates the duplicate-source-of-truth risk. Adds `data-i18n`
tags to each role, each dates string, and each highlight `<li>`.

### `src/components/Projects.astro`

The existing `PROJECTS` array keeps title/link/github/image/tags as before
(unchanged, not translated). Description strings are sourced from
`translations.js`'s `en.projects.*.description` instead of being inline
string literals in the `PROJECTS` array, and each rendered description gets
a `data-i18n` tag.

### `src/components/Contact.astro`, `src/components/Footer.astro`

Tag their translatable text/attributes as listed in the Content section.

## Explicitly out of scope

- Astro-native i18n routing (`/es/` URL prefix) — the client-side toggle
  approach was chosen instead.
- Translating tech names, company names, project titles, or the email
  address — all treated as proper nouns/universal terms.
- A third language — the architecture (dot-path keys under a top-level
  language code) generalizes to more languages later, but only `en`/`es`
  are being built now.
- Fixing the sub-frame flash-of-English limitation with an SSR-aware
  approach — explicitly accepted as out of scope for a personal portfolio.
- The new project the user will share separately — still a distinct,
  smaller follow-up, unaffected by this spec.

## Testing / Verification Plan

Consistent with the project's existing approach (no automated test
framework; verification via `npm run build` plus manual/browser checks):

1. `npm run build` succeeds.
2. On first load (no stored preference): site is in English by default.
3. Clicking the language toggle: every tagged string switches to Spanish
   instantly, no page reload, including the About Me bio's embedded tech
   spans rendering correctly (not escaped/broken), the Footer's `{year}`
   substitution still working, and the CV link's `href` switching to
   `/cv_spanish.pdf`.
4. Toggling back to English restores every string exactly to its original
   English text (no leftover Spanish fragments, no missing text).
5. Reloading the page after choosing Spanish: the stored preference
   persists (site loads in Spanish, `document.documentElement.lang="es"`).
6. Both themes (light/dark) and both languages combine correctly — spot
   check Spanish content in dark mode.
7. Mobile (375px) and desktop (1440px): the language toggle button is
   reachable and operable in both the header and (if applicable) doesn't
   break the mobile menu's layout.
8. Keyboard: the toggle button is focusable with a visible focus ring and
   operable via Enter/Space, matching `ThemeToggle`'s existing pattern.
9. Page title and meta description both update on toggle (check via
   browser tab text and view-source).
