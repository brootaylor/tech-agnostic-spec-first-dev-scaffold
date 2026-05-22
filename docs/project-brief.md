# Project Brief

This file is the single source of truth for project context.

It is tech-agnostic — do not add anything here that is specific to any particular
tool, framework, or AI agent beyond what is selected in the Stack section below.

Agent-specific configs live in `.agents/`.

**Last updated:** <!-- e.g. 2025-01-15 — update whenever a meaningful change is made -->

---

## Your setup checklist

These are the only sections you need to fill in before starting. Everything else is reference material — for you to read if you need it, and for any AI agent working on the project.

- [ ] **What this project is** — describe the project, who it's for, its goals, and any known constraints
- [ ] **Stack** — mark one option per category as `[active]`
- [ ] **Browser support** — update the targets table if the defaults don't match your project
- [ ] **Accessibility standard** — review the default and update it if your project has different requirements

---

## What this project is

> *Fill in each field below. The more specific you are, the more useful this file
> is as a reference — for you and for any AI agent working on the project.*

**What it is:**
> One or two sentences describing the product. For example:
> 
> *"A static documentation site for a component library. It covers usage guidelines, code examples, and design principles."*

**Who it's for:**
> Who uses it and in what context. For example:
> 
> *"Front-end developers at the company who need a reference when building new features."*

**Key goals:**
> What does success look like? For example:
> 
> *"Fast load times, accessible to WCAG 2.1 AA, easy to update without a developer."*

**Known constraints:**
> Anything that limits what can be built or how. This includes technical constraints, business rules, and environmental requirements. For example:
> 
> - *"Must work without JavaScript enabled for core content"*
> - *"No external analytics or third-party scripts"*
> - *"Must support Safari 15 and above due to the user base — note any required polyfills in the Browser support section"*
> - *"Cannot use a CDN — all assets must be self-hosted"*
> - *"Bundle size must stay under 100kb uncompressed"*

---

## Stack

This section defines the framework and tooling for the project.
Mark exactly one option per category as `[active]`. Leave all others blank.

### Framework

The core framework for this project. Pick one.

| Framework | Active |
|-----------|--------|
| **Static Site Generators (SSG)** | |
| Eleventy *(11ty)* | |
| **Web Frameworks** | |
| Astro | |
| **Component Frameworks** | |
| React | |
| React + Next.js | |
| Svelte | |
| Svelte + SvelteKit | |
| **No framework** | |
| Vanilla | `[active]` |

### Language

| Language | Active |
|----------|--------|
| TypeScript | |
| JavaScript | `[active]` |

### Styles

| Approach | Active |
|----------|--------|
| Plain CSS | `[active]` |
| CSS Modules | |
| Sass | |
| Tailwind | |

### Unit testing

| Tool | Active |
|------|--------|
| Vitest | |
| Jest | |
| None | `[active]` |

### E2E testing

| Tool | Active |
|------|--------|
| Playwright | |
| None | `[active]` |

### Build

Applies to **Vanilla only**. SSG, web, and component frameworks manage their own build pipeline.

| Tool | Active |
|------|--------|
| Vite | `[active]` |
| None *(framework handles it)* | |

### Service worker

Optional. Defaults to `None`. See `docs/service-worker.md` for
full details on caching strategies and framework-specific implementation notes.

| Option | Active |
|--------|--------|
| None | `[active]` |
| Cache first *(assets)* + Network first *(pages)* — recommended for static sites | |
| Network first *(all)* — better for frequently updated content | |
| Stale while revalidate — serves cache immediately, updates in the background | |
| Custom — define your own strategy in `docs/service-worker.md` | |

### Storybook

Optional. Defaults to `None`. See `docs/storybook.md` for
setup instructions and framework-specific implementation notes.

| Option | Active |
|--------|--------|
| None | `[active]` |
| Storybook | |

### Linting

Optional. Defaults to `None`. When active, ESLint is configured with rules that
catch both code quality issues and common security vulnerabilities. See
`docs/security.md` for the secure coding rules that linting enforces.

| Option | Active |
|--------|--------|
| None | `[active]` |
| ESLint | |

### Security

Optional. Defaults to `None`. See `docs/security.md` for header definitions,
CSP configuration, and framework-specific setup guidance.

| Option | Active |
|--------|--------|
| None | `[active]` |
| Headers only | |
| Headers + CSP | |

---

## Stack compatibility notes

> Find your active selections in the sections below before generating any config files.
> Each section is independent — only read what applies to your stack.
> Humans should review the notes and follow any instructions before setup. Agents must follow the instructions when generating config files.

---

### Build pipeline

Notes for frameworks that manage their own build pipeline. The **Build** selection
in the Stack section above does not apply to these.

- **Eleventy** — includes its own build pipeline; ignore the Build selection
- **Astro** — includes its own build pipeline; ignore the Build selection
- **React** — if `React + Next.js` is not active, ask before setup:
  *"Do you want a simple Vite setup, or a full meta-framework with Next.js?"*
  Mark the answer active in the Framework table before continuing. Ignore the Build selection either way.
- **Svelte** — if `Svelte + SvelteKit` is not active, ask before setup:
  *"Do you want a simple Vite setup, or a full meta-framework with SvelteKit?"*
  Mark the answer active in the Framework table before continuing. Ignore the Build selection either way.
- **React + Next.js** — Next.js manages its own build pipeline; ignore the Build selection
- **Svelte + SvelteKit** — SvelteKit manages its own build pipeline; ignore the Build selection

---

### Tool combinations

Notes for specific combinations that require extra setup steps or have known
conflicts. Check here whenever two or more selections interact.

- **Vanilla + Vite + Jest** — Jest requires additional config to handle ESM modules in a Vite project. Prefer Vitest for Vite-based stacks to avoid this complexity
- **Tailwind** — requires PostCSS config. Install `tailwindcss`, `postcss`, and `autoprefixer` as dev dependencies and generate `tailwind.config.js`
- **Tailwind + Astro** — use the official `@astrojs/tailwind` integration (`npx astro add tailwind`) rather than configuring PostCSS manually; the integration handles the config automatically
- **CSS Modules** — supported natively by Vite, Next.js, and SvelteKit. No additional config needed for those stacks. For Vanilla without Vite, additional build config is required
- **CSS Modules + Eleventy** — Eleventy has no native CSS Modules support. A separate bundler is required, which significantly complicates the setup. Consider plain CSS unless there is a strong reason to use modules
- **ESLint + TypeScript** — add `@typescript-eslint/parser` and `@typescript-eslint/eslint-plugin` as dev dependencies alongside ESLint
- **ESLint + Svelte** — add `eslint-plugin-svelte` as a dev dependency
- **ESLint + Astro** — add `eslint-plugin-astro` and `astro-eslint-parser` as dev dependencies. The `.eslintrc` must set `astro-eslint-parser` as the parser for `.astro` files

---

### Testing setup

Testing tools require framework-specific wiring beyond a standard install. Read
the relevant entry below before generating any test config file.

**React + Vitest**
- Install `@testing-library/react`, `@testing-library/jest-dom`, and `jsdom`
- `vitest.config.ts` must include the `react()` plugin from `@vitejs/plugin-react`
- Set `environment: 'jsdom'` in the Vitest config
- Set `globals: true` to avoid importing `describe`, `it`, and `expect` in every test file

**React + Jest**
- Install `@testing-library/react`, `@testing-library/jest-dom`, `babel-jest`, and `@babel/preset-react`
- Configure Jest to use `jsdom` as the test environment
- A Babel config is required to transform JSX

**Svelte + Vitest**
- Install `@testing-library/svelte`, `@testing-library/jest-dom`, and `jsdom`
- `vitest.config.ts` must include the `svelte()` plugin from `@sveltejs/vite-plugin-svelte`
- Set `environment: 'jsdom'` in the Vitest config

**Svelte + Jest**
- Install `@testing-library/svelte` and configure Jest to transform `.svelte` files using `svelte-jester`

**Astro + Vitest**
- Do not create a standalone `vitest.config.ts` — Astro must supply its Vite plugin chain to
  Vitest or component rendering will fail
- Use `getViteConfig` from `astro/config` as the base config:
  ```ts
  // vitest.config.ts
  import { getViteConfig } from 'astro/config';
  export default getViteConfig({ test: { environment: 'jsdom' } });
  ```
- Install `vitest` and `jsdom` as dev dependencies
- For TypeScript, also install `@astrojs/check` — this handles type checking of `.astro` files
  separately from Vitest, which only runs `.ts` and `.js` unit tests

**Eleventy + Vitest**
- Eleventy manages its own build; Vitest is configured independently with no special integration needed
- Vitest will only run tests against JS/TS utility modules — it cannot test Eleventy templates directly

---

### TypeScript setup

**Astro + TypeScript**
- Install `@astrojs/check` and `typescript` as dev dependencies
- `tsconfig.json` must extend Astro's preset — do not write one from scratch:
  ```json
  { "extends": "astro/tsconfigs/strict" }
  ```
- Use `@astrojs/check` for type checking (it understands `.astro` files); `tsc` alone does not

**Eleventy + TypeScript**
- Eleventy does not natively process TypeScript source files
- A separate compilation step is required — compile `src/` with `tsc` and point Eleventy at
  the output, or use a bundler plugin
- This significantly increases setup complexity. If TypeScript is only needed for the Eleventy
  config file itself, use a `.eleventy.ts` approach with `ts-node` instead

---

### Sass setup

**Astro + Sass**
- Install `sass` only — Astro's Vite integration handles compilation automatically. No PostCSS
  or additional config is needed

**Eleventy + Sass**
- Eleventy does not process Sass natively. A separate watch script or build step is required
  (e.g. the `sass` CLI with `--watch`, or a Gulp task). Add the compiled CSS output directory
  to `.gitignore` and to Eleventy's passthrough copy config

---

### Storybook

**Astro + Storybook** and **Eleventy + Storybook**
- Do not run `npx storybook@latest init` without checking compatibility first — support for
  these frameworks is evolving and auto-detection may produce an incorrect setup
- Consult the [Storybook documentation](https://storybook.js.org/docs) for the current
  recommended approach before generating any Storybook config

---

## Browser support

The following defines the minimum browser targets for this project. These targets
determine which JavaScript and CSS features are safe to use without polyfills or
fallbacks.

| Target | Version |
|--------|---------|
| Chrome | Latest 2 versions |
| Firefox | Latest 2 versions |
| Safari | Latest 2 versions |
| Edge | Latest 2 versions |

> *Update this table to reflect the actual support requirements for your project.
> For example, if legacy browser support is required, note the specific versions here
> and add any polyfill or transpilation requirements to the **Known constraints** field
> in "What this project is" above.*

- No Internet Explorer support unless explicitly listed above
- Agents must not use browser APIs or CSS features that fall outside these targets
  without flagging it first

---

## Accessibility standard

> *Review the standard below and update it if your project has different requirements. The default is WCAG 2.1 AA.*

All components, pages, and layouts produced for this project must meet
**WCAG 2.1 Level AA** as a minimum.

This applies to:

- Colour contrast (text and interactive elements)
- Keyboard navigation and focus management
- Screen reader support (semantic HTML, ARIA roles, labels, and live regions)
- Motion (respect `prefers-reduced-motion` where animations are used)

Individual specs may define accessibility requirements beyond this baseline.
Agents must not consider a component complete until its spec's accessibility
requirements are met and the project-wide standard is satisfied.

---

## Setup instructions

Before writing any implementation code, the project dependencies and config files
need to be in place.

**If building by hand:**

- Set up `package.json` and any required config files
  (e.g. `vite.config.js`, `jest.config.js`) based on your active stack selections
- Check your chosen framework's documentation for the recommended Node.js version
  and update `.nvmrc` accordingly
- Refer to your chosen framework's documentation for the exact setup

**If using an AI agent:**

1. Check for the latest stable version of the active framework and use that version when populating `package.json`
2. Check the active framework's Node.js requirements and use the recommended LTS version of Node
3. Populate `package.json` with the correct scripts and dependencies for the active stack
4. Generate any required config files based on the active selections
5. Populate `.nvmrc` with the correct Node.js version for the active framework
6. Update the stack-specific section of `.gitignore` with any entries required by the active stack
7. If a service worker option is active, implement it following `docs/service-worker.md`
8. If Storybook is active, set it up following `docs/storybook.md`
9. If ESLint is active, install ESLint and the plugins listed in `docs/security.md` and generate `.eslintrc`
10. If a security option is active, apply the configuration following `docs/security.md`
11. Do not install any dependencies not directly required by the active stack selections

---

## Spec conventions

Specs use a status field to communicate whether they are ready for implementation.

| Status | Meaning | Who acts on it |
|--------|---------|----------------|
| `Draft` | Spec is incomplete — do not implement | Human only |
| `Ready` | Spec is complete — proceed with implementation | Human + agent |
| `Complete` | Implemented and tested | Human only |

**Agents must not implement a spec whose status is `Draft`.**

**Agents must not re-implement or overwrite a spec whose status is `Complete`.** If
changes are needed to a completed component, the human must first update the spec
and reset its status to `Ready` before asking the agent to revise the implementation.

If an agent encounters a `Draft` spec that is required by a feature it is working
on, it should stop and ask the user to complete the spec before continuing.

The spec template lives at `docs/specs/_component-template.spec.md`. All new
specs must follow this template.

---

## Coding conventions

These conventions apply across the whole codebase regardless of framework or
language. Agents must follow them when generating any file.

### Naming

- **Component files and folders** — PascalCase: `Button/Button.js`
- **Page and layout files and folders** — PascalCase: `Home/Home.js`
- **Utility / helper files** — camelCase: `formatDate.js`
- **Style files** — match the component they belong to: `Button.css`
- **Test files** — match the file under test with a `.test` suffix: `Button.test.js`
- **Asset files** — kebab-case: `icon-sun.svg`, `hero-image.webp`
- **CSS class names** — BEM: `.btn`, `.btn--primary`, `.btn__label`
- **CSS custom properties** — kebab-case with semantic prefix: `--color-primary`, `--space-md`
- **JavaScript variables and functions** — camelCase
- **TypeScript types and interfaces** — PascalCase

### Imports

- External dependencies first, then internal modules, then relative imports
- No default exports from utility files — named exports only
- Components may use default exports

### Comments

- Prefer self-documenting code over inline comments
- Use comments to explain *why*, not *what*
- JSDoc comments on any exported function or component that isn't self-evident

### General

- No raw values without a name — any number, size, duration, or colour used in code must be extracted to a named constant or design token so its purpose is clear and it can be changed in one place. For example, use `var(--space-md)` rather than `24px`, and `const retryDelayMs = 5000` rather than `5000`
- No hardcoded colour values anywhere in stylesheets — always reference a CSS custom property
- Prefer explicit over implicit — if something isn't obvious from the surrounding code, name it
- **Modern platform first** — read `docs/modern-platform-guide.md` before writing any HTML, CSS, or JavaScript. Use native platform APIs unless that file explicitly permits a fallback.

### Secure coding

- Never use `eval()`, `new Function()`, or pass a string as the first argument to `setTimeout` or `setInterval` — these execute arbitrary code and are flagged by the linter
- Never use `innerHTML` with any value that originates from user input or an external source — use `textContent` or DOM methods instead
- Never hardcode API keys, tokens, or credentials in source files — use environment variables, and never commit `.env` files containing real values
- Never store sensitive data (tokens, passwords, personally identifiable information) in `localStorage` or `sessionStorage`
- If a component or feature needs to load a resource from an external origin, declare it in the component's Dependencies table and update the CSP exceptions table in `docs/security.md` before marking the component `Complete`

These rules are enforced by ESLint when the Linting option is active. See `docs/security.md` for plugin and rule details.

> *Adjust or extend these conventions to match your team's preferences. Keeping them
> here means all agents pick them up automatically.*

---

## Agent behaviour rules

These rules govern how agents must behave when working on this project.
They apply regardless of which agent is used.

- **Read this file first** — before doing anything else, read `docs/project-brief.md` in full
- **Read the spec before implementing** — never generate implementation code without first reading the relevant spec
- **Do not implement `Draft` specs** — see Spec conventions above
- **Do not re-implement `Complete` specs** — if a spec is marked `Complete`, skip it. If changes are needed, the human must update the spec and reset its status to `Ready` first
- **Do not modify spec files** — specs are written by humans. Agents read them; they do not edit them. If a spec is ambiguous or incomplete, stop and ask
- **Do not delete files without confirmation** — always ask before removing any file that was not created in the current session
- **Do not install unlisted dependencies** — only install packages directly required by the active stack selections or an explicit spec requirement
- **Design tokens before styles** — read `docs/design-tokens.md` before writing any CSS. If the file is empty or incomplete, stop and ask the user to fill it in
- **Read security config before generating HTML or deployment config** — read `docs/security.md` before generating `index.html`, `_headers`, `vercel.json`, or any middleware file. The header values and CSP directives defined there are the source of truth
- **Modern platform before implementation** — read `docs/modern-platform-guide.md` before writing any HTML, CSS, or JavaScript. Use native platform APIs and features unless that file explicitly permits a fallback.
- **Tests before implementation** — write tests first, then implement until they pass
- **One spec at a time** — unless explicitly asked to scaffold multiple specs at once, implement one spec per session and confirm before moving to the next
- **Read compatibility notes before setup** — before generating any config file, check the Stack compatibility notes section above for the active stack combination and follow any instructions there
- **Stop and report when setup fails** — if initial project setup produces errors or a tool cannot be configured correctly after a single attempt, stop immediately. Report exactly what failed, the full error message, and what was tried. Do not attempt further fixes in a loop. Wait for the human to review and advise before continuing
- **Ask, don't assume** — if a spec is ambiguous, a constraint is unclear, or a decision would affect the whole project, ask rather than guess

---

## Features and components

Understanding the difference between features and components is important for
working with this scaffold effectively.

A **feature** describes a piece of functionality from the user's perspective —
what they can do and why. Feature specs live in `docs/features/` and are written
in terms of user stories and acceptance criteria.

A **component** is a discrete, reusable piece of UI that implements part of a
feature. Component specs live in `docs/specs/components/` and define the
interface, behaviour, states, and test cases needed to build it.

A feature will typically depend on one or more components. The feature spec
lists which components are required before implementation can begin.

---

## Other spec types

Beyond features and components, the following spec types may be added to
`docs/specs/` as a project grows:

| Type | Location | Describes |
|------|----------|-----------|
| Pages / Views | `docs/specs/pages/` | A full page composed of multiple components |
| Layouts | `docs/specs/layouts/` | Reusable page structure wrapping content |
| Hooks | `docs/specs/hooks/` | Reusable logic shared across components |

The following are better suited to standalone reference docs in `docs/` rather
than specs:

| Type | Location | Describes |
|------|----------|-----------|
| Services / API | `docs/services.md` | How the frontend communicates with the backend |
| Design tokens | `docs/design-tokens.md` | Colours, spacing, typography, and other design constants |

---

## File & folder conventions

File extensions follow the active stack selection in the Stack section above.

```bash
src/
  components/
    <ComponentName>/
      <ComponentName>.{js|ts|jsx|tsx|svelte}              # ← implementation
      <ComponentName>.stories.{js|jsx|ts|tsx|svelte}      # ← stories (Storybook only)
      <ComponentName>.test.{js|ts}                        # ← unit tests
      <ComponentName>.{css|scss}                          # ← styles (extension matches active Styles selection)
      <ComponentName>.spec.md                             # ← co-located spec (copied from docs/specs/components/)
  pages/
    <PageName>/
      <PageName>.{js|ts|jsx|tsx|svelte}                   # ← implementation
      <PageName>.stories.{js|jsx|ts|tsx|svelte}           # ← stories (Storybook only)
      <PageName>.test.{js|ts}                             # ← unit tests
      <PageName>.{css|scss}                               # ← styles
      <PageName>.spec.md                                  # ← co-located spec
  layouts/
    <LayoutName>/
      <LayoutName>.{js|ts|jsx|tsx|svelte}                 # ← implementation
      <LayoutName>.stories.{js|jsx|ts|tsx|svelte}         # ← stories (Storybook only)
      <LayoutName>.{css|scss}                             # ← styles
      <LayoutName>.spec.md                                # ← co-located spec
  scripts/
    main.js                                               # ← default starting file (Vanilla + Vite only)
  styles/
    tokens.{css|scss}                                     # ← design token values (extension matches active Styles selection)
    main.{css|scss}                                       # ← global styles, imports tokens
  assets/
    icons/                                                # ← SVG icons (inlined in components, not via <img>)
    images/                                               # ← raster images (png, jpg, webp) referenced via <img>
  lib/                                                    # ← shared utilities
  types/                                                  # ← global types (TypeScript only)
docs/
  project-brief.md                                        # ← this file
  modern-platform-guide.md                                # ← which web platform APIs and features to use
  design-tokens.md                                        # ← design token definitions
  service-worker.md                                       # ← service worker configuration
  storybook.md                                            # ← storybook configuration
  security.md                                             # ← security headers and CSP configuration
  features/                                               # ← user-facing feature specs
  specs/
    _component-template.spec.md                           # ← spec template
    components/                                           # ← authoritative component specs
    pages/                                                # ← page / view specs
    layouts/                                              # ← layout specs
    hooks/                                                # ← hook specs
```

### Assets

- **SVG icons** — place in `src/assets/icons/` and inline them directly in the component. Do not reference via `<img>`
- **Raster images** — place in `src/assets/images/` and reference via `<img>` with appropriate `alt` text
- Do not place assets directly in `src/assets/` root — always use the subdirectories above

### Default starting files

- **Vanilla + Vite** — `src/index.html` is the default home page and `src/scripts/main.js` is the JavaScript file it references. Both are included as minimal starting files to build out
- **React** — `src/index.html` and `src/scripts/main.js` are the default starting files. Update `main.js` to mount the React app
- **React + Next.js** — pages and routing are managed by Next.js. Remove `src/index.html` and `src/scripts/main.js` if switching to Next.js
- **Svelte** — `src/index.html` and `src/scripts/main.js` are the default starting files. Update `main.js` to mount the Svelte app
- **Svelte + SvelteKit** — pages and routing are managed by SvelteKit. Remove `src/index.html` and `src/scripts/main.js` if switching to SvelteKit
- **Astro** — pages and templating are managed by Astro's own file-based routing. Remove `src/index.html` and `src/scripts/main.js` if switching to Astro
- **Eleventy** — pages and templating are managed by Eleventy's own templating system. Remove `src/index.html` and `src/scripts/main.js` if switching to Eleventy

---

## Where to look

| Question | File |
|----------|------|
| What does a feature need to do? | `docs/features/<feature>.md` |
| What should a component do? | `docs/specs/components/<name>.spec.md` |
| What should a page look like? | `docs/specs/pages/<name>.spec.md` |
| What should a layout do? | `docs/specs/layouts/<name>.spec.md` |
| What does a hook do? | `docs/specs/hooks/<name>.spec.md` |
| What are the design tokens? | `docs/design-tokens.md` |
| How is the service worker configured? | `docs/service-worker.md` |
| How is Storybook configured? | `docs/storybook.md` |
| What are the security headers and CSP? | `docs/security.md` |
