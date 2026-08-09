# English/Spanish Language Toggle Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add an instant, no-reload English/Spanish language toggle (matching the existing dark/light theme toggle's UX), translating all body content site-wide, with the CV link switching between the two CV PDFs per language.

**Architecture:** A single centralized translation data module (`src/i18n/translations.js`), a generic `data-i18n`/`data-i18n-attr` tagging system applied across every content component, and a bundled module script in `Layout.astro` that applies the active language on load and on toggle. A new `LanguageToggle.astro` component mirrors `ThemeToggle.astro`'s structure.

**Tech Stack:** Astro 5, Tailwind CSS v4, plain JS (no i18n library, no new dependencies).

## Global Constraints

- No new npm dependencies.
- No automated test framework is added. Verification per task is `npm run build` succeeding plus an explicit manual browser check.
- Every color in any new/touched markup must use the existing token classes — no raw hex.
- Tech names, company names, project titles, and email addresses are never translated — see the design spec's Content section for the exact, approved English/Spanish text for everything that *is* translated. Do not paraphrase or re-translate; use the strings below verbatim.
- This repo has pre-existing tracked build-artifact files (`.astro/*`, `dist/*`) that regenerate on `npm run build`/`npm run dev` and show as modified in `git status`. These are never part of any task's commit — always `git checkout -- .astro dist` to revert them before committing.
- The new project the user will share separately remains explicitly out of scope for this plan.

---

## File Structure

| File | Action | Responsibility |
|---|---|---|
| `src/i18n/translations.js` | Create | Single source of truth for every bilingual string (`{ en: {...}, es: {...} }`) |
| `src/components/LanguageToggle.astro` | Create | EN/ES toggle button, modeled on `ThemeToggle.astro` |
| `src/layouts/Layout.astro` | Modify | Adds the bundled `applyTranslations()` module script, initial-load wiring |
| `src/components/ThemeToggle.astro` | Modify | Dynamic aria-label sourced from translated `data-label-*` attributes instead of hardcoded strings |
| `src/components/Header.astro` | Modify | Adds `LanguageToggle`; tags nav labels; hamburger aria-label sourced from translated `data-label-*` attributes |
| `src/pages/index.astro` | Modify | Tags `<h1>`, badge, and the four section headings |
| `src/components/AboutMe.astro` | Modify | Tags subheadline, bio (rich HTML), CV link `href` |
| `src/components/Experience.astro` | Modify | Refactored to source role/dates/highlights from `translations.js`; tags each |
| `src/components/Projects.astro` | Modify | Refactored to source descriptions from `translations.js`; tags each; tags button labels |
| `src/components/Contact.astro` | Modify | Tags aria-label and "Copied!" text; fixes stale email to match `AboutMe.astro` |
| `src/components/Footer.astro` | Modify | Tags copyright text (with `{year}` substitution) |

---

### Task 1: Translation data

**Files:**
- Create: `src/i18n/translations.js`

**Interfaces:**
- Produces: `export const translations = { en: {...}, es: {...} }`, a plain JS object with the dot-path structure below. Consumed by every other task (imported directly in `.astro` frontmatter for SSG defaults, and by `Layout.astro`'s client-side module script for the live swap).

- [ ] **Step 1: Create `src/i18n/translations.js`**

```js
export const translations = {
  en: {
    nav: {
      home: "Home",
      about: "About",
      skills: "Skills",
      experience: "Experience",
      projects: "Projects",
      contact: "Contact",
    },
    header: {
      openMenu: "Open menu",
      closeMenu: "Close menu",
    },
    theme: {
      switchToDark: "Switch to dark theme",
      switchToLight: "Switch to light theme",
    },
    lang: {
      switchToSpanish: "Switch to Spanish",
      switchToEnglish: "Switch to English",
    },
    about: {
      heading: 'Hey, I\'m <span class="text-accent">Johan</span> Boshell',
      badge: "Available for hire",
      subheadline: "Full Stack Developer from Pereira, Colombia",
      bio: 'I design and build web applications across the full stack — from responsive interfaces to the APIs and services behind them. My day-to-day toolkit spans <span class="text-accent font-medium">Angular</span>, <span class="text-accent font-medium">TypeScript</span>, <span class="text-accent font-medium">React</span>, <span class="text-accent font-medium">Node.js</span> and <span class="text-accent font-medium">NestJS</span>, working with REST APIs, service-oriented architectures, and relational databases like PostgreSQL. I have hands-on experience modernizing legacy systems, migrating Firebase-based apps to custom backend architectures, and keeping cloud deployments running smoothly. I love learning, solving problems, and building products that make a difference.',
      cvHref: "/cv_english.pdf",
    },
    experience: {
      prestemonos: {
        role: "Full Stack Developer",
        dates: "Dec 2023 – Present",
        highlights: [
          "Led the evolution of a Firebase-based architecture toward a custom Node.js backend, reducing backend coupling by ~30% and improving maintainability across 6 core modules.",
          "Designed and deployed a marketing-automation microservice on Google Cloud Platform (GCP), contributing to a 23% increase in requested credits.",
          "Modernized 2 legacy Angular 6 applications to current Angular versions, reducing technical debt and improving frontend tooling compatibility.",
          "Designed RESTful APIs supporting 40+ endpoints, and managed cloud deployments maintaining ~99% service availability.",
        ],
      },
      utp: {
        role: "Backend Developer",
        dates: "Aug 2025 – Present",
        highlights: [
          "Contributed to the UTP Mobile API's backend, supporting an app with 10,000+ downloads on Google Play, also distributed on the App Store.",
          "Designed reusable REST APIs and decoupled service modules consumed by multiple institutional applications.",
          "Developed and maintained PostgreSQL/Oracle database integrations across 30+ institutional modules.",
          "Contributed to legacy modernization and delivered 20+ features/releases via Scrum and GitLab-based merge-request workflows.",
        ],
      },
    },
    projects: {
      viewCode: "View Code",
      visitWebsite: "Visit Website",
      weatherQuery: {
        description: "Enter a city and get its weather! This app uses the OpenWeather API to show a quick summary of the weather forecast.",
      },
      groceryBud: {
        description: "A minimal React app to manage your grocery list. Add, edit, or delete items with ease — all saved in your browser using local storage.",
      },
      tierlistMaker: {
        description: "An interactive Tier List Maker built with HTML, CSS, and JavaScript that lets users upload images, organize them into ranked tiers, and export the final tier list as an image. Ideal for ranking characters, games, foods, or anything visual—completely offline and fully customizable",
      },
      cssMemorama: {
        description: "A memory matching game built entirely with HTML and CSS — no JavaScript. Uses pseudoclasses and CSS selectors to manage card flipping and matching logic.",
      },
      minesweeper: {
        description: "A classic Minesweeper game built using vanilla JavaScript, HTML, and CSS — no frameworks, just raw logic and DOM manipulation.",
      },
    },
    contact: {
      copyAriaLabel: "Copy email address to clipboard",
      copied: "Copied!",
    },
    footer: {
      copyright: '© {year} <a href="#" class="hover:underline">Johan Boshell™</a>. All Rights Reserved.',
    },
    meta: {
      title: "Johan Boshell - Full Stack Developer",
      description: "Full Stack Developer from Pereira, Colombia. Angular, TypeScript, React, Node.js, NestJS, PostgreSQL.",
    },
  },
  es: {
    nav: {
      home: "Inicio",
      about: "Sobre mí",
      skills: "Habilidades",
      experience: "Experiencia",
      projects: "Proyectos",
      contact: "Contacto",
    },
    header: {
      openMenu: "Abrir menú",
      closeMenu: "Cerrar menú",
    },
    theme: {
      switchToDark: "Cambiar a tema oscuro",
      switchToLight: "Cambiar a tema claro",
    },
    lang: {
      switchToSpanish: "Cambiar a español",
      switchToEnglish: "Cambiar a inglés",
    },
    about: {
      heading: 'Hola, soy <span class="text-accent">Johan</span> Boshell',
      badge: "Disponible para contratar",
      subheadline: "Desarrollador Full Stack de Pereira, Colombia",
      bio: 'Diseño y desarrollo aplicaciones web a través de todo el stack — desde interfaces responsivas hasta las APIs y servicios detrás de ellas. Mi día a día incluye <span class="text-accent font-medium">Angular</span>, <span class="text-accent font-medium">TypeScript</span>, <span class="text-accent font-medium">React</span>, <span class="text-accent font-medium">Node.js</span> y <span class="text-accent font-medium">NestJS</span>, trabajando con APIs REST, arquitecturas orientadas a servicios y bases de datos relacionales como PostgreSQL. Tengo experiencia práctica modernizando sistemas legacy, migrando aplicaciones basadas en Firebase hacia arquitecturas de backend propias, y manteniendo despliegues en la nube funcionando sin problemas. Me encanta aprender, resolver problemas y construir productos que marquen la diferencia.',
      cvHref: "/cv_spanish.pdf",
    },
    experience: {
      prestemonos: {
        role: "Desarrollador Full Stack",
        dates: "Dic 2023 – Actualidad",
        highlights: [
          "Lideré la evolución de una arquitectura basada en Firebase hacia un backend propio en Node.js, reduciendo el acoplamiento del backend en ~30% y mejorando la mantenibilidad en 6 módulos principales.",
          "Diseñé e implementé un microservicio de automatización de marketing en Google Cloud Platform (GCP), contribuyendo a un incremento del 23% en las solicitudes de crédito.",
          "Moderné 2 aplicaciones legacy de Angular 6 a versiones actuales de Angular, reduciendo la deuda técnica y mejorando la compatibilidad con las herramientas frontend actuales.",
          "Diseñé APIs RESTful soportando más de 40 endpoints, y gestioné despliegues en la nube manteniendo ~99% de disponibilidad del servicio.",
        ],
      },
      utp: {
        role: "Desarrollador Backend",
        dates: "Ago 2025 – Actualidad",
        highlights: [
          "Contribuí al backend de la API Mobile de la UTP, dando soporte a una aplicación con más de 10.000 descargas en Google Play, también disponible en App Store.",
          "Diseñé APIs REST reutilizables y módulos de servicios desacoplados consumidos por múltiples aplicaciones institucionales.",
          "Desarrollé y mantuve integraciones de bases de datos PostgreSQL/Oracle en más de 30 módulos institucionales.",
          "Contribuí a la modernización de sistemas legacy y entregué más de 20 funcionalidades/releases mediante Scrum y flujos de merge requests en GitLab.",
        ],
      },
    },
    projects: {
      viewCode: "Ver Código",
      visitWebsite: "Visitar Sitio",
      weatherQuery: {
        description: "¡Ingresa una ciudad y obtén su clima! Esta aplicación usa la API de OpenWeather para mostrar un resumen rápido del pronóstico del tiempo.",
      },
      groceryBud: {
        description: "Una aplicación minimalista en React para gestionar tu lista de compras. Agrega, edita o elimina artículos fácilmente — todo guardado en tu navegador usando local storage.",
      },
      tierlistMaker: {
        description: "Un creador de Tier Lists interactivo construido con HTML, CSS y JavaScript que permite a los usuarios subir imágenes, organizarlas en niveles y exportar la lista final como imagen. Ideal para clasificar personajes, juegos, comidas o cualquier cosa visual—completamente offline y totalmente personalizable",
      },
      cssMemorama: {
        description: "Un juego de memoria construido completamente con HTML y CSS — sin JavaScript. Usa pseudoclases y selectores CSS para gestionar el volteo de cartas y la lógica de coincidencias.",
      },
      minesweeper: {
        description: "Un juego clásico de Buscaminas construido con JavaScript puro, HTML y CSS — sin frameworks, solo lógica pura y manipulación del DOM.",
      },
    },
    contact: {
      copyAriaLabel: "Copiar dirección de correo al portapapeles",
      copied: "¡Copiado!",
    },
    footer: {
      copyright: '© {year} <a href="#" class="hover:underline">Johan Boshell™</a>. Todos los derechos reservados.',
    },
    meta: {
      title: "Johan Boshell - Desarrollador Full Stack",
      description: "Desarrollador Full Stack de Pereira, Colombia. Angular, TypeScript, React, Node.js, NestJS, PostgreSQL.",
    },
  },
};
```

- [ ] **Step 2: Verify the build succeeds**

Run: `npm run build`
Expected: Build completes with no errors. (Not imported anywhere yet, so this only validates the module's JS syntax.)

- [ ] **Step 3: Commit**

```bash
git add src/i18n/translations.js
git commit -m "feat: add centralized EN/ES translation data"
```

---

### Task 2: Layout.astro translation-apply mechanism

**Files:**
- Modify: `src/layouts/Layout.astro`

**Interfaces:**
- Consumes: `translations` from `../i18n/translations.js` (Task 1).
- Produces: `window.applyTranslations(lang)`, callable by `LanguageToggle.astro` (Task 3) after it updates the stored preference. Walks `[data-i18n]` (sets `innerHTML`) and `[data-i18n-attr]` (sets one or more `"attribute:key"` pairs, semicolon-separated, e.g. `data-i18n-attr="data-label-dark:theme.switchToDark;data-label-light:theme.switchToLight"`) elements. Also updates `document.title` and `<meta name="description">`. Any resolved string containing the literal text `{year}` has it substituted from that element's own `data-year` attribute before being applied.

- [ ] **Step 1: Replace `src/layouts/Layout.astro`**

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
		<script>
			import { translations } from '../i18n/translations.js';

			function resolveKey(lang, key) {
				return key.split('.').reduce(function (obj, part) {
					return obj && obj[part] !== undefined ? obj[part] : undefined;
				}, translations[lang]);
			}

			function applyTranslations(lang) {
				document.documentElement.lang = lang;

				document.querySelectorAll('[data-i18n]').forEach(function (el) {
					var key = el.getAttribute('data-i18n');
					var value = resolveKey(lang, key);
					if (typeof value !== 'string') return;
					if (value.indexOf('{year}') !== -1) {
						var year = el.getAttribute('data-year');
						if (year) value = value.replace('{year}', year);
					}
					el.innerHTML = value;
				});

				document.querySelectorAll('[data-i18n-attr]').forEach(function (el) {
					var spec = el.getAttribute('data-i18n-attr');
					spec.split(';').forEach(function (pair) {
						var idx = pair.indexOf(':');
						if (idx === -1) return;
						var attr = pair.slice(0, idx).trim();
						var key = pair.slice(idx + 1).trim();
						var value = resolveKey(lang, key);
						if (typeof value === 'string') el.setAttribute(attr, value);
					});
				});

				var titleValue = resolveKey(lang, 'meta.title');
				if (titleValue) document.title = titleValue;

				var descValue = resolveKey(lang, 'meta.description');
				var metaDesc = document.querySelector('meta[name="description"]');
				if (metaDesc && descValue) metaDesc.setAttribute('content', descValue);
			}

			window.applyTranslations = applyTranslations;

			var storedLang = localStorage.getItem('lang') || 'en';
			applyTranslations(storedLang);
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
		scroll-padding-top: 6rem;
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

The second `<script>` (the one with the `import`) deliberately has **no** `is:inline` — Astro bundles it as an ES module via Vite, which defers its execution until after the document has been parsed (the same timing as `DOMContentLoaded`, per the HTML module-script spec), so by the time it runs, every `data-i18n`/`data-i18n-attr` element in the body already exists in the DOM. The theme script directly above it stays `is:inline` and unchanged — it only needs to flip a CSS class, not touch body content, so its current before-paint timing is still correct and doesn't need this treatment.

- [ ] **Step 2: Verify the build succeeds**

Run: `npm run build`
Expected: Build completes with no errors. (No elements are tagged `data-i18n`/`data-i18n-attr` yet — that starts in Task 3 — so this only validates the script's syntax and that the import resolves.)

- [ ] **Step 3: Manual verification**

Run `npm run dev`, open the browser console, and confirm no errors are thrown on page load. Run `window.applyTranslations('es')` directly in the console — it should execute without throwing (even though nothing visible changes yet, since no elements are tagged).

- [ ] **Step 4: Commit**

```bash
git add src/layouts/Layout.astro
git commit -m "feat: add applyTranslations mechanism to Layout"
```

---

### Task 3: Toggle button components

**Files:**
- Modify: `src/components/ThemeToggle.astro`
- Create: `src/components/LanguageToggle.astro`

**Interfaces:**
- Consumes: `window.applyTranslations` (Task 2, for `LanguageToggle` only — `ThemeToggle` doesn't call it, it only reacts to translated `data-label-*` attributes that Task 2's mechanism already updates generically).
- Produces: `<LanguageToggle />`, consumed by `Header.astro` (Task 4).

- [ ] **Step 1: Replace `src/components/ThemeToggle.astro`**

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
    data-label-dark="Switch to dark theme"
    data-label-light="Switch to light theme"
    data-i18n-attr="data-label-dark:theme.switchToDark;data-label-light:theme.switchToLight"
>
    <Sun class="size-5 hidden dark:block" aria-hidden="true" />
    <Moon class="size-5 block dark:hidden" aria-hidden="true" />
</button>

<script is:inline>
    (function () {
        var STORAGE_KEY = 'theme';
        var toggle = document.getElementById('theme-toggle');

        function labelFor(theme) {
            var attr = theme === 'dark' ? 'data-label-light' : 'data-label-dark';
            return toggle.getAttribute(attr);
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

Only the `aria-label`-related lines changed: two new `data-label-*` attributes plus `data-i18n-attr` on the button, and `labelFor()` now reads from those attributes instead of returning a hardcoded string. Nothing else in the file changes. Note: on initial page load, if the stored language preference is Spanish, this button's `aria-label` is set once (synchronously, immediately, before Task 2's deferred module script has translated the `data-label-*` attributes) using their still-English default values — meaning the very first `aria-label` read by a screen reader could be in English for a moment. This is a screen-reader-only, non-visual instance of the same "accepted sub-frame flash" limitation already documented in the design spec for visible text, and isn't worth extra complexity to close.

- [ ] **Step 2: Create `src/components/LanguageToggle.astro`**

```astro
---
---

<button
    id="lang-toggle"
    type="button"
    class="inline-flex items-center justify-center size-11 rounded-xl hover:bg-muted transition-colors cursor-pointer text-sm font-semibold focus:outline-none focus-visible:ring-2 focus-visible:ring-accent"
    aria-label="Switch to Spanish"
    data-label-es="Switch to Spanish"
    data-label-en="Switch to English"
    data-i18n-attr="data-label-es:lang.switchToSpanish;data-label-en:lang.switchToEnglish"
>ES</button>

<script is:inline>
    (function () {
        var STORAGE_KEY = 'lang';
        var toggle = document.getElementById('lang-toggle');

        function labelFor(lang) {
            var attr = lang === 'es' ? 'data-label-en' : 'data-label-es';
            return toggle.getAttribute(attr);
        }

        function currentLang() {
            return document.documentElement.lang === 'es' ? 'es' : 'en';
        }

        function updateButton(lang) {
            toggle.textContent = lang === 'es' ? 'EN' : 'ES';
            toggle.setAttribute('aria-label', labelFor(lang));
        }

        document.addEventListener('DOMContentLoaded', function () {
            updateButton(currentLang());
        });

        toggle.addEventListener('click', function () {
            var next = currentLang() === 'es' ? 'en' : 'es';
            localStorage.setItem(STORAGE_KEY, next);
            if (window.applyTranslations) {
                window.applyTranslations(next);
            }
            updateButton(next);
        });
    })();
</script>
```

The button's own visible text ("ES"/"EN") and its `aria-label` are set correctly on load via a `DOMContentLoaded` listener rather than immediately — this is deliberately different from `ThemeToggle`'s immediate approach, because this button's displayed text directly indicates the current active language (unlike an aria-label, this is visible to every user, not just screen readers), so showing the wrong one even briefly would be a real, visible bug. `DOMContentLoaded` fires after Task 2's deferred module script has already run (module scripts always execute before `DOMContentLoaded` per the HTML spec), so by the time this listener fires, `document.documentElement.lang` and the `data-label-*` attributes are already correctly set for the stored preference.

- [ ] **Step 3: Verify the build succeeds**

Run: `npm run build`
Expected: Build completes with no errors. (`LanguageToggle` isn't rendered anywhere yet — Task 4 does that.)

- [ ] **Step 4: Commit**

```bash
git add src/components/ThemeToggle.astro src/components/LanguageToggle.astro
git commit -m "feat: add LanguageToggle component; translate ThemeToggle's dynamic aria-label"
```

---

### Task 4: Wire LanguageToggle and nav translation into Header

**Files:**
- Modify: `src/components/Header.astro`

**Interfaces:**
- Consumes: `LanguageToggle` (Task 3).
- Produces: translated nav labels and hamburger aria-label, reachable by `data-i18n`/`data-i18n-attr` from Task 2's mechanism.

- [ ] **Step 1: Replace `src/components/Header.astro`**

```astro
---
import ThemeToggle from './ThemeToggle.astro'
import LanguageToggle from './LanguageToggle.astro'
import Menu from './icons/Menu.astro'
import Close from './icons/Close.astro'

const NAV_LINKS = [
    { href: '#about', label: 'About', key: 'nav.about' },
    { href: '#skills', label: 'Skills', key: 'nav.skills' },
    { href: '#experience', label: 'Experience', key: 'nav.experience' },
    { href: '#projects', label: 'Projects', key: 'nav.projects' },
    { href: '#contact', label: 'Contact', key: 'nav.contact' },
]
---

<header class="fixed top-4 left-1/2 -translate-x-1/2 z-50 w-[calc(100%-2rem)] sm:w-auto">
    <nav
        aria-label="Primary"
        class="flex items-center justify-between sm:justify-start gap-2 sm:gap-4 bg-card/95 backdrop-blur border border-border px-3 py-2 rounded-2xl text-sm text-card-foreground shadow-sm"
    >
        <a href="#" data-i18n="nav.home" class="font-bold px-3 py-1.5 rounded-xl hover:bg-muted transition-colors">Home</a>

        <ul class="hidden sm:flex items-center gap-1">
            {NAV_LINKS.map(({ href, label, key }) => (
                <li>
                    <a href={href} data-nav-link data-i18n={key} class="font-medium px-3 py-1.5 rounded-xl hover:bg-muted transition-colors" aria-current="false">{label}</a>
                </li>
            ))}
        </ul>

        <div class="flex items-center gap-1">
            <LanguageToggle />
            <ThemeToggle />
            <button
                id="menu-toggle"
                type="button"
                class="sm:hidden inline-flex items-center justify-center size-11 rounded-xl hover:bg-muted transition-colors cursor-pointer"
                aria-label="Open menu"
                aria-expanded="false"
                aria-controls="mobile-menu"
                data-label-open="Open menu"
                data-label-closed="Close menu"
                data-i18n-attr="data-label-open:header.openMenu;data-label-closed:header.closeMenu"
            >
                <Menu id="menu-icon-open" class="size-5" aria-hidden="true" />
                <Close id="menu-icon-close" class="size-5 hidden" aria-hidden="true" />
            </button>
        </div>
    </nav>

    <ul id="mobile-menu" class="hidden sm:hidden flex-col gap-1 mt-2 bg-card border border-border rounded-2xl p-2 shadow-sm">
        {NAV_LINKS.map(({ href, label, key }) => (
            <li>
                <a href={href} data-nav-link data-i18n={key} class="block font-medium px-3 py-2 rounded-xl hover:bg-muted transition-colors" aria-current="false">{label}</a>
            </li>
        ))}
    </ul>
</header>

<script is:inline>
    document.addEventListener('DOMContentLoaded', function () {
        var menuToggle = document.getElementById('menu-toggle');
        var mobileMenu = document.getElementById('mobile-menu');
        var iconOpen = document.getElementById('menu-icon-open');
        var iconClose = document.getElementById('menu-icon-close');

        function closeMenu() {
            mobileMenu.classList.add('hidden');
            mobileMenu.classList.remove('flex');
            menuToggle.setAttribute('aria-expanded', 'false');
            menuToggle.setAttribute('aria-label', menuToggle.getAttribute('data-label-open'));
            iconOpen.classList.remove('hidden');
            iconClose.classList.add('hidden');
        }

        function openMenu() {
            mobileMenu.classList.remove('hidden');
            mobileMenu.classList.add('flex');
            menuToggle.setAttribute('aria-expanded', 'true');
            menuToggle.setAttribute('aria-label', menuToggle.getAttribute('data-label-closed'));
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

        var sectionIds = ['about', 'skills', 'experience', 'projects', 'contact'];
        var sections = sectionIds
            .map(function (id) { return document.getElementById(id); })
            .filter(Boolean);

        function setActive(id) {
            navLinks.forEach(function (link) {
                var isActive = link.getAttribute('href') === '#' + id;
                link.setAttribute('aria-current', isActive ? 'true' : 'false');
                link.classList.toggle('bg-accent/10', isActive);
                link.classList.toggle('text-accent', isActive);
            });
        }

        function isAtBottom() {
            return window.innerHeight + window.scrollY >= document.documentElement.scrollHeight - 2;
        }

        function updateActiveSection() {
            if (!sections.length) {
                return;
            }
            if (isAtBottom()) {
                setActive(sections[sections.length - 1].id);
                return;
            }
            var triggerY = window.scrollY + window.innerHeight * 0.4;
            var current = sections[0];
            for (var i = 0; i < sections.length; i++) {
                if (sections[i].offsetTop <= triggerY) {
                    current = sections[i];
                }
            }
            setActive(current.id);
        }

        var ticking = false;
        window.addEventListener('scroll', function () {
            if (!ticking) {
                ticking = true;
                window.requestAnimationFrame(function () {
                    updateActiveSection();
                    ticking = false;
                });
            }
        }, { passive: true });

        window.addEventListener('resize', updateActiveSection);

        updateActiveSection();
    });
</script>
```

Changes from the previous version: `LanguageToggle` import + render (placed before `ThemeToggle`); `NAV_LINKS` entries gain a `key` field and each rendered nav `<a>` (both desktop and mobile lists) gets `data-i18n={key}`; the "Home" link gets `data-i18n="nav.home"`; the hamburger button gains `data-label-open`/`data-label-closed`/`data-i18n-attr`; `closeMenu()`/`openMenu()` now read the aria-label from those attributes instead of hardcoded `'Open menu'`/`'Close menu'` strings. The scrollspy logic (`setActive`, `isAtBottom`, `updateActiveSection`, the scroll/resize listeners) is completely unchanged.

- [ ] **Step 2: Verify the build succeeds**

Run: `npm run build`
Expected: Build completes with no errors.

- [ ] **Step 3: Manual verification**

Run `npm run dev` and confirm: the language toggle button ("ES") appears in the header next to the theme toggle, in both desktop nav and (at 375px) alongside the hamburger; clicking it doesn't yet visibly translate anything else on the page (that starts in later tasks) but the button itself flips to "EN" and back correctly, and its own aria-label updates (inspect via dev tools).

- [ ] **Step 4: Commit**

```bash
git add src/components/Header.astro
git commit -m "feat: wire LanguageToggle into Header; tag nav and hamburger for translation"
```

---

### Task 5: Tag index.astro headings

**Files:**
- Modify: `src/pages/index.astro`

**Interfaces:**
- Consumes: `nav.skills`/`nav.experience`/`nav.projects`/`nav.contact` keys (Task 1) — reused directly for the section headings rather than separate keys, since the strings are identical in both places.

- [ ] **Step 1: Replace `src/pages/index.astro`**

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
	title="Johan Boshell - Full Stack Developer"
	description="Full Stack Developer from Pereira, Colombia. Angular, TypeScript, React, Node.js, NestJS, PostgreSQL."
>
	<SectionContainer id="about">
		<div class="flex flex-wrap items-center gap-4 mb-4 font-semibold">
			<h1 class="text-4xl" data-i18n="about.heading">
				Hey, I'm <span class="text-accent">Johan</span> Boshell
			</h1>
			<Badge data-i18n="about.badge">Available for hire</Badge>
		</div>
		<AboutMe />
	</SectionContainer>

	<SectionContainer id="skills">
		<h2 class="text-4xl font-semibold mb-6" data-i18n="nav.skills">Skills</h2>
		<Skills />
	</SectionContainer>

	<SectionContainer id="experience">
		<h2 class="text-4xl font-semibold mb-6" data-i18n="nav.experience">Experience</h2>
		<Experience />
	</SectionContainer>

	<SectionContainer id="projects">
		<h2 class="text-4xl font-semibold mb-6 flex gap-x-3 items-center">
			<CodeIcon class="size-10" aria-hidden="true" />
			<span data-i18n="nav.projects">Projects</span>
		</h2>
		<Projects />
	</SectionContainer>

	<SectionContainer id="contact">
		<h2 class="text-4xl font-semibold mb-6 flex gap-x-3 items-center">
			<Mail class="size-10" aria-hidden="true" />
			<span data-i18n="nav.contact">Contact</span>
		</h2>
		<Contact />
	</SectionContainer>
</Layout>
```

Note the `<h1>` and the Skills/Experience `<h2>`s are tagged directly (their entire content is just the translatable text), while the Projects/Contact `<h2>`s wrap only the text portion in a `<span data-i18n="...">` — tagging the whole heading there would wipe out the `<CodeIcon />`/`<Mail />` SVG icon when `innerHTML` gets replaced. `<Badge data-i18n="about.badge">` works because `Badge.astro` spreads `{...Astro.props}` onto its rendered `<span>`, so the attribute lands correctly.

- [ ] **Step 2: Verify the build succeeds**

Run: `npm run build`
Expected: Build completes with no errors.

- [ ] **Step 3: Manual verification**

Run `npm run dev`, click the language toggle, and confirm: the `<h1>` becomes "Hola, soy Johan Boshell" with "Johan" still accent-colored; the "Available for hire" badge becomes "Disponible para contratar"; all four section headings translate; the `CodeIcon`/`Mail` icons next to Projects/Contact remain visible (not wiped out). Toggle back to English and confirm everything restores exactly.

- [ ] **Step 4: Commit**

```bash
git add src/pages/index.astro
git commit -m "feat: tag index.astro headings and badge for translation"
```

---

### Task 6: AboutMe translation

**Files:**
- Modify: `src/components/AboutMe.astro`

**Interfaces:**
- No structural change to the `SOCIAL` array's shape or content beyond adding one attribute to the CV entry's rendering.

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
          <p class="text-2xl font-semibold mb-4" data-i18n="about.subheadline">Full Stack Developer from Pereira, Colombia</p>
          <p class="text-base leading-relaxed max-w-xl text-muted-foreground" data-i18n="about.bio">
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
                    {tag.name == "CV" && <a href={tag.link} data-i18n-attr="href:about.cvHref" target="_blank" download="CV-JohanBoshell">{tag.name}</a>}
                </span>
            </li>
        ))}
    </ul>
</article>
```

Three additions: `data-i18n="about.subheadline"` on the subheadline `<p>`; `data-i18n="about.bio"` on the bio `<p>` (the translated value already includes the 5 accent-colored spans, so `innerHTML` replacement re-creates them correctly per language); `data-i18n-attr="href:about.cvHref"` on the CV `<a>` only (the LinkedIn/GitHub/email links are untouched, their `href`s never change with language).

- [ ] **Step 2: Verify the build succeeds**

Run: `npm run build`
Expected: Build completes with no errors.

- [ ] **Step 3: Manual verification**

Run `npm run dev`, toggle to Spanish, and confirm: subheadline and bio both translate (bio's 5 tech-name spans remain accent-colored, text reads naturally, no literal `<span>` tags visible as text); clicking "CV" now downloads/opens `/cv_spanish.pdf`. Toggle back to English — bio/subheadline restore exactly, CV link goes back to `/cv_english.pdf`.

- [ ] **Step 4: Commit**

```bash
git add src/components/AboutMe.astro
git commit -m "feat: translate About Me subheadline, bio, and CV link"
```

---

### Task 7: Experience translation (refactor to shared data source)

**Files:**
- Modify: `src/components/Experience.astro`

**Interfaces:**
- Consumes: `translations.en.experience` (Task 1), for the component's own default (English) static render — replaces the previous locally-duplicated `EXPERIENCE` array content.
- Company names and locations (Prestémonos / Dosquebradas, Colombia; Universidad Tecnológica de Pereira (UTP) / Pereira, Colombia) stay as local literals — not translated, not sourced from `translations.js`.

- [ ] **Step 1: Replace `src/components/Experience.astro`**

```astro
---
import { translations } from "../i18n/translations.js";

const en = translations.en.experience;

const EXPERIENCE = [
  {
    key: "prestemonos",
    role: en.prestemonos.role,
    company: "Prestémonos",
    dates: en.prestemonos.dates,
    location: "Dosquebradas, Colombia",
    highlights: en.prestemonos.highlights,
  },
  {
    key: "utp",
    role: en.utp.role,
    company: "Universidad Tecnológica de Pereira (UTP)",
    dates: en.utp.dates,
    location: "Pereira, Colombia",
    highlights: en.utp.highlights,
  },
];
---

<ol class="relative border-l border-border ml-3">
  {EXPERIENCE.map((job) => (
    <li class="mb-10 ml-6 last:mb-0">
      <span class="absolute -left-[7px] mt-1.5 size-3 rounded-full border-2 border-background bg-accent"></span>
      <div class="flex flex-col gap-2 md:flex-row md:gap-8">
        <div class="md:w-64 md:shrink-0">
          <h3 class="text-lg font-bold text-accent" data-i18n={`experience.${job.key}.role`}>{job.role}</h3>
          <p class="font-semibold">{job.company}</p>
          <p class="text-sm text-muted-foreground" data-i18n={`experience.${job.key}.dates`}>{job.dates}</p>
          <p class="text-sm text-muted-foreground">{job.location}</p>
        </div>
        <ul class="list-outside list-disc pl-5 space-y-1.5 text-sm leading-relaxed text-muted-foreground">
          {job.highlights.map((highlight, index) => (
            <li data-i18n={`experience.${job.key}.highlights.${index}`}>{highlight}</li>
          ))}
        </ul>
      </div>
    </li>
  ))}
</ol>
```

The rendered markup (classes, structure, the dot/line timeline mechanism) is completely unchanged from before — only the frontmatter's data source changed (from a local literal array to importing `translations.en.experience`), plus each role/dates/highlight element now carries a `data-i18n` key. The dot-path keys resolve correctly against `translations.js`'s nested structure, including the numeric highlight index (e.g. `experience.prestemonos.highlights.0`) against the `highlights` array.

- [ ] **Step 2: Verify the build succeeds**

Run: `npm run build`
Expected: Build completes with no errors.

- [ ] **Step 3: Manual verification**

Run `npm run dev`, confirm the Experience section still renders identically to before (English, same content, same timeline visuals) — this refactor should be visually invisible on its own. Then toggle to Spanish and confirm: both roles' titles, dates, and all 8 highlight bullets (4 per role) translate correctly; company names (Prestémonos, Universidad Tecnológica de Pereira (UTP)) and locations stay in their original form in both languages. Toggle back to English and confirm everything restores exactly.

- [ ] **Step 4: Commit**

```bash
git add src/components/Experience.astro
git commit -m "feat: translate Experience roles/dates/highlights, sourced from translations.js"
```

---

### Task 8: Projects translation (descriptions + button labels)

**Files:**
- Modify: `src/components/Projects.astro`

**Interfaces:**
- Consumes: `translations.en.projects` (Task 1) for the 5 project descriptions' default (English) static render, plus `projects.viewCode`/`projects.visitWebsite` for the button labels.
- Titles, links, github URLs, images, and tags are completely unchanged — not translated, not touched by this task.

- [ ] **Step 1: Replace `src/components/Projects.astro`**

```astro
---
import Javascript from "./icons/Javascript.astro";
import CSS from "./icons/CSS.astro";
import HTML from "./icons/HTML.astro";
import Tailwind from "./icons/Tailwind.astro";
import NextJS from "./icons/NextJS.astro";
import React from "./icons/React.astro";
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
};

const en = translations.en.projects;

const PROJECTS = [
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
  }
];
---

<ul class="w-full grid grid-cols-1 md:grid-cols-2 gap-6">
  {
    PROJECTS.map(({ key, image, title, description, tags, link, github }) => (
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

          <div class="flex gap-3 mt-auto">
            <Button href={github} target="_blank" rel="noopener noreferrer" data-i18n="projects.viewCode">
              View Code
            </Button>
            <Button href={link} target="_blank" rel="noopener noreferrer" data-i18n="projects.visitWebsite">
              Visit Website
            </Button>
          </div>
        </article>
      </li>
    ))
  }
</ul>
```

Each `PROJECTS` entry gains a `key` field matching its `translations.js` key; `description` values are sourced from `en.<key>.description` instead of inline literals; the description `<p>` and the two `Button` components get `data-i18n` tags. Titles, links, github URLs, images, and tags are untouched.

- [ ] **Step 2: Verify the build succeeds**

Run: `npm run build`
Expected: Build completes with no errors.

- [ ] **Step 3: Manual verification**

Run `npm run dev`, confirm Projects still renders identically to before in English (visual no-op on its own). Toggle to Spanish and confirm: all 5 project descriptions translate; "View Code"/"Visit Website" become "Ver Código"/"Visitar Sitio" on every card; project titles, tag chip labels (JavaScript, React, etc.), and links remain unchanged. Toggle back to English and confirm everything restores exactly, and the links still point to the correct (unchanged) URLs.

- [ ] **Step 4: Commit**

```bash
git add src/components/Projects.astro
git commit -m "feat: translate Projects descriptions and button labels, sourced from translations.js"
```

---

### Task 9: Contact and Footer translation, plus stale-email fix

**Files:**
- Modify: `src/components/Contact.astro`
- Modify: `src/components/Footer.astro`

**Interfaces:**
- No structural change beyond the tags and the email-text fix.

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
        >boshelljohan@gmail.com</p>
        <button
            id="copyEmailButton"
            type="button"
            aria-label="Copy email address to clipboard"
            data-i18n-attr="aria-label:contact.copyAriaLabel"
            class="cursor-pointer rounded-lg flex justify-center items-center py-2.5 bg-accent text-accent-foreground hover:opacity-90 focus:outline-none focus-visible:ring-2 focus-visible:ring-accent focus-visible:ring-offset-2 focus-visible:ring-offset-background w-2/10"
        >
            <Copy class="size-5" aria-hidden="true" />
        </button>
    </div>

    <!-- Contenedor para el badge -->
    <div id="badgeContainer" class="self-start invisible" role="status">
        <Badge data-i18n="contact.copied">Copied!</Badge>
    </div>
</article>

<script is:inline>
    (function () {
        var button = document.getElementById("copyEmailButton");
        var emailPar = document.getElementById("emailPar");
        var badgeContainer = document.getElementById("badgeContainer");
        var hideTimeout;

        button.addEventListener("click", function () {
            if (!navigator.clipboard) {
                return;
            }
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

Three changes: the displayed/copied email text changes from the stale `johan.boshell@utp.edu.co` to `boshelljohan@gmail.com` (matching `AboutMe.astro`, per the approved fix); the copy button's `aria-label` gets `data-i18n-attr="aria-label:contact.copyAriaLabel"` — this one is a genuinely **static** attribute (unlike `ThemeToggle`'s or the hamburger's, which are set dynamically by JS based on state), so it can be tagged directly with `data-i18n-attr` on the real `aria-label` attribute itself, no `data-label-*` indirection needed; the "Copied!" `<Badge>` gets `data-i18n="contact.copied"`.

- [ ] **Step 2: Replace `src/components/Footer.astro`**

```astro
---
const year = new Date().getFullYear()
---

<footer class="bg-background border-t border-border w-full">
    <div class="w-full mx-auto max-w-screen-xl p-4 flex flex-col md:flex-row md:items-center md:justify-between gap-2">
      <span class="text-sm text-muted-foreground" data-i18n="footer.copyright" data-year={year}>© {year} <a href="#" class="hover:underline">Johan Boshell™</a>. All Rights Reserved.</span>
      <ul class="flex flex-wrap items-center gap-x-6 text-sm font-medium text-muted-foreground">
        <li>
            <a href="#about" data-i18n="nav.about" class="hover:underline">About</a>
        </li>
        <li>
            <a href="#contact" data-i18n="nav.contact" class="hover:underline">Contact</a>
        </li>
      </ul>
    </div>
</footer>
```

The copyright `<span>` gets `data-i18n="footer.copyright"` and a `data-year={year}` attribute holding the build-time-computed year — Task 2's `applyTranslations()` reads this `data-year` attribute to substitute `{year}` in the translated string (both the `en` and `es` copyright strings contain the literal `{year}` placeholder). The footer's "About"/"Contact" links reuse `nav.about`/`nav.contact`.

- [ ] **Step 3: Verify the build succeeds**

Run: `npm run build`
Expected: Build completes with no errors.

- [ ] **Step 4: Manual verification**

Run `npm run dev` and confirm: the Contact section shows/copies `boshelljohan@gmail.com` (not the old email) in both languages; the copy button's aria-label translates (inspect via dev tools) and clicking it still works (paste to confirm it copies `boshelljohan@gmail.com`); the "Copied!" badge translates to "¡Copiado!" in Spanish; the Footer's copyright line translates while keeping the correct current year and the "Johan Boshell™" link intact; Footer's "About"/"Contact" links translate.

- [ ] **Step 5: Commit**

```bash
git add src/components/Contact.astro src/components/Footer.astro
git commit -m "feat: translate Contact and Footer; fix stale email in Contact to match AboutMe"
```

---

### Task 10: Full integration pass and manual QA

**Files:** none (verification-only task)

**Interfaces:** none — this task exercises the whole page as assembled by Tasks 1–9.

- [ ] **Step 1: Build check**

Run: `npm run build`
Expected: Build completes with no errors.

- [ ] **Step 2: Full-page translation QA**

Run `npm run dev` (or `npm run preview` after a build) and, starting from a fresh `localStorage` (no stored `lang` or `theme`), check:
- Page loads in English by default.
- Clicking the language toggle switches every translatable string on the page to Spanish, instantly, with no page reload: nav (desktop + mobile), the `<h1>`, "Available for hire" badge, About Me subheadline/bio (tech spans still colored), all 4 section headings (Skills/Experience/Projects/Contact — Projects/Contact icons still visible), both Experience roles/dates/all 8 highlights, all 5 Projects descriptions + both button labels, Contact's copy button aria-label + "Copied!"/"¡Copiado!" badge, Footer's copyright line (correct year, "Johan Boshell™" link intact) + About/Contact links, and the browser tab title + meta description (view-source or dev tools).
- The CV link switches to `/cv_spanish.pdf` when in Spanish, back to `/cv_english.pdf` in English — verify both resolve 200 and their content actually matches (English PDF shows "PROFILE", Spanish shows "PERFIL" — this pairing was fixed in a prior plan; only the *link switching per language* is new here).
- Toggling back to English restores every single string exactly, with nothing left in Spanish.
- Reloading the page after selecting Spanish: the page loads in Spanish again (stored preference persisted), and `document.documentElement.lang === "es"`.

- [ ] **Step 3: Cross-cutting checks (theme × language × responsive)**

- Toggle dark mode on, then toggle language — confirm both work together with no visual issues (e.g. accent-colored spans in the bio still use the correct token color in dark mode after a Spanish translation swap).
- At 375px width: language toggle button is reachable and operable in the header; mobile menu still opens/closes correctly and its nav links are translated too; toggling language while the mobile menu happens to be open doesn't break anything (acceptable per the design spec if the *label* doesn't instantly refresh — but the *menu itself* must not error or become unusable).
- At 1440px width: same checks, desktop nav.
- Keyboard: Tab to the language toggle button, confirm a visible focus ring, and confirm Enter/Space activates it the same as a click.

- [ ] **Step 4: Regression check on prior functionality**

Confirm nothing from the existing site broke: the scrollspy still correctly highlights the active section while scrolling through all 5 sections in both languages; the theme toggle still works correctly on its own; anchor-nav clicks still scroll the target section's heading below the fixed header (not hidden behind it).

- [ ] **Step 5: Final commit (if any QA fixes were needed)**

If any of the above turned up issues, fix them in the relevant file(s) and commit:

```bash
git add -A
git commit -m "fix: address issues found in language toggle integration QA"
```

If no issues were found, no commit is needed for this task.
