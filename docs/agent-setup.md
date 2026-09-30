# Agent setup

How each "Ai" coding agent is wired to this project: where its config lives, the
pointers it needs, and how to add, switch, or remove one. The rule these all
serve is in `AGENTS.md` → Agent configuration: **nothing tool-specific is
committed**. Read this once, while setting up, and again when adding an agent.
Nothing in the build loop needs it.

## Structure

Any agent that needs a config of its own gets a directory:

```bash
.agents/
├── claude/     # Claude Code
├── skills/     # the shared workflow skills, read by any capable agent
└── ...         # add any agent that has a config file convention
```

Every file inside an agent directory does two things only:

1. Points the agent at the context files listed in `AGENTS.md`, starting with
   `docs/project-brief.md`
2. Adds anything genuinely specific to that agent (custom commands, model
   settings)

Nothing else belongs in them. `.agents/skills/` is shared, not tool-specific:
every agent works from the same tree, whether it discovers it through a pointer
or is sent there by `AGENTS.md`.

## How the pointers are wired

An agent that reads `AGENTS.md` needs no pointer at all - that file is already at
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

## Switching between agents

There is no project-level switch. Every agent reads the same
`docs/project-brief.md` and the same specs, so they always share one
understanding of the project. For most agents there is nothing to do but open the
project; for one that needs a pointer, create it first.

## Adding a new agent

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
nothing else to duplicate. Do not duplicate the skills under `.cursor/`,
`.opencode/skills/` or any other tool's tree either; each of those discovers a
compatible tree already.

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

## Removing an agent

Delete the pointer and the directory:

```bash
rm CLAUDE.md
rm -rf .agents/claude
```

Nothing else changes. A project using no Claude Code can delete the `CLAUDE.md`
pointer, `.claude/` and `.agents/claude/`, but must keep `AGENTS.md` and
`.agents/skills/` - the first is what every other tool reads, the second is where
the skills actually live.

Unused pointers can be removed, but **`.agents/` is never one of them.** Those
pointer paths are gitignored; `.agents/` holds the only real copies of both the
agent config and the skills tree. Deleting it leaves every pointer dangling - no
config and no skills, with `ls -l` still showing links that look healthy.

## Troubleshooting

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
