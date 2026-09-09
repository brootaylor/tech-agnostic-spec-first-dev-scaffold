# Project Brief

This file is the single source of truth for project context.

It is tech-agnostic — do not add anything here that is specific to any particular
tool, framework, or "Ai" agent beyond what is selected in the Stack section below.

Agent-specific configs live in `.agents/`.

**Last updated:** <!-- e.g. 2025-01-15 — update whenever a meaningful change is made -->

---

## Your setup checklist

These are the only sections you need to fill in before starting. Everything else is reference material — for you to read if you need it, and for any "Ai" agent working on the project.

> [!IMPORTANT]
> *Adopting the scaffold into a codebase that already exists? Do not fill this in
> by hand. `docs/adopting-an-existing-project.md` Step B has `/survey` draft the
> whole file from the code for you to review, and the Stack section below then
> records what the project already uses rather than what you would have chosen.*

1. [ ] [**What this project is**](#what-this-project-is) — describe the project, who it's for, its goals, and any known constraints
2. [ ] [**Stack**](#stack) — the longest of the four. Mark one option per category as `[active]`, replacing the defaults the scaffold ships with. Ten categories in all, five of which need a decision:
   - **Decide these:** [Framework](#framework), [Language](#language), [Styles](#styles), [Unit testing](#unit-testing), [Build](#build) *(Vanilla only — every other framework brings its own)*
   - **Optional, fine left as `None`:** [End-to-end testing](#end-to-end-e2e-testing), [Service worker](#service-worker), [Storybook](#storybook), [Linting](#linting), [Security](#security)
3. [ ] [**Browser support**](#browser-support) — update the targets table if the defaults don't match your project
4. [ ] [**Accessibility standard**](#accessibility-standard) — review the default and update it if your project has different requirements

---

## What this project is

> *Fill in each field below. The more specific you are, the more useful this file
> is as a reference — for you and for any "Ai" agent working on the project.*

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
> *"Fast load times, accessible to Web Content Accessibility Guidelines (WCAG) 2.2 AA, easy to update without a developer."*

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

> [!IMPORTANT]
> **The `[active]` marks below are the scaffold's shipped default, not a
> recommendation.** *They sit on Vanilla, JavaScript, plain CSS, and Vite because
> the example specs and the `src/` starting files are written against that
> combination — nothing about it has been chosen for your project. Replace them
> with your own selections (`docs/workflow.md` Step 2) before any setup runs, clearing
> the shipped mark rather than adding a second one beside it. Nothing errors if
> you don't: an agent reads a shipped default exactly as it reads a settled
> decision, and will install and scaffold against it.*

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

### End-to-end (E2E) testing

Optional. Defaults to `None`. **Unlike every other category here, nothing in the
scaffold wires this one up** - `/tests` sets up unit testing only, and there is
no configuration document for it. Marking Playwright active records the intent
and nothing more; installing and configuring it is yours to do by hand. `/check`
will use it if it finds it already installed.

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
Content Security Policy (CSP) configuration, and framework-specific setup guidance.

| Option | Active |
|--------|--------|
| None | `[active]` |
| Headers only | |
| Headers + CSP | |

### Before you build from these selections

Some combinations need extra wiring, and some frameworks override the Build
selection entirely. `docs/setup.md` holds both the compatibility notes for those
combinations and the one-time procedure that puts dependencies and config files
in place. Read it once, after these selections are settled and before any
implementation code exists; nothing in the build loop needs it afterwards.

---

## Browser support

> *Update this table to reflect the actual support requirements for your project.
> For example, if legacy browser support is required, note the specific versions here
> and add any polyfill or transpilation requirements to the **Known constraints** field
> in "What this project is" above.*

The following defines the minimum browser targets for this project. These targets
determine which JavaScript and CSS features are safe to use without polyfills or
fallbacks.

| Target | Version |
|--------|---------|
| Chrome | Latest 2 versions |
| Firefox | Latest 2 versions |
| Safari | Latest 2 versions |
| Edge | Latest 2 versions |

- No Internet Explorer support unless explicitly listed above
- Agents must not use browser APIs or CSS features that fall outside these targets
  without flagging it first

---

## Accessibility standard

> *Review the standard below and update it if your project has different requirements. The default is WCAG 2.2 AA.*

All components, pages, and layouts produced for this project must meet
**WCAG 2.2 Level AA** as a minimum.

This applies to:

- Colour contrast (text and interactive elements)
- Keyboard navigation and focus management
- Screen reader support — semantic HTML, plus Accessible Rich Internet
  Applications (ARIA) roles, labels, and live regions
- Text alternatives (every `<img>` carries an `alt`; decorative images use `alt=""`)
- Motion (respect `prefers-reduced-motion` where animations are used)
- Target size — every control operated by pointer is at least 24 by 24 CSS
  pixels, or spaced so that a 24-pixel circle centred on it does not overlap
  another target. Links inside a sentence are exempt
- Focus visibility — a control that takes keyboard focus is never *entirely*
  hidden behind a sticky header, footer, or other content the page puts over it

WCAG 2.2 adds two further Level AA criteria that apply only where the project
has the interaction in question: any drag movement needs a single-pointer
alternative (2.5.7), and any login must not depend on a cognitive test such as
remembering or transcribing a value (3.3.8).

Individual specs may define accessibility requirements beyond this baseline.
Agents must not consider a component complete until its spec's accessibility
requirements are met and the project-wide standard is satisfied.

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

All new specs must follow the template for their kind:
`docs/features/_feature-template.md` for a feature,
`docs/specs/_component-template.spec.md` for a component, page, or layout. See
Features and components below if it is not obvious which one applies.

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
- **CSS class names** — Block-Element-Modifier (BEM): `.btn`, `.btn--primary`, `.btn__label`
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
- Never use `innerHTML` with any value that originates from user input or an external source — use `textContent` or Document Object Model (DOM) methods instead
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
- **Do not promote a spec to `Ready`** — moving a spec from `Draft` to `Ready` is the human's signal that the contract is settled. An agent that grants itself that signal has removed the gate this project is built around. Drafting a brand-new spec is allowed on explicit request, but only as `Draft`
- **Do not modify spec files** — specs are written by humans. Agents read them; they do not edit them. If a spec is ambiguous or incomplete, stop and ask. The one exception is `/complete`, which sets a finished spec's `**Status:**` and `**Last updated:**` lines and nothing else
- **Use the workflow skills when one covers the task** — `/feature`, `/implement`, `/check`, `/audit`, `/complete` and the rest live in `.agents/skills/` and are listed in `AGENTS.md`. Prefer them to freehand work: they encode the review gates that make the output reviewable
- **Do not create, switch, merge, or delete branches** — the build loop commits to whatever branch is already checked out. Branch management is the human's decision
- **Do not delete files without confirmation** — always ask before removing any file that was not created in the current session
- **Do not install unlisted dependencies** — only install packages directly required by the active stack selections or an explicit spec requirement
- **Design tokens before styles** — read `docs/design-tokens.md` before writing any CSS. **It ships complete, so its being full is not evidence anyone settled it**: check its `**Last updated:**` line, and while that still holds the placeholder comment, treat every value as the scaffold's unreviewed baseline and stop and ask the user to settle them. The one exception is a work order whose first build step ports an approved `prototypes/theme.css` into it — that step is how the file gets settled, and it is reviewed like any other
- **Read security config before generating HTML or deployment config** — read `docs/security.md` before generating `index.html`, `_headers`, `vercel.json`, `render.yaml`, `next.config.js`, or any middleware file. The header values and CSP directives defined there are the source of truth
- **Modern platform before implementation** — read `docs/modern-platform-guide.md` before writing any HTML, CSS, or JavaScript. Use native platform APIs and features unless that file explicitly permits a fallback.
- **Tests before implementation, when the project has a test runner** — a real `Test` command under Commands in `AGENTS.md` is the switch. While one is declared, write the test first and implement until it passes. Unit testing ships as `None`, so on a fresh clone there is nothing to write tests with: point the human at `/tests` rather than installing a runner yourself or claiming a step was tested
- **One spec at a time** — unless explicitly asked to scaffold multiple specs at once, implement one spec per session and confirm before moving to the next
- **Confirm the stack before setup** — the `[active]` marks in the Stack section ship pre-filled with the scaffold's default, and nothing distinguishes a default left untouched from a decision the human made. Before running initial project setup, or generating any config file or dependency list from those marks, count the marks in every category, then state the active selections back to the human and confirm they are this project's actual choices. **A category with two or more `[active]` marks, or with none, is unresolved — stop and ask which one applies rather than picking one.** Two marks usually means a shipped default was never cleared, so do not assume the newer or lower entry is the intended one
- **Read compatibility notes before setup** — before generating any config file, check `docs/setup.md` → Stack compatibility notes for the active stack combination and follow any instructions there
- **Stop and report when setup fails** — if initial project setup produces errors or a tool cannot be configured correctly after a single attempt, stop immediately. Report exactly what failed, the full error message, and what was tried. Do not attempt further fixes in a loop. Wait for the human to review and advise before continuing
- **Ask, don't assume** — if a spec is ambiguous, a constraint is unclear, or a decision would affect the whole project, ask rather than guess

---

## Features and components

A **feature** is a unit of user value — something a person can do, and why that
matters to them. A **component** is a unit of code — a reusable piece of user
interface built so that a feature can happen.

|  | Feature | Component |
|---|---------|-----------|
| Lives in | `docs/features/<name>.md` | `docs/specs/components/<name>.spec.md` |
| Template | `docs/features/_feature-template.md` | `docs/specs/_component-template.spec.md` |
| Written as | User stories (`US-##`) and acceptance criteria (`AC-##`) | Interface, states, behaviour, test cases |
| Answers | Why does this exist, and how do we know it is done? | What does it take in, render, and do? |
| Reusable | No — one feature, one goal | Yes — many features can use the same one |

**The decision rule.** Can you write it as *"As a user, I want… so that…"*? If
yes, it is a feature. If the honest answer is *"nobody wants this on its own,
they want the thing it enables"*, it is a component. Nobody wants a toggle; they
want their colour scheme remembered.

Neither implies the other. A purely visual element such as `Button` can have a
component spec with no feature above it, and a feature that is entirely logic may
need no components at all.

**A feature spec depends on its components.** It lists them in a Components
required table, and none of the work can start until every one of them has
reached `Ready`. The dependency runs one way only — a component can be specified
and built on its own.

**Shared values belong to the feature.** Any value more than one component has to
agree on — an attribute name, a storage key, a precedence order, a route path —
is fixed once in the feature spec's Implementation notes table. Component specs
reference it and must never restate it.

> [!IMPORTANT]
> *A shared value copied into a component spec rather than referenced is a silent
> failure. Both files read as correct and nothing errors; they drift apart the
> first time either is edited on its own, and the mismatch surfaces later as a
> bug with no obvious cause.*

**Three things are called "feature" in this scaffold.** They are not
interchangeable:

| Name | What it is |
|------|------------|
| `docs/features/<name>.md` | A feature spec — a durable description of user value |
| `context/current-feature.md` | The work order being built right now — a generated file, not a spec |
| `/feature` | The skill that turns the first into the second |

Only the first is written by hand. `context/current-feature.md` is regenerated
every time work starts and reset by `/complete`, so nothing durable should ever
be written there.

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
| Service worker | `docs/service-worker.md` | Caching strategy and offline behaviour |
| Storybook | `docs/storybook.md` | Storybook setup and configuration |
| Security | `docs/security.md` | Security headers, CSP, and secure coding rules |

The last three are written in the shape of a spec — user stories, acceptance
criteria, out of scope — because that shape is a good way to pin down what a
configuration has to achieve. **They are still reference docs, not specs.**
They carry no `**Status:**` line, they are never promoted to `Ready`, and
`/status` does not count them: what turns each one on is its `[active]` mark in
the Stack section above, not a status line of its own.

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
    main.js                                               # ← default starting file (Vanilla + Vite, React, Svelte)
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
  setup.md                                                # ← one-time setup: procedure and stack compatibility notes
  workflow.md                                             # ← the ten-step human guide, setup to deployment
  adopting-an-existing-project.md                         # ← setup guide for a codebase that already exists
  features/                                               # ← user-facing feature specs
    _feature-template.md                                  # ← feature spec template
  specs/
    _component-template.spec.md                           # ← component / page / layout spec template
    components/                                           # ← authoritative component specs
    pages/                                                # ← page / view specs
    layouts/                                              # ← layout specs
    hooks/                                                # ← reusable-logic specs (generated, only if the project needs them)
  reference/                                              # ← reference images a spec points at (generated by /feature)
prototypes/                                               # ← throwaway mockups from /prototype, deleted by /complete (generated)
context/                                                  # ← the build loop's working state (generated)
  sessions.md                                             # ← where things stand, read when starting cold (gitignored)
  decisions.md                                            # ← why choices were made and what was rejected (gitignored)
  current-feature.md                                      # ← the work order in flight, or a stub when idle
  findings.md                                             # ← review findings ledger, written by /audit
  history/                                                # ← archived work orders: features, fixes, rollbacks
AGENTS.md                                                 # ← cross-tool agent instructions and skill reference
```

### Assets

- **Scalable Vector Graphics (SVG) icons** — place in `src/assets/icons/` and inline them directly in the component. Do not reference via `<img>`
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
- **Every stack** — `src/assets/icons/spinner.svg` is framework-neutral and stays in place whichever stack is active. `docs/specs/components/button.spec.md` lists it as a dependency and requires it inlined, so removing it breaks that spec

---

## Where to look

| Question | File |
|----------|------|
| Which kind of spec do I write? | `docs/project-brief.md` → Features and components |
| Which platform APIs and CSS features should I use? | `docs/modern-platform-guide.md` |
| How do I write a feature spec? | `docs/features/_feature-template.md` |
| How do I write a component, page, or layout spec? | `docs/specs/_component-template.spec.md` |
| What does a feature need to do? | `docs/features/<feature>.md` |
| What should a component do? | `docs/specs/components/<name>.spec.md` |
| What should a page look like? | `docs/specs/pages/<name>.spec.md` |
| What should a layout do? | `docs/specs/layouts/<name>.spec.md` |
| What are the design tokens? | `docs/design-tokens.md` |
| How is the service worker configured? | `docs/service-worker.md` |
| How is Storybook configured? | `docs/storybook.md` |
| What are the security headers and CSP? | `docs/security.md` |
| How do I set the project up for the first time? | `docs/setup.md` |
| Does my stack combination need extra wiring? | `docs/setup.md` → Stack compatibility notes |
| Where does the work stand, and what is still open? | `context/sessions.md` → Where things stand |
| Why was a past decision made that way? | `context/decisions.md` |
| What is being built right now? | `context/current-feature.md` |
| What review findings are open? | `context/findings.md` |
| What has been built already, and in what order? | `context/history/` |
| Which skill do I run, and when? | `AGENTS.md` |
