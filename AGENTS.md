# AGENTS.md

Instructions for "Ai" coding agents working in this project.

`AGENTS.md` is a cross-tool convention that most coding agents read directly, so
this is the entry point for any of them. Claude Code reads `CLAUDE.md`, which
imports this file, so there is a single source of truth either way.

Most agents read this file natively - Codex, Cursor, GitHub Copilot, Gemini CLI,
Jules, Aider, Zed, Windsurf, Devin and OpenCode among them - and need no config of
their own, so they work as soon as they open the project. Claude Code is the
notable holdout, and the single ready-made config under `.agents/` is its. For any
other agent that expects a config file of its own, see "Adding a new agent" below.

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
> files this one already has - and because `CLAUDE.md` here is a symlink, that
> write follows the link and destroys the tracked `.agents/claude/CLAUDE.md`.
> Nor do they reliably refuse: `create-vite` offers "Remove existing files and
> continue" as one of three choices. Step 3 writes the framework config directly
> instead.*

For a codebase that already exists, the order is reversed - the code is there
first and the scaffold merges in on top - and the early steps differ: see
`docs/adopting-an-existing-project.md`.

The workflow is defined by the local skills and context files below.

## Read these for full context

- `context/sessions.md` - where the work stands. With `decisions.md` below, one of
  the only two files that survive a `/compact` or `/clear`. One **Where things
  stand** block and nothing else: queue, branches, what is open, the next action.
  Gitignored and personal to you, so it will not exist in a fresh clone; create it
  on first use. **See "Keep the state file current" below**
- `context/decisions.md` - why a choice was made and what was rejected, newest
  first. Append-only, and one entry per decision rather than per session. The
  only thing here that git history cannot reconstruct. Gitignored and personal
  to you as well; create it on first use
- `context/current-feature.md` - the work order for the one feature, fix, or
  rollback being built right now, or the stub when nothing is in flight
- `context/findings.md` - the review ledger `/audit` writes and `/complete` clears
- `context/history/` - archived work orders: what was built, in what order, and why

## Keep the state file current

**`context/sessions.md` and `context/decisions.md` are the only things that
survive a context reset.** A `/compact` or `/clear` discards the conversation,
and nothing warns you - there is no error, just a later session that has to
rediscover what was known.

**Read both files before acting when starting cold**, and after any `/clear` or
`/compact`. Neither is loaded automatically - `CLAUDE.md` imports `AGENTS.md`,
`docs/project-brief.md`, `context/current-feature.md` and `context/findings.md`,
but not these - so they have to be opened deliberately. `sessions.md` is short by
design; read it every time. Read `decisions.md` when a choice looks settled and
you are about to revisit it.

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

The project's own documentation set is authoritative for everything else:

- `docs/project-brief.md` - **the single source of truth.** Read it in full
  before doing anything else: stack selection, browser targets, accessibility
  standard, coding conventions, and agent behaviour rules
- `docs/modern-platform-guide.md` - read before writing any HTML, CSS, or JS
- `docs/design-tokens.md` - read before writing any CSS
- `docs/security.md` - read before generating HTML or deployment config
- `docs/service-worker.md` and `docs/storybook.md` - optional features, only when
  marked active in `docs/project-brief.md`
- `docs/setup.md` - read once, before any implementation code exists: the
  initial setup procedure, and the compatibility notes for stack combinations
  that need extra wiring. Nothing in the build loop reads it afterwards
- `docs/workflow.md` - the ten-step human guide from setup through to deployment
- `docs/adopting-an-existing-project.md` - the setup half of that guide for a
  project that already has code; it rejoins `docs/workflow.md` at Step 4

## Specs are contracts

Specs live in `docs/features/` (user-facing) and `docs/specs/` (components,
pages, layouts). Each carries a status line:

| Status | Meaning | Who acts |
|--------|---------|----------|
| `Draft` | Incomplete - do not implement | Human only |
| `Ready` | Complete - proceed with implementation | Human + agent |
| `Complete` | Implemented and tested | Human only |

**The `**Status:**` line is the work queue.** `/feature` builds the next `Ready`
spec, `/status` reports the queue by status, and `/complete` writes `Complete`
back when the work is done.

**Agents read specs and never edit them.** Do not implement a `Draft` spec; stop
and ask. Do not re-implement or overwrite a `Complete` spec; the human resets it
to `Ready` first.

New specs follow the template for their kind — `docs/features/_feature-template.md`
for something a user can do, `docs/specs/_component-template.spec.md` for a
component, page, or layout. A feature spec's Implementation notes table is
authoritative for any value its components must agree on; component specs
reference those values and never restate them.

Two edits are the only exceptions in the whole workflow:

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

### Structure

Any agent that needs a config of its own gets a directory:

```bash
.agents/
├── claude/     # Claude Code
├── skills/     # the shared workflow skills, read by any capable agent
└── ...         # add any agent that has a config file convention
```

Every file inside an agent directory does two things only:

1. Points the agent at the context files listed above, starting with
   `docs/project-brief.md`
2. Adds anything genuinely specific to that agent (custom commands, model
   settings)

Nothing else belongs in them. `.agents/skills/` is shared, not tool-specific:
every agent works from the same tree, whether it discovers it through a pointer
or is sent there by `AGENTS.md`.

### How the pointers are wired

An agent that reads `AGENTS.md` needs no pointer at all - this file is already at
the project root. Claude Code looks for project config at `./CLAUDE.md` or
`./.claude/CLAUDE.md`, with no setting that repoints it at `.agents/` - and
`.claude/` is gitignored here, so anything put there stays personal to one
machine. It therefore needs a pointer at each location it expects.

| Location | Points to |
|----------|-----------|
| `CLAUDE.md` | `.agents/claude/CLAUDE.md` |
| `.claude/skills` | `../.agents/skills` |

**macOS and Linux** - symlink:

```bash
ln -s .agents/claude/CLAUDE.md CLAUDE.md
mkdir -p .claude && ln -s ../.agents/skills .claude/skills
```

> [!IMPORTANT]
> *The `.claude/skills` pointer needs both the `mkdir -p` and the leading `../`,
> and each guards a different failure. `.claude/` is gitignored, so it does not
> exist in a fresh clone and `ln -s` will not create it. And a symlink's target is
> resolved relative to the link's own directory, so `.agents/…` without the `../`
> creates a link pointing at `.claude/.agents/…` - which `ln` reports as success
> and `ls -l` displays as if it were correct. `ls .claude/skills` is what proves
> it resolves. Only the root-level `CLAUDE.md` line needs neither guard.*

**Windows** - `ln -s` needs Developer Mode or an elevated terminal. If neither is
available, copy instead, then keep the copies in sync by hand:

```bat
copy .agents\claude\CLAUDE.md CLAUDE.md
if not exist .claude mkdir .claude
xcopy /E /I .agents\skills .claude\skills
```

**Every pointer is gitignored**, so a Windows copy and a macOS symlink never
collide in git, and no one inherits another developer's agent choice. Symlink
where you can: a copy drifts from its source, which is how the shared skills tree
ended up symlinked rather than duplicated per tool.

### Switching between agents

There is no project-level switch. Every agent reads the same
`docs/project-brief.md` and the same specs, so they always share one
understanding of the project. For most agents there is nothing to do but open the
project; for one that needs a pointer, create it first.

### Adding a new agent

First check whether the agent reads `AGENTS.md` - most now do, and those need
none of the steps below. For one that insists on a config file of its own:

1. Create `.agents/<agent-name>/`
2. Create the agent's required config file inside it
3. In that file, tell the agent to read `docs/project-brief.md` first. See
   `.agents/claude/CLAUDE.md` for a working example
4. Add any agent-specific config below that instruction
5. Create the pointer at the location the agent expects, per the table above
6. Add that pointer path to `.gitignore`, **anchored with a leading slash** -
   `/<CONFIG>.md`, not `<CONFIG>.md`

All project conventions are already in `docs/project-brief.md`, so there is
nothing else to duplicate.

> [!IMPORTANT]
> **A `.gitignore` pattern containing no slash matches at every depth, not just
> the root.** *Unanchored, the pointer's filename ignores the pointer **and** the
> real config at `.agents/<agent-name>/` - the only tracked copy. This project's
> own `.gitignore` carried an unanchored `CLAUDE.md` until it was fixed; nothing
> was lost only because that file was already tracked, and an ignore rule cannot
> untrack. A config you add now has no such protection. Nothing reports it:
> `git add -A` skips the file in silence, `git status` cannot list what it never
> staged, and the pointer resolves perfectly for whoever created it. The failure
> surfaces only in someone else's clone, as a dangling link. Anchor the pattern,
> then prove it with `git check-ignore -v .agents/<agent-name>/<file>` - it
> should print nothing.*

### Removing an agent

Delete the pointer and the directory:

```bash
rm CLAUDE.md
rm -rf .agents/claude
```

Nothing else changes.

### Troubleshooting

**The agent isn't reading `docs/project-brief.md`.** Check the pointer exists
where the agent expects it. `ls -la` should show an entry like
`CLAUDE.md -> .agents/claude/CLAUDE.md`. If it's missing, recreate it.

**The agent reads its config but ignores the project brief.** Some agents need an
explicit instruction to read external files; a path alone isn't always enough.
Check the agent's documentation and copy how the existing configs in `.agents/`
handle it.

**The agent produces output that contradicts the project brief.** The brief is
probably incomplete or ambiguous in that area. Clarify the relevant section and
re-run. Don't hand-edit the agent's output to paper over a brief that needs
fixing.

**The agent implemented the wrong thing.** Check the spec it built against. The
spec is the contract, so a wrong result usually means a spec that was promoted to
`Ready` before it was settled.

## Workflow

Build one feature, fix, or rollback at a time, behind review gates. This is the
automated form of `docs/workflow.md` Steps 4-10, not a second workflow: the spec
`**Status:**` line is still the queue, and Steps 4-6 (writing feature specs,
component specs, and design tokens) stay human work. Each skill is plain markdown
any capable agent can read and follow. Where each tool finds them:

- Claude Code: discovers `.claude/skills/<skill>/SKILL.md` on its own, and
  invokes them as `/feature`, `/implement` and so on
- Every other agent: `AGENTS.md` names the path,
  `.agents/skills/<skill>/SKILL.md`, and you name the skill in your prompt

> [!IMPORTANT]
> *Only Claude Code loads this tree by itself. For every other agent the path is
> written down where the agent will read it, and **naming the skill is what runs
> it** - there is no auto-discovery to rely on. A tool that has its own skills
> convention will not find these at `.agents/skills/`. The review gates are in the
> `SKILL.md`, so a skill followed this way behaves the same as one invoked.*

Unused pointers can be removed, but **`.agents/` is never one of them.** Those
pointer paths are gitignored; `.agents/` holds the only real copies of
both the agent config and the skills tree. Deleting it leaves every pointer
dangling - no config and no skills, with `ls -l` still showing links that look
healthy.

A project using no Claude Code can delete the `CLAUDE.md` pointer, `.claude/` and
`.agents/claude/`, but must keep `AGENTS.md` and `.agents/skills/` - the first is
what every other tool reads, the second is where the skills actually live. Do not
duplicate the skills under `.cursor/`, `.opencode/skills/` or any other tool's
tree; each of those discovers a compatible tree already.

When changing shared workflow behavior, edit
`.agents/skills/<skill>/SKILL.md` - the one tracked copy. Every tool reaches it
through its own pointer, so there is nothing to keep in sync and no second tree
to create.

### The build loop

These run in order, once per spec. Each stops at a review gate rather than
running on into the next.

| Skill | What it does |
|-------|--------------|
| `brief` | Read-only briefing on a spec before you build it: what it involves, what it depends on, what would block it |
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

**Two rules hold however a skill is invoked.** No skill moves a spec from
`Draft` to `Ready` - that is the human's signal that the contract is settled, and
a rollback's `Complete` -> `Ready` is a restoration rather than that move (see
"Specs are contracts" above). And no skill creates, switches, merges, or deletes
a branch; the loop commits to whatever branch is checked out, matching the
"commit your work" checkpoints in `docs/workflow.md`.

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

Automatic GitHub checks are a separate explicit setup. Running `/ci` inspects the
real project and defines one `Verify` command from checks that already exist.
Use this order when available: typecheck, tests, then build. Never invent a test
runner or another check just to fill the command.

For JavaScript and TypeScript projects, prefer a package script such as `verify`
and use the detected package manager. For other stacks, use the native task
runner or exact combined command. Record the exact command under Commands below.

The optional `.github/workflows/verify.yml` must run that same command for pull
requests and pushes to the default branch. Preserve existing workflows, use the
project's real runtime and install command, and grant only `contents: read` by
default. This setup does not add local git hooks, coverage, browser tests,
security scans, or version matrices. Those remain later project choices.

GitHub branch protection or a ruleset can require the check after the repository
is pushed, but that is a separate remote setting. A project with no automatic
GitHub checks still works; the loop falls back to the documented build and test
commands.

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
