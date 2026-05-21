# Tech-Agnostic Spec-First Development Scaffold

A starter template for building web projects — framework-agnostic, spec-driven, and works whether you build by hand, use an AI coding agent, or both.

---

## Quick start

```bash
git clone https://github.com/brootaylor/tech-agnostic-spec-first-dev-scaffold.git my-project
cd my-project
```

Then follow [WORKFLOW.md](./WORKFLOW.md) for the full setup guide. If you have a GitHub account, use the **"Use this template"** button to start from a clean copy of the repo.

---

## Why spec-first?

Writing specs before code forces clarity. Before anyone — human or agent — writes a single line of implementation, the spec defines exactly what a component should do, what states it has, how it should behave, and what tests it needs to pass.

The spec becomes the shared language for the project. If the output isn't right, the spec is the first place to look.

---

## Two ways to build

| | Handcrafted | AI-assisted |
|---|---|---|
| **How** | Use the specs and workflow as directions for building yourself | Use an AI coding agent to read specs and generate implementation |
| **Setup** | No extra config needed | See [AGENTS.md](./AGENTS.md) for agent setup |

Both paths follow the same workflow and use the same specs.

---

## Features

- **Spec-first workflow** — specs are written before any code is produced; the spec is the source of truth for humans and agents alike
- **Tech-agnostic** — currently supports 'Vanilla', Astro, Eleventy, React, and Svelte. More tech stack options can be added if needed.
- **Agent-agnostic** — currently includes config for Claude Code, Cursor, and GitHub Copilot, with a clear pattern for adding others
- **Modern platform guide** — a reference for humans and agents for which web platform APIs and features to use, and when a fallback is acceptable.
- **Optional: service worker** — offline and caching support with a strategy selector and framework-specific guidance
- **Optional: Storybook** — component development and documentation environment for React, Svelte, and plain JavaScript (aka, 'Vanilla')
- **Optional: security** — HTTP security headers, CSP configuration, and secure coding guidelines with framework-specific notes
- **Living documentation** — specs double as project documentation; keep them up to date and the whole project stays coherent

---

## Key files

| File | Purpose |
|------|---------|
| [`docs/project-brief.md`](./docs/project-brief.md) | Single source of truth — stack selector, conventions, agent rules |
| [`docs/modern-platform-guide.md`](./docs/modern-platform-guide.md) | Which web platform APIs and features to use |
| [`docs/design-tokens.md`](./docs/design-tokens.md) | Colour, spacing, and typography definitions |
| [`WORKFLOW.md`](./WORKFLOW.md) | Step-by-step guide from setup through to deployment |
| [`AGENTS.md`](./AGENTS.md) | How AI agents are configured in this project |

Optional configuration docs:

| File | Enables |
|------|---------|
| [`docs/service-worker.md`](./docs/service-worker.md) | Offline support — set active strategy in `project-brief.md` |
| [`docs/storybook.md`](./docs/storybook.md) | Storybook — enable in `project-brief.md` |
| [`docs/security.md`](./docs/security.md) | Security headers and CSP — set active option in `project-brief.md` |

---

## How specs work

Each spec defines the interface, behaviour, states, accessibility requirements, and test cases for what's being built. A status field controls whether an agent may act on it:

| Status | Meaning |
|--------|---------|
| `Draft` | Incomplete — do not implement |
| `Ready` | Complete — proceed with implementation |
| `Complete` | Implemented and tested |

Spec files live in `docs/specs/`. Use `docs/specs/_component-template.spec.md` as your starting point for any new spec.

---

## Project structure

```
my-project/
├── docs/
│   ├── project-brief.md                    ← single source of truth
│   ├── modern-platform-guide.md
│   ├── design-tokens.md
│   ├── service-worker.md
│   ├── storybook.md
│   ├── security.md
│   ├── features/                           ← user-facing feature specs
│   └── specs/                              ← technical specs for components, pages, layouts
│       ├── _component-template.spec.md
│       ├── components/
│       ├── pages/
│       └── layouts/
├── src/
│   ├── components/
│   ├── pages/
│   ├── layouts/
│   ├── styles/
│   ├── assets/
│   └── scripts/
└── .agents/
    ├── claude/
    ├── cursor/
    └── copilot/
```

> The example specs in `docs/features/` and `docs/specs/` are real, working examples that follow the same conventions you'd use in a production project. Use them as a reference or replace them with your own.
