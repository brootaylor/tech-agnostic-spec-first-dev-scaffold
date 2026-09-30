---
name: fix
description: Document an ad-hoc bug fix or small change (one with no spec of its own) into context/current-feature.md so it runs through the same build loop. Writes a short fix work order and stops; then /implement builds it and /complete logs it to context/history/fixes/ and commits it. Use when the user runs /fix, reports a bug, or asks to fix or change something that has no spec of its own.
---

# fix - document an ad-hoc fix, then build it like anything else

Where this sits in the workflow:

    /fix  ->  /implement  ->  /complete  ->  back to your features
    (document (build it,      (log to context/history/fixes/
     the fix)  reviewed)       + commit)

A fix is a bug or small change with no spec of its own. It runs through the same
loop as a feature (build with review gates, iterate, then commit); it just starts
here instead of `/feature`, and is logged separately.

## Input

A description of the bug or change, for example `/fix "password reset email never
sends"`. If the user just reported the problem in chat, use that.

The input may also be a finding ID from `context/findings.md`, alone
or with a description, for example `/fix F-03`. Pull the problem statement from
that ledger entry.

## Step 1 - write the fix work order

**Stop first if `context/current-feature.md` holds anything but the reset
stub.** This skill overwrites that file, and a work order in flight would lose
its ticked steps and its `**Base commit:**` without a word. Name it and point at
`/implement` or `/complete`; a finding against active work is repaired through
`/implement` instead.

Pull context from `docs/project-brief.md` and `AGENTS.md`, then write a short
work order to `context/current-feature.md` (this file holds whatever is being built
now, feature or fix), following `reference/fix-work-order-template.md`. Keep it
lighter than a feature work order: the problem, the fix, build steps, files, and how
to verify.

Four fields in that header are load-bearing, and each fails quietly if you get
it wrong:

- **`Type: Fix`** - what `/complete` reads to log this to
  `context/history/fixes/` rather than `context/history/features/`.
- **`Work status:`** - not `Status:`. That name belongs to a spec's own status
  line, which only `/complete` may write; a work order that borrows it invites
  an agent to edit the wrong file.
- **No `Spec:` line at all.** A fix has no source spec, and both `/implement`
  and `/complete` treat its absence as correct. `Spec: none` is worse than
  omitting it - it reads as a path that will not resolve.
- **`Fixes:`** - the finding ID, whenever the input was one (`/fix F-03`).
  `/implement` reads it to know which ledger entry to mark `fixed` when the
  repairing step lands. Without it the link survives only in the conversation,
  and a context clear breaks it: the repair ships, the finding stays `open`, and
  `/complete` refuses to finish for a defect that was already fixed. Delete the
  line entirely for a fix that came from a bug report rather than the ledger.

Then stop. Tell the user to review the fix work order, then run `/implement` to build it.

## Rules

- Keep it small. If it's really a new feature, stop and say so. It needs a spec
  in `docs/features/` or `docs/specs/`, and that spec has to be promoted to
  `Ready` by the human before `/feature` will touch it. Drafting one is
  `/feature`'s no-spec intake, on explicit approval and only ever as `Draft`.
  **Never mark a spec `Ready` to unblock your own work** - that promotion is the
  human's signal that the contract is settled, and it is the gate this whole
  project is built around.
- Same conventions as everything else (`docs/project-brief.md`).

## Formatting

Output follows `AGENTS.md` - Output conventions.
