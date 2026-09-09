# Adopting an existing project

Bringing a codebase that already exists under the build loop — a prototype that outgrew itself, an inherited repository, or an "Ai"-generated app that now has to become production-ready.

**Only the setup differs.** Steps A and B replace `docs/workflow.md` Steps 1 to 3, Steps C to E are the loop pointed at repairs rather than features, and from Step F this is `docs/workflow.md` Steps 4 to 10 unchanged. There is no second workflow to learn.

This page is the sequence, not the execution. Each step names the skill that carries it — `/survey`, `/audit`, `/tests`, `/fix` — and holds what no individual skill can know: the order they run in, why one must precede another, and which route a given problem takes. It stands to those skills as `docs/workflow.md` stands to `/feature` and `/implement`.

`docs/workflow.md` assumes an empty folder: the scaffold is cloned into it first, and the app is then built inside it. This path is the inverse — the code is already there and the scaffold merges in on top — and the difference is not only the install. A greenfield project writes a spec and then builds to it. Here the code came first, so the early work is finding out what is there, establishing a standard to measure it against, and building a safety net before changing anything.

---

## Who does what

> This document addresses the **human** running the adoption. Throughout, "you" is that person and never the agent. The `Who acts` column below uses the same vocabulary as the spec status table in `AGENTS.md`.

Steps A and B are human work. Everything from C onward is agent-driven, with a human review at each gate.

That split is structural, not a preference. **Step A cannot be agent work, because the agent does not exist in that project until Step A finishes** — `.claude/skills` and the `CLAUDE.md` pointer are created at A6, and until then a session opened there has no skills and no project conventions loaded. It is the bootstrap, so it is done by hand.

| Step | Who acts | Why |
|------|----------|-----|
| A — merge the scaffold in | Human only | Nothing is configured yet; this is what configures it |
| B — fill in the brief | Human + agent | `/survey` drafts it from the codebase; the brief stays a human-owned contract, so nothing is written without approval |
| C — audit | Agent | `/audit` |
| D — safety net | Agent | `/tests`, `/ci` |
| E — repair | Agent, gated | `/fix`, `/implement`, `/audit`, `/complete` — one human review gate per step |
| F — rebuild | Human + agent | Specs are human work; the loop that builds them is the agent's |
| G — deployment readiness | Agent prepares, human authorises | `/release` stops before any deploy or push |

---

## The whole thing at a glance

```
SETUP  ·  once  ·  A by hand, B drafted by agent and approved by you

  ├─  Step A   merge the scaffold in ····  seven substeps; copy by name, never wholesale
  └─  Step B   /survey ·················  drafts the brief; the yardstick for all of C–G
      │
      ▼
BASELINE  ·  once, and it is the deliverable  ·  agent

  └─  Step C   /audit full ··············  context/findings.md, graded P0–P3
      │
      ▼
SAFETY NET  ·  before changing a line of the project's code  ·  agent

  └─  Step D   /tests  then  /ci ········  a suite, and one Verify command
      │
      ▼
REPAIR LOOP  ·  per finding, worst first  ·  agent, gated

  └─  Step E   /fix F-nn ──▶ /implement ──▶ /audit ──▶ /complete
      │                                      │
      └──────────────────────────────────────┘
                 re-review moves it fixed ──▶ closed
      │
      ▼
REBUILD  ·  only for what is replaced  ·  specs by hand, build by agent

  ├─  Step F   write specs, promote to Ready, run the normal loop
  └─  Step G   /release ·················  deployment readiness
```

Steps E and F are different jobs. **A repair keeps the existing design and removes a defect — that is `/fix`. A rebuild replaces the design — that needs a spec**, because there is no contract to hold the new version to otherwise.

---

## Step A — Merge the scaffold into the existing project

> **Human only.** There is no agent in this project yet — A6 is what gives it one.

Seven substeps, which assume the project is already cloned and running locally. `PROJECT` below is its path.

### A1 — Take a clean baseline

Everything else here is recoverable only if there is something to recover to.

```bash
cd "$PROJECT"
git status --short        # expect no output
```

If anything is listed, commit or stash it before going further. A repository built quickly often has uncommitted work in it, and that work has no other copy.

Whether to do this on a branch is your call — no part of the loop creates, switches, or merges one, so the scaffold is content either way.

### A2 — Preserve the project's existing agent instructions

A project built with an "Ai" assistant usually carries instructions of its own, and two of them do not survive this step. A4 copies the scaffold's `AGENTS.md` over the project's, and A6 deletes its `CLAUDE.md` to make room for the pointer. Those two must be copied before you go any further.

Nothing here touches any other tool's instructions — `.cursor/rules`, `.github/copilot-instructions.md`, `.windsurfrules`, `.clinerules` — and A5 says explicitly not to untrack them. Copy them anyway, so Steps B and F have one folder holding everything the project said about itself.

Run this from inside the project, replacing `my-project` with your project's name in all three places:

```bash
mkdir -p ~/adopted-agent-instructions/my-project
for f in CLAUDE.md AGENTS.md .cursor/rules .github/copilot-instructions.md .windsurfrules .clinerules; do
  [ -e "$f" ] && cp -R "$f" ~/adopted-agent-instructions/my-project/
done
ls -aR ~/adopted-agent-instructions/my-project
```

Missing files are skipped, so run it whatever the project has. `cp -R` because `.cursor/rules` is a directory in current Cursor, not a file; `ls -aR` because `.windsurfrules` and `.clinerules` are dotfiles a plain `ls -R` would hide.

**Your home directory, not `/tmp`.** Step B reads these copies and Step F reads them again, which on a real engagement is weeks later. macOS deletes anything in `/tmp` left untouched for three days, nightly and without reporting it (`man 8 tmp_cleaner`); Linux distributions clear it on schedules of their own. Naming the folder after the project also keeps two adoptions from mixing one client's instructions into another's.

These files are worth more than they look. Claude Code's `/init` writes `CLAUDE.md` by analysing the codebase, so it frequently records the real run commands, the architecture, and the traps someone hit — facts that exist in no other file and that the code itself only implies.

Keep the copies until Step F, not just Step B. `/survey` reads them at B as **claims to verify against the code**, never as evidence — an instruction file describing a stack the project abandoned two refactors ago is common, and catching that is exactly what a survey is for. But these files usually carry a second thing the brief has no room for: statements about how the software is meant to behave. That is the raw material for any spec you write at Step F, and frequently the only record of intent that exists anywhere.

### A3 — Take a pristine copy of the scaffold

Clone it somewhere outside the project rather than copying from a working copy you already have on disk.

```bash
git clone --depth 1 https://github.com/brootaylor/tech-agnostic-spec-first-dev-scaffold.git /tmp/scaffold
rm -rf /tmp/scaffold/.git
```

> [!IMPORTANT]
> **Do not copy from your own working copy of the scaffold.** *It contains gitignored files that a fresh clone does not: `context/sessions.md` and `context/decisions.md` are your personal state and decision logs, and they describe whatever else you have been working on — frequently another client. `cp -R` ignores `.gitignore` entirely, so those files travel, land in the project, and are committed by the next `git add .` with nothing reporting it. A clone cannot carry them, which is the whole reason to take one.*

### A4 — Check for name collisions, then copy

Look before merging into `docs/`. Most of the scaffold's filenames are distinctive, but `workflow.md` and `security.md` are plausible in any project — a project's own process notes are as likely to be called `workflow.md` as anything else. List every file at any depth, because the copy below merges recursively and a nested collision would not show up in a top-level listing:

```bash
find "$PROJECT/docs" -type f 2>/dev/null
```

If any of the scaffold's filenames appear in that listing — `project-brief.md`, `setup.md`, `design-tokens.md`, `security.md`, `service-worker.md`, `storybook.md`, `modern-platform-guide.md`, `workflow.md`, `adopting-an-existing-project.md` — rename the project's copy before you run the commands below, because `cp -R` overwrites silently and the project's version is the one that holds real content. `git mv docs/security.md docs/security-original.md` keeps it in history and out of the way; fold anything worth keeping into the scaffold's version at Step B, then delete it.

Then copy the workflow layer across:

```bash
cd "$PROJECT"
cp -R /tmp/scaffold/.agents .
cp -R /tmp/scaffold/context .
cp    /tmp/scaffold/AGENTS.md .
mkdir -p docs && cp -R /tmp/scaffold/docs/. docs/
```

Copy `.editorconfig` and `.markdownlint.json` too if the project has none.

> [!IMPORTANT]
> **Never copy the scaffold's `package.json`, `src/`, `README.md`, `LICENSE` or `.nvmrc`.** *The scaffold ships a placeholder `package.json` naming no dependencies and a three-file `src/` (`index.html`, `scripts/main.js`, `assets/icons/spinner.svg`) that exists to give a new project somewhere to start. Copying either over a real project destroys the dependency list or the application entry point, and a broad `cp -R /tmp/scaffold/. .` does exactly that in one stroke — which is why the commands above name each path. The other three are the scaffold's own identity and belong to it, not to the project.*

### A5 — Merge `.gitignore` rather than replacing it

Append the scaffold's two ignore blocks to whatever is already there:

```bash
cat >> .gitignore <<'EOF'

# ── "Ai" tool config ──────────────────────────────────
.claude/
/CLAUDE.md

# ── Personal context ──────────────────────────────────
context/sessions.md
context/decisions.md
EOF
```

> [!IMPORTANT]
> **The leading slash on `/CLAUDE.md` is load-bearing, and dropping it fails silently.** *A pattern with no slash in it matches at every depth, so a bare `CLAUDE.md` also ignores `.agents/claude/CLAUDE.md` — the config A4 has just copied in, and the only real copy of it. A7's `git add -A` then skips that file without a word, `git status` cannot list what it never staged, and A6's two checks still pass because they read the working tree rather than the index. The adoption commits without the Claude config, works perfectly for you, and reaches the next person as a dangling symlink. Prove the anchor took with `git check-ignore -v .agents/claude/CLAUDE.md`, which should print nothing.*

An ignore rule does not untrack a file that is already tracked. If the project already commits a `CLAUDE.md` or a `.claude/` directory, git keeps carrying it and your new rule has no effect. Check, and untrack anything it finds:

```bash
git ls-files | grep -E 'CLAUDE\.md|^\.claude/'
git rm --cached <each path listed>
```

> [!IMPORTANT]
> *That grep names only the two paths the block above ignores. Do not widen it to `.cursor/`, `copilot-instructions` or any other tool's config: the scaffold takes no position on those, and a project that deliberately commits them for its team is entitled to keep doing so. `git rm --cached` on one of those untracks a file nobody asked you to remove.*

### A6 — Create the agent pointer

Delete the project's own `CLAUDE.md` first, if it has one. A2 already holds the copy, so nothing is lost by removing it now — and if it is still there, `ln -s` refuses to create the pointer:

```
ln: CLAUDE.md: File exists
```

That is one line among several commands, and everything after it keeps running. The result is a project where the agent reads the old instructions and none of the scaffold's, which looks like the scaffold not working rather than a link that was never made.

```bash
rm -f CLAUDE.md                 # A2 has the copy
ln -s .agents/claude/CLAUDE.md CLAUDE.md
mkdir -p .claude && ln -s ../.agents/skills .claude/skills
head -3 CLAUDE.md               # proves the CLAUDE.md link resolves
ls .claude/skills               # proves the skills link resolves
```

Those last two lines are the check, and each must print something: `head` should show the first lines of the scaffold's config, and `ls` should list the skill directories — `audit`, `feature`, `implement`, and the rest. Silence or an error from either means that link is broken. Verify this way rather than with `ls -l`, which displays a symlink pointing at a path that does not exist exactly as it displays a working one.

The project's own `AGENTS.md` has been replaced by the scaffold's at A4, and its `CLAUDE.md` by the pointer just created. Both survive in `~/adopted-agent-instructions/my-project/` from A2, which is where Step B picks them up.

### A7 — Commit the adoption on its own

One commit containing nothing but the scaffold, so the diff of every later commit is only project code:

```bash
git add -A
git diff --cached --name-only | grep '^\.agents/'   # must list the skills AND .agents/claude/
git status                                          # read it before committing
```

That `grep` is the one check `git status` cannot do for you. A file an ignore rule swallowed never reaches the index, so it appears in no status output and no diff — the only way to notice is to ask what *did* get staged. Expect `.agents/claude/CLAUDE.md` and the whole `.agents/skills/` tree. If the config line is missing, A5's ignore block lost its leading slash.

Then open your agent in the project and run `/status`. It should report the spec queue and find nothing in flight. If it does not recognise the command, the `.claude/skills` link from A6 is wrong.

---

## Step B — Fill in `docs/project-brief.md`

> **Human + agent.** `/survey` drafts; the human reviews and approves. Nothing is written without that approval.

This is the step to resist skipping, and the one most likely to be skipped. It is `docs/workflow.md` Step 2, with one change of posture: **the Stack section records what the project already uses, not what you would have chosen.** If you intend to change a choice later, that is a migration with its own spec, not a line edit here.

> [!IMPORTANT]
> **An empty brief produces an audit that finds almost nothing and says so confidently.** *`/audit` measures the code against the conventions, browser targets, and accessibility standard recorded here. With the shipped placeholders still in place it falls back to generic review, reports few findings, and gives no sign that the yardstick was blank. On a client engagement that is worse than no audit: it is a clean bill of health you cannot support.*

### Draft it with `/survey`

One message. Everything after the command is its argument, and here it names the instructions you preserved at A2 — the project's own `AGENTS.md` and `CLAUDE.md` are no longer there, so `/survey` cannot reach those on its own:

```
/survey  Also read ~/adopted-agent-instructions/my-project/ as claims to verify against the code.
```

Say it in a sentence like that rather than passing the path alone: `/survey <path>` means *survey only this part of the project*, so passing the directory on its own would survey the copies instead of the codebase.

`/survey` reads the manifests, the lockfile, the configs, the source layout, and the declared scripts, then proposes the brief with every statement marked by how well the code supports it — proven by a file, inferred from a pattern, or not determined at all. It writes nothing until you approve it, and it never runs the project's scripts to find out what they do.

Anything in the preserved instructions that the code confirms belongs in the brief. Anything the code contradicts is worth knowing too: it means the project's own documentation had already drifted, which is useful to be able to tell whoever owns it.

Keep the copies after this step. `/survey` will also point out anything in them that reads as a statement of intended behaviour rather than a fact about the stack — that belongs to Step F, not to the brief.

Read the draft back against the repository rather than accepting it. The separation of evidence from inference is the point of the skill, but an inference is still an inference, and this file is the yardstick for everything in C through G.

`/survey` also proposes the Commands rows in `AGENTS.md` — the project's real dev-server, build, production-server and lint commands, read off its manifest. Approve those alongside the brief. **This is the only step on this path that fills them.** `docs/workflow.md` Step 3 is where a greenfield project writes them, and you skipped it by already existing; Step D's `/tests` and `/ci` own only the `Test` and `Verify` rows. Left as the shipped `<command>` placeholders they produce no error — `/check` and `/try` cannot start the app, `/debug` cannot reproduce a failure, and `/implement` and `/complete` build nothing before committing, each reporting a gap and carrying on.

### What you settle yourself

Two fields `/survey` reports on but will not decide:

- **Browser support** — which browsers the project must work in
- **Accessibility standard** — which standard, at which level, it must meet

It will tell you what the project currently targets, where that is declared — a `browserslist` value, an accessibility linter in devDependencies — because that is readable evidence. What it will not do is record that as the requirement. The two are different questions, and on a client engagement the second is contractual: a project targeting `browserslist: defaults` may be owed far wider support than that, and nothing in the repository would say so.

Where they differ, `/survey` says so. That gap is worth raising with whoever owns the project before you agree what the brief records.

`/discovery` is the wrong tool here — it interviews you about a product you intend to build, not one that already exists.

---

## Step C — Take the baseline

> **Agent step.** From here on the human invokes skills and reviews what comes back, rather than editing files by hand.

```
/audit full
```

This is the deliverable the rest of the engagement is scoped from. It reviews every project-owned source, test, and configuration file through five lenses — quality, security, performance, accessibility, and tests — grades each finding `P0` to `P3`, and writes them to `context/findings.md` with stable identifiers.

On a large or unfamiliar codebase, run it by area instead of all at once (`/audit src/api security`, then `/audit src/api quality`, and so on). One pass over a whole application through five lenses is a lot to hold at once, and a focused pass reports what it did not cover.

Three things `/audit` deliberately does not do, so plan for them separately:

- **It does not scan dependencies for known vulnerabilities.** It is forbidden from running network-backed tools without your explicit approval, and from implying that reading a manifest is a real scan. Run `npm audit`, or whatever the project's ecosystem provides, alongside it.
- **It does not tell you what the application does.** Nothing in the scaffold reverse-engineers behaviour from code. If the client needs a written account of the system they already own, that is analysis you do by hand, and it is worth quoting for separately.
- **It reviews accessibility statically, not in a browser.** The accessibility lens reads markup, styles and tokens, and computes contrast where the values are readable. Focus order, screen-reader output and anything that only appears once the app is running are `/check`'s job, and `/audit` hands them over by name rather than guessing. On an engagement where accessibility is contractual, budget for that pass as well as this one.

`context/findings.md` survives a context reset; a report in the chat does not. Everything from here on is worked out of that ledger.

---

## Step D — Build the safety net, before any repair

```
/tests
/ci
```

`/tests` adds a unit test runner for the detected stack and turns the testing gate on. `/ci` defines one `Verify` command from checks that already exist — typecheck, tests, build — and the matching GitHub Actions workflow.

**Run both before Step E, not after.** A vibe-coded project characteristically has no tests, and the findings from Step C are largely invitations to refactor. Refactoring an untested application is how an audit turns into an outage, and on a client engagement it is your outage. If the findings are severe enough that some repair cannot wait, take those few first, then stop and build the net.

Test coverage of the existing behaviour is also the only thing that distinguishes a bug from a feature in a codebase with no specs. Write the test for what the code does now, then repair against it.

---

## Step E — Burn the findings down

Per finding, worst severity first:

```
/fix F-01        writes the work order from the ledger entry
/implement       builds it, one reviewed diff at a time
/audit           re-reviews the repair
/complete        archives the work order, one commit
```

`/fix` takes a finding identifier directly, so the ledger entry becomes the problem statement and `/implement` marks that finding `fixed` when the repairing step lands.

A repair does not close itself. `/implement` can only move a finding to `fixed`; only a review pass moves it to `closed`, and `/complete` refuses to finish while any `P0` or `P1` sits at `open` or `fixed`. That is the gate doing its job — the point at which a repair is done is when someone has looked at the result, not when the code changed.

Findings the client decides not to fix are `accepted`, with their reason recorded in the ledger. Only they can make that call, and the record of who made it is worth having.

---

## Step F — Specs for what you rebuild

> **Split step.** Writing the specs and promoting them to `Ready` is human work, as it is on any project. The loop that builds them is the agent's.

**A project with no specs at all is the expected starting state here**, and it stays that way for most of the codebase. Nothing in this path backfills specs for working code. You write one only when you are replacing something, and only for the thing you are replacing.

So the usual answer to "there are no specs" is: correct, and none are needed yet.

When you do write one, follow `docs/workflow.md` Steps 4 to 6 as written: a feature spec in `docs/features/`, component specs in `docs/specs/`, and design tokens if the visual layer is in scope. Promote the specs to `Ready` yourself — the tokens document carries no status and is never promoted — then run the normal loop.

### Where the intent comes from

A vibe-coded project often has no specification anywhere, but it usually has *something* — a `CLAUDE.md`, an instructions file, a long README section, a prompt someone kept. A2 set the instruction files aside for exactly this. They are the closest thing to a statement of intent the project has, and they are the right place to start a spec from.

Treat them the way `/survey` treats them, though: as claims, not as the contract. Three things are commonly true of such a file, and each changes what you write:

- **It describes behaviour that was never built.** An instruction is not evidence the code followed it.
- **It describes behaviour the code has since drifted from.** Then the spec has to record which one is correct — and that is a decision for whoever owns the project, not one to settle by reading.
- **It omits everything nobody had to say out loud.** Error states, empty states, keyboard access and focus order are routinely absent, because the assistant that built it was never asked. The spec template's sections are what surface those.

Read the code alongside the file. Where they agree, you have a specification of the existing behaviour and can decide whether to keep it. Where they disagree, you have a question worth asking before any code is written.

The rule holds here exactly as it does on a new project: **no skill moves a spec from `Draft` to `Ready`.** On a client engagement that promotion is also the moment scope is agreed, which makes it a good place to have the conversation.

Specs written now cover only what you rebuild. There is no requirement — and no value — in retrospectively specifying code you are leaving alone.

---

## Step G — Deployment readiness

`/release` reviews local deployment configuration, environment variables, and smoke-test planning for Render or Vercel. It stops before any deploy, push, or remote change unless you say yes separately in that conversation.

---

## What this path does not give you

Worth knowing before quoting the work:

| Gap | What it means in practice |
|-----|---------------------------|
| No reverse-engineering | Nothing reads the codebase and describes its behaviour. Producing that account is manual work |
| No dependency scanning | `/audit` reviews code, not the supply chain. Run the ecosystem's own scanner alongside it |
| The brief is manual | Recording the stack accurately is your reading of the codebase, and everything downstream depends on it |
| Specs only for new work | The loop does not retrospectively specify code you are not touching, by design |

None of these blocks the work. These are the parts that stay yours.
