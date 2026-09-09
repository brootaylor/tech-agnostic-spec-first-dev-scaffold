---
name: survey
description: "Read an existing codebase and draft docs/project-brief.md, plus the Commands rows in AGENTS.md, from what is actually there - stack, package manager, real run commands, conventions, entry points - separating what the code proves from what it only suggests, and naming what cannot be determined from code at all. Shows the proposed drafts and writes only on explicit approval. Use when the user runs /survey, has just adopted the scaffold into a project that already has code, asks what stack a codebase uses, or needs docs/project-brief.md filled in for a project nobody has documented. Not for a new project with no code yet - that is /discovery."
---

# survey - read a codebase, draft the brief from the evidence

Where this sits in the workflow:

    an existing codebase  ->  [survey]  ->  you approve  ->  /audit
    (no brief yet)            (read it,     (the brief is    (now has a
                               draft the     yours to edit)   yardstick)
                               brief)

`docs/project-brief.md` is the single source of truth every other skill measures
against. On a project the scaffold has just been added to, it is still the shipped
placeholder - and an empty brief is worse than an obvious gap, because `/audit`
will review against it, find little, and give no sign the yardstick was blank.

This skill closes that gap the only way it can be closed honestly: by reading the
code and reporting what is there, marked by how well each claim is supported.

It is `docs/workflow.md` Step 2 for a project that skipped Steps 1 to 3 by
already existing. See `docs/adopting-an-existing-project.md`, Step B.

How it differs from its neighbors:

- `/discovery` interviews the user about a product they *intend* to build.
  `/survey` reads a codebase that already exists. Discovery asks; survey observes.
  A project with no code yet wants `/discovery`.
- `/audit` judges code against the brief. `/survey` produces the brief so
  `/audit` has something to judge against. **Survey never grades, never reports
  defects, and never opens a finding.**
- `/brief` explains one spec. Unrelated, despite the similar name.

## Input

None, normally: survey the project rooted at the working directory.

A path narrows it - `/survey src/api` - which is worth doing on a monorepo or
when only one application in the tree is in scope. State the scope either way.

**A path argument always means "survey only this".** Reference material to read
alongside the project - agent instructions set aside during an adoption, for
instance - is named in the request instead, and never narrows the scope. If an
argument is a path outside the project, say which reading you took before
proceeding rather than guessing.

## Step 1 - read the evidence

Read the project's own artifacts. Prefer `rg` and targeted reads; never dump
whole files into the response.

1. **Manifests and lockfiles** - `package.json`, `requirements.txt`,
   `pyproject.toml`, `go.mod`, `Cargo.toml`, `composer.json`, `Gemfile`, or
   whatever the ecosystem uses. The lockfile names the package manager: a
   `pnpm-lock.yaml` settles it in a way a README sentence does not.
2. **Configuration** - build, bundler, framework, TypeScript, linting, styling,
   and test configs. These are the strongest single signal of the real stack.
   Two are worth naming: a `browserslist` field or `.browserslistrc` declares
   what the project currently targets, and an accessibility linter or test tool
   (`eslint-plugin-jsx-a11y`, `axe-core`, `pa11y`) shows someone was aiming at a
   standard. Read both - and see Step 2 for what they do and do not settle.
3. **Scripts** - the actual dev, build, test, and lint commands, read off the
   manifest rather than reconstructed. These fill the Commands section of
   `AGENTS.md` - see Step 3.
4. **Source layout** - entry points, directory structure, routing convention,
   where components live, how styles are authored (plain CSS, modules, a
   preprocessor, CSS-in-JS, utility classes in markup).
5. **Runtime version** - `.nvmrc`, an `engines` field, a container image, a
   continuous integration workflow's setup step.
6. **Tests** - whether any exist at all, what runner, and roughly what they
   cover. Presence or absence, not quality; quality is `/audit`.
7. **Existing prose** - `README.md`, and an `AGENTS.md`, `CLAUDE.md` or
   `.cursor/rules` the project already had. Read these for *claims to verify*,
   never as evidence. A README describing a stack the code abandoned two
   refactors ago is a common and entirely silent failure.

   In a project the scaffold was just added to, its own `AGENTS.md` and
   `CLAUDE.md` have been replaced, and copies of those - plus whatever other
   instruction files it carried - set aside outside the repository; see
   `docs/adopting-an-existing-project.md`, A2. Any other tool's instructions,
   `.cursor/rules` among them, are still in place. Read them if the user names the
   path, and treat a disagreement between them and the code as a finding *about
   the documentation*, worth reporting to the user, not a defect in the code.

   Such a file usually mixes two kinds of statement, and they have different
   destinations. Facts about the stack, the commands and the conventions belong
   in the brief. Statements about how the software should *behave* do not -
   they are the raw material for a spec, and often the only record of intent the
   project has. Separate them, and report the behavioural ones under their own
   heading (Step 4) so the user still has them when they reach Step F. **Do not
   write a spec from them, and do not put them in the brief.**
8. **Git**, lightly - whether there is history at all, and roughly how active it
   is. A repository with one initial commit is a different object from one with
   two years of work, and it changes how much the conventions can be trusted.

## Step 2 - separate what you know from what you are guessing

Every statement the draft makes falls into one of three buckets, and keeping them
apart is this skill's whole value:

| Bucket | Means | Example |
|---|---|---|
| **Evidence** | A file proves it | `vite.config.js` plus `vite` in dependencies |
| **Inference** | The code strongly suggests it, without stating it | Component conventions read off six files that agree |
| **Unknown** | Cannot be determined from code | Who the users are; the accessibility standard |

Never promote an inference to evidence because it is probably right, and never
quietly drop an unknown because the brief has a field for it. A brief that says
"not determined - needs your answer" in three places is far more useful than one
that reads as complete and is wrong in three places.

Two fields need this separation handled with particular care, because a project
often carries evidence that looks like an answer and is not one:

- **Browser support** - a `browserslist` value is evidence of what the project
  *currently targets*. It is not evidence of what it *must* support, which is a
  commercial question code cannot answer.
- **Accessibility standard** - an accessibility linter in devDependencies shows
  an intent. It does not name a standard or a level, and passing some automated
  checks is not the same as being held to the Web Content Accessibility
  Guidelines (WCAG) 2.2 at conformance level AA.

So report what was found, then ask for the requirement, and keep the two apart:

    Browser support
      Found:    browserslist "> 0.5%, last 2 versions, not dead" (package.json)
      Needed:   not determined - what is this project actually required to
                support? The scaffold's default is a starting proposal, not
                an answer.

**Never let the found value become the recorded requirement.** A project
targeting `defaults` may be owed a much wider range by contract, and a project
targeting a wide range may have inherited it from a framework template nobody
revisited. Where the two turn out to differ, say so plainly - that gap is a
finding the user will want, and it is invisible from either side alone.

## Step 3 - draft the brief

Produce the proposed contents of `docs/project-brief.md`, plus the Commands rows
for `AGENTS.md`, without writing anything yet. In the brief, fill in the
setup-checklist sections and leave everything below them alone - that is
reference material the scaffold ships, and it is not this skill's to rewrite.

**Under "What this project is"** - describe what the application actually does,
read from its routes, entry points, and user-facing strings. Say plainly that
this is a description of the code as found, not a statement of product intent,
and that the user should correct it.

**The Stack section** - mark exactly one `[active]` per category, and give the
evidence for each on the same line. Clear the shipped default in a category
before marking the real choice; two `[active]` marks in one category is not an
error anything reports, and a leftover default can pull in a framework the
project never used.

Where a category genuinely has no answer - no end-to-end test tool, no linting -
say "none found" rather than marking anything. That is a true and useful fact.

> [!IMPORTANT]
> **Do not widen the Stack table to fit the project.** It is written for
> front-end web work, and a project may be Python, Go, Rust, or anything else.
> That is not a gap: `/tests`, `/ci`, and `/release` detect the real stack
> independently of the table. Record the actual stack in plain prose under the
> table and leave the table's own categories alone.

**The Commands section of `AGENTS.md`** - not the brief; that file has no
Commands section. This is the second and last file this skill writes, and only
those rows. Fill them with the project's real commands, exactly as the manifest
declares them, with the package manager the lockfile proves. Delete rows that do
not apply, and leave `Test` and `Verify` absent - `/tests` and `/ci` own those
two. **Never invent a command to fill a row**; a missing test command is a fact
`/tests` exists to change.

These rows are the one part of the survey the rest of the loop executes rather
than reads. `/check` and `/try` start the app from them, `/debug` reproduces a
failure with them, and `/implement` and `/complete` build from them. Left as the
shipped `<command>` placeholders they raise no error - each skill reports a gap
and carries on with less evidence than it should have - and on an adopted project
nothing else ever fills them.

Then present the draft with three lists beside it:

- what the code proves
- what was inferred, and from how many files
- what could not be determined and needs the user's answer

If the preserved instructions carried statements of intended behaviour, list
them separately under **Behavioural intent found - not part of the brief**, with
the file each came from. Say plainly that these are unverified claims about what
the software should do, that this skill has not checked them against the code,
and that they are worth keeping for whoever writes a spec later. Nothing here
becomes a spec, and nothing here goes into the brief.

End by asking for review. **Do not write in the same response that first presents
the draft.**

## Step 4 - write only after approval

Write `docs/project-brief.md`, and the Commands rows in `AGENTS.md`, only on
explicit approval. If the user asks for changes, revise and show the affected
sections again first.

If the brief already contains real content - the user filled some in, or a
previous survey ran - never replace it silently. Show what would change and why,
and preserve anything the user wrote. The scaffold's own shipped placeholders are
the one safe thing to overwrite.

After writing:

- report what changed
- list every remaining unknown, so none of them quietly become defaults
- point at the next step: `/audit full` now has a yardstick to measure against

## Rules

- **`docs/project-brief.md` and the Commands section of `AGENTS.md` are the only
  files this skill writes**, and only on explicit approval. Nothing else in
  `AGENTS.md` is this skill's to touch. Never edit source, config, specs,
  `context/` files, or `.gitignore`. Never install, build, commit, branch, or
  push.
- **Never run the project's scripts to find out what they do.** Read them. A
  build or test command in an unfamiliar repository can do anything, and this
  skill has no reason to execute it. Reading is enough to record it.
- **Never grade, never open a finding.** Noticing that something looks wrong is
  `/audit`'s job. If something genuinely alarming surfaces - a committed secret,
  for instance - say so once, plainly, and point at `/audit`; do not start
  reviewing.
- **Never trust prose over code.** An existing README or `AGENTS.md` is a claim
  to check, and a disagreement between the two is itself worth reporting.
- **Never guess browser support or the accessibility standard.** Ask.
- **Never write a spec, and never set any `**Status:**` line.** This skill does
  not touch the work queue.
- Say "not determined" as often as it is true. The value here is a brief the
  user can trust, not a brief that looks finished.

## Formatting

Format the output to match the project's conventions in `AGENTS.md`: concise,
scannable markdown, with lists for enumerations and tables for matrices rather
than dense paragraphs. Lead with the survey findings; the proposed file contents
come after them.
