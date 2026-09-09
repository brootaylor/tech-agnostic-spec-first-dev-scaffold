---
name: status
description: "Show where the project stands: the spec queue by status, the current work order's checked and unchecked steps, git state, drift warnings, and the exact next action. Read-only. Use when the user runs /status, asks where things stand, what's next, what's in progress, or is picking work back up after a break or a context clear."
---

# status - where the project stands right now

Where this sits in the workflow:

    any time  ->  [status]  ->  reads the spec queue + current-feature + git
                  (read-only)   prints a short "you are here"

This skill answers one question: *where am I?* It reads the files that already
track progress and prints a short orientation. It is the fast way back in after a
break, a context clear, or a day away. It never changes anything: no edits, no
commits, no installs, no builds, no branch changes.

Progress in this workflow lives in files, not the chat, so everything this skill
reports comes from disk and git. That is the point: a fresh session can run
`/status` and know exactly as much as the last one did.

For setup problems - an unfilled `docs/project-brief.md`, missing commands in
`AGENTS.md`, or agent pointers that were never wired up - see `docs/workflow.md`
Steps 1-3 rather than this skill.

## Input

None. `/status` takes no argument.

## What it reads

Gather these, then summarize. Don't dump file contents; report the distilled
state.

1. **The spec queue** - the `**Status:**` line of every spec in `docs/features/`
   and `docs/specs/`. **This is the project's real progress tracker.** Count them
   by status and name the next `Ready` spec, the same target `/feature` would
   pick. **Skip both templates** - `docs/features/_feature-template.md` and
   `docs/specs/_component-template.spec.md`. Each carries a `**Status:** Draft`
   line of its own, so counting them reports phantom work and names a template as
   something a human has to promote.
   Call out any `Ready` spec blocked by a `Draft` dependency, since `/feature`
   will refuse it. Check every spec type, not just features - a page can be
   blocked by a `Draft` layout, and a component by another `Draft` component.
   Dependencies come from the spec's own table: **Components required** in a
   feature spec, **Dependencies** in a component, page, or layout spec.
2. **Current work** - `context/current-feature.md`. Is something in
   progress, or is it the reset stub? If a work order is present, report its type
   and name, the source spec named on its `Spec:` line, which build steps are
   checked, the first unchecked step where `/implement` resumes, and its
   `**Work status:**` line.

   **That line is the only record of whether the work was ever proved.**
   `/implement` and `/complete` write `verified` to it, and `/check` writes
   `verification failed` or `verification incomplete` when it could not. None of
   that is visible in the checkboxes: every step can be ticked on a work order
   whose last `/check` failed. Report the value, and never infer verification
   from checked boxes alone. It is the work order's own field - the spec's
   `**Status:**` line is a different thing in a different file, and this skill
   reads both and writes neither.
3. **Findings** - `context/findings.md`. Count findings by status and
   report open and fixed counts next to the spec queue. Call out any P0 or
   P1 still `open` or `fixed` by ID, since those block `/complete`. A missing
   file means no findings.
4. **Git** - current branch, whether the working tree is clean or has uncommitted
   changes, roughly how many files changed, last commit subject, and whether the
   branch is ahead of its remote. If the directory is not a git repo, say so and
   skip this part rather than failing.
5. **Where things stand** - `context/sessions.md`, when it exists. Items 1-4 can
   be rederived from disk at any time; this one cannot. Read its **Where things
   stand** block for the open items and the next action it recorded, and carry
   anything still true into this report. **A missing file is normal, not a
   gap** - it is gitignored, so a fresh clone has none and `AGENTS.md` tells the
   user to create it on first use. Say nothing about it in that case.

   Then check it against what items 1-4 actually found. It is written by hand at
   the end of a session and nothing updates it afterwards, so it is the one input
   here that can be confidently wrong: a queue that has moved on, a branch line
   from before the last commit, an open item already closed. **Where it disagrees
   with the files, the files win.** Put the disagreement on the `Watch:` line and
   carry on - a stale state file is drift like any other, and this skill reads it
   without ever writing it back.
6. **Progress drift** - flag all steps checked but not completed, a `Spec:` line
   pointing at a spec that no longer exists, a spec marked `Complete` with
   nothing implemented, or a spec still `Ready` after its work was completed. A
   rollback legitimately targets a `Complete` spec until `/complete` resets it,
   so do not flag that as drift.

## Output

A short, scannable summary, not a wall of text. Aim for something like:

    Status: Building docs/specs/components/button.spec.md
    Specs: 1 Complete, 1 Ready, 4 Draft. Next Ready: theme-toggle.
    Current work: Step 2 of 3 done, work status "in progress". Next step:
                  focus and hover states.
    Findings: 1 open P2 (F-04), 1 fixed P1 awaiting re-review (F-02).
    Git: 3 uncommitted files, last commit "feat: base button".
    Watch: dark-mode.md is Ready but theme-toggle.spec.md is still Draft, so
           /feature will refuse it.

    Next action: run /implement for Step 3.

End with a single suggested next action, chosen in this order:

- A work order is in progress with unchecked steps -> `/implement` and name the step.
- A work order is in progress and all implementation steps are checked -> read
  `**Work status:**` to choose. `verification failed` or `verification
  incomplete` -> `/implement` to repair, naming what `/check` could not prove.
  Anything short of `verified` -> `/check`, since nothing has recorded proof.
  `/implement` when a P0 or P1 finding is still `open` (the repair is an extra
  reviewed step), `/audit` when one is `fixed` and awaiting re-review (both
  block `/complete`). `/try` if the user wants a manual review path. Otherwise
  `/complete`.
- `current-feature.md` is the reset stub and a P0 or P1 finding is `open` ->
  `/fix <finding id>`; when one is `fixed`, `/audit` to re-review and close it.
- `current-feature.md` is the reset stub and a spec is `Ready` -> `/feature` and
  name that spec.
- `current-feature.md` is the reset stub and every spec is `Draft` -> say the
  queue is blocked on human review, name the `Draft` specs, and suggest `/brief`
  to see what each one still needs before it can be promoted.
- Every spec is `Complete` -> say the current milestone is complete; suggest
  hardening, release, or docs when appropriate, or writing the next spec.

If something is off, include a `Watch:` line before the next action. Catching
drift is half the value of the command.

## Rules

- **Read-only, always.** This skill never writes a file, never commits, never runs
  installs, never runs builds or tests, and never switches branches. If the user
  wants to act on what it reports, they run the relevant skill next.
- **Prefer exact next actions.** Do not end with vague advice like "continue the
  workflow". Name the command and, when useful, the file or step.
- **Distill, don't dump.** Report the state in a few lines. Do not paste file
  contents back unless the user asks for them.
- **Be honest about gaps.** If a file is missing or the repo is not initialized,
  say that plainly instead of guessing.

## Formatting

Format the output to match the project's conventions in `AGENTS.md`: concise,
scannable markdown, with lists for enumerations and tables for matrices rather
than dense paragraphs.
