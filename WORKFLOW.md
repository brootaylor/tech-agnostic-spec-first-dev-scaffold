# WORKFLOW.md

A step-by-step guide to using this scaffold to build a web project — by hand or with an AI coding agent.

---

## Prerequisites

| Tool | Notes |
|------|-------|
| A terminal | macOS and Linux have one built in. On Windows, [Git Bash](https://gitforwindows.org) or [Windows Terminal](https://aka.ms/terminal) are good options |
| [Git](https://git-scm.com) | Required to clone this scaffold. Recommended for version control throughout development |
| [Node.js](https://nodejs.org) | Required by most tooling — install the LTS version |
| [npm](https://www.npmjs.com) | Comes with Node.js |
| A code editor | [VS Code](https://code.visualstudio.com) is a good option |

> A GitHub account is only required if you want to use the **"Use this template"** button. Cloning or downloading the ZIP does not require one.

---

## Before you start

The scaffold comes with a full set of starter files: root config (`README.md`, `package.json`, `.gitignore`, `.nvmrc`), docs (`project-brief.md`, `design-tokens.md`, feature and spec examples), default source files (`src/index.html`, `src/scripts/main.js`), and agent configs under `.agents/`.

The spec and feature files are illustrative examples — replace or modify them to suit your project.

> **Commit everything before making any changes.** This gives you a clean baseline to return to.

---

## Step 1 — Configure your agent

> **Agent-only step.** Skip this if you're building by hand.

Make sure your agent has what it needs:

| Agent | Prerequisites |
|-------|--------------|
| Claude Code | Node.js + an [Anthropic API key](https://console.anthropic.com) |
| Cursor | The [Cursor app](https://cursor.sh) |
| GitHub Copilot | A GitHub account with [Copilot access](https://github.com/features/copilot) + the relevant IDE extension |

Open `.agents/<your-agent>/` and follow the setup notes there to point your agent at the config file. For Claude Code:

```bash
ln -s .agents/claude/CLAUDE.md CLAUDE.md
```

See `AGENTS.md` for a full explanation of how agent config files are wired up.

---

## Step 2 — Fill in `project-brief.md`

Open `docs/project-brief.md` and complete two things before anything else:

**Describe your project** — replace the placeholder under "What this project is" with a plain description of what you're building and who it's for.

**Choose your stack** — mark exactly one option per category as `[active]`:

- Framework
- Language
- Styles
- Unit testing
- E2E testing
- Build
- Service worker *(optional)*
- Storybook *(optional)*
- Linting *(optional)*
- Security *(optional)*

`project-brief.md` is the first thing the agent reads. Getting it right before writing any specs avoids problems later.

---

## Step 3 — Set up your stack

With your stack selected, `package.json` needs to be populated with the correct dependencies.

**By hand:**

- Set up `package.json` and any required config files (e.g. `vite.config.js`, `jest.config.js`) based on your active stack selections
- Update `.nvmrc` with the Node.js version your framework recommends
- Update the stack-specific section of `.gitignore` (e.g. `dist/` for Vite, `_site/` for Eleventy, `.astro/` for Astro)

**With an agent:**

```
Read `docs/project-brief.md` and complete the initial project setup.
```

The agent will populate `package.json`, generate any required config files, and update `.nvmrc` and `.gitignore`. It covers setup only — specs and design tokens come in later steps.

**Starting files:** `src/index.html` and `src/scripts/main.js` are included for Vanilla, React, and Svelte stacks. For React and Svelte, `main.js` needs to be updated to mount the app. For Astro and Eleventy, remove both files — those frameworks manage their own pages and templating.

> **Commit your work** once setup is complete and dependencies are in place.

---

## Step 4 — Write a feature spec

Pick the first feature you want to build and create a spec for it in `docs/features/`.

A feature spec describes what a user can do, not how it's built. Use `docs/features/dark-mode.md` as a reference — it covers the overview, user stories, acceptance criteria, and which components are required.

Finish the feature spec before moving on to component specs.

> `docs/features/` is for user-facing feature specs only. Technical configuration docs (service worker, Storybook) live directly in `docs/`.

---

## Step 5 — Write your component specs

Look at the "Components required" section of your feature spec. For each item listed, create a spec using `docs/specs/_component-template.spec.md` as your starting point. This step covers components primarily, but the same process applies to pages and layouts too:

| Spec type | Write the spec here | Code will be generated here |
|-----------|--------------------|-----------------------------|
| Component | `docs/specs/components/` | `src/components/<Name>/` |
| Page | `docs/specs/pages/` | `src/pages/<Name>/` |
| Layout | `docs/specs/layouts/` | `src/layouts/<Name>/` |

See `docs/specs/components/button.spec.md` for a complete worked example.

Set the status to `Draft` while writing. Change it to `Ready` only when every section is complete — the agent will not proceed with a `Draft` spec.

---

## Step 6 — Define your design tokens

`docs/design-tokens.md` is a template for defining your project's visual language — colours, spacing, typography, and other design constants. This step doesn't have to happen before writing specs — it just needs to be done before the agent can write any styles. It can also be revisited at any stage as the project evolves, and run multiple times if you're using an agent to generate the style files.

**By hand:**

- Create `src/styles/tokens.css` (or `tokens.scss` if Sass is active) and implement the values from `docs/design-tokens.md`
- Create `src/styles/main.css` (or `main.scss`) and import the token file at the top

**With an agent:**

```
Read `docs/design-tokens.md` and create the token and main style files.
```

If `docs/design-tokens.md` is empty, the agent will stop and ask you to fill it in first.

> **Commit your work** once your tokens are defined and implemented.

---

## Step 7 — Build

Once a spec's status is `Ready`, it's time to build.

**By hand:**

- Use the spec as your blueprint
- Work through it in order: interface first, tests next, then implementation

**With an agent** — prompt for a single spec:

```
Read `docs/specs/components/button.spec.md` and scaffold the implementation.
```

Or scaffold all ready specs at once:

```
Read all specs in `docs/specs/` with a status of `Ready` and scaffold the implementation for each one.
```

Claude Code users can also use the custom command shorthand:

```bash
/create-component Button
```

> `/create-component` is defined in `.agents/claude/commands/create-component.md` and is Claude Code-specific. Other agents don't support it — use the prompt examples above instead.

The agent will read the spec, derive the interface, write the tests first, then implement until all tests pass.

If the agent stops to ask a question, the spec is likely ambiguous in that area. Go back, clarify the relevant section, and re-run.

---

## Step 8 — Review the output

Generated code appears in `src/` under the relevant directory (see the table in Step 5). Check it against the spec:

- Does every `TC-##` in the spec have a corresponding passing test?
- Does the implementation match the behaviour described?
- Are the accessibility requirements met?

**If something is wrong**, there are two likely causes:

- **The spec is ambiguous** — update the spec first, then ask the agent to fix the implementation
- **The spec is clear but the output is wrong** — re-prompt with the relevant section highlighted:
  ```
  Re-read the Behaviour section of `docs/specs/components/button.spec.md` and correct the implementation.
  ```

Don't edit the implementation directly without also updating the spec. The spec is the source of truth — if the two drift apart, the agent's output becomes unpredictable.

Once everything checks out, update the spec status to `Complete`.

> **Commit your work** once the spec is marked `Complete` and all tests are passing.

---

## Step 9 — Run it locally or deploy

**Most frameworks** (Vanilla + Vite, React, Svelte, Astro):

```bash
npm install
npm run dev
```

**Eleventy:**

```bash
npm install
npx @11ty/eleventy --serve
```

For deployment, [Netlify](https://www.netlify.com) and [Vercel](https://vercel.com) work well with all of these frameworks. Connect your Git repository, set the build command and output directory for your framework, and they handle the rest.

---

## Step 10 — Iterate

Once a spec is marked `Complete` and committed, return to Step 4 and repeat the cycle for the next feature — feature spec first, then component specs, then build and review.

As the project grows, update `docs/project-brief.md` with any new conventions or constraints the agent needs to know about.

---

## The spec is the source of truth

The spec files are the living documentation of your project. Keep them up to date and the agent stays useful. Let them drift and the agent becomes unpredictable.

If the output isn't right, the spec is always the first place to look. If the spec is clear and correct, but the output is wrong, re-prompt with the relevant section highlighted. If the spec is ambiguous, update it first to clarify, then re-run.
