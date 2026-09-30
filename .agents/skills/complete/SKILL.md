---
name: complete
description: "Close out a finished feature, fix, or rollback - final safety pass, the spec's Status written back, the work order archived under context/history/, one work-level commit, then ask before pushing. Use when the user runs /complete, or asks to finish, wrap up, or close out the current feature, fix, or rollback after it is built and reviewed."
---

# complete - log the finished work and make the work commit

Where this sits in the workflow:

    /feature, /fix, or /rollback  ->  /implement  ->  [complete]  ->  next
    (the work order)                   (build it)      (commit + log)

`/implement` built the feature, fix, or rollback, with optional per-step commit
checkpoints. This skill closes it out: it writes the spec's status back, logs the
work, and makes the single work-level commit. Step 0 decides whether the work is
finished enough to close.

## Quality gates

Run these before logging or committing:

- **Check** - run `/check` when any "done when" needs observed runtime behaviour:
  a click, request, command-line command, download, background job, or
  multi-screen flow.
  Also run it whenever the work rendered or changed user interface, whatever the
  "done when" criteria happen to say. Keyboard operability, focus and accessible
  names are observable behaviour that no build step and no static diff proves,
  and `docs/project-brief.md` applies its accessibility standard project-wide
  whether or not the spec restates it.
- **Audit** - run `/audit current` when the work touched a security boundary:
  authentication, authorization, payments, secrets, personal or user data,
  migrations, destructive operations, or external side effects. Run
  `/audit current accessibility` when the work added or changed markup, styles,
  or design tokens - that lens is where contrast and semantics get measured.
- **Try guide** - run `/try` when the change affects UI, navigation, copy, a
  public API or CLI, output, or another workflow a person uses directly.

Apply them in that order: `/check`, `/audit current`, then `/try`. Reuse adequate
evidence produced during the current work item instead of repeating it. A gate
that is needed but cannot run is a blocker. `/try` only generates instructions
for human review; never claim the user performed them. The user can always ask
for any of these explicitly, and P0/P1 finding blockers apply either way.

## Step 0 - final safety pass

Before logging or committing, run a short safety pass and report blockers only:

- an active work order exists and its build steps are all checked. Uncommitted
  step work is expected - per-step checkpoints are optional, and this skill
  commits it
- changed files are tied to the active spec, with no unrelated dirty work mixed
  in (a dirty `context/findings.md` is expected, since `/audit` writes it)
- the exact `Verify` command from `AGENTS.md` passed in this session, when one is
  declared; otherwise the build passed, and tests passed when the project has a
  declared test command and the change touched logic
- every gate named above that applied to this work has evidence, and there is a
  clear manual try path
- when the work rendered or changed user interface, the spec's accessibility
  requirements **and** the project-wide standard in `docs/project-brief.md` both
  have evidence. That file forbids treating a component as complete until both
  are satisfied, so an untested accessibility requirement is a blocker here, not
  a note for later
- when the project declares a test runner, logic changes have passing focused
  tests; when it does not, say so rather than implying the logic is tested
- if workflow files changed, they were edited in the tracked `.agents/` tree
  rather than inside a gitignored pointer directory such as `.claude/`, where
  git would never see them
- the index holds no unmerged entries. Check with `git ls-files -u`; a
  conflicted path also shows as `UU` in `git status --short`. This is a hard
  stop, not a note. Step 2 stages everything, so an unresolved conflict from a
  rollback's reverse patch would be committed as product code with its
  `<<<<<<<` markers intact
- no P0 or P1 finding in `context/findings.md` is `open` or `fixed`.
  `fixed` still blocks on purpose: the repair exists but no review has looked at
  it - run `/audit` to close it. The only waivers are `accepted` (the user's
  explicit decision in the current chat, reason recorded; never set it for
  them) or `invalid` (an `/audit` re-examination verdict with recorded
  evidence, or the user's explicit call). A missing ledger file means no
  findings.
- when the work consumed `prototypes/` (its Design reference points there), the
  theme port is finished. Step 1 deletes that folder, and `theme.css` exists
  nowhere else, so this is checked here, before anything is written. Compare
  `docs/design-tokens.md` against `prototypes/theme.css` value by value: every
  colour, type and spacing value must have a matching entry, and the file's
  `**Last updated:**` line must no longer hold the template's placeholder
  comment. "Are there tokens in it" proves nothing - that file ships complete,
  so a skipped port looks exactly like a finished one, and the project would
  keep the scaffold's baseline palette with nothing reporting it. If either
  fails, hand back to `/implement` to finish the port

Do not claim "passed", "verified", or "working" without naming the command,
route, screenshot, or output that proves it. Stop before Step 1 if required
evidence is missing.

After this safety pass succeeds, set the work order's `**Work status:**` to
`verified` before archiving it. If it was already `verified`, rerun the required
final checks anyway because `/complete` owns the final safety pass.

That is the work order's own field in `context/current-feature.md`. The source
spec's `**Status:**` line is written separately in Step 1, and `Complete` is the
only value this skill may put there.

## Step 1 - log the work

Read the work order's `Type:` line: `Feature`, `Fix`, or `Rollback`. A fix has no
source spec. A rollback records the exact target feature, archive, commit, and
parent. A work order written before this line existed may have none at all -
treat the absence of `Type: Fix` and `Type: Rollback` as a feature, and say you
inferred it.

### Write the spec status back

**This is the one place in the workflow allowed to edit a spec file, and it may
touch only two lines.** Read the work order's `Spec:` line to find the source
spec in `docs/features/` or `docs/specs/`, then:

- Set `**Status:**` to `Complete` - or, for a rollback, back to `Ready`: the
  contract still stands and only the implementation was withdrawn. Retiring the
  spec for good is a separate human decision.
- Set `**Last updated:**` to today's date, in the format the file already uses.
  On a spec's first completion the line is still the template's placeholder
  comment, so there is no format to match — write ISO 8601 (`YYYY-MM-DD`),
  which is what that comment shows and what every spec in the project uses.
  Replace the whole comment; don't leave it beside the date.

Change nothing else. Not the interface, not the behaviour, not the test cases,
not a typo you noticed. The spec's body is a human-owned contract, and an agent
editing it silently is the failure this status machine exists to prevent.

If the work order has no `Spec:` line, or the path does not resolve, **stop and
say so** rather than guessing which spec to mark. A fix legitimately has no
source spec; a feature that lost one is drift worth reporting.

If a feature spec's components each have their own specs, mark a component spec
`Complete` only when that component was actually built and verified in this work.
A feature spec goes `Complete` when its own acceptance criteria are met.

### Archive the work order

- **Feature** - archive `context/current-feature.md` to
  `context/history/features/NN-name.md`. Number sequentially from what is
  already in that folder. The spec status written above is the authority on what
  is built; this archive is the record of how it was built.
- **Fix** - archive it to `context/history/fixes/NN-name.md`, numbering
  sequentially from what is already in that folder, the same way features are
  numbered. **The number is what makes the write safe.** Fixes are named from a
  short description of the bug, so two of them collide far more readily than two
  features do - and an unnumbered write would overwrite the older archive with
  nothing reporting it. A fix has no source spec, so there is no status to write
  back.
- **Rollback** - archive it to
  `context/history/rollbacks/YYYY-MM-DD-NN-name.md`, preserving the original
  completed feature archive. Create `context/history/rollbacks/` first if an
  older installation does not have it yet.

**Archive resolved findings.** If `context/findings.md` holds any
findings, append a `## Findings` section to the archive file just written with
every `closed`, `accepted`, or `invalid` entry at its final status (`accepted`
entries keep their recorded reason). Prefix each ID with the archive name for
global uniqueness: feature 12's `F-03` becomes `12/F-03`; fixes and rollbacks
use their archive filename as the prefix. An entry carried forward from earlier
work archives with the item that resolved it; its **Found** line preserves
where it came from. Only `closed`, `accepted`, and `invalid` entries are
resolved for archival. A `fixed` entry is not resolved at any severity: never
append it to the archive or remove it from the live ledger.

Then remove only the archived entries from the ledger. Entries with `open`,
`fixed`, or `unverified` status stay verbatim with their IDs, never rewritten
and never silently dropped - a fixed P2/P3 finding does not block completion,
but a later `/audit` still has to re-review it. Only when none of those remain,
reset the ledger to exactly this stub, and create it the same way if the file is
missing (an older install):

    # Findings

    > **Generated file.** The findings ledger: review findings raised by `/audit`
    > against the work in progress, each with a durable ID, severity (P0-P3), and
    > status. `/implement` marks repaired findings `fixed`, a later `/audit` pass
    > moves them to `closed`, and `/complete` refuses to finish while any P0 or
    > P1 finding is `open` or `fixed`, then archives resolved findings with the
    > work and resets this file.

    _No findings recorded. `/audit` appends findings here when it finds them._

Then replace `context/current-feature.md` with the canonical stub below. Do not
paraphrase it or substitute an abbreviated "no work" stub. Before committing,
read the file and confirm it exactly matches:

    # Current Feature

    > **Generated file.** Holds the work order for the one feature, fix, or rollback
    > being built right now. Run `/feature <spec>` to sequence a `Ready` spec from
    > `docs/features/` or `docs/specs/`, or `/fix "<bug>"` for an ad-hoc fix. Use
    > `/rollback <completed-feature>` to plan a safe reversal. Build one thing at a
    > time; `/complete` writes the spec's status back, archives this under
    > `context/history/`, and resets this file.

    _Nothing in progress. Run `/feature`, `/fix`, or `/rollback` to start._

If the ledger was reset, confirm `context/findings.md` exactly matches the
Findings stub above as well.

Don't commit yet; the next step makes one work commit covering the code and these
documentation changes. The archive is the build history.

**Discard consumed prototypes.** If this feature built the look from
`prototypes/`, delete the folder now - Step 0 has already confirmed the theme
reached `docs/design-tokens.md` - and fold the deletion into this feature's
commit. The HTML mockups were always throwaway.

## Step 2 - make the work commit

Stage everything for this work item (any uncommitted step work plus the Step 1
logging changes) and make one conventional work commit (for example
`feat: <feature>`, `fix: <name>`, or `revert: roll back <feature>`).

If the work carried per-step checkpoint commits, leave them as they are. They are
this work item's history on the current branch, and rewriting them is not this
skill's job.

## Step 3 - offer to push

Stop and ask whether to push. Completing is not permission to push: it needs a
separate explicit yes in the current chat. If the repo has no remote or upstream,
say so instead of guessing.

Then point the user at `/feature`, `/fix`, or `/rollback` for the next thing,
and at `/try latest` for a manual walkthrough - it reads the archive, so it
works after `current-feature.md` is reset.

## Rules

- The work item is the unit of history: one work commit that closes out the
  feature, fix, or rollback, plus any checkpoints made along the way.
- A rollback preserves the original feature archive and adds a separate rollback
  archive. Never rewrite history to make the feature look as if it never existed.
- Don't complete unfinished or failing work; Step 0 is the gate.
- Never complete while a P0 or P1 finding is `open` or `fixed` in the ledger. The
  recorded ways past the gate without code are `accepted` (only by the user's
  explicit decision, with their reason) or `invalid` (only from re-examination
  evidence or the user's explicit call); both travel into the archive, never a
  silent drop.
- Never create, switch, merge, or delete a branch. This skill commits to the
  branch that is already checked out.
- Pushing is the user's call. Do not treat `/complete`, an approved work commit,
  or "looks good" as permission to push; ask, and push only after an explicit yes
  in the current chat.

## Formatting

Output follows `AGENTS.md` - Output conventions.
