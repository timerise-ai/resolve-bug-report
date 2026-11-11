# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

An [Agent Skill](https://agentskills.io) package: markdown only. There is no `package.json` here and nothing
in this repository executes. It teaches an agent to resolve one bug report a customer filed through an in-app
reporting bridge: intake and a sufficiency decision, read-only production evidence, root cause, fix with a
sibling sweep, fail-then-pass proof, commit, and a reply to the reporter.

Keep the two straight: the commands and code in `references/` (`gh`, `supabase db query`, SQL, the paging
helper, the guard test) run in the **app** whose report is being resolved, not in this repository.

The skill was written by the engineer who runs the reporting bridge of a multi-tenant booking product, from
ten recorded sessions. `references/provenance.md` is the rationale layer: what the sessions corrected, what
was kept deliberately and what was added without having run.

## Structure

- `SKILL.md`: entry point, loaded whole on every activation, so it stays between 130 and 160 lines, the
  closing index line aside. Frontmatter `description` is the trigger surface; the body carries the flow
  diagram and the seam paragraph that points to `adaptation.md`, eight critical facts, six hard rules, the
  quick start, the reference directory and the closing index line.
- `README.md`: the human-facing front door, in the section order of the index's STANDARD.md. It carries the
  "current release is" line that `bumpv` updates with `CHANGELOG.md`.
- `references/intake.md`, `evidence.md`, `root-causes.md`, `fix-and-verify.md`, `reply.md`, `adaptation.md`,
  `provenance.md`: one topic each, loaded on demand. The five flow files end with a `## Checklist`.
- `references/provenance.md`: three sections, *Fixed in the flow*, *Kept deliberately* and *Added*, with one
  numbering that runs through all three. `CHANGELOG.md` cites entries by number, so a new entry goes at the
  end of its section and a renumbering is named in the changelog.
- `evals/`: `prompts.md` holds what an operator types after installing. The skill builds no code, so each
  prompt carries the report it works on and the checks only confirm the fixture app was left intact; the notes
  carry the score against the hard rules. Every other file there is one eval run, committed to `main` by the
  workflow, so pull before pushing. Prompts are addressed by position: add a prompt at the end rather than
  rewording or reordering ones that have results.
- `.github/workflows/agent-eval.yml`: the caller of the index's reusable eval workflow, copied verbatim. Do
  not edit it, and never add a trigger on `push` or `pull_request`. It runs prompt 1 on every published
  release across all three agents.

## Editing conventions

- **Code blocks name their destination on the first line**: `// lib/supabase/paginate.ts`, `-- run read only`,
  `# run in the app repository`; a reply template's first line is the `/reply` prefix itself.
- **The TypeScript blocks compile** under `strict` and `noUncheckedIndexedAccess`. They have no compiler here:
  paste the blocks in `references/fix-and-verify.md` into a scratch project with `vitest` and `@types/node`
  after editing them.
- **These checks stand in for build and test** before a commit or a release:

  ```bash
  # SKILL.md body stays between 130 and 160 lines (the closing index line aside)
  wc -l SKILL.md

  # Lines over 110 columns; table rows and the README badge lines are the only allowed hits
  awk 'length > 110 {print FILENAME":"FNR": "length}' SKILL.md README.md CLAUDE.md CHANGELOG.md \
    references/*.md evals/*.md

  # Typographic punctuation; must print nothing
  grep -nP '[\x{2013}\x{2014}\x{2192}\x{00B7}\x{2018}\x{2019}\x{201C}\x{201D}]' SKILL.md README.md \
    CLAUDE.md CHANGELOG.md references/*.md evals/*.md

  # Run one eval by hand; the prompt number is its position in evals/prompts.md
  gh workflow run agent-eval.yml -f agent=claude-code -f prompt=1
  ```
- **Identifiers are shared across files**: `fetchAllRows`, `fetchAllByIds`, `PAGE_SIZE`, `ID_CHUNK`,
  `MAX_TOTAL_ROWS`, the table names in the examples (`issue_reports`, `issue_report_attachments`, `audit_log`,
  `employee_services`), the `/reply` prefix and the option names of the user question. Change them in all or
  none.
- **Keep the three tables in sync** with `references/`: the reference directory and the quick start in
  `SKILL.md`, and the file table in `README.md`. Links are relative. A new section in a reference also gets
  its trigger keywords added to that file's row in the reference directory.
- **The hard rules and the non-negotiables move together**: the six hard rules in `SKILL.md`, the six
  non-negotiables in `README.md` and the restatement in `adaptation.md`, same order, never optional.
- **The odd-looking parts stay.** Asking before a blind fix, the file-scanning guard instead of a loader test,
  the scoped read instead of paging: each has an entry under *Kept deliberately* in `provenance.md`.
- **Mark additions as additions** under *Added* in `provenance.md`: anything no recorded session ran.
- **The host renames the vocabulary** in the table in `adaptation.md`; the step order, the matrix and the
  rules are the authoring contract.
- **Plain punctuation, wrapped at 110 columns.** No em-dashes, en-dashes, arrows, middle dots or smart quotes.
  Table rows and the README badge lines are the only lines allowed past 110.
- **Changes land under `[Unreleased]` in `CHANGELOG.md`** in Keep a Changelog form; `bumpv` turns that into
  the release section.
- **Evals are not skill content.** A new prompt or an eval result is committed as `chore(evals): ...`, never
  causes a bump and never rides in the release commit; a result's frontmatter is never edited.
- Commits follow Conventional Commits with no AI attribution trailers. Commit with the `commit` skill and
  release with `bumpv`; both follow the index's STANDARD.md.
