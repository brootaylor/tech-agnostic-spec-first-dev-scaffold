# Tech-Agnostic Spec-First Development Scaffold

A starter template for building web projects — tech-agnostic, spec-first, and works whether you build by hand, use an "Ai" coding agent, or both.

> [!IMPORTANT]
> **Status:** *This is an active, evolving experiment, not a finished product. It'll keep changing as the idea gets tested against real projects — issues and discussion are welcome.*

---

## Quick start

**Get the files.** Either route lands you the same repo:

- **[Use this template](https://github.com/brootaylor/tech-agnostic-spec-first-dev-scaffold/generate)** — *starts a clean copy under your own GitHub account, with none of this repo's history. Needs a GitHub account.*
- **Or clone it**:

  ```bash
  git clone https://github.com/brootaylor/tech-agnostic-spec-first-dev-scaffold.git my-project
  cd my-project
  ```

**Then take the route that matches what you have:**

| You have | Start here | Your first moves |
|----------|-----------|------------------|
| An empty folder | [docs/workflow.md](./docs/workflow.md) — the ten-step guide | Point your agent at the scaffold, describe the project in `docs/project-brief.md`, pick your stack and install it. |
| A codebase that already exists | [docs/adopting-an-existing-project.md](./docs/adopting-an-existing-project.md) | Merge the scaffold in by hand, then `/survey` drafts the brief and `/audit` grades what's already there. |

Only the setup differs. Both routes reach the same specs, the same build loop, and the same gates.

You'll need [Git](https://git-scm.com) and [Node.js](https://nodejs.org) (LTS) — the full list is in [docs/workflow.md → Prerequisites](./docs/workflow.md#prerequisites).

---

## Why spec-first?

Writing specs before code forces clarity. Before anyone — human or agent — writes a single line of implementation, the spec defines exactly what a component should do, what states it has, how it should behave, and what tests it needs to pass.

The spec becomes the shared language for the project. If the output isn't right, the spec is the first place to look.

It also means the tool choice comes last, not first — React, Astro, an "Ai" agent, whatever — because none of that decides what needs building. The spec does.

---

## Two ways to build

| | Handcrafted | Ai-assisted |
|---|---|---|
| **How** | Use the specs and workflow as directions for building yourself | Use an "Ai" coding agent to read specs and generate implementation |
| **Setup** | No extra config needed | See [AGENTS.md](./AGENTS.md) for agent setup |
| **Building** | Work through the spec in order — interface, tests, implementation | Run the build loop: `/feature` → `/implement` → `/check` → `/audit` → `/complete` |

Both paths follow the same workflow and use the same specs.

---

## Features

- **Spec-first workflow** — specs are written before any code is produced; the spec is the source of truth for humans and agents alike
- **Tech-agnostic** — currently supports 'Vanilla', Astro, Eleventy, React, React + Next.js, Svelte, and Svelte + SvelteKit. More tech stack options can be added if needed.
- **Agent-agnostic** — any tool that reads the `AGENTS.md` convention works with no setup at all; Claude Code reads a `CLAUDE.md` that imports the same file, so every agent gets one source of truth, and there's a clear pattern for adding any other
- **A build loop with review gates** — 18 shared workflow skills take a `Ready` spec through work order, implementation, verification, audit, and completion, stopping for human review at each step. They're plain markdown shared by every agent, not one tool's feature
- **Modern platform guide** — a reference for humans and agents for which web platform APIs and features to use, and when a fallback is acceptable.
- **Optional: service worker** — offline and caching support with a strategy selector and framework-specific guidance
- **Optional: Storybook** — component development and documentation environment for React, Svelte, and plain JavaScript (aka, 'Vanilla')
- **Optional: security** — HTTP security headers, Content Security Policy (CSP) configuration, and secure coding guidelines with framework-specific notes
- **Living documentation** — specs double as project documentation; keep them up to date and the whole project stays coherent

---

## Key files

In the order you meet them:

| File | Purpose |
|------|---------|
| [`docs/workflow.md`](./docs/workflow.md) | Step-by-step guide from setup through to deployment — start here |
| [`docs/project-brief.md`](./docs/project-brief.md) | Single source of truth — stack selector, conventions, agent rules |
| [`docs/setup.md`](./docs/setup.md) | One-time setup — the procedure, and compatibility notes per stack combination |
| [`docs/modern-platform-guide.md`](./docs/modern-platform-guide.md) | Which web platform APIs and features to use |
| [`docs/design-tokens.md`](./docs/design-tokens.md) | Colour, spacing, and typography definitions |
| [`AGENTS.md`](./AGENTS.md) | How "Ai" agents are configured in this project |

Optional configuration docs:

| File | Enables |
|------|---------|
| [`docs/service-worker.md`](./docs/service-worker.md) | Offline support — set active strategy in `project-brief.md` |
| [`docs/storybook.md`](./docs/storybook.md) | Storybook — enable in `project-brief.md` |
| [`docs/security.md`](./docs/security.md) | Security headers and CSP — set active option in `project-brief.md` |

The build loop's working state, generated as you go — useful to read, no need to edit by hand:

| File | Purpose |
|------|---------|
| [`context/current-feature.md`](./context/current-feature.md) | The work order in flight, or a stub when nothing is being built |
| [`context/findings.md`](./context/findings.md) | Review findings raised by `/audit`, cleared by `/complete` |
| [`context/history/`](./context/history/) | Archived work orders — the record of what was built |

---

## How specs work

Each spec defines the interface, behaviour, states, accessibility requirements, and test cases for what's being built. A status field controls whether an agent may act on it:

| Status | Meaning |
|--------|---------|
| `Draft` | Incomplete — do not implement |
| `Ready` | Complete — proceed with implementation |
| `Complete` | Implemented and tested |

> [!IMPORTANT]
> **Promoting a spec from `Draft` to `Ready` is always a human act.** *No agent or skill grants itself that signal — it's how you say the contract is settled before anything gets built.*

Specs come in two kinds, and which one you write depends on what you are describing:

| You are describing | Start from | It lands in |
|--------------------|-----------|-------------|
| Something a **user can do**, and why it matters | `docs/features/_feature-template.md` | `docs/features/` |
| A reusable **component, page, or layout** | `docs/specs/_component-template.spec.md` | `docs/specs/` |

The test: can you write it as *"As a user, I want… so that…"*? If yes, it's a feature. If nobody wants it on its own — they want the thing it enables — it's a component. See [project-brief.md](./docs/project-brief.md) → Features and components for the full distinction, including which spec owns values the two must agree on.

---

## The build loop

If you're building with an agent, the workflow is a set of shared skills in `.agents/skills/` — plain markdown any capable agent can read and follow. They run in order, once per spec, and each one stops at a review gate rather than running on into the next:

```
  a spec you have promoted to  Ready
       │
       ├─  /feature ───▶  work order   context/current-feature.md
       ├─  /implement ─▶  code         src/** + checkpoint commits
       ├─  /check ─────▶  evidence     every "done when" proven
       ├─  /audit ─────▶  findings     context/findings.md
       └─  /complete ──▶  done         Status: Complete, archived
       │
       └──────────────▶  the next Ready spec
```

| Skill | What it does |
|-------|--------------|
| `/brief` | Preview a spec before committing to it — what it involves, what would block it |
| `/feature` | Turn a `Ready` spec into a work order with small, reviewable build steps |
| `/implement` | Build those steps one at a time — diff, plain-English explanation, approval |
| `/check` | Prove each "done when" against the running app |
| `/audit` | Review the code against the project's standards |
| `/try` | Get a manual walkthrough to click through yourself |
| `/complete` | Write `Complete` back to the spec, archive the work, make one commit |
| `/status` | See the queue, what's in flight, and the exact next action |

Others sit outside the loop — `/discovery`, `/survey`, `/fix`, `/rollback`, `/debug`, `/prototype`, `/tests`, `/ci`, and `/release` — plus `/autopilot`, which carries a settled spec the whole way instead of stopping at each gate.

> [!IMPORTANT]
> *Two rules hold however a skill is invoked: **no skill moves a spec from `Draft` to `Ready`**, and **no skill creates, switches, merges, or deletes a branch** — the loop commits to whatever branch you're already on. See [AGENTS.md](./AGENTS.md) for the full reference.*

---

## Project structure

```
my-project/
├── README.md
├── AGENTS.md                               ← cross-tool agent instructions and skill reference
├── docs/
│   ├── workflow.md                         ← the ten-step guide, setup to deployment
│   ├── adopting-an-existing-project.md     ← setup route for an existing codebase
│   ├── project-brief.md                    ← single source of truth
│   ├── setup.md                            ← one-time setup: procedure and stack compatibility notes
│   ├── modern-platform-guide.md
│   ├── design-tokens.md
│   ├── service-worker.md
│   ├── storybook.md
│   ├── security.md
│   ├── features/                           ← user-facing feature specs
│   │   └── _feature-template.md
│   ├── specs/                              ← technical specs for components, pages, layouts
│   │   ├── _component-template.spec.md
│   │   ├── components/
│   │   ├── pages/
│   │   ├── layouts/
│   │   └── hooks/                          ← reusable-logic specs, only if you need them
│   └── reference/                          ← reference images a spec points at
├── src/                                    ← three files ship; the subdirectories below are yours to create
│   ├── index.html                          ← ships; some stacks manage their own — see workflow.md Step 3
│   ├── scripts/main.js                     ← ships; the application entry point
│   ├── assets/icons/spinner.svg            ← ships; a spec depends on it — keep it whatever your stack
│   ├── components/
│   ├── pages/
│   ├── layouts/
│   ├── styles/
│   ├── lib/                                ← shared utilities
│   └── types/                              ← global types (TypeScript only)
├── prototypes/                             ← not shipped; /prototype creates it, /complete deletes it
├── context/                                ← the build loop's working state
│   ├── sessions.md                         ← where things stand, rewritten each session (gitignored)
│   ├── decisions.md                        ← why choices were made, append-only (gitignored)
│   ├── current-feature.md                  ← the work order in flight
│   ├── findings.md                         ← review findings ledger
│   └── history/                            ← archived work orders
└── .agents/
    ├── claude/                             ← the one agent needing its own config
    └── skills/                             ← shared workflow skills, read by any agent
```

> [!IMPORTANT]
> *The example specs in `docs/features/` and `docs/specs/` are real, working examples that follow the same conventions you'd use in a production project. Use them as a reference or replace them with your own.*
