---
name: implement
description: "Build the feature, fix, or rollback described by the work order in context/current-feature.md, one small reviewable step at a time. Implements each step, shows the diff and explains it in plain English, tests, and iterates until it works. A Type: Rollback work order uses a guarded reverse patch that preserves the loop's own history. After each approved step it offers an optional commit checkpoint; the work-level commit and logging are /complete's job. Use when the user runs /implement, or asks to build, implement, or start the current feature, fix, or rollback once its work order is ready."
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

If the work order's Design reference points at `prototypes/*.html`, those mockups
are the visual target - build components to match them, and treat
`prototypes/theme.css` as the token source. The work order's first step ports it
into `docs/design-tokens.md` **and** the app's global stylesheet before any
component is built. Both halves are required: `/complete` deletes `prototypes/`
afterwards, and a theme that only reached the stylesheet leaves
`docs/design-tokens.md` - the file every agent is told to read before writing
CSS - an empty template for the rest of the project.

**Resuming?** If the **work order** already has some build steps checked off
(`- [x]`), this feature was started earlier and interrupted (often a cleared
context). Read the boxes in `context/current-feature.md` and nowhere else: a spec
carries `- [x]` boxes of its own, in its test cases and its `Draft -> Ready`
checklist, and reading those as build progress will skip work that was never
done. The work order and
its ticked steps are files, so pick up where it left off: read which steps are done,
check `git status` and the log to see what is committed and what is still
in the working tree, then continue from the **first unchecked step** instead of
starting over. No separate save/load is needed - the project instructions load
`current-feature.md` every session.

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

For a rollback work order, do not hand-delete the old feature and do not run a whole
commit `git revert`. Completed feature commits also contain loop history and
plan bookkeeping, while `current-feature.md` now contains the active rollback
work order. Reversing the whole commit would damage that state.

Before the first rollback build step:

1. Read the approved work order's `Target commit` and `Target parent` fields. Stop
   unless both values match `^[0-9a-f]{40}$`. Do not accept abbreviated,
   uppercase, or otherwise malformed SHAs.
2. Resolve the archive's introducing commit and verify it has exactly one parent.
   Stop on a merge target. Resolve that single parent to a full SHA value.
   Confirm the resolved commit exactly equals `Target commit` and the resolved
   parent exactly equals `Target parent`. Stop on any mismatch.
3. Confirm the resolved target is an ancestor of `HEAD` and the only dirty path
   before applying the patch is the approved rollback work order. Stop on drift.
4. Preview the resolved target's product diff while excluding `.agents/**`,
   `.claude/**`, `context/**`, `docs/**`, `AGENTS.md`, `CLAUDE.md`, and
   `prototypes/**`. Confirm the preview is non-empty and matches the Product
   paths in the work order.
5. Apply that resolved product diff in reverse with three-way conflict detection
   and stage it. Use only the resolved full SHA values before running:

       git diff --binary <target-parent> <target-commit> -- . \
         ':(exclude).agents/**' \
         ':(exclude).claude/**' ':(exclude)context/**' \
         ':(exclude)docs/**' \
         ':(exclude)AGENTS.md' ':(exclude)CLAUDE.md' \
         ':(exclude)prototypes/**' |
         git apply --reverse --3way --index

   Never omit the protected pathspec exclusions for convenience.
6. Show both `git diff --cached` and `git status`. Confirm no protected path is
   staged or modified before presenting the step for review.

> [!IMPORTANT]
> A conflicting reverse apply does not leave the tree untouched. `--3way` writes
> conflict markers into the working tree files and leaves the index at unmerged
> stages, and git reports it as `Applied patch to '<file>' with conflicts.` -
> the word *Applied*, on a command that exited non-zero. Leaving it for a later
> skill to notice is not a plan: `/complete` stages everything for the work
> commit, so anything still conflicted here is committed as product code with
> its markers intact. Its safety pass rejects an unmerged index for exactly this
> reason, but that is a backstop, not the fix - resolve it or report it here.

If the reverse patch conflicts, say so explicitly and report three things: the
exact paths, the later commit that appears involved, and the fact that the
working tree now holds conflict markers. Confirm the damage with `git status`
(conflicted paths show as `UU`) and `git ls-files -u`. Do not auto-resolve,
discard, stash, reset, or switch to a broad checkout - those are the user's call
precisely because the tree is already dirty. Ask whether to resolve only the
conflict allowed by the approved work order or abandon the attempt, and if the
answer is to abandon, ask before running the cleanup rather than choosing one. A
cascade into another completed feature needs a new rollback plan.

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
4. **Verify the step.** If `AGENTS.md` declares a `Verify` command, run that exact
   command as the automated gate. It is only an umbrella for checks the project
   actually has, so do not invent tests or other checks to satisfy it. If no
   `Verify` command exists, run the documented build command, and the test
   command when the project declares one. **A real `test` command under Commands
   in `AGENTS.md` is the switch**, and `/tests` is what adds one: while one is
   declared, a step that adds logic must ship a passing test in the same diff and
   the suite must be green before the step is approved; while none is declared,
   say so plainly rather than claiming the step is tested. The Unit testing
   selection in `docs/project-brief.md` records the human's intent, not the
   gate - a runner marked `[active]` there with no command in `AGENTS.md` means
   the setup has not been run, so point at `/tests` rather than installing one
   mid-step. UI and integration-only steps ride on
   screenshot plus build evidence. Run a focused test separately when it gives
   faster feedback, then use `Verify` as the final automated gate. Create focused
   test files next to the source they cover. Never install a runner mid-step
   unless the current work order is explicitly the unit-testing setup itself (for
   example `/fix "add unit testing"`); point at `/tests` instead. If a step
   surfaces non-trivial logic the spec did not foresee, add a focused test then,
   or note why not. Run `/check` when a "done when" needs observed runtime
   behaviour - a click, download, request, command-line command, background job,
   or flow across screens - and prove it against the real app rather than
   eyeballing it.
5. **Iterate until it works.** If it fails or the user wants changes, revise the
   step (re-prompt or hand-edit the code), show the updated diff, and re-test.
   Repeat until the user approves.
6. **Mark it done, then prompt when required.** After the applicable gate is
   satisfied, check the step off (`- [x]`) in
   `context/current-feature.md` so progress survives a context
   clear. If the step repaired a finding tracked in
   `context/findings.md`, set that finding's status to `fixed` now too
   and note the repair in its **Resolution** line. **Read the work order's
   `Fixes:` line to find which finding that is** - `/fix F-03` stamps the ID
   there precisely so the link survives a context clear. Fall back to matching
   the repair against the ledger only when the line is absent, and say you did.
   Never set `closed`: a repair
   is re-reviewed by `/audit` before it clears, because a fix can introduce a
   worse defect than the one it removed. Then offer a short choice, noting that
   checkpoints are optional since `/complete` makes the real work-level
   commit. Use the current tool's short
   user-input prompt when available; when you've just produced a long block to
   read (a deep explanation, a big
   walk-through), ask in plain text instead, so the prompt doesn't cover what the
   user is still reading:
   - **Continue** (default) - roll into the next step without committing.
   - **Commit checkpoint** - commit just this step with a conventional message
     (a cheap rollback point).
   - **Walk me through it** - give a deeper, line-level explanation of the new or
     changed code (why this approach, what each part does, any gotchas), then
     re-ask this checkpoint prompt. A loop-back, not a terminal choice.
   - **Stop here** - pause the loop so the user can review or come back later.

   On **Continue** or after **Commit checkpoint**, go to the next step. On **Walk
   me through it**, explain in depth and then re-ask this prompt in plain text (the
   explanation is long, so a modal would cover it). On **Stop here**, stop and say
   where things stand: the work is intact on disk; run `/implement` again to
   resume, or `/complete` to wrap up what's built so far.

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
- Build only what the spec says. If the spec is wrong or thin, stop and ask the
  human to change it — do not improvise, and do not edit the spec yourself.
- Never create, switch, merge, or delete a branch. Per-step commits are optional
  checkpoints; the work-level commit is `/complete`'s job, and any push needs the
  user's explicit yes.
- For Type: Rollback, reverse only the approved product diff and preserve all
  protected loop paths.

## Formatting

Format the output to match the project's conventions in `AGENTS.md`: concise,
scannable markdown, with lists for enumerations and tables for matrices rather
than dense paragraphs.
