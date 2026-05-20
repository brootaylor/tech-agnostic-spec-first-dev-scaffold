# Tech-Agnostic Spec-First Development Scaffold

A starter template for building web projects — tech-agnostic, spec-first, and works whether you build by hand, use an AI agent, or both.

## Quick start

```bash
git clone https://github.com/brootaylor/tech-agnostic-spec-first-dev-scaffold.git my-project
cd my-project
```

Then follow [WORKFLOW.md](./WORKFLOW.md) for the full setup guide. For AI agent setup, see [AGENTS.md](./AGENTS.md).

If you have a GitHub account, you can also click the **"Use this template"** button at the top of the repo to create a new repository with all the scaffold files as a clean starting point.

---

## Why spec-first?

Writing specs before code forces clarity. Before anyone — human or agent — writes a single line of implementation, the spec defines exactly what a component should do, what states it has, how it should behave, and what tests it needs to pass.

The spec becomes the shared language for the project. If the output isn't right, the spec is the first place to look.

---

## Two ways to build

**Handcrafted** — use the specs and workflow as directions for building the project yourself. The specs tell you exactly what to build and the workflow guides you through the process.

**AI-assisted** — use an AI coding agent of your choice to read the specs and generate the implementation. The `.agents/` directory contains configuration for Claude Code, Cursor, and GitHub Copilot. See [AGENTS.md](./AGENTS.md) for setup instructions.

Both paths follow the same workflow and use the same specs.

---

## Features

- **Spec-first workflow** — specs are written before any code is produced. The spec is the source of truth throughout, for humans and agents alike
- **Tech-agnostic** — works with Vanilla, Astro, Eleventy, React, or Svelte out of the box. Framework, language, styles, testing tools, and build tool are all configurable via a simple stack selector in `docs/project-brief.md`
- **AI agent agnostic** — not tied to any specific AI agent. Includes config for Claude Code, Cursor, and GitHub Copilot out of the box, with a clear pattern for adding others
- **Modern platform guide** — a reference for which web platform APIs and features to use, and when a fallback is acceptable. Agents must read it before writing any HTML, CSS, or JavaScript
- **Optional: service worker** — offline and caching support with a strategy selector and framework-specific guidance
- **Optional: Storybook** — component development and documentation environment, with setup guidance for React, Svelte, and plain JavaScript
- **Optional: security guidelines** — configurable security measures and best practices to follow during development, with framework-specific notes
- **Living documentation** — specs double as project documentation. Keep them up to date and the whole project stays coherent for humans and agents alike

---

## Choosing your stack

[`docs/project-brief.md`](./docs/project-brief.md) is the single source of truth for the project. It includes a stack selector where you mark your preferred framework, language, styles, testing tools, and build tool. Both humans and agents read it before doing anything.

---

## Modern platform guide

[`docs/modern-platform-guide.md`](./docs/modern-platform-guide.md) defines which web platform APIs and features to use, and when a fallback is acceptable. Agents must read it before writing any HTML, CSS, or JavaScript. It's the source of truth for which modern features to use and when.

---

## Design tokens

[`docs/design-tokens.md`](./docs/design-tokens.md) defines the colours, spacing, and typography for your project. Fill it in before writing any styles — agents will stop and ask if it's empty. Values are implemented in `src/styles/tokens.{css|scss}` and referenced throughout the codebase as CSS custom properties.

---

## Optional configuration

| Doc | Purpose | How to enable |
|-----|---------|---------------|
| [`docs/service-worker.md`](./docs/service-worker.md) | Offline support and caching strategy, with framework-specific implementation notes | Set active strategy in `docs/project-brief.md` |
| [`docs/storybook.md`](./docs/storybook.md) | Component development and documentation environment | Enable in `docs/project-brief.md` |
| [`docs/security.md`](./docs/security.md) | Security guidelines and best practices to follow during development | Set active security measures in `docs/project-brief.md` |

---

## What's in this scaffold

```bash
my-project/
├── README.md                             # ← you are here
├── WORKFLOW.md                           # ← step-by-step guide to using this scaffold
├── AGENTS.md                             # ← AI agent setup
│
├── docs/
│   ├── project-brief.md                  # ← single source of truth: stack, conventions, agent rules
│   ├── modern-platform-guide.md          # ← which web platform APIs and features to use
│   ├── design-tokens.md                  # ← colour, spacing, and typography definitions
│   ├── service-worker.md                 # ← optional: caching strategy configuration
│   ├── storybook.md                      # ← optional: component documentation environment
│   ├── security.md                       # ← optional: security guidelines and best practices
│   ├── features/
│   │   └── dark-mode.md                  # ← example feature spec
│   └── specs/
│       ├── _component-template.spec.md   # ← spec template
│       ├── components/
│       │   ├── button.spec.md            # ← example component spec
│       │   └── theme-toggle.spec.md      # ← example component spec
│       ├── pages/
│       │   └── home.spec.md              # ← example page spec
│       └── layouts/
│           └── main-layout.spec.md       # ← example layout spec
│
└── .agents/                              # ← AI agent config (see AGENTS.md)
    ├── claude/
    ├── cursor/
    └── copilot/
```

The example specs are real, working examples that follow the same conventions you'd use in a production project. Use them as a reference or replace them with your own.

---

## How specs work

Each spec defines the interface, behaviour, states, accessibility requirements, and test cases for what's being built. A status field on every spec controls whether an agent may act on it:

| Status | Meaning |
|--------|---------|
| `Draft` | Incomplete — do not implement |
| `Ready` | Complete — proceed with implementation |
| `Complete` | Implemented and tested |

See [WORKFLOW.md](./WORKFLOW.md) for the full step-by-step guide from scaffold setup through to deployment.
