# Stack setup

> **This is a one-time setup guide, not a spec.** It carries no `**Status:**`
> line and is never promoted to `Ready`. Read it once, before the first build;
> the build loop never needs it again.

It holds the parts of `docs/project-brief.md` that only ever apply before
any implementation code exists: the procedure that puts dependencies and config
files in place, which starting files each framework keeps or removes, and the
compatibility notes for stack combinations that need extra wiring. They live here rather than in the brief so the file every agent
loads on every session stays the part that governs the work, not the part that
started it.

Fill in `docs/project-brief.md` first — the stack selections below are read from
there, and nothing here makes sense until they are settled.

---

## Setup instructions

> [!IMPORTANT]
> **This section is for a project with no code yet.** *In a codebase the scaffold
> was adopted into, the dependencies and config files already exist, and every
> instruction below would overwrite working configuration — `package.json`,
> `.nvmrc`, the framework config, `.gitignore`. Follow
> `docs/adopting-an-existing-project.md` instead, which starts from what is
> already there. Nothing here checks first.*

Before writing any implementation code, the project dependencies and config files
need to be in place. Check **Stack compatibility notes** below for your active
combination first — several combinations need wiring that neither path spells
out, and some frameworks override the Build selection entirely.

**If building by hand:**

- Set up `package.json` and any required config files
  (e.g. `vite.config.js`, `jest.config.js`) based on your active stack selections
- Check your chosen framework's documentation for the recommended Node.js version
  and update `.nvmrc` accordingly
- Update the stack-specific section of `.gitignore` with any entries your stack
  needs — typically the build output directory your framework writes to
- Fill in the Commands section of `AGENTS.md` with the real dev-server, build,
  production-server and lint commands you just wrote into `package.json`,
  deleting any row that does not apply. Leave `Test` and `Verify` absent —
  `/tests` and `/ci` own those two rows
- Refer to your chosen framework's documentation for the exact setup

**If using an "Ai" agent:**

1. Check for the latest stable version of the active framework and use that version when populating `package.json`
2. Check the active framework's Node.js requirements and use the recommended Long Term Support (LTS) version of Node
3. Populate `package.json` with the correct scripts and dependencies for the active stack
4. Generate any required config files based on the active selections
5. Populate `.nvmrc` with the correct Node.js version for the active framework
6. Update the stack-specific section of `.gitignore` with any entries required by the active stack
7. Fill in the Commands section of `AGENTS.md` with the real dev-server, build, production-server and lint commands just written into `package.json`, deleting any row that does not apply. Leave the `Test` and `Verify` rows absent — `/tests` and `/ci` own those. A missing `Test` row is what keeps the testing gate off; a missing `Verify` row simply means the skills that would run it fall back to the build and test commands
8. If a service worker option is active, implement it following `docs/service-worker.md`
9. If Storybook is active, set it up following `docs/storybook.md`
10. If ESLint is active, install ESLint and the plugins listed in `docs/security.md` and generate `eslint.config.mjs` — flat config, never `.eslintrc`, which ESLint v10 does not read at all
11. If a security option is active, apply the configuration following `docs/security.md`
12. Do not install any dependencies not directly required by the active stack selections

Two rules hold throughout:

- **Read the compatibility notes first** — before generating any config file,
  check Stack compatibility notes below for the active stack combination and
  follow any instructions there
- **Stop and report when setup fails** — if setup produces errors, or a tool
  cannot be configured correctly after a single attempt, stop immediately.
  Report exactly what failed, the full error message, and what was tried. Do not
  attempt further fixes in a loop. Wait for the human to review and advise
  before continuing

### Default starting files

- **Vanilla + Vite** — `src/index.html` is the default home page and `src/scripts/main.js` is the JavaScript file it references. Both are included as minimal starting files to build out
- **React** — `src/index.html` and `src/scripts/main.js` are the default starting files. Update `main.js` to mount the React app
- **React + Next.js** — pages and routing are managed by Next.js. Remove `src/index.html` and `src/scripts/main.js` if switching to Next.js
- **Svelte** — `src/index.html` and `src/scripts/main.js` are the default starting files. Update `main.js` to mount the Svelte app
- **Svelte + SvelteKit** — pages and routing are managed by SvelteKit. Remove `src/index.html` and `src/scripts/main.js` if switching to SvelteKit
- **Astro** — pages and templating are managed by Astro's own file-based routing. Remove `src/index.html` and `src/scripts/main.js` if switching to Astro
- **Eleventy** — pages and templating are managed by Eleventy's own templating system. Remove `src/index.html` and `src/scripts/main.js` if switching to Eleventy
- **Every stack** — `src/assets/icons/spinner.svg` is framework-neutral and stays in place whichever stack is active. `docs/specs/components/button.spec.md` lists it as a dependency and requires it inlined, so removing it breaks that spec

---

## Stack compatibility notes

> Find your active selections in the sections below before generating any config files.
> Each section is independent — only read what applies to your stack.
> Humans should review the notes and follow any instructions before setup. Agents must follow the instructions when generating config files.

---

### Build pipeline

Notes for frameworks that manage their own build pipeline. The **Build** selection
in `docs/project-brief.md` → Stack does not apply to these.

- **Eleventy** — includes its own build pipeline; ignore the Build selection
- **Astro** — includes its own build pipeline; ignore the Build selection
- **React** — if `React + Next.js` is not active, ask before setup:
  *"Do you want a simple Vite setup, or a full meta-framework with Next.js?"*
  Mark the answer active in the Framework table in `docs/project-brief.md` before continuing. Ignore the Build selection either way.
- **Svelte** — if `Svelte + SvelteKit` is not active, ask before setup:
  *"Do you want a simple Vite setup, or a full meta-framework with SvelteKit?"*
  Mark the answer active in the Framework table in `docs/project-brief.md` before continuing. Ignore the Build selection either way.
- **React + Next.js** — Next.js manages its own build pipeline; ignore the Build selection
- **Svelte + SvelteKit** — SvelteKit manages its own build pipeline; ignore the Build selection

---

### Tool combinations

Notes for specific combinations that require extra setup steps or have known
conflicts. Check here whenever two or more selections interact.

- **Vanilla + Vite + Jest** — Jest requires additional config to handle ECMAScript modules (ESM) in a Vite project. Prefer Vitest for Vite-based stacks to avoid this complexity
- **Tailwind** — Tailwind 4 is CSS-first. Install `tailwindcss` plus the adapter for the build tool (`@tailwindcss/vite` for Vite, or `@tailwindcss/postcss` and `postcss` for a PostCSS pipeline), then `@import "tailwindcss";` at the top of the main stylesheet. There is no `tailwind.config.js`, and `autoprefixer` is not needed
- **Tailwind + Astro** — run `npx astro add tailwind`, which on Astro 5.2 and later installs the `@tailwindcss/vite` plugin and writes the config. Do not reach for the `@astrojs/tailwind` integration: it is legacy, kept only to keep Tailwind 3 projects working
- **CSS Modules** — supported natively by Vite, Next.js, and SvelteKit. No additional config needed for those stacks. For Vanilla without Vite, additional build config is required
- **CSS Modules + Eleventy** — Eleventy has no native CSS Modules support. A separate bundler is required, which significantly complicates the setup. Consider plain CSS unless there is a strong reason to use modules
- **ESLint + TypeScript** — add `@typescript-eslint/parser` and `@typescript-eslint/eslint-plugin` as dev dependencies alongside ESLint
- **ESLint + Svelte** — add `eslint-plugin-svelte` as a dev dependency
- **ESLint + Astro** — add `eslint-plugin-astro` as a dev dependency and spread `...eslintPluginAstro.configs.recommended` into `eslint.config.mjs`. Its shared config wires `astro-eslint-parser` up for `.astro` files; do not set the parser by hand

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
  (e.g. the `sass` command-line tool with `--watch`, or a Gulp task). Add the compiled CSS output directory
  to `.gitignore` and to Eleventy's passthrough copy config

---

### Storybook

**Astro + Storybook** and **Eleventy + Storybook**
- Do not run `npx storybook@latest init` without checking compatibility first — support for
  these frameworks is evolving and auto-detection may produce an incorrect setup
- Consult the [Storybook documentation](https://storybook.js.org/docs) for the current
  recommended approach before generating any Storybook config
