# Claude Code — Agent Config

> `docs/project-brief.md` is imported below, so it is already in context - do
> not read it again. All project conventions, workflow rules, and context live there.
> This file only contains additions specific to Claude Code.

Shared cross-tool instructions live in `AGENTS.md`, imported below along with the
context files Claude Code keeps loaded each session.

@AGENTS.md

@docs/project-brief.md

@context/current-feature.md
@context/findings.md

---

## Setup note

Claude Code expects this file at the project root as `CLAUDE.md`. It lives here
instead at `.agents/claude/CLAUDE.md` as part of this project's agent-agnostic
structure. Symlinking is one option, though it isn't guaranteed to work across all
operating systems or agent versions:

```bash
ln -s .agents/claude/CLAUDE.md CLAUDE.md
```

The workflow skills need one more pointer:

```bash
mkdir -p .claude && ln -s ../.agents/skills .claude/skills
```

`.claude/` is gitignored, so it won't exist in a fresh clone — without the
`mkdir` the symlink fails with `No such file or directory`.

Both are gitignored, so they never travel with the repo.

Do not run `/init` in this project. It would write a new `CLAUDE.md` at the
root, where that filename is a symlink to this file — overwriting the tracked
original or shadowing it with a copy that drifts. Nothing needs generating: the
imports above already load the project's context every session. If someone asks
for an orientation, run `/status`.

---

## Claude Code-specific notes

Workflow skills are in `.agents/skills/` and are shared with every other agent —
there are no Claude-only commands.

This scaffold ships no `settings.json`. Claude Code reads one from `.claude/`,
which is gitignored, so anything put there stays personal to whoever set it up —
which is the right home for machine-specific paths, permission rules, and hooks.
The context files above are loaded by the `@` imports at the top of this file,
not by a settings key, so a fresh clone needs no settings file to arrive briefed.

When a skill covers the task (`/feature`, `/implement`, `/check`, `/complete`,
and so on), use it rather than improvising — the skills encode the review gates.
`AGENTS.md` lists them in the order you'd run them.

Spec files live in:

- `docs/features/` — user-facing feature specs
- `docs/specs/components/` — component specs
- `docs/specs/pages/` — page / view specs
- `docs/specs/layouts/` — layout specs

Always read the relevant spec before generating or editing anything. Co-located
`*.spec.md` files in `src/` are copies written by `/implement`; the `docs/specs/`
version is the source of truth if they ever differ.

The loop's working state lives in `context/`:

- `current-feature.md` — the work order in flight, or the stub when idle
- `findings.md` — the review ledger `/audit` writes
- `history/` — archived work orders, the record of what was built
