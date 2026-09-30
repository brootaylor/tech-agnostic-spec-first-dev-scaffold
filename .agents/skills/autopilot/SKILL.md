---
name: autopilot
description: "Optional explicit mode that runs one bounded pass of the build loop - work order, build steps, gates, checkpoint commits, repairs of confirmed high-severity findings - without pausing at each review point, then stops with a review packet. Never completes, pushes, deploys, publishes, or does anything destructive without explicit approval. Use only when the user explicitly runs /autopilot or directly asks for it."
---

# autopilot - run the loop without stopping at every gate

Where this sits in the workflow:

    /status  ->  [autopilot]  ->  review packet  ->  /complete
    (where       (work order,      (human review,     (log, commit,
     are we?)     build, gates)     fixes if needed)   reset)

Autopilot is an explicit opt-in path. A single user request is permission to run
one bounded pass until the work is ready for review, blocked, or unsafe to
continue.

It does **not** replace the normal workflow. `/feature`, `/implement`, `/check`
and `/complete` remain the conservative default, and the human review between
each of them is the point of a spec-first loop. Do not suggest Autopilot as the
default next action. Use it only when the user explicitly asks for it.

The explicit request is permission to make checkpoint commits on the current
branch after passing implementation steps. It is not permission to push, deploy,
publish, send, delete data, or run destructive actions.

## What Autopilot changes

Autopilot **runs the other skills as written**. It is not a second copy of their
rules, and where this file is silent the skill that owns the step decides. These
are the only differences:

| Step | Skill that owns it | Autopilot's difference |
|------|-------------------|------------------------|
| Write the work order | `/feature`, or `/fix` for an ad-hoc repair | Continue past the red-team gate without asking; report what the critique changed |
| Build each step | `/implement` Step 2 | No per-step approval prompt |
| Checkpoint commits | `/implement` Step 2 | Always commit a passing step, rather than offering it as a choice |
| Quality gates | `/complete` "Quality gates" | A needed gate that cannot run is a hard stop, not a note |
| Findings | `/audit` | Repair confirmed P0 and P1 in scope, then hand the recheck back to `/audit` |
| Finishing | `/complete` | Autopilot stops before it, with a review packet |

> [!IMPORTANT]
> Read the owning skill before running its step. Working from this table alone
> means building without the rules it points at - and a mode that skips the
> approval prompts is exactly the one that cannot afford to skip the rules too.

## Input

- **No argument** - resume the current work order if one exists, otherwise target
  the next `Ready` spec in `docs/features/`, then `docs/specs/`. Never a `Draft`
  spec: stop and report which specs need human review. Never re-implement a
  `Complete` one. Promoting a spec to `Ready` is a human act and is not available
  to this mode.
- **A name or path** - target that spec, for example `/autopilot button`.
- **`fix "<issue>"`** - write and build an ad-hoc fix work order.
- **`resume`** - continue the work order already in `context/current-feature.md`.

If the requested target conflicts with work already in progress, stop and ask
which one should win. Do not overwrite `context/current-feature.md` silently.

Rollback is intentionally excluded. If the request is a rollback, or
`current-feature.md` is marked `Type: Rollback`, stop and direct the user to the
reviewed `/implement` path. Reversing completed work requires the explicit
dependency and conflict gates in `/rollback` and `/implement`.

## Step 1 - preflight

Read the state `/status` reads: `AGENTS.md`, `docs/project-brief.md`, the target
spec and anything it depends on, `context/current-feature.md`,
`context/findings.md`, and the git branch, status and recent log.

Then decide whether it is safe to run at all. Stop before changing any file when:

- the repo is not a git repo
- the working tree is dirty and there is no work order tying those changes to
  this run
- `current-feature.md` has real work and the user requested a different target
- the target spec is `Draft`, or a `Ready` feature spec depends on a `Draft`
  component spec
- the work is visual or replication-heavy and no design reference exists
- the task needs product, data, auth, billing, or destructive decisions that
  `docs/project-brief.md` and the spec do not answer

## Step 2 - run the loop

Follow the skills in order, with the differences named in the table above:

1. **Work order** - resume the active one from its first unchecked step, or write
   one with `/feature` (planned spec) or `/fix` (ad-hoc repair), red-team
   included.
2. **Base commit** - `/implement` Step 1, unchanged. Autopilot does not create,
   switch, merge, or delete branches; it works on whatever branch is checked out.
3. **Build** - `/implement` Step 2 for every build step, in order, with its
   verification, self-review, step ticking and `**Work status:**` updates exactly
   as written. Skip only the approval prompt, and commit each passing step rather
   than offering the choice. Use a conventional message about the step, not about
   Autopilot: `feat: button focus and hover states`, not `feat: autopilot step 2`.
4. **Gates** - once every step is checked, apply `/complete`'s Quality gates in
   its order: `/check`, `/audit current`, `/try`. Reuse evidence already produced
   in this run. `/try` only writes instructions for a human; never claim the user
   performed them.
5. **Findings** - for each one `/audit` records, repair confirmed P0 and P1
   findings that stay inside the approved scope and need no product or
   architecture decision, and set them `fixed`. Report P2 and P3; fix one only
   when it is small, caused by this work, and required by the project's
   conventions. A confirmed P0 or P1 that cannot be repaired safely in scope is a
   stop, not a note in the packet.

After any repair, rerun the `Verify` command and the acceptance evidence it
affected, then run `/audit` again over the repaired area and name the result in
the packet. That pass is what moves a `fixed` finding to `closed`.

> [!IMPORTANT]
> **Never close your own repair by rechecking it yourself.** A repair is done
> when a review has looked at the result, not when the code changes - and
> Autopilot runs with the least human oversight in the loop, so this is where a
> self-graded repair does the most damage.

Stop after two failed repair attempts on the same issue. Do not widen the work
into a general refactor or a whole-project hardening pass; that is a separate
`/audit` followed by planned `/fix` work.

## Step 3 - final review packet

Stop with a concise packet - useful enough for `/complete`, not a full audit
report:

- target spec or fix, and whether the work order was created or resumed
- what the red-team critique changed
- changed files and why each changed
- build, test and check commands run, with pass or fail
- which gates ran and which did not apply
- screenshots or output paths, when relevant
- how to try it manually, or a pointer to `/try` for the full walkthrough
- checkpoint commits made
- audit scope, findings, repairs made, and checks rerun, when the gate ran
- P0/P1 findings still `open` or `fixed` in `context/findings.md`, which block
  `/complete`
- unresolved risks or skipped checks
- exact next action

If everything is green, the next action is usually: review the diff, run `/try`
if a walkthrough is wanted, then `/complete`. If something failed, name the
failing check and the next fix target.

## Hard stops

Stop immediately and report instead of continuing when Autopilot would need to:

- push, deploy, publish, merge, or send anything
- delete data, reset a database, run irreversible migrations, kill processes, or
  change system settings
- install dependencies or use network access without the current tool's approval
  flow
- make a product decision the spec and `docs/project-brief.md` do not cover
- promote a `Draft` spec to `Ready` to give itself work
- run a gate the work needs but the project cannot support
- continue after two failed fix attempts on the same issue
- hide, skip, or hand-wave a failing check

## Rules

- One Autopilot run handles one feature or one fix.
- Autopilot stops before `/complete`. Writing a spec's `**Status:**` back to
  `Complete` stays a reviewed act.
- Keep `context/current-feature.md` accurate as each step completes. Nobody is
  watching this run, so that file is the only thing an interrupted one can be
  resumed from.
- Prefer fewer, higher-quality changes over broad coverage, and report
  uncertainty plainly. A blocked run is useful if it tells the truth.

## Formatting

Output follows `AGENTS.md` - Output conventions.
