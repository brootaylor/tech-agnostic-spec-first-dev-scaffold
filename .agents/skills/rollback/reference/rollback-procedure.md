# Rollback procedure - the guarded reverse patch

`/implement` follows this for the first build step of a `Type: Rollback` work
order, and `/rollback` uses its protected-path list when it plans one. It is the
only copy of both.

Do not hand-delete the old feature and do not run a whole-commit `git revert`.
Completed feature commits also contain loop history and plan bookkeeping, while
`current-feature.md` now contains the active rollback work order. Reversing the
whole commit would damage that state.

## Protected paths

Never reversed, whatever the target commit touched:

- `.agents/**`
- `.claude/**`
- `context/**`
- `docs/**`
- `AGENTS.md`
- `CLAUDE.md`
- `prototypes/**`

They hold the original feature archive, the specs, the active rollback work
order, the skills, and throwaway prototype history. Root `README.md` and
application code are product files unless the project says otherwise.

## Before the first rollback build step

1. Read the approved work order's `Target commit` and `Target parent` fields. Stop
   unless both values match `^[0-9a-f]{40}$`. Do not accept abbreviated,
   uppercase, or otherwise malformed SHAs.
2. Resolve the archive's introducing commit and verify it has exactly one parent.
   Stop on a merge target. Resolve that single parent to a full SHA value.
   Confirm the resolved commit exactly equals `Target commit` and the resolved
   parent exactly equals `Target parent`. Stop on any mismatch.
3. Confirm the resolved target is an ancestor of `HEAD` and the only dirty path
   before applying the patch is the approved rollback work order. Stop on drift.
4. Preview the resolved target's product diff while excluding the protected
   paths. Confirm the preview is non-empty and matches the Product paths in the
   work order.
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

## If the reverse patch conflicts

> [!IMPORTANT]
> A conflicting reverse apply does not leave the tree untouched. `--3way` writes
> conflict markers into the working tree files and leaves the index at unmerged
> stages, and git reports it as `Applied patch to '<file>' with conflicts.` -
> the word *Applied*, on a command that exited non-zero. Leaving it for a later
> skill to notice is not a plan: `/complete` stages everything for the work
> commit, so anything still conflicted here is committed as product code with
> its markers intact. Its safety pass rejects an unmerged index for exactly this
> reason, but that is a backstop, not the fix - resolve it or report it here.

Say so explicitly and report three things: the exact paths, the later commit that
appears involved, and the fact that the working tree now holds conflict markers.
Confirm the damage with `git status` (conflicted paths show as `UU`) and
`git ls-files -u`. Do not auto-resolve, discard, stash, reset, or switch to a
broad checkout - those are the user's call precisely because the tree is already
dirty. Ask whether to resolve only the conflict allowed by the approved work
order or abandon the attempt, and if the answer is to abandon, ask before running
the cleanup rather than choosing one. A cascade into another completed feature
needs a new rollback plan.
