# Workflow

A step-by-step guide to using this scaffold to build a web project — by hand or with an "Ai" coding agent.

---

## The whole thing at a glance

Set up once, then loop. The ten steps below are the long form of this:

```
SETUP  ·  once per project

  clone the template
      │
      ├─  Step 1   configure agent ····  .agents/<tool>/, linked to it
      ├─  Step 2   project-brief.md ····  describe it, pick your stack
      └─  Step 3   set up the stack ····  dependencies and configuration
                                          + /tests  /ci
      │
      ▼
SPEC  ·  per feature, and always human work

      ├─  Step 4   feature spec ·······  docs/features/<name>.md
      ├─  Step 5   component specs ····  docs/specs/**/<name>.spec.md
      └─  Step 6   design tokens ······  docs/design-tokens.md
      │
      │           Steps 4 and 5 are written as   Status: Draft
      │           Step 6 has no status and skips the gate below —
      │           it is not a spec, it just has to be done before
      │           any styles are written
      ▼
  ┌────────────────────────────────────────────────────────────┐
  │ HUMAN GATE     you promote   Draft ──▶ Ready               │
  │ No skill ever does this. It is how you say the             │
  │ contract is settled and building may begin.                │
  └────────────────────────────────────────────────────────────┘
      │
      ▼
BUILD LOOP  ·  per spec, one spec at a time      ← Steps 7 and 8

      ├─  + /brief      preview a spec before committing to it
      │
      ├─  /feature ───▶  context/current-feature.md      « review gate »
      │                 the work order: small build steps
      │
      ├─  /implement ─▶  src/** + checkpoint commits     « gate per step »
      │                 diff + plain-English explanation
      │
      ├─  /check ─────▶  evidence against each "done when"
      │
      ├─  /audit ─────▶  context/findings.md
      │                 the worst two severities block /complete
      │
      ├─  + /try        manual walkthrough to click through yourself
      │
      └─  /complete ──▶  Status: Complete  ·  archived to context/history/
                        one commit covering code + bookkeeping
      │
      └──────────────▶  next Ready spec, back to the top of the loop
                        (or back to Step 4 for a whole new feature)


Optional routes  ·  each replaces work above rather than adding to it

  /discovery   ▶  Steps 2 and 4, drafted from a conversation
  /prototype   ▶  Step 6, arrived at visually instead of abstractly

Anytime         /status  where things stand    /fix       bug with no spec
                /debug   why is this failing   /rollback  undo a feature
```

`+` marks an optional addition — run it as well, or not at all. The two
optional routes are worth knowing about properly:

**`/discovery` is another way through Steps 2 and 4.** Instead of filling in the
brief and your first feature specs from a blank page, it interviews you — one
question at a time, over as many turns as the project needs — then drafts both
files from that conversation and shows you everything before it writes. What it
writes is always `Draft`, so promotion stays yours. Use it when you'd rather
talk the product through than write it cold; writing by hand is equally valid,
and skipping it changes nothing downstream.

**`/prototype` is another way through Step 6.** Instead of choosing colour,
spacing, and type values abstractly, it writes static mockups of your real
screens into `prototypes/`, all sharing one `theme.css`, so you can look at
actual screens and adjust until the look is right. Run it once you have a
feature spec (Step 4) to prototype against. The mockups are throwaway — the
theme is the keeper, and your first user interface (UI) feature ports it into
`docs/design-tokens.md` and your stylesheet.

Everything from `/feature` down is the build loop — see
[AGENTS.md](../AGENTS.md) for the full skill reference.

---

## Prerequisites

| Tool | Notes |
|------|-------|
| A terminal | macOS and Linux have one built in. On Windows, [Git Bash](https://gitforwindows.org) or [Windows Terminal](https://aka.ms/terminal) are good options |
| [Git](https://git-scm.com) | Required to clone this scaffold. Recommended for version control throughout development |
| [Node.js](https://nodejs.org) | Required by most tooling — install the Long Term Support (LTS) version |
| [npm](https://www.npmjs.com) | Comes with Node.js |
| A code editor | [VS Code](https://code.visualstudio.com) is a good option |

> A GitHub account is only required if you want to use the **"Use this template"** button. Cloning or downloading the ZIP does not require one.

---

## Before you start

The scaffold comes with a full set of starter files: root config (`README.md`, `package.json`, `.gitignore`, `.nvmrc`), docs (`project-brief.md`, `design-tokens.md`, feature and spec examples), default source files (`src/index.html`, `src/scripts/main.js`, `src/assets/icons/spinner.svg`), agent configs under `.agents/`, and the build loop's empty working state under `context/`.

The spec and feature files are illustrative examples — replace or modify them to suit your project. So are the token values in `docs/design-tokens.md`: that file arrives complete rather than blank, and every value in it is a baseline waiting to be replaced at Step 6, not a choice made for you.

> **Commit everything before making any changes.** This gives you a clean baseline to return to.

> [!IMPORTANT]
> **Already have a codebase?** *These ten steps assume you are starting from an empty folder. To bring an existing project under the loop — an inherited repository, a prototype that outgrew itself, an "Ai"-generated app that needs to become production-ready — read [`docs/adopting-an-existing-project.md`](./adopting-an-existing-project.md) instead. It covers merging the scaffold into a repository that is already occupied, then rejoins this guide at Step 4.*

---

## Step 1 — Configure your agent

> **Agent-only step.** Skip this if you're building by hand.

Make sure your agent has what it needs:

| Agent | Prerequisites | Setup |
|-------|--------------|-------|
| Codex, Cursor, GitHub Copilot, Gemini CLI, Jules, Aider, Zed, Windsurf, Devin, OpenCode | Whatever the tool itself needs to run | **None.** They read `AGENTS.md` from the project root, which points them at the workflow skills — you invoke those by name (Step 7) |
| Claude Code | A [Claude](https://claude.ai) Pro, Max, Team or Enterprise plan to sign in with, or an [Anthropic Console account](https://console.anthropic.com) billed by usage — the free plan does not include Claude Code. The installer ships a self-contained binary, so Claude Code itself needs no Node.js | The two links below |

`AGENTS.md` is an open convention that most coding agents now read directly from the project root, so for them there is nothing to set up at all. Claude Code is the exception: [it reads `CLAUDE.md`, not `AGENTS.md`](https://code.claude.com/docs/en/memory), and gives you no way to change that filename. The scaffold's `CLAUDE.md` imports `AGENTS.md`, which is the approach Anthropic's own documentation recommends, so Claude Code ends up reading the same instructions as everything else — it just needs a pointer to reach them.

This scaffold keeps that configuration in `.agents/` instead, so cloning it never forces one developer's tool on everybody else. If you're using Claude Code, your one setup task is to create links at the filenames it expects, pointing back into `.agents/`:

```bash
ln -s .agents/claude/CLAUDE.md CLAUDE.md
mkdir -p .claude && ln -s ../.agents/skills .claude/skills
```

> [!IMPORTANT]
> *Two ways this step goes wrong:*
>
> - *`.claude/` is gitignored, so it does not exist in a fresh clone. Without the `mkdir -p`, that second command fails with `No such file or directory` and none of the workflow skills are available to you.*
> - **Do not run Claude Code's built-in `/init`.** *It generates a `CLAUDE.md` by analysing the codebase, but here that filename is a symlink into `.agents/`. Running it either writes straight through the link and overwrites the tracked original, or replaces your pointer with a regular file that shadows it and drifts from it silently. If you have already run it: delete the root `CLAUDE.md`, then run `git status`. If it reports `.agents/claude/CLAUDE.md` as modified, the original was overwritten — restore it with `git restore .agents/claude/CLAUDE.md`. Then recreate the link. Deleting the root file alone does not undo the overwrite: the link will resolve happily to the generated content, so check `git status` before assuming you have recovered.*

Both links are gitignored, so your choice of agent never travels with the repository. On Windows, where `ln -s` needs Developer Mode or an elevated terminal, copy the files instead and keep them in sync by hand.

See `AGENTS.md` for the pointer table and the notes on adding an agent that expects a config file of its own.

### Create the two state files

**This part applies to every agent, including the ones that needed no setup above.**

The build loop keeps its long-term memory in two files under `context/`. Both are gitignored — they accumulate a record of *your* work on *your* project, so they are yours and never travel with the repository, which is also why a fresh clone does not have them and the scaffold cannot create them for you. Make them now:

```bash
printf '# Where things stand\n\n_Nothing recorded yet. Rewrite this block in full at the end of each session._\n' > context/sessions.md
printf '# Decisions\n\n_Newest first. One entry per decision, not per session._\n' > context/decisions.md
```

| File | Holds | How it is written |
|------|-------|-------------------|
| `context/sessions.md` | Where things stand right now: the queue, the branch, what is open, the next action | Rewritten **in full** at the end of a session of substance, never appended to. Twenty lines is the budget |
| `context/decisions.md` | Why a choice was made, and what was rejected and why | Append-only, newest first. One entry per decision, not per session — most sessions add nothing |

> [!IMPORTANT]
> **These two files are the only things that survive a cleared context.** *A `/compact` or `/clear` discards the conversation, and nothing warns you — there is no error, just a later session that has to rediscover what was known, and a decision you already settled being reopened because no record of it exists. Nothing writes these files for you: ask your agent to update both whenever you finish a session of substance, and to read both before acting when it starts cold. `AGENTS.md` → "Keep the state file current" has the full convention.*

---

## Step 2 — Fill in `project-brief.md`

Open `docs/project-brief.md` and complete two things before anything else:

**Describe your project** — replace the placeholder under "What this project is" with a plain description of what you're building and who it's for.

**Choose your stack** — the `[active]` marks arrive pre-filled with the scaffold's shipped default (Vanilla, JavaScript, plain CSS, Vite), because the example specs and the `src/` starting files are written against that combination. **Replace them rather than adding to them** — clear the shipped mark in a category before marking your own, so exactly one option per category ends up `[active]`:

- Framework
- Language
- Styles
- Unit testing
- End-to-end (E2E) testing *(optional)*
- Build
- Service worker *(optional)*
- Storybook *(optional)*
- Linting *(optional)*
- Security *(optional)*

`project-brief.md` is the first thing the agent reads. Getting it right before writing any specs avoids problems later.

**No tool checks this for you.** A category left with two `[active]` marks — the shipped default plus the one you added — or with none at all is not an error anything reports. An agent reads whatever it finds and installs against it, so a leftover default can pull in a framework you never chose. Your agent is required to read the selections back to you before it generates any config or dependency list (see "Confirm the stack before setup" in `project-brief.md` → Agent behaviour rules); check that list against what you actually picked, and say so if a category names two.

> **Not sure yet?** `/discovery` runs a guided interview — one question at a time — and drafts this file and your first feature specs from the conversation, showing you everything before it writes. It's optional, and it never promotes a spec past `Draft`. Writing them by hand is equally valid.

---

## Step 3 — Set up your stack

With your stack selected, `package.json` needs to be populated with the correct dependencies.

The procedure itself lives in `docs/setup.md` rather than being repeated here, so there's no second copy to fall out of date — and it's the file your agent reads anyway. Two sections there matter:

- **Setup instructions** — the procedure in two versions: a list to work through by hand, and the numbered version an agent executes. Follow whichever suits you.
- **Stack compatibility notes** — the combinations that need extra wiring. Check yours before generating any config.

**With an agent, this is the whole prompt:**

```
Read `docs/project-brief.md`, then follow `docs/setup.md` to complete the initial project setup.
```

It covers setup only — specs and design tokens come in later steps.

> [!IMPORTANT]
> **The Commands section of `AGENTS.md` is read by the loop, not just written by you.** *`/check` and `/try` use it to start the app, `/debug` to reproduce a failure, and `/implement` and `/complete` to run the build before anything is committed. Left as the shipped `<command>` placeholders it produces no error — each skill simply reports the command as a gap and carries on with less evidence than it should have.*
>
> *Leave the `Test` and `Verify` rows alone for now — `/tests` and `/ci` write those two. They behave differently when absent:*
>
> - *`Test` — a real command here is the single switch that turns the testing gate on, so no row means no gate.*
> - *`Verify` — missing just means those skills fall back to the build and test commands you filled in above.*

**Starting files.** `src/index.html` and `src/scripts/main.js` ship with the scaffold. What you do with them depends on the framework you marked `[active]`:

| Framework | What to do |
|-----------|------------|
| Vanilla | Keep both as they are |
| React, Svelte | Keep both — and update `main.js` to mount the app |
| Astro, Eleventy, React + Next.js, Svelte + SvelteKit | Remove both — these four manage their own pages and routing |

`src/assets/icons/spinner.svg` stays whichever framework you picked: an icon file is framework-neutral, and `docs/specs/components/button.spec.md` lists it as a dependency and requires it inlined.

> **Testing and automatic checks are opt-in.** `/tests` adds a unit test runner for your active stack and turns the testing gate on; `/ci` defines one `Verify` command and the matching GitHub Actions workflow. Run either now or later — the loop works without them.

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

> [!IMPORTANT]
> *Set the status to `Draft` while writing, and change it to `Ready` only when every section is complete. An agent will not proceed with a `Draft` spec, and it will never promote one for you — that decision is yours alone, and it is how you say the contract is settled.*

---

## Step 6 — Define your design tokens

`docs/design-tokens.md` is a template for defining your project's visual language — colours, spacing, typography, and other design constants.

This step doesn't have to happen before writing specs — it just needs to be done before the agent can write any styles. It can also be revisited at any stage as the project evolves, and run multiple times if you're using an agent to generate the style files.

**By hand:**

- Create `src/styles/tokens.css` (or `tokens.scss` if Sass is active) and implement the values from `docs/design-tokens.md`
- Create `src/styles/main.css` (or `main.scss`) and import the token file at the top

**With an agent:**

```
Read `docs/design-tokens.md` and create the token and main style files.
```

The file arrives complete rather than blank, so "filled in" is not the test. Its `**Last updated:**` line is: while that still holds the placeholder comment, the values are the scaffold's baselines and the agent will stop and ask you to settle them first. Replace the comment with a date once they are yours.

> **Want to settle the look first?** Run `/prototype` yourself — nothing invokes it for you — any time after Step 4 and before you build. It locks one theme and mocks up screens against it, but does not write this file itself: the theme reaches it in three hops:
>
> 1. **`/prototype`** writes the theme as CSS variables in `prototypes/theme.css`, plus static mockups that link it. Commit the folder — until the theme is ported, it lives nowhere else.
> 2. **`/feature`** spots `prototypes/` on your first UI feature and makes porting `theme.css` into `docs/design-tokens.md` **and** your stylesheet the first build step.
> 3. **`/complete`** deletes `prototypes/` once that feature is built.

> **Commit your work** once your tokens are defined and implemented.

---

## Step 7 — Build

Once a spec's status is `Ready`, it's time to build.

**By hand:**

- Use the spec as your blueprint
- Work through it in order: interface first, tests next, then implementation

**With an agent** — two commands, in order:

```
/feature button
```

`/feature` reads the spec, sizes the work, and writes small reviewable build steps to `context/current-feature.md`. It refuses a `Draft` spec, so promote it to `Ready` first. Read what it wrote — and adjust it if you need to — before running `/implement` below. That pause is the review gate: `/feature` plans, it never starts building.

```
/implement
```

`/implement` needs no argument — it builds whatever work order `/feature` left in `context/current-feature.md`, one step at a time. For each step it shows the diff, explains it in plain English, runs whatever checks the project has, and waits for you to approve before moving on. It commits to whatever branch you're on and never creates or merges branches.

To preview a spec before committing to it, run `/brief` — it explains what the spec involves, what it depends on, and what would block it, without writing anything.

If the agent stops to ask a question, the spec is likely ambiguous in that area. Go back, clarify the relevant section, and re-run.

> **`/autopilot`** is for the times you want the agent to carry a settled spec the whole way — one bounded pass, no pausing at each review point.
>
> - **Runs** the lot: work order, build steps, verification, gates, checkpoint commits — then stops with a review packet for you.
> - **Never** runs `/complete`, pushes, or deploys.
> - **Opt-in only.** The step-at-a-time path above is the default, because the review between steps is the point of a spec-first loop.

> These commands live in `.agents/skills/` and work the same in any agent. Prefer them to freehand prompts — they encode the review gates.

---

## Step 8 — Review the output

Generated code appears in `src/` under the relevant directory (see the table in Step 5). Check it against the spec:

- Does every `TC-##` in the spec have a corresponding passing test? *(Only once the project has a test runner — unit testing ships as `None`, so until you have run `/tests` there is nothing to write tests with, and the agent will say so rather than claim a case was covered.)*
- Does the implementation match the behaviour described?
- Are the accessibility requirements met?

**With an agent**, three commands cover this. Run them in this order, and only the ones your change calls for:

| Command | What it does | Run it when |
|---------|--------------|-------------|
| `/check` | Proves each "done when" against the running app | A criterion needs observed behaviour — a click, request, command, download, background job, multi-screen flow — or the work rendered or changed any UI at all |
| `/audit` | Reviews the code against the project's standards, writing findings to `context/findings.md` | The work touched a security boundary — authentication, payments, secrets, personal data, migrations, destructive or external operations — or changed markup, styles, or design tokens |
| `/try` | Writes a step-by-step manual walkthrough for you to click through | The change affects UI, navigation, copy, a public API or command-line interface, output, or another workflow someone uses directly |

`/complete` applies these same three as gates before it will log anything, so running them here is how you find problems early. A gate the work needs but the project cannot run is a blocker, not a note — and the two worst audit severities block `/complete` outright.

**If something is wrong**, there are three likely causes:

- **The spec is ambiguous** — update the spec first, then ask the agent to fix the implementation
- **You can't tell why it's failing** — `/debug` reproduces the symptom, isolates the failing path, and hands the evidence to `/fix` or `/implement`. It edits nothing itself
- **The spec is clear but the output is wrong** — re-prompt with the relevant section highlighted:
  ```
  Re-read the Behaviour section of `docs/specs/components/button.spec.md` and correct the implementation.
  ```

> [!IMPORTANT]
> *Don't edit the implementation directly without also updating the spec. The spec is the source of truth — if the two drift apart, the agent's output becomes unpredictable.*

Once everything checks out, close the work out — by hand or with `/complete`.

**By hand:** set the spec's `**Status:**` to `Complete` and its `**Last updated:**` to today's date, archive `context/current-feature.md` to `context/history/features/`, reset it to its stub, then commit.

**With an agent:** `/complete` does all of that. It is the only part of the loop allowed to edit a spec file, and only those two lines. It makes one commit covering the code and the bookkeeping, and stops to ask before pushing — finishing is not permission to push.

> **Commit your work** once the spec is marked `Complete` and all tests are passing.

---

## Step 9 — Run it locally or deploy

**Most frameworks** (Vanilla + Vite, React, React + Next.js, Svelte, Svelte + SvelteKit, Astro):

```bash
npm install
npm run dev
```

**Eleventy:**

```bash
npm install
npx @11ty/eleventy --serve
```

For deployment, [Netlify](https://www.netlify.com), [Vercel](https://vercel.com), and [Render](https://render.com) all work well with these frameworks. Connect your Git repository, set the build command and output directory for your framework, and they handle the rest.

**With an agent**, `/release` prepares a deployment for **Render or Vercel** — the two providers whose config files it knows the exact shape of. It sits outside the build loop: run it after `/complete`, or not at all. Netlify isn't unsupported, it just needs no skill: connect the repository as described above, and `docs/security.md` covers its headers.

- **Picks the target and shape.** With no argument it recommends a provider when the choice is obvious and asks when it isn't, then settles the shape — static site, web service, worker or cron job on Render; framework app, static output, serverless functions or a monorepo project on Vercel. It says so plainly when a provider is a poor fit. `/release check` gives a read-only readiness report instead.
- **Runs your real commands.** The build, the tests where they're declared, and the start or preview command, then smoke-tests the health path. A missing command is reported as a gap, never guessed at.
- **Writes local config only.** `render.yaml`, or `vercel.json` where Vercel's defaults aren't enough. Environment variables go in by name — it never writes a secret value — and when the Security selection in `docs/project-brief.md` is active it carries the headers from `docs/security.md` in unchanged.
- **Ends with a release packet:** target, shape, files changed, checks run, environment names needed, the exact smoke test, blockers, and the next action.

It stops before any actual deploy, remote service change, remote environment variable, or push — each needs a separate yes from you in that conversation.

---

## Step 10 — Iterate

**Ready for the next feature?** Once a spec is marked `Complete` and committed, return to Step 4 and repeat the cycle — feature spec first, then component specs, then build and review.

**Picking up after a break, or a cleared context?** Run `/status`. It reports the spec queue, what's in progress, and the exact next action, all read from files on disk — so a fresh session knows exactly as much as the last one did.

**Need to know *why* a choice was made?** That is the one thing `/status` cannot rebuild from disk: which approaches you already rejected, and what you were part-way through deciding. `context/sessions.md` and `context/decisions.md` hold it (Step 1) — ask your agent to update both at the end of any session that settled something.

**Found a bug that has no spec?** `/fix "<description>"` writes a short fix work order and runs it through the same build loop, logged separately under `context/history/fixes/`.

**Built something you now want gone?** `/rollback "<feature>"` finds the commit that introduced it, checks what has been built on top of it since, and writes a reversal plan for you to review before any code changes. The original spec goes back to `Ready` rather than disappearing — the contract stands, only the implementation is withdrawn.

**Learned a new convention or constraint?** Add it to `docs/project-brief.md` as the project grows. It is the first thing every agent reads.

---

## The spec is the source of truth

The spec files are the living documentation of your project. Keep them up to date and the agent stays useful. Let them drift and the agent becomes unpredictable.

If the output isn't right, the spec is always the first place to look. If the spec is clear and correct, but the output is wrong, re-prompt with the relevant section highlighted. If the spec is ambiguous, update it first to clarify, then re-run.
