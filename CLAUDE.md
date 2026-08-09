# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

A single-page personal portfolio site for Johan Boshell, built with Astro 5 and Tailwind CSS 4. No backend, no client-side framework — content is static Astro components with a couple of small inline `<script>` snippets for interactivity.

## Commands

```sh
npm install       # install dependencies
npm run dev       # start dev server at localhost:4321
npm run build     # build production site to ./dist/
npm run preview   # preview the production build locally
npm run astro ...             # run Astro CLI commands (e.g. `npm run astro check`)
```

There is no test suite and no linter configured in this repo.

## Architecture

**Single page, section-based.** `src/pages/index.astro` is the only route. It wraps three `SectionContainer` blocks (`#about`, `#projects`, `#contact`) in `src/layouts/Layout.astro`, which renders the fixed `Header` (anchor-link nav) and `Footer` around a `<slot />`. Navigation is plain in-page anchor scrolling (`html { scroll-behavior: smooth }` in `Layout.astro`), not client-side routing.

**Content lives in data arrays inside components, not in a content collection.** `Projects.astro`, `AboutMe.astro`, and `Contact.astro` each define their content as a local `const` array/object at the top of the file (e.g. `PROJECTS`, `TAGS`, `SOCIAL`) and map over it in the template. To add a project or social link, edit the corresponding array in place rather than introducing a new data layer. `ProjectsOld.astro` is a superseded, unused version of `Projects.astro` kept around but not imported anywhere — don't wire it back up without checking with the user first.

**Tech icons are one component per icon.** `src/components/icons/*.astro` each wrap a single inline SVG (e.g. `React.astro`, `Javascript.astro`, `Tailwind.astro`) and accept a `class` prop for sizing. `Projects.astro`'s `TAGS` map pairs a tag name/color class with one of these icon components — follow that pattern when adding support for a new technology badge.

**Styling is Tailwind v4, config-free.** There is no `tailwind.config.js`; Tailwind is wired in purely via the `@tailwindcss/vite` plugin in `astro.config.mjs` and `@import "tailwindcss";` in `src/styles/global.css`. The site uses a fixed dark purple palette expressed as literal hex values in class names (`#10002B` page background, `#240046` cards, `#5A189A`/`#7B2CBF` accents/buttons, `#3C096C` nav pill) rather than Tailwind's dark-mode variants as the primary theming mechanism — match these existing hex values for new UI rather than inventing new ones.

**Client-side JS is minimal and inline.** `Contact.astro` uses an `is:inline` `<script>` with a plain `onclick="handleClick()"` attribute (copy-email-to-clipboard) rather than an Astro island or framework component. Follow this lightweight pattern for small interactive bits; there's no React/Vue/Svelte integration installed.

**Language toggle uses a centralized translation-data convention.** All English/Spanish strings live in `src/i18n/translations.js` (`translations.en.*` / `translations.es.*`, dot-path-addressable, including numeric array indices for lists like Experience's highlights). `Layout.astro`'s bundled `<script type="module">` exposes `window.applyTranslations(lang)`, which walks the DOM for two attribute conventions: `data-i18n="dot.path"` replaces an element's `innerHTML` with the resolved string (a same-element `data-year` attribute substitutes a literal `{year}` token in the result), and `data-i18n-attr="attr:dot.path;attr2:dot.path2"` sets one or more literal HTML attributes. For labels that existing JS sets dynamically at runtime rather than as a static attribute (e.g. the theme toggle's and hamburger menu's `aria-label`), tag the underlying static `data-label-*` attribute instead of the live one, and have that script read the live value from `data-label-*` rather than a hardcoded string. To add a new translatable string: add the key to both `en` and `es` in `translations.js`, then tag the consuming element.

**Static assets** live in `public/` (referenced with absolute paths like `/img/GroceryBud.png`, `/cv.pdf`) versus `src/assets/` (imported SVGs used directly in components, e.g. `background.svg`).

**`dist/` is committed to the repository** even though `.gitignore` excludes it — the ignore rule was added after `dist/` was already tracked, so it still shows up in `git status`/diffs for existing files. Don't assume changes there are just build noise; check whether the user actually wants a fresh build committed before touching it.
