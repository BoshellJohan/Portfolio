# Portfolio Redesign Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Redesign the Astro portfolio site with a token-based light/dark design system, add a Skills section, and fix the concrete bugs found during review.

**Architecture:** CSS custom-property color tokens (Tailwind v4 `@theme`) swapped via a `.dark` class on `<html>`, toggled by a new `ThemeToggle` component and persisted to `localStorage`. Existing component structure is preserved (Astro components, data-driven `.astro` files) — this is a restyle + targeted fixes, not a framework change.

**Tech Stack:** Astro 5, Tailwind CSS v4 (`@tailwindcss/vite`), TypeScript (strict, via `astro/tsconfigs/strict`). No new dependencies.

## Global Constraints

- No new npm dependencies (spec: "Explicitly out of scope" — no icon library swap, no font swap).
- Keep `@fontsource-variable/onest` as the only font; do not add Inter or any other font.
- Every color in every touched component must come from the token set defined in Task 1 (`bg-background`, `text-foreground`, `bg-card`, `text-card-foreground`, `bg-muted`, `text-muted-foreground`, `border-border`, `bg-accent`, `text-accent-foreground`) — no raw hex values in component markup after this plan is complete.
- No automated test framework is added (spec: static site, no existing tests). Verification per task is `npm run build` (must succeed) plus an explicit manual browser check.
- Existing `PROJECTS` and `SOCIAL` data arrays keep their exact content (titles, links, descriptions) — restyle only, per the approved spec.
- `dist/` and any build/tooling config are out of scope — do not touch.

---

## File Structure

| File | Action | Responsibility |
|---|---|---|
| `src/styles/global.css` | Modify | Color tokens (light + dark), `dark` custom variant, reduced-motion base rule |
| `src/layouts/Layout.astro` | Modify | FOUC-prevention theme-init script, `<main>` landmark, token-based body styling |
| `src/components/icons/Sun.astro` | Create | Icon for `ThemeToggle` (light mode state) |
| `src/components/icons/Moon.astro` | Create | Icon for `ThemeToggle` (dark mode state) |
| `src/components/icons/Menu.astro` | Create | Hamburger icon for mobile nav |
| `src/components/icons/Close.astro` | Create | Close (X) icon for mobile nav |
| `src/components/icons/Astro.astro` | Create | Skill icon (Astro framework) |
| `src/components/icons/TypeScript.astro` | Create | Skill icon (TypeScript) |
| `src/components/icons/Git.astro` | Create | Skill icon (Git) |
| `src/components/icons/CodeIcon.astro` | Modify | Fix missing `{...Astro.props}` spread (size prop currently silently ignored) |
| `src/components/ThemeToggle.astro` | Create | Light/dark toggle button + persistence script |
| `src/components/Button.astro` | Modify | Polymorphic: renders `<a>` when given `href`, `<button>` otherwise |
| `src/components/Badge.astro` | Modify | Token colors; fixes invalid `class=\`...\`` syntax (unbraced template literal) |
| `src/components/Skills.astro` | Create | New Skills/Tech Stack section content |
| `src/components/Header.astro` | Modify | Mobile hamburger menu, active-section highlighting, `ThemeToggle`, fixes mobile `gap` bug, removes dead commented-out image |
| `src/components/AboutMe.astro` | Modify | Token colors, fixes CV link path, fixes empty mailto link, fixes heading level |
| `src/components/Projects.astro` | Modify | Grid layout (replaces sticky-stack), token colors, polymorphic `Button` usage, removes unused `--index`, fixes heading levels |
| `src/components/Contact.astro` | Modify | Token colors, real `<button>` instead of `<span onclick>`, fixes clipboard bug (`.textContent` not `.value`), removes duplicate `id="contact"` |
| `src/components/Footer.astro` | Modify | Token colors, dynamic copyright year, removes unused icon imports |
| `src/pages/index.astro` | Modify | Adds `Skills` section (`id="skills"`), fixes heading level (page `<h1>`) |
| `src/components/ProjectsOld.astro` | Delete | Dead code, unused, superseded by `Projects.astro` |
| `src/components/SectionContainer.astro` | No change | Reviewed against the spec's "restyle onto tokens" list — it has no color classes today (only `w-full md:w-[740px] mt-30 p-6`), so there is nothing to migrate. Its fixed 740px width comfortably fits the new 2-column Projects/Skills grids (~360px per column), so no width change is needed either. |

---

### Task 1: Design tokens, dark-mode variant, and Layout shell

**Files:**
- Modify: `src/styles/global.css`
- Modify: `src/layouts/Layout.astro`

**Interfaces:**
- Produces: CSS custom properties `--color-background`, `--color-foreground`, `--color-card`, `--color-card-foreground`, `--color-muted`, `--color-muted-foreground`, `--color-border`, `--color-accent`, `--color-accent-foreground` (and their Tailwind utility classes: `bg-background`, `text-foreground`, `bg-card`, `text-card-foreground`, `bg-muted`, `text-muted-foreground`, `border-border`, `bg-accent`, `text-accent-foreground`).
- Produces: `.dark` class toggle target on `<html>`, read by every later task's `dark:` usages and by `ThemeToggle` (Task 3).

- [ ] **Step 1: Replace `global.css` with the token system**

```css
@import "tailwindcss";

@custom-variant dark (&:where(.dark, .dark *));

@theme {
  --color-background: #FAFAFA;
  --color-foreground: #09090B;
  --color-card: #FFFFFF;
  --color-card-foreground: #09090B;
  --color-muted: #E8ECF0;
  --color-muted-foreground: #64748B;
  --color-border: #E4E4E7;
  --color-accent: #7C3AED;
  --color-accent-foreground: #FFFFFF;
}

:root.dark {
  --color-background: #0A0A0C;
  --color-foreground: #F5F5F7;
  --color-card: #17171B;
  --color-card-foreground: #F5F5F7;
  --color-muted: #1F1F24;
  --color-muted-foreground: #A1A1AA;
  --color-border: #27272A;
  --color-accent: #A78BFA;
  --color-accent-foreground: #09090B;
}

@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
    scroll-behavior: auto !important;
  }
}
```

- [ ] **Step 2: Add the FOUC-prevention theme-init script and fix the landmark/token wiring in `Layout.astro`**

Replace the full contents of `src/layouts/Layout.astro` with:

```astro
---
import Header from '../components/Header.astro'
import Footer from '../components/Footer.astro'
import '@fontsource-variable/onest';

interface Props {
	title: string
	description: string
}

const {description, title} = Astro.props
---

<!doctype html>
<html lang="en">
	<head>
		<meta charset="UTF-8" />
		<meta name="description" content={description}/>
		<meta name="viewport" content="width=device-width, initial-scale=1" />
		<link rel="icon" type="image/svg+xml" href="/logo.svg" />
		<meta name="generator" content={Astro.generator} />
		<title>{title}</title>
		<script is:inline>
			(function () {
				const stored = localStorage.getItem('theme');
				const prefersDark = window.matchMedia('(prefers-color-scheme: dark)').matches;
				const theme = stored || (prefersDark ? 'dark' : 'light');
				if (theme === 'dark') {
					document.documentElement.classList.add('dark');
				}
			})();
		</script>
	</head>
	<body class="scroll-smooth bg-background text-foreground">
		<Header />
		<main class="w-full flex flex-col items-center">
			<slot />
		</main>
		<Footer />
	</body>
</html>

<style>
	:root {
		color-scheme: light;
	}

	:root.dark {
		color-scheme: dark;
	}

	* {
		margin: 0;
		padding: 0;
		box-sizing: border-box;
	}

	html {
		scroll-behavior: smooth;
	}

	body {
		margin: 0 auto;
		width: 100%;
		min-height: 100vh;
		font-family: 'Onest Variable', system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, 'Open Sans', 'Helvetica Neue', sans-serif;
		display: flex;
		flex-flow: column nowrap;
		justify-content: center;
		align-items: center;
		transition: background-color 200ms ease, color 200ms ease;
	}
</style>
```

Note: `<main>` now wraps the page's `<slot />` at the layout level — it was previously only wrapping the projects list inside `Projects.astro`, which is an invalid landmark structure (a page should have exactly one `<main>` around its primary content). Task 9 removes the now-redundant inner `<main>` from `Projects.astro`.

- [ ] **Step 3: Verify the build succeeds**

Run: `npm run build`
Expected: Build completes with no errors (there are no consumers of the new tokens yet, so no visual difference is expected — this step just confirms the CSS/layout syntax is valid).

- [ ] **Step 4: Commit**

```bash
git add src/styles/global.css src/layouts/Layout.astro
git commit -m "feat: add light/dark color token system and theme-init script"
```

---

### Task 2: New icon components

**Files:**
- Create: `src/components/icons/Sun.astro`
- Create: `src/components/icons/Moon.astro`
- Create: `src/components/icons/Menu.astro`
- Create: `src/components/icons/Close.astro`
- Create: `src/components/icons/Astro.astro`
- Create: `src/components/icons/TypeScript.astro`
- Create: `src/components/icons/Git.astro`
- Modify: `src/components/icons/CodeIcon.astro`

**Interfaces:**
- Produces: 7 new icon components, each accepting the same prop shape as existing icons (`{...Astro.props}` spread, so callers can pass `class`, `aria-hidden`, etc.).
- Consumes: none (leaf components).

All new icons follow the existing stroke-icon pattern already used by `Copy.astro`/`Mail.astro`/`Github.astro`'s sibling files: 24×24 viewBox, `stroke="currentColor"`, `stroke-width="2"`, and `{...Astro.props}` spread so `class="size-*"` from call sites works.

- [ ] **Step 1: Create `src/components/icons/Sun.astro`**

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
    <path d="M12 12m-4 0a4 4 0 1 0 8 0a4 4 0 1 0 -8 0"></path>
    <path d="M12 3l0 1"></path>
    <path d="M12 20l0 1"></path>
    <path d="M3 12l1 0"></path>
    <path d="M20 12l1 0"></path>
    <path d="M5.6 5.6l0.7 0.7"></path>
    <path d="M17.7 17.7l0.7 0.7"></path>
    <path d="M5.6 18.4l0.7 -0.7"></path>
    <path d="M17.7 6.3l0.7 -0.7"></path>
</svg>
```

- [ ] **Step 2: Create `src/components/icons/Moon.astro`**

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
    <path d="M12 3c.132 0 .263 0 .393 0a7.5 7.5 0 0 0 7.92 12.446a9 9 0 1 1 -8.313 -12.454z"></path>
</svg>
```

- [ ] **Step 3: Create `src/components/icons/Menu.astro`**

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
    <path d="M4 6l16 0"></path>
    <path d="M4 12l16 0"></path>
    <path d="M4 18l16 0"></path>
</svg>
```

- [ ] **Step 4: Create `src/components/icons/Close.astro`**

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
    <path d="M18 6l-12 12"></path>
    <path d="M6 6l12 12"></path>
</svg>
```

- [ ] **Step 5: Create `src/components/icons/Astro.astro`**

A simplified rocket glyph (generic representation, not the official trademarked Astro logo — consistent with the rest of this project's hand-rolled, simplified tech icons):

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
    <path d="M4 13a1 1 0 0 0 1 1a5 5 0 0 0 5 5a1 1 0 0 0 1 -1v-1a2 3 0 0 1 2 -3"></path>
    <path d="M12 4a17 17 0 0 1 -3.222 9.723l-2.828 2.828a2.121 2.121 0 0 1 -3 -3l2.828 -2.828a17 17 0 0 1 9.722 -3.223a1 1 0 0 1 1 1a17 17 0 0 1 -1 5"></path>
    <path d="M16 10a2 2 0 1 0 4 0a2 2 0 1 0 -4 0"></path>
</svg>
```

- [ ] **Step 6: Create `src/components/icons/TypeScript.astro`**

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
    <path d="M9 12.5h3" stroke-width="1.6"></path>
    <path d="M10.5 12.5v4.5" stroke-width="1.6"></path>
    <path d="M14.5 12.75c.3 -.2 .7 -.3 1.1 -.25c.6 .1 1.1 .5 1.1 1.1c0 .9 -2.2 .8 -2.2 1.9c0 .7 .6 1.1 1.2 1.1c.4 0 .8 -.1 1.1 -.3" stroke-width="1.6"></path>
</svg>
```

- [ ] **Step 7: Create `src/components/icons/Git.astro`**

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
    <path d="M7 18m-2 0a2 2 0 1 0 4 0a2 2 0 1 0 -4 0"></path>
    <path d="M7 6m-2 0a2 2 0 1 0 4 0a2 2 0 1 0 -4 0"></path>
    <path d="M17 12m-2 0a2 2 0 1 0 4 0a2 2 0 1 0 -4 0"></path>
    <path d="M7 8l0 8"></path>
    <path d="M7 8a4 4 0 0 0 4 4h4"></path>
</svg>
```

- [ ] **Step 8: Fix `src/components/icons/CodeIcon.astro`**

This icon is currently missing the `{...Astro.props}` spread, so `class="size-10"` passed from `index.astro` is silently dropped and the icon always renders at its hardcoded 30×30 size. Replace its contents with:

```astro
<svg
    {...Astro.props}
    xmlns="http://www.w3.org/2000/svg"
    viewBox="0 0 24 24"
    fill="none"
    stroke="currentColor"
    stroke-linecap="round"
    stroke-linejoin="round"
    width="30"
    height="30"
    stroke-width="2"
>
    <path d="M7 8l-4 4l4 4"></path>
    <path d="M17 8l4 4l-4 4"></path>
    <path d="M14 4l-4 16"></path>
</svg>
```

- [ ] **Step 9: Verify the build succeeds**

Run: `npm run build`
Expected: Build completes with no errors. (These icons aren't imported anywhere yet, so this only validates the `.astro` syntax.)

- [ ] **Step 10: Commit**

```bash
git add src/components/icons/Sun.astro src/components/icons/Moon.astro src/components/icons/Menu.astro src/components/icons/Close.astro src/components/icons/Astro.astro src/components/icons/TypeScript.astro src/components/icons/Git.astro src/components/icons/CodeIcon.astro
git commit -m "feat: add theme-toggle, mobile-menu, and skill icons; fix CodeIcon sizing"
```

---

### Task 3: `ThemeToggle` component

**Files:**
- Create: `src/components/ThemeToggle.astro`

**Interfaces:**
- Consumes: `Sun` and `Moon` from `src/components/icons/` (Task 2); `.dark` class convention on `<html>` and `localStorage['theme']` key established in Task 1.
- Produces: `<ThemeToggle />` component with no props, used by `Header.astro` (Task 7).

- [ ] **Step 1: Create `src/components/ThemeToggle.astro`**

```astro
---
import Sun from './icons/Sun.astro'
import Moon from './icons/Moon.astro'
---

<button
    id="theme-toggle"
    type="button"
    class="inline-flex items-center justify-center size-11 rounded-xl hover:bg-muted transition-colors cursor-pointer focus:outline-none focus-visible:ring-2 focus-visible:ring-accent"
    aria-label="Switch to dark theme"
>
    <Sun class="size-5 hidden dark:block" aria-hidden="true" />
    <Moon class="size-5 block dark:hidden" aria-hidden="true" />
</button>

<script is:inline>
    (function () {
        var STORAGE_KEY = 'theme';
        var toggle = document.getElementById('theme-toggle');

        function labelFor(theme) {
            return theme === 'dark' ? 'Switch to light theme' : 'Switch to dark theme';
        }

        function currentTheme() {
            return document.documentElement.classList.contains('dark') ? 'dark' : 'light';
        }

        toggle.setAttribute('aria-label', labelFor(currentTheme()));

        toggle.addEventListener('click', function () {
            var next = currentTheme() === 'dark' ? 'light' : 'dark';
            document.documentElement.classList.toggle('dark', next === 'dark');
            localStorage.setItem(STORAGE_KEY, next);
            toggle.setAttribute('aria-label', labelFor(next));
        });
    })();
</script>
```

- [ ] **Step 2: Verify the build succeeds**

Run: `npm run build`
Expected: Build completes with no errors. (Not wired into `Header.astro` yet — Task 7 does that — so no visual change yet.)

- [ ] **Step 3: Commit**

```bash
git add src/components/ThemeToggle.astro
git commit -m "feat: add ThemeToggle component"
```

---

### Task 4: Polymorphic `Button` component

**Files:**
- Modify: `src/components/Button.astro`

**Interfaces:**
- Produces: `<Button>` now accepts an optional `href` prop. When present, renders an `<a>` (forwarding any other passed attributes like `target`/`rel`); when absent, renders a `<button type="button">` as before.
- Consumed by: `Projects.astro` (Task 9), which currently wraps `<Button>` inside `<a>` — an invalid nested-interactive-element pattern this task removes the need for.

- [ ] **Step 1: Replace `src/components/Button.astro`**

```astro
---
interface Props {
    href?: string
    [key: string]: unknown
}

const { href, ...rest } = Astro.props as Props

const classes = "cursor-pointer inline-flex items-center justify-center gap-2 bg-accent text-accent-foreground hover:opacity-90 focus:outline-none focus-visible:ring-2 focus-visible:ring-accent focus-visible:ring-offset-2 focus-visible:ring-offset-background font-medium rounded-lg text-sm px-5 py-3 text-center transition-opacity"
---

{href ? (
    <a href={href} class={classes} {...rest}>
        <slot />
    </a>
) : (
    <button type="button" class={classes} {...rest}>
        <slot />
    </button>
)}
```

- [ ] **Step 2: Verify the build succeeds**

Run: `npm run build`
Expected: Build completes with no errors. (`ProjectsOld.astro` still calls `<Button><a href="">...</a></Button>` at this point — that remains valid Astro/TypeScript since `href` is optional and unknown extra props are allowed, so `Button` just renders as a `<button>` wrapping an `<a>`, same as before. It's still dead code either way; Task 12 deletes the file.)

- [ ] **Step 3: Commit**

```bash
git add src/components/Button.astro
git commit -m "feat: make Button polymorphic (renders <a> when given href)"
```

---

### Task 5: `Badge` token restyle

**Files:**
- Modify: `src/components/Badge.astro`

**Interfaces:**
- Produces: `<Badge>` renders with token-based colors; no prop shape change (still just `<slot />` + passthrough).
- Consumed by: `index.astro` ("Available for hire"), `Contact.astro` ("Copied!").

- [ ] **Step 1: Replace `src/components/Badge.astro`**

The current file has a real bug: `class=\`bg-purple-100...\`` is a bare backtick template literal directly after `=` with no `{}` wrapper, which Astro does not evaluate as an expression — the class attribute ends up containing literal backtick characters, so none of the intended Tailwind classes ever match and the badge renders unstyled. Fixed here by using a plain string:

```astro
<span
    {...Astro.props}
    class="bg-accent/10 text-accent text-xs font-semibold px-3 py-1.5 rounded-full border border-accent/20"
>
    <slot />
</span>
```

- [ ] **Step 2: Verify the build succeeds**

Run: `npm run build`
Expected: Build completes with no errors.

- [ ] **Step 3: Commit**

```bash
git add src/components/Badge.astro
git commit -m "fix: Badge unbraced class template literal; restyle onto color tokens"
```

---

### Task 6: `Skills` section

**Files:**
- Create: `src/components/Skills.astro`
- Modify: `src/pages/index.astro`

**Interfaces:**
- Consumes: icon components `Javascript`, `CSS`, `HTML`, `Tailwind`, `NextJS`, `React` (existing, from `src/components/icons/`) plus `Astro`, `TypeScript`, `Git` (Task 2). Imported as `AstroIcon` locally to avoid shadowing Astro's global `Astro` object available in component frontmatter.
- Produces: `<Skills />` component with no props, rendered inside a `<SectionContainer id="skills">` in `index.astro`.

- [ ] **Step 1: Create `src/components/Skills.astro`**

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

- [ ] **Step 2: Wire `Skills` into `index.astro`**

In `src/pages/index.astro`, add the import and a new section between About and Projects:

```astro
---
import Badge from "../components/Badge.astro";
import SectionContainer from "../components/SectionContainer.astro";
import Layout from "../layouts/Layout.astro";
import Projects from "../components/Projects.astro";
import Skills from "../components/Skills.astro";
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

Note the heading-level fix bundled in: the page's intro ("Hey, I'm Johan Boshell") changes from `<h2>` to `<h1>` — every other page heading was already correctly one level below it (`<h2>`), but there was no `<h1>` anywhere in the document before this change, which is an accessibility/heading-hierarchy defect. `Skills`, `Projects`, `Contact` stay `<h2>`.

- [ ] **Step 3: Verify the build succeeds**

Run: `npm run build`
Expected: Build completes with no errors.

- [ ] **Step 4: Manual verification**

Run `npm run dev`, open `localhost:4321`, and confirm:
- A "Skills" section renders between About and Projects with 9 labeled tech badges in a grid.
- The page has exactly one `<h1>` (view page source or browser dev tools) — "Hey, I'm Johan Boshell".

- [ ] **Step 5: Commit**

```bash
git add src/components/Skills.astro src/pages/index.astro
git commit -m "feat: add Skills section; fix missing page-level h1"
```

---

### Task 7: `Header` — mobile menu, active-section highlighting, theme toggle

**Files:**
- Modify: `src/components/Header.astro`

**Interfaces:**
- Consumes: `ThemeToggle` (Task 3), `Menu`/`Close` icons (Task 2). Reads section ids `about`, `skills`, `projects`, `contact` (present since Task 6) via `IntersectionObserver`.
- Produces: no props; self-contained nav.

- [ ] **Step 1: Replace `src/components/Header.astro`**

```astro
---
import ThemeToggle from './ThemeToggle.astro'
import Menu from './icons/Menu.astro'
import Close from './icons/Close.astro'

const NAV_LINKS = [
    { href: '#about', label: 'About' },
    { href: '#skills', label: 'Skills' },
    { href: '#projects', label: 'Projects' },
    { href: '#contact', label: 'Contact' },
]
---

<header class="fixed top-4 left-1/2 -translate-x-1/2 z-50 w-[calc(100%-2rem)] sm:w-auto">
    <nav
        aria-label="Primary"
        class="flex items-center justify-between sm:justify-start gap-2 sm:gap-4 bg-card/95 backdrop-blur border border-border px-3 py-2 rounded-2xl text-sm text-card-foreground shadow-sm"
    >
        <a href="#" class="font-bold px-3 py-1.5 rounded-xl hover:bg-muted transition-colors">Home</a>

        <ul class="hidden sm:flex items-center gap-1">
            {NAV_LINKS.map(({ href, label }) => (
                <li>
                    <a href={href} data-nav-link class="font-medium px-3 py-1.5 rounded-xl hover:bg-muted transition-colors" aria-current="false">{label}</a>
                </li>
            ))}
        </ul>

        <div class="flex items-center gap-1">
            <ThemeToggle />
            <button
                id="menu-toggle"
                type="button"
                class="sm:hidden inline-flex items-center justify-center size-11 rounded-xl hover:bg-muted transition-colors cursor-pointer"
                aria-label="Open menu"
                aria-expanded="false"
                aria-controls="mobile-menu"
            >
                <Menu id="menu-icon-open" class="size-5" aria-hidden="true" />
                <Close id="menu-icon-close" class="size-5 hidden" aria-hidden="true" />
            </button>
        </div>
    </nav>

    <ul id="mobile-menu" class="hidden sm:hidden flex-col gap-1 mt-2 bg-card border border-border rounded-2xl p-2 shadow-sm">
        {NAV_LINKS.map(({ href, label }) => (
            <li>
                <a href={href} data-nav-link class="block font-medium px-3 py-2 rounded-xl hover:bg-muted transition-colors" aria-current="false">{label}</a>
            </li>
        ))}
    </ul>
</header>

<script is:inline>
    (function () {
        var menuToggle = document.getElementById('menu-toggle');
        var mobileMenu = document.getElementById('mobile-menu');
        var iconOpen = document.getElementById('menu-icon-open');
        var iconClose = document.getElementById('menu-icon-close');

        function closeMenu() {
            mobileMenu.classList.add('hidden');
            mobileMenu.classList.remove('flex');
            menuToggle.setAttribute('aria-expanded', 'false');
            menuToggle.setAttribute('aria-label', 'Open menu');
            iconOpen.classList.remove('hidden');
            iconClose.classList.add('hidden');
        }

        function openMenu() {
            mobileMenu.classList.remove('hidden');
            mobileMenu.classList.add('flex');
            menuToggle.setAttribute('aria-expanded', 'true');
            menuToggle.setAttribute('aria-label', 'Close menu');
            iconOpen.classList.add('hidden');
            iconClose.classList.remove('hidden');
        }

        menuToggle.addEventListener('click', function () {
            var isOpen = menuToggle.getAttribute('aria-expanded') === 'true';
            if (isOpen) {
                closeMenu();
            } else {
                openMenu();
            }
        });

        var navLinks = document.querySelectorAll('[data-nav-link]');
        navLinks.forEach(function (link) {
            link.addEventListener('click', closeMenu);
        });

        var sectionIds = ['about', 'skills', 'projects', 'contact'];
        var sections = sectionIds
            .map(function (id) { return document.getElementById(id); })
            .filter(Boolean);

        function setActive(id) {
            navLinks.forEach(function (link) {
                var isActive = link.getAttribute('href') === '#' + id;
                link.setAttribute('aria-current', isActive ? 'true' : 'false');
                link.classList.toggle('bg-muted', isActive);
            });
        }

        var observer = new IntersectionObserver(
            function (entries) {
                entries.forEach(function (entry) {
                    if (entry.isIntersecting) {
                        setActive(entry.target.id);
                    }
                });
            },
            { rootMargin: '-40% 0px -50% 0px', threshold: 0 }
        );

        sections.forEach(function (section) {
            observer.observe(section);
        });
    })();
</script>
```

This also removes the dead commented-out base64 image blob that was at the top of the previous file, and fixes the mobile-width bug where nav links had no `gap` below the `lg` breakpoint (the old markup only set `gap` classes at `lg:`) — links below `sm` now live in the hamburger menu instead of a cramped inline row.

- [ ] **Step 2: Verify the build succeeds**

Run: `npm run build`
Expected: Build completes with no errors.

- [ ] **Step 3: Manual verification**

Run `npm run dev` and confirm, at both a mobile width (375px) and desktop width (1440px):
- Desktop: nav shows Home/About/Skills/Projects/Contact inline plus the theme toggle; no hamburger button visible.
- Mobile: nav shows Home + theme toggle + hamburger button; clicking the hamburger opens a dropdown with the four section links, and the icon swaps to an X; clicking a link closes the menu and scrolls to the section.
- Scrolling the page updates which nav link is visually highlighted (`aria-current="true"`) to match the section in view, in both the desktop nav and (open) mobile menu.
- Tab through the nav with the keyboard — every link, the theme toggle, and the hamburger button receive a visible focus ring.

- [ ] **Step 4: Commit**

```bash
git add src/components/Header.astro
git commit -m "feat: add mobile menu and active-section highlighting to Header"
```

---

### Task 8: `AboutMe` restyle and link fixes

**Files:**
- Modify: `src/components/AboutMe.astro`

**Interfaces:**
- No props change. Fixes two real bugs in the `SOCIAL` data array.

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
        link: "/cv.pdf",
        icon: null,
        class: "bg-accent text-accent-foreground hover:opacity-90"
    },

    {
        name: "johan.boshell@utp.edu.co",
        link: "mailto:johan.boshell@utp.edu.co",
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
          <p class="text-2xl font-semibold mb-4">Web Developer from Pereira, Colombia</p>
          <p class="text-base leading-relaxed max-w-xl text-muted-foreground">
            I'm passionate about web development, focused on creating engaging and functional experiences using modern technologies like <span class="text-accent font-medium">React</span>, <span class="text-accent font-medium">JavaScript</span>, <span class="text-accent font-medium">Astro</span>, <span class="text-accent font-medium">Tailwind</span> and more.
            I love learning, solving problems, and building products that make a difference.
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

Changes: `/public/cv.pdf` → `/cv.pdf` (Astro serves `public/`'s contents from the site root — the old path 404s); the Mail entry's `link: ""` → `link: "mailto:johan.boshell@utp.edu.co"` (previously went nowhere); the "Web Developer from Pereira, Colombia" line changes from `<h1>` to `<p>` (the page's single `<h1>` is now "Hey, I'm Johan Boshell" from Task 6 — a second `<h1>` here was a heading-hierarchy violation); tokens replace hardcoded purple/white hex values; `rel="noopener noreferrer"` added to the LinkedIn/GitHub external links (previously missing, `target="_blank"` without it is a minor security/perf issue).

- [ ] **Step 2: Verify the build succeeds**

Run: `npm run build`
Expected: Build completes with no errors.

- [ ] **Step 3: Manual verification**

Run `npm run dev` and confirm:
- Clicking "CV" downloads/opens the PDF (not a 404).
- Clicking the email chip opens the system mail client addressed to `johan.boshell@utp.edu.co`.
- View page source: only one `<h1>` exists on the page.

- [ ] **Step 4: Commit**

```bash
git add src/components/AboutMe.astro
git commit -m "fix: broken CV link and empty mailto link; restyle AboutMe onto tokens"
```

---

### Task 9: `Projects` grid restyle

**Files:**
- Modify: `src/components/Projects.astro`

**Interfaces:**
- Consumes: `Button` (Task 4, now polymorphic — called with `href` instead of being wrapped in `<a>`).
- No change to the `PROJECTS`/`TAGS` data shape or content.

- [ ] **Step 1: Replace `src/components/Projects.astro`**

```astro
---
import Javascript from "./icons/Javascript.astro";
import CSS from "./icons/CSS.astro";
import HTML from "./icons/HTML.astro";
import Tailwind from "./icons/Tailwind.astro";
import NextJS from "./icons/NextJS.astro";
import React from "./icons/React.astro";

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
};

const PROJECTS = [
  {
    title: "Weather Query",
    description:
      "Enter a city and get its weather! This app uses the OpenWeather API to show a quick summary of the weather forecast.",
    link: "https://weatherapi-boshell.netlify.app/",
    github: "https://github.com/BoshellJohan/Weather_API",
    image: "/img/weatherQuery.png",
    tags: [TAGS.NEXTJS, TAGS.TAILWIND],
  },
  {
    title: "Grocery Bud",
    description:
      "A minimal React app to manage your grocery list. Add, edit, or delete items with ease — all saved in your browser using local storage.",
    link: "https://grocerybud-boshelljohan.netlify.app/",
    github: "https://github.com/BoshellJohan/GroceryBud",
    image: "/img/GroceryBud.png",
    tags: [TAGS.REACT],
  },
  {
    title: "Tierlist Maker",
    description:
      "An interactive Tier List Maker built with HTML, CSS, and JavaScript that lets users upload images, organize them into ranked tiers, and export the final tier list as an image. Ideal for ranking characters, games, foods, or anything visual—completely offline and fully customizable",
    link: "https://tierlistmaker-boshelljohan.netlify.app/",
    github: "https://github.com/BoshellJohan",
    image: "/img/tierlist-maker.png",
    tags: [TAGS.JAVASCRIPT, TAGS.HTML, TAGS.CSS],
  },
  {
    title: "CSS Memorama",
    description:
      "A memory matching game built entirely with HTML and CSS — no JavaScript. Uses pseudoclasses and CSS selectors to manage card flipping and matching logic.",
    link: "https://memorama-boshelljohan.netlify.app/",
    github: "https://github.com/BoshellJohan/Memograma-Final-Version",
    image: "/img/Memorama.png",
    tags: [TAGS.HTML, TAGS.CSS],
  },
  {
    title: "MineSweeper",
    description:
      "A classic Minesweeper game built using vanilla JavaScript, HTML, and CSS — no frameworks, just raw logic and DOM manipulation.",
    link: "https://minesweeper-boshell.netlify.app/",
    github: "https://github.com/BoshellJohan/Minesweeper",
    image: "/img/Minesweeper.png",
    tags: [TAGS.JAVASCRIPT, TAGS.HTML, TAGS.CSS],
  }
];
---

<ul class="w-full grid grid-cols-1 md:grid-cols-2 gap-6">
  {
    PROJECTS.map(({ image, title, description, tags, link, github }) => (
      <li>
        <article class="flex flex-col h-full bg-card text-card-foreground border border-border rounded-xl p-6 shadow-sm hover:shadow-md transition-shadow">
          <div class="rounded-lg overflow-hidden border border-border mb-4 aspect-video bg-muted">
            <img
              class="w-full h-full object-cover"
              src={image}
              alt={title}
              loading="lazy"
              width="640"
              height="360"
            />
          </div>

          <h3 class="text-xl font-bold mb-2">{title}</h3>
          <p class="text-muted-foreground leading-relaxed mb-4">{description}</p>

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

          <div class="flex gap-3 mt-auto">
            <Button href={github} target="_blank" rel="noopener noreferrer">
              View Code
            </Button>
            <Button href={link} target="_blank" rel="noopener noreferrer">
              Visit Website
            </Button>
          </div>
        </article>
      </li>
    ))
  }
</ul>
```

Changes from the previous version: the `<main>` wrapper is removed (the layout now provides the page's single `<main>`, per Task 1); the sticky-stack scroll classes (`sticky top-[10vh]`, `id={`card_${idx + 1}`}`, `style={`--index:${idx + 1};`}`) are removed in favor of a plain responsive grid, since `--index` was never consumed by any CSS rule; card titles change from `<h1>` to `<h3>` (they're nested under the page's "Projects" `<h2>`); `Button` is now called with `href`/`target`/`rel` directly instead of being wrapped in a separate `<a>` (fixes the invalid nested-interactive-elements markup).

- [ ] **Step 2: Verify the build succeeds**

Run: `npm run build`
Expected: Build completes with no errors.

- [ ] **Step 3: Manual verification**

Run `npm run dev` and confirm:
- Projects render as a grid: 1 column at 375px width, 2 columns at 1024px+.
- "View Code" and "Visit Website" are real links (inspect element: an `<a>` tag, not a `<button>` nested inside an `<a>`) that open the correct URLs in a new tab.
- Card hover shows a shadow transition.

- [ ] **Step 4: Commit**

```bash
git add src/components/Projects.astro
git commit -m "feat: replace sticky-stack scroll with responsive grid; fix invalid Button-in-anchor nesting"
```

---

### Task 10: `Contact` restyle and bug fixes

**Files:**
- Modify: `src/components/Contact.astro`

**Interfaces:**
- No props change.

- [ ] **Step 1: Replace `src/components/Contact.astro`**

```astro
---
import Copy from "./icons/Copy.astro";
import Badge from "./Badge.astro";
---

<article class="w-full flex flex-col items-center gap-2 py-0 mb-16">
    <div class="w-full flex gap-2 flex-row mb-2">
        <p
            id="emailPar"
            class="text-card-foreground bg-card p-2.5 rounded-lg border border-border w-8/10"
        >johan.boshell@utp.edu.co</p>
        <button
            id="copyEmailButton"
            type="button"
            aria-label="Copy email address to clipboard"
            class="cursor-pointer rounded-lg flex justify-center items-center py-2.5 bg-accent text-accent-foreground hover:opacity-90 focus:outline-none focus-visible:ring-2 focus-visible:ring-accent w-2/10"
        >
            <Copy class="size-5" aria-hidden="true" />
        </button>
    </div>

    <!-- Contenedor para el badge -->
    <div id="badgeContainer" class="self-start invisible">
        <Badge>Copied!</Badge>
    </div>
</article>

<script is:inline>
    (function () {
        var button = document.getElementById("copyEmailButton");
        var emailPar = document.getElementById("emailPar");
        var badgeContainer = document.getElementById("badgeContainer");
        var hideTimeout;

        button.addEventListener("click", function () {
            navigator.clipboard.writeText(emailPar.textContent.trim())
                .then(function () {
                    badgeContainer.classList.remove("invisible");
                    clearTimeout(hideTimeout);
                    hideTimeout = setTimeout(function () {
                        badgeContainer.classList.add("invisible");
                    }, 3000);
                })
                .catch(function (err) {
                    console.error("Error al copiar el correo:", err);
                });
        });
    })();
</script>
```

Fixes: the copy control is now a real `<button>` (was a `<span onclick="...">` with no keyboard support or button semantics); the click handler reads `emailPar.textContent` instead of `emailPar.value` (`.value` doesn't exist on a `<p>`, so the previous handler ran `navigator.clipboard.writeText(undefined)` and silently did nothing useful); the root `<article id="contact">` loses its `id` since `index.astro`'s `<SectionContainer id="contact">` already provides that id — two elements sharing `id="contact"` was invalid duplicate-ID HTML; the global `handleClick()` function on `window` is replaced with a scoped `addEventListener` (no more global namespace pollution); tokens replace hardcoded purple hex values.

- [ ] **Step 2: Verify the build succeeds**

Run: `npm run build`
Expected: Build completes with no errors.

- [ ] **Step 3: Manual verification**

Run `npm run dev`, open the Contact section, and confirm:
- Clicking the copy button actually places `johan.boshell@utp.edu.co` on the system clipboard (paste somewhere to confirm) and shows the "Copied!" badge for ~3 seconds.
- Tab to the copy button with the keyboard and press Enter/Space — it activates the same way a click does.
- Inspect element: only one element on the page has `id="contact"` (the `SectionContainer`).

- [ ] **Step 4: Commit**

```bash
git add src/components/Contact.astro
git commit -m "fix: clipboard copy reading .value on a <p> (now .textContent); span-as-button a11y; duplicate id=contact"
```

---

### Task 11: `Footer` restyle and cleanup

**Files:**
- Modify: `src/components/Footer.astro`

**Interfaces:**
- No props change.

- [ ] **Step 1: Replace `src/components/Footer.astro`**

```astro
---
const year = new Date().getFullYear()
---

<footer class="bg-background border-t border-border w-full">
    <div class="w-full mx-auto max-w-screen-xl p-4 flex flex-col md:flex-row md:items-center md:justify-between gap-2">
      <span class="text-sm text-muted-foreground">© {year} <a href="#" class="hover:underline">Johan Boshell™</a>. All Rights Reserved.</span>
      <ul class="flex flex-wrap items-center gap-x-6 text-sm font-medium text-muted-foreground">
        <li>
            <a href="#about" class="hover:underline">About</a>
        </li>
        <li>
            <a href="#contact" class="hover:underline">Contact</a>
        </li>
      </ul>
    </div>
</footer>
```

Changes: the `LinkedIn`/`Github`/`Mail` icon imports are removed — they were imported but never used anywhere in the template (dead code); copyright year is now computed via `new Date().getFullYear()` instead of hardcoded `2025`; tokens replace hardcoded colors.

- [ ] **Step 2: Verify the build succeeds**

Run: `npm run build`
Expected: Build completes with no errors.

- [ ] **Step 3: Commit**

```bash
git add src/components/Footer.astro
git commit -m "fix: remove unused icon imports from Footer; dynamic copyright year; restyle onto tokens"
```

---

### Task 12: Delete dead `ProjectsOld.astro`

**Files:**
- Delete: `src/components/ProjectsOld.astro`

**Interfaces:**
- None — verifying no other file imports it is the entire point of this task.

- [ ] **Step 1: Confirm nothing imports it**

Run: `grep -r "ProjectsOld" src/` (or equivalent search)
Expected: No results other than the file itself.

- [ ] **Step 2: Delete the file**

```bash
git rm src/components/ProjectsOld.astro
```

- [ ] **Step 3: Verify the build succeeds**

Run: `npm run build`
Expected: Build completes with no errors.

- [ ] **Step 4: Commit**

```bash
git commit -m "chore: remove unused ProjectsOld.astro"
```

---

### Task 13: Full integration pass and manual QA

**Files:** none (verification-only task)

**Interfaces:** none — this task exercises the whole page as assembled by Tasks 1–12.

- [ ] **Step 1: Build check**

Run: `npm run build`
Expected: Build completes with no errors, no warnings about unresolved imports or unused props.

- [ ] **Step 2: Preview the production build**

Run: `npm run preview`, open the printed local URL.

- [ ] **Step 3: Theme QA**

- Toggle dark/light via the header button; confirm every section (About, Skills, Projects, Contact, Footer, nav) recolors correctly with no leftover hardcoded-purple elements.
- Reload the page after toggling to dark — confirm it stays dark (no flash of light theme, `localStorage` persisted).
- Clear `localStorage`, set the OS to dark mode, reload — confirm the site defaults to dark.

- [ ] **Step 4: Responsive QA**

Check at 375px, 768px, 1024px, and 1440px widths (browser dev tools device toolbar):
- No horizontal scroll at any width.
- Header collapses to the hamburger menu below `sm` (640px) and expands to the inline nav above it.
- Projects grid is 1 column below `md` (768px), 2 columns at/above it.
- Skills grid reflows from 2 to 3 columns per its breakpoints.

- [ ] **Step 5: Accessibility QA**

- Tab through the entire page with the keyboard only (no mouse): every interactive element (nav links, theme toggle, hamburger, project buttons/links, copy-email button, social/CV links) must show a visible focus ring and be operable with Enter/Space.
- Enable "reduce motion" in OS accessibility settings, reload, and confirm transitions are effectively instant (per the `prefers-reduced-motion` rule in `global.css`).
- Confirm exactly one `<h1>` exists on the page and heading levels descend without skipping (h1 → h2 → h3).

- [ ] **Step 6: Bug-fix verification**

- CV link downloads/opens the PDF.
- Email chip in About opens a mail client addressed correctly.
- Copy-email button in Contact actually copies the address (paste to confirm).
- No duplicate `id="contact"` in the rendered DOM.

- [ ] **Step 7: Final commit (if any QA fixes were needed)**

If Steps 3–6 turned up any issues, fix them in the relevant component file(s) and commit:

```bash
git add -A
git commit -m "fix: address issues found in redesign integration QA"
```

If no issues were found, no commit is needed for this task.
