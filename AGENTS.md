# AGENTS.md

Instructions for "Ai" coding agents working in this project.

`AGENTS.md` is a cross-tool convention that most coding agents read directly from
the project root - Claude Code, Codex, Cursor, GitHub Copilot, Gemini CLI and
OpenCode among them - so this is the entry point for all of them, with no config
file of their own. For an agent that expects one anyway, see
`docs/agent-setup.md`.

## What this is

This project's own description - what it is, who it's for, its goals and
constraints - lives in `docs/project-brief.md`.

This project is spec-first: a spec defines interface, behaviour, states,
accessibility, and test cases before any implementation exists, so the spec is
the contract a human or an agent is held to. Tool choice comes last. The stack
is selected in `docs/project-brief.md`, not assumed here.

The build loop is a workflow layer, not an app skeleton. The workflow files come
first: the scaffold is cloned into an empty folder, and the stack is then set up
*inside* that clone at `docs/workflow.md` Step 3 - `package.json` populated and
the framework's config files written from the Stack selections in
`docs/project-brief.md`.

> [!IMPORTANT]
> *Do not run a framework scaffolder (`create-next-app`, `npm create vite`, and
> so on) inside the clone. They expect an empty directory and overwrite the
> files this one already has, and they do not reliably refuse: `create-vite`
> offers "Remove existing files and continue" as one of three choices. Step 3
> writes the framework config directly instead.*

For a codebase that already exists, the order is reversed - the code is there
first and the scaffold merges in on top - and the early steps differ: see
`docs/adopting-an-existing-project.md`.

The workflow is defined by the local skills and context files below.

## Read these for full context

Start with these three, before acting. An agent that expands `@` imports -
Claude Code does - already has them in context and should not read them again:

- @docs/project-brief.md - **the single source of truth**: stack selection,
  browser targets, accessibility standard, coding conventions, and agent
  behaviour rules
- @context/current-feature.md - the work order for the one feature, fix, or
  rollback being built right now, or the stub when nothing is in flight
- @context/findings.md - the review ledger `/audit` writes and `/complete` clears

Then, when they apply:

- `context/sessions.md` - where the work stands. With `decisions.md` below, one of
  the only two files that survive a `/compact` or `/clear`. One **Where things
  stand** block and nothing else: queue, branches, what is open, the next action.
  Gitignored and personal to you, so it will not exist in a fresh clone; create it
  on first use. **See "Keep the state file current" below**
- `context/decisions.md` - why a choice was made and what was rejected, newest
  first. Append-only, and one entry per decision rather than per session. The
  only thing here that git history cannot reconstruct. Gitignored and personal
  to you as well; create it on first use
- `context/history/` - archived work orders: what was built, in what order, and why

## Keep the state file current

**`context/sessions.md` and `context/decisions.md` are the only things that
survive a context reset.** A `/compact` or `/clear` discards the conversation,
and nothing warns you - there is no error, just a later session that has to
rediscover what was known.

**Read them before acting when starting cold**, and after any `/clear` or
`/compact`. Neither is imported with this file, the way the three above are, so
they have to be opened deliberately. Read `sessions.md` in full - it is one
twenty-line block. From `decisions.md`, read only the headings (`grep '^## '
context/decisions.md`), and open the full entry when a choice looks settled
and you are about to revisit it. The whole file grows without limit, so reading
all of it costs more with every decision recorded.

**Rewrite it wholesale, every session; never append to it.** It is a snapshot of
now, not a changelog. Appending leaves a confident account of a state that
stopped being true some sessions ago, and nothing detects that - the stale line
is indistinguishable from the fresh ones. **Twenty lines is the budget.** If
something will not fit, it belongs in `context/decisions.md`, or nowhere.

**Write a decision down when you make one**, in `context/decisions.md`: what was
chosen, and what was rejected and why. Most sessions add nothing there, and that
is correct - it records decisions, not activity. Knowing an approach was tried
and dropped is what stops a later session repeating it.

**Do not narrate the work.** Git history already holds what changed. If the user
asks to update memory, that always includes both files.

Beyond `docs/project-brief.md`, the project's own documentation set is
authoritative for everything else:

- `docs/modern-platform-guide.md` - read before writing any HTML, CSS, or JS
- `docs/design-tokens.md` - read before writing any CSS
- `docs/security.md` - read before generating HTML or deployment config
- `docs/service-worker.md` and `docs/storybook.md` - optional features, only when
  marked active in `docs/project-brief.md`
- `docs/stack-setup.md` - read once, before any implementation code exists: the
  initial setup procedure, and the compatibility notes for stack combinations
  that need extra wiring. Nothing in the build loop reads it afterwards
- `docs/workflow.md` - the ten-step human guide from setup through to deployment
- `docs/adopting-an-existing-project.md` - the setup half of that guide for a
  project that already has code; it rejoins `docs/workflow.md` at Step 4

Read each of these once per session, not once per step. If one is already in
context, use that copy rather than reading it again.

## Specs are contracts

Specs live in `docs/features/` (user-facing) and `docs/specs/` (components,
pages, layouts). What each `**Status:**` value allows - `Draft`, `Ready`,
`Complete` - and the template each kind of spec follows are in
`docs/project-brief.md` → Spec conventions; which spec owns a value its
components share is in → Features and components.

**The `**Status:**` line is the work queue.** `/feature` builds the next `Ready`
spec, `/status` reports the queue by status, and `/complete` writes `Complete`
back when the work is done.

**Agents read specs and never edit them.** Two edits are the only exceptions in
the whole workflow:

- `/complete` sets a finished spec's `**Status:**` and `**Last updated:**` lines,
  and nothing else. That is `Complete` for a feature or a fix, and `Complete`
  back to `Ready` for a rollback.
- `/feature` may draft a brand-new spec, on explicit approval, and only as
  `**Status:** Draft`. **Moving a spec from `Draft` to `Ready` is a human act** -
  that is the signal the contract is settled, and an agent that grants itself
  that signal has removed the gate the project is built around.

**Promotion and restoration are different moves.** `Draft` -> `Ready` is a
promotion: a contract nobody has agreed to yet becomes one the loop may build,
and only a human makes that call. `Complete` -> `Ready` on a rollback is a
restoration: the human settled that contract once already, the spec still
describes what the project wants, and only the implementation is being withdrawn.
`/complete` makes the second move and never the first.

### The work order's own status

A work order in `context/current-feature.md` carries a `**Work status:**` line.
**It is a different field, in a different file, from the spec `**Status:**` line
above**, and confusing the two writes to a human-owned contract. These are its
only values:

| Work status | Meaning | Who writes it |
|-------------|---------|---------------|
| `not started` | Work order written, no code yet | `/feature`, `/fix`, `/rollback` |
| `in progress` | Building; any earlier proof is now stale | `/implement` |
| `verified` | Every done-when proven with evidence | `/implement`, `/check`, `/complete` |
| `verification failed` | `/check` disproved a done-when | `/check` |
| `verification incomplete` | `/check` could not prove one either way | `/check` |

> [!IMPORTANT]
> *Checked build steps are not proof. Every step can be ticked on a work order
> whose last `/check` failed, so this line is the only record of whether the work
> was ever proven. `/status` reports it, and `/complete` runs its own final pass
> regardless of what it says.*

### Which file wins

| Question | Authority |
|----------|-----------|
| Stack, conventions, browser targets, accessibility, agent rules | `docs/project-brief.md` |
| What one thing must do, and when it is done | its spec in `docs/features/` or `docs/specs/` |
| What is built, in progress, or not started | the specs' `**Status:**` lines |
| Build steps for the one thing in flight | `context/current-feature.md` (disposable) |
| Open review findings | `context/findings.md` (generated) |

Higher rows win. A generated file never overrides a human-owned contract.

## Agent configuration

**Nothing tool-specific is committed to this repository.** `.agents/<agent>/` is
the only committed home for agent config. Every tool's hardwired location is a
gitignored pointer created during setup. Before adding or un-ignoring any path,
ask whether it names a specific tool. If it does, it belongs in `.agents/` with a
pointer, not in the repo.

That rule is the whole premise: committing one tool's config forces that tool on
everyone who clones the template.

The pointer table, the setup commands, adding, switching or removing an agent,
and troubleshooting are all in `docs/agent-setup.md`, which you read only when
changing that wiring. **Never delete `.agents/`**: it holds the only real copy
of the skills tree, and of any agent config, and every pointer dangles without it.

## Workflow

Build one feature, fix, or rollback at a time, behind review gates. This is the
automated form of `docs/workflow.md` Steps 4-10, not a second workflow: the spec
`**Status:**` line is still the queue, and Steps 4-6 (writing feature specs,
component specs, and design tokens) stay human work. Each skill is plain markdown
any capable agent can read and follow, at `.agents/skills/<skill>/SKILL.md` -
the one tracked copy, so that is where shared workflow behaviour is changed. How
each tool finds and runs them is in `docs/agent-setup.md`.

### The build loop

These run in order, once per spec. Each stops at a review gate rather than
running on into the next. Every skill reads its state from disk, so each can
start in a fresh session; after `/complete`, suggest the user clear the
conversation before the next `/feature`.

| Skill | What it does |
|-------|--------------|
| `preview` | Read-only preview of a spec before you build it: what it involves, what it depends on, what would block it |
| `feature` | Turns the next `Ready` spec into a work order at `context/current-feature.md`, with small reviewable build steps |
| `implement` | Builds those steps one at a time - diff, plain-English explanation, verification, approval - on the branch you are already on |
| `check` | Proves each "done when" against the running app and captures the evidence |
| `audit` | Reviews the code against the project's standards; records findings in `context/findings.md`, where open or fixed P0/P1 findings block `complete` |
| `try` | Read-only manual walkthrough: what to start, where to click, what to expect |
| `complete` | Final safety pass, writes `Complete` back to the spec, archives the work order under `context/history/`, makes one work commit |
| `status` | Read-only: the spec queue, what is in flight, drift warnings, and the exact next action |

Outside the loop:

| Skill | What it does |
|-------|--------------|
| `discovery` | Optional guided interview that fills in `docs/project-brief.md` and drafts the first feature specs - the conversational form of Steps 2 and 4 |
| `survey` | Reads a codebase that already exists and drafts `docs/project-brief.md` from the evidence in it, marking what is proven, what is inferred, and what cannot be determined from code. `/discovery`'s counterpart for a project that was not started here |
| `fix` | Documents an ad-hoc bug or change with no spec of its own, then runs it through the same loop |
| `rollback` | Plans a safe reversal of a completed feature from its archive and commit, then hands the work order to `implement` |
| `debug` | Reproduces and isolates a failure without editing code, then hands the evidence to `fix` or `implement` |
| `prototype` | Pre-build static mockups to lock the look before any spec is built |
| `tests` | Adds or normalizes unit testing and turns on the test gate |
| `ci` | Sets up one project-specific `Verify` command and matching automatic GitHub checks |
| `release` | Render or Vercel deployment readiness: local config, env review, smoke-test planning |

`tests`, `ci`, and `release` detect the project's real stack, so they handle
runtimes the Stack table in `docs/project-brief.md` does not list - Python, Go,
Rust, and others. That is deliberate headroom, not a gap in the table: the
scaffold's specs, design tokens, and platform guide are written for front-end
web work, while the skills that touch build, test, and deploy tooling are
written to detect whatever is actually there. Do not narrow them to match the
table, and do not widen the table to match them.

One more sits above the loop rather than inside it. `autopilot` runs a single
bounded pass - work order, build steps, verification, gates, checkpoint commits -
without pausing at each review point, then stops with a review packet for a
human. It never runs `complete`. It exists for the times you want the agent to
carry a settled spec the whole way, and it is opt-in only: the step-at-a-time
path above is the default, and the review between each step is the point of a
spec-first loop.

In Claude Code, invoke these as slash commands (`/feature`, `/implement`, and so
on). In Codex, invoke them as skills (`$feature`, `$implement`). In Cursor,
GitHub Copilot, OpenCode, and any other tool with no dedicated syntax for these,
name the skill and ask the agent to follow its `SKILL.md` - the gates are in the
file, so a skill followed manually behaves the same as one invoked.

**Two rules hold however a skill is invoked:** no skill promotes a spec from
`Draft` to `Ready` (see "Specs are contracts" above), and no skill creates,
switches, merges, or deletes a branch - the loop commits to whatever branch is
checked out.

Deployment is also explicit. `/release` can prepare local Render or Vercel config
and run readiness checks, but it must stop before deploy, remote service changes,
push, or publish unless the user gives a separate yes in the current chat.

## Output conventions

How a skill reports back. Every `SKILL.md` ends by pointing here, so this is the
one place to change the house style for all of them at once.

- **Concise and scannable, not a wall of text.** Lead with the answer or the
  state, then the reasoning, and only as far as it is needed.
- **Lists for enumerations, tables for matrices.** A set of things is a list. A
  set of things each carrying the same few attributes is a table. Dense
  paragraphs are neither.
- **Name files by path** - `docs/specs/components/button.spec.md`, not "the
  button spec" - so the reader can open what is being discussed.
- **Separate what is proven from what is assumed**, and never report the second
  as the first. "Not checked" is a legitimate thing to say.
- **Expand every acronym and initialism on first use** in a report, then use the
  short form freely. The documentation set does this throughout - Web Content
  Accessibility Guidelines (WCAG), Content Security Policy (CSP) - and skill
  output is read by the same people.
- **End with the next action** where the skill has one, phrased as the command to
  run.

A skill's own file may add to this. Where the two differ, the skill's file wins
for that skill.

## Automatic verification

Automatic GitHub checks are a separate, explicit setup, and `/ci` owns their
rules. A project with no automatic GitHub checks still works: the loop falls
back to the documented build and test commands.

## Commands

Fill these in for your stack, from the selections in `docs/project-brief.md`, as
part of `docs/workflow.md` Step 3. Delete any row that does not apply, and do
not invent a command to fill a gap.

- Dev server: `<command>` (http://localhost:<port>)
- Build: `<command>`
- Production server: `<command>`
- Lint: `<command>`
- Test: `<command>` - written by `/tests`; leave absent until then
- Verify: `<command>` - written by `/ci`; leave absent until then

Those last two labels are read, not just written. Keep them exactly as spelled:
`/implement` looks for a `Test` command to decide whether the testing gate is on,
and `/implement`, `/complete` and `/autopilot` all run the `Verify` command when
one is documented.

Testing is opt-in. If this project does not already have a unit test runner, run
`/tests` or `$tests` to add one and fill in the `Test` row. The presence of a
real test command there is the single switch that turns the testing gate on.

Automatic GitHub checks are a separate opt-in. Run `/ci` or `$ci` to define one
`Verify` command and the matching workflow.
