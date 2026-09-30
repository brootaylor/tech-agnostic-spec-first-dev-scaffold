# Agent setup

How each "Ai" coding agent is wired to this project: where its config lives, the
pointers it needs, and how to add, switch, or remove one. The rule these all
serve is in `AGENTS.md` → Agent configuration: **nothing tool-specific is
committed**. Read this once, while setting up, and again when adding an agent.
Nothing in the build loop needs it.

## Structure

`.agents/` is the only committed home for anything an agent reads besides
`AGENTS.md` itself:

```bash
.agents/
├── skills/     # the shared workflow skills, read by any capable agent
└── ...         # a directory for any agent that insists on a config file of its own
```

The scaffold ships no agent directory, because every agent it documents reads
`AGENTS.md` directly. One is needed only for an agent that will not, and every
file inside it does two things only:

1. Points the agent at `AGENTS.md`, which carries everything else
2. Adds anything genuinely specific to that agent (custom commands, model
   settings)

Nothing else belongs in them. `.agents/skills/` is shared, not tool-specific:
every agent works from the same tree, whether it discovers it through a pointer
or is sent there by `AGENTS.md`.

## How the pointers are wired

An agent that reads `AGENTS.md` needs no pointer for its instructions - that file
is already at the project root, and it imports `docs/project-brief.md`,
`context/current-feature.md` and `context/findings.md` for any agent that
expands `@` imports. Codex, Cursor, GitHub Copilot, Gemini CLI, Jules, Aider,
Zed, Windsurf, Devin and OpenCode all read it, and so does Claude Code from
v2.1.277 ([its documentation](https://code.claude.com/docs/en/memory#agents-md)).

Claude Code needs one pointer, for the skills. It discovers them only under
`.claude/skills/`, which is gitignored here, so anything put there stays personal
to one machine:

| Location | Points to |
|----------|-----------|
| `.claude/skills` | `../.agents/skills` |

**macOS and Linux** - symlink:

```bash
mkdir -p .claude && ln -s ../.agents/skills .claude/skills
```

> [!IMPORTANT]
> *The pointer needs both the `mkdir -p` and the leading `../`, and each guards a
> different failure. `.claude/` is gitignored, so it does not exist in a fresh
> clone and `ln -s` will not create it. And a symlink's target is resolved
> relative to the link's own directory, so `.agents/…` without the `../` creates a
> link pointing at `.claude/.agents/…` - which `ln` reports as success and `ls -l`
> displays as if it were correct. `ls .claude/skills` is what proves it resolves.*

**Windows** - `ln -s` needs Developer Mode or an elevated terminal. If neither is
available, copy instead, then keep the copy in sync by hand:

```bat
if not exist .claude mkdir .claude
xcopy /E /I .agents\skills .claude\skills
```

**Every pointer is gitignored**, so a Windows copy and a macOS symlink never
collide in git, and no one inherits another developer's agent choice. Symlink
where you can: a copy drifts from its source, which is how the shared skills tree
ended up symlinked rather than duplicated per tool.

> [!IMPORTANT]
> **Claude Code reads `AGENTS.md` only while the project has no `CLAUDE.md`.**
> *A `CLAUDE.md` or `CLAUDE.local.md` at the root, or a `.claude/CLAUDE.md`,
> makes it read that file instead, and `AGENTS.md` silently drops out - with it
> the brief and every rule in this workflow. Claude Code's built-in `/init`
> creates exactly that file, so do not run it here. If one has appeared, delete
> it, or reduce it to a single line, `@AGENTS.md`, so it imports this file rather
> than replacing it. `/memory` lists which instruction files a session loaded.*

**On a Claude Code older than v2.1.277**, or with its built-in `agents-md` plugin
disabled, nothing loads on its own. Create a root `CLAUDE.md` whose only line is
`@AGENTS.md`. `/CLAUDE.md` is gitignored, so that file stays yours.

## How each agent finds the skills

- Claude Code: discovers `.claude/skills/<skill>/SKILL.md` on its own, and
  invokes them as `/feature`, `/implement` and so on
- Codex: invokes them as skills (`$feature`, `$implement`)
- Every other agent - Cursor, GitHub Copilot, OpenCode, and any tool with no
  dedicated syntax: `AGENTS.md` names the path, `.agents/skills/<skill>/SKILL.md`,
  and you name the skill in your prompt and ask the agent to follow it

> [!IMPORTANT]
> *Only Claude Code loads this tree by itself. For every other agent the path is
> written down where the agent will read it, and **naming the skill is what runs
> it** - there is no auto-discovery to rely on. A tool that has its own skills
> convention will not find these at `.agents/skills/`. The review gates are in the
> `SKILL.md`, so a skill followed this way behaves the same as one invoked.*

## Switching between agents

There is no project-level switch. Every agent reads the same `AGENTS.md`, the
same `docs/project-brief.md` and the same specs, so they always share one
understanding of the project. For most agents there is nothing to do but open the
project; for one that needs a pointer, create it first.

## Adding a new agent

First check whether the agent reads `AGENTS.md` - most now do, and those need
none of the steps below. For one that insists on a config file of its own:

1. Create `.agents/<agent-name>/`
2. Create the agent's required config file inside it
3. In that file, point the agent at `AGENTS.md` - an `@AGENTS.md` import where
   the agent supports imports, otherwise an instruction to read it first
4. Add any agent-specific config below that line
5. Create the pointer at the location the agent expects
6. Add that pointer path to `.gitignore`, **anchored with a leading slash** -
   `/<CONFIG>.md`, not `<CONFIG>.md`

All project conventions are already in `AGENTS.md` and `docs/project-brief.md`,
so there is nothing else to duplicate. Do not duplicate the skills under
`.cursor/`, `.opencode/skills/` or any other tool's tree either; each of those
discovers a compatible tree already.

> [!IMPORTANT]
> **A `.gitignore` pattern containing no slash matches at every depth, not just
> the root.** *Unanchored, the pointer's filename ignores the pointer **and** the
> real config at `.agents/<agent-name>/` - the only tracked copy. This project's
> own `.gitignore` once carried an unanchored `CLAUDE.md`; nothing was lost only
> because the config it matched was already tracked, and an ignore rule cannot
> untrack. A config you add now has no such protection. Nothing reports it:
> `git add -A` skips the file in silence, `git status` cannot list what it never
> staged, and the pointer resolves perfectly for whoever created it. The failure
> surfaces only in someone else's clone, as a dangling link. Anchor the pattern,
> then prove it with `git check-ignore -v .agents/<agent-name>/<file>` - it
> should print nothing.*

## Removing an agent

Delete its pointer and, if it has one, its directory under `.agents/`. For
Claude Code that is just the skills pointer:

```bash
rm -rf .claude
```

Nothing else changes. Every project must keep `AGENTS.md` and `.agents/skills/` -
the first is what every tool reads, the second is where the skills actually live.

Unused pointers can be removed, but **`.agents/` is never one of them.** Those
pointer paths are gitignored; `.agents/` holds the only real copy of the skills
tree, and any agent config. Deleting it leaves every pointer dangling - no skills,
with `ls -l` still showing links that look healthy.

## Troubleshooting

**The agent isn't reading `AGENTS.md`.** For Claude Code, check for a
`CLAUDE.md` it would read instead (see above) and that `claude --version` is
v2.1.277 or later; `/memory` lists the files a session loaded. For an agent with
a config of its own, check the pointer exists where the agent expects it.

**The agent reads `AGENTS.md` but not the project brief.** Not every agent
expands `@` imports; one that does not sees the brief as a path to read, and the
agent behaviour rules tell it to. If it still skips the brief, ask it to read
`docs/project-brief.md` in full before continuing.

**The agent produces output that contradicts the project brief.** The brief is
probably incomplete or ambiguous in that area. Clarify the relevant section and
re-run. Don't hand-edit the agent's output to paper over a brief that needs
fixing.

**The agent implemented the wrong thing.** Check the spec it built against. The
spec is the contract, so a wrong result usually means a spec that was promoted to
`Ready` before it was settled.
