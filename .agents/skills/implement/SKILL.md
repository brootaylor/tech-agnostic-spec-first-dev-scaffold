---
name: implement
description: "Build the feature, fix, or rollback in context/current-feature.md one small reviewed step at a time - diff, plain-English explanation, verification, and an optional commit checkpoint per step. A Type: Rollback work order uses a guarded reverse patch. The work-level commit is /complete's job. Use when the user runs /implement, or asks to build, implement, or start the current feature, fix, or rollback once its work order is ready."
---

# implement - build the current work order, one reviewed step at a time

Where this sits in the workflow:

    /feature, /fix, or /rollback  ->  [implement]  ->  /complete  ->  next
    (the work order)                   (build it,       (commit + log)
                                        reviewed)

**Two documents, and this skill must never confuse them.** The **work order** is
`context/current-feature.md` - generated, disposable, and where the build steps
live. The **spec** is the human-owned contract in `docs/features/` or
`docs/specs/`, named on the work order's `Spec:` line. Throughout this file,
"the spec" always means that second file, and it is never where progress gets
recorded.

`/feature`, `/fix`, or `/rollback` wrote the work order to
`context/current-feature.md` and stopped. This skill turns that work order into code,
without vibe coding: small steps, a visible diff plus a plain-English explanation
for each, testing, and iteration until it works. It commits on the branch you are
already on - this loop does not create or switch branches. The work-level commit
and the logging are `/complete`'s job.

## Before you start

Read `context/current-feature.md`. **Then read the spec named on its
`Spec:` line**, in `docs/features/` or `docs/specs/`. The work order carries the
sequencing; the spec carries the contract - interface, behaviour, states,
accessibility requirements, and test cases. Build against both, and where they
disagree, the spec wins and the work order is wrong: stop and say so rather than
building the discrepancy.

A `Type: Fix` work order legitimately has no `Spec:` line. A feature one that
lacks it is drift; report it before building.

**Never edit a spec file.** Not to fix a typo, not to record progress, not to
adjust a criterion you found inconvenient. If the spec is wrong or ambiguous,
stop and ask the human to change it. Only `/complete` writes to a spec, and only
its `**Status:**` and `**Last updated:**` lines.

If it has no real work order (still the reset stub),
stop and tell the user to run `/feature` (for a planned feature), `/fix` (for an
ad-hoc bug or change), or `/rollback` (for a completed feature reversal) first.
Pull the conventions, active stack, browser targets, and accessibility standard
from `docs/project-brief.md` so the code matches them.

If the work order's Design reference points at `prototypes/*.html`, those
mockups are the visual target - build components to match them, and treat
`prototypes/theme.css` as the token source. The work order's first step ports it
into `docs/design-tokens.md` **and** the app's global stylesheet; hold that step
to its "done when" exactly, because `/complete` refuses to delete `prototypes/`
until both halves match.

**Resuming?** If the **work order** already has some build steps checked off
(`- [x]`), this feature was started earlier and interrupted (often a cleared
context). Read the boxes in `context/current-feature.md` and nowhere else: a spec
carries `- [x]` boxes of its own, in its test cases and its `Draft -> Ready`
checklist, and reading those as build progress will skip work that was never
done. Check `git status` and the log to see what is committed and what is still
in the working tree, then continue from the **first unchecked step** instead of
starting over. No separate save/load is needed - the ticked steps are on disk.

## Step 1 - record where this work starts

Before the first product edit, write the current `HEAD` to the work order's
`**Base commit:**` line. `/audit` uses it to find what this work changed, and
`/complete` uses it to scope the work commit. If the project isn't a git repo
yet, say so and ask the user to run `git init` first; the loop needs commit
history to work with. On resume, the line is already filled - leave it alone.

This loop does not create, switch, merge, or delete branches. It commits to
whatever branch is checked out, matching the "commit your work" checkpoints in
`docs/workflow.md`. If the user wants feature branches, that is their call to
make before running `/implement`.

### Type: Rollback safeguard

For a `Type: Rollback` work order, the first build step is the guarded reverse
patch in `.agents/skills/rollback/reference/rollback-procedure.md`. Read it in
full before that step and follow it exactly - its SHA checks, protected-path
exclusions and conflict handling are not optional, and nothing else in this file
replaces them.

## Step 2 - build one step, review, iterate, checkpoint

Before the first product edit in this run, set `**Work status:**` to
`in progress` in `context/current-feature.md`. This invalidates any older
verification state. Do the same whenever implementation resumes after a passing
check and changes product code again.

That is the work order's own field. It is **not** the spec's `**Status:**` line:
the spec stays `Ready` throughout, only `/complete` ever writes to it, and
`Complete` is the only value it may write. A spec set to `in progress` corrupts
the queue `/status` and `/feature` read.

Work through the work order's build steps in order, one at a time, using the
review and approval gate below after every step.

1. Implement just that step: the smallest change that satisfies its "done when."
2. Show the **diff**, not whole files.
3. **Explain it, and prove it.** Give a short summary: what the step delivered,
   one line per changed file on what it does and why, then confirm the step's
   "done when" is met with evidence (build output, a screenshot, a passing
   assertion). This summary is the comprehension gate, so keep it concrete, not
   ceremonial. Include a short **How to try it** note when the step has a manual
   path: the command, URL, click, endpoint, or output the user can check.
4. **Verify the step.**
   - **Automated gate.** Run the `Verify` command from `AGENTS.md` when one is
     declared - it wraps only checks the project really has, so never invent
     one to satisfy it. Without it, run the documented build command, plus the
     test command when one is declared. A focused test may run first for faster
     feedback; the gate still runs last.
   - **Testing switch.** A real `Test` command under Commands in `AGENTS.md`
     turns testing on. While it is on, a step that adds logic ships a passing
     test in the same diff, placed next to the source it covers, and the suite
     is green before approval; logic the spec did not foresee gets a test too,
     or a note saying why not. While it is off, say so plainly rather than
     claiming the step is tested. UI and integration-only steps ride on
     screenshot plus build evidence either way.
   - **Never install a runner mid-step** - point at `/tests`, even when the
     brief marks a runner `[active]` (that records intent; the missing command
     means setup never ran). The one exception is a work order that is itself
     the testing setup, such as `/fix "add unit testing"`.
   - **Runtime behaviour.** When a "done when" needs the running app - a click,
     download, request, command, background job, or multi-screen flow - run
     `/check` rather than eyeballing it.
5. **Iterate until it works.** If it fails or the user wants changes, revise the
   step (re-prompt or hand-edit the code), show the updated diff, and re-test.
   Repeat until the user approves.
6. **Mark it done, then offer a checkpoint.** Once the gate passes, tick the
   step (`- [x]`) in `context/current-feature.md` so progress survives a context
   clear. If it repaired a finding in `context/findings.md`, set that finding
   `fixed` and note the repair in its **Resolution** line. The work order's
   `Fixes:` line names the finding - `/fix F-03` stamps it there so the link
   survives a context clear; match against the ledger only when the line is
   absent, and say you did. Never set `closed`: `/audit` re-reviews every
   repair, because a fix can introduce a worse defect than the one it removed.

   Then offer a short choice. Use the current tool's short user-input prompt,
   except straight after a long block of reading (such as a walk-through), where
   plain text keeps the prompt from covering it:
   - **Continue** (default) - roll into the next step without committing.
   - **Commit checkpoint** - commit just this step with a conventional message,
     a cheap rollback point. Optional: `/complete` makes the work-level commit.
   - **Walk me through it** - a deeper, line-level explanation of the new or
     changed code (why this approach, what each part does, any gotchas), then
     ask this again.
   - **Stop here** - pause. Say where things stand: the work is intact on disk,
     and running `/implement` again resumes from the first unchecked step.

### Where the code goes, and the co-located spec copy

Generated code follows the project's convention, one directory per contract:

| Spec | Code |
|------|------|
| `docs/specs/components/<name>.spec.md` | `src/components/<Name>/` |
| `docs/specs/pages/<name>.spec.md`      | `src/pages/<Name>/`      |
| `docs/specs/layouts/<name>.spec.md`    | `src/layouts/<Name>/`    |

When a step creates one of those directories, drop a copy of the source spec in
it as `<Name>.spec.md`, so anyone opening the folder has the contract next to the
code. The copy is a convenience only: `docs/specs/` stays the source of truth,
`/complete` writes the status back there and never to the copy, and if the two
ever differ the `docs/specs/` version wins. Refresh the copy if the source spec
changes while the work is in flight.

File extensions follow the active Language and Styles selections in
`docs/project-brief.md`. Never add a prop, event, or behaviour the spec does not
name.

Never batch the whole thing into one diff. If a step's diff is too big to read,
split it. The documented `Verify` command, or the fallback build and tests, must
pass before any commit.

## Step 3 - hand off to /complete

Before handing off, check `context/findings.md`. A P0 or P1 finding
still `open` or `fixed` there means `/complete` will refuse to finish, so close
the loop now:

- Repair each `open` P0 or P1 as an extra reviewed step. First append it to the
  work order's build steps in `current-feature.md` (`- [ ] Repair F-03 - <title>`) so
  the repair is on the record and survives a context clear, then run the same
  loop as Step 2: smallest change, diff, plain-English explanation, evidence.
  Check the step off and mark the finding `fixed` together.
- Then run `/audit` so the repairs are re-reviewed and can move to `closed`.
  A repair this skill made never closes itself.
- If the user decides a finding should not be fixed, only they can set
  `accepted` (reason recorded). A finding that looks wrong goes back to
  `/audit` to invalidate with recorded evidence; this skill never sets
  `accepted` or `invalid`.

When every step is built and `Verify`, or the fallback build and tests, passes
(committed as checkpoints or not), stop with a compact review packet:

Set the work order's `**Work status:**` to `verified` immediately before that
packet. This is durable workflow evidence for `/status`. Do not set it when a
required command, observable done-when, or configured gate failed or could not
run. Again, this is the work order's field in `context/current-feature.md`, never
the spec's `**Status:**` line.

- what changed, grouped by file or area
- checks run, with the exact command or proof used
- how to try it manually, or a pointer to `/try`
- ledger state: any findings still `open` or `fixed`, by ID
- known risks, skipped checks, or follow-up notes
- next action, usually `/complete`

Then tell the user `/complete` makes the one work-level commit and logs the work
(writes the spec's status back, archives the work order, resets the ledger).

## Rules

- One small step per diff, reviewed and approved before the next one starts.
- Explain every change in plain English. Understanding the code is the point.
- Iterate until each step works; never commit code the user hasn't approved.
- Follow the conventions in `docs/project-brief.md`, the platform guidance in
  `docs/modern-platform-guide.md`, and the tokens in `docs/design-tokens.md`.
- Read `docs/security.md` before any step generates `index.html`, `_headers`,
  `vercel.json`, `render.yaml`, `next.config.js`, or a middleware file. Its
  header values and Content Security Policy (CSP) directives are the source of
  truth; do not invent or vary them here. A step that loads a resource from an
  external origin also adds it to that file's CSP exceptions table.
- Build only what the spec says. If the spec is wrong or thin, stop and ask the
  human to change it — do not improvise, and do not edit the spec yourself.
- Never create, switch, merge, or delete a branch. Per-step commits are optional
  checkpoints; the work-level commit is `/complete`'s job, and any push needs the
  user's explicit yes.
- For Type: Rollback, reverse only the approved product diff and preserve all
  protected loop paths.

## Formatting

Output follows `AGENTS.md` - Output conventions.
