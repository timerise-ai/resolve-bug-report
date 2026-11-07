# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

An [Agent Skill](https://agentskills.io) package: markdown only. There is no `package.json` here and nothing
in this repository executes. It teaches an agent to resolve one bug report a customer filed through an in-app
reporting bridge: intake and a sufficiency decision, read-only production evidence, root cause, fix with a
sibling sweep, fail-then-pass proof, commit, and a reply to the reporter.

Keep the two straight: the commands and code in `references/` (`gh`, `supabase db query`, SQL, the paging
helper, the guard test) run in the **app** whose report is being resolved, not in this repository.

The skill was written by the engineer who runs the reporting bridge of a multi-tenant booking product, from a
recorded session. `references/provenance.md` is the rationale layer: what the session corrected, what was kept
deliberately and what was added without having run.

## Structure

- `SKILL.md`: entry point, loaded whole on every activation, so it stays between 130 and 160 lines, the
  closing index line aside. Frontmatter `description` is the trigger surface; the body carries the flow
  diagram and the seam paragraph that points to `adaptation.md`, seven critical facts, six hard rules, the
  quick start, the reference directory and the closing index line.
- `README.md`: the human-facing front door, in the section order of the index's STANDARD.md.
- `references/intake.md`, `evidence.md`, `root-causes.md`, `fix-and-verify.md`, `reply.md`, `adaptation.md`,
  `provenance.md`: one topic each, loaded on demand.
- `evals/`: `prompts.md` holds what an operator types after installing. The skill builds no code, so each
  prompt carries the report it works on and the checks only confirm the fixture app was left intact; the notes
  carry the score against the hard rules. Every other file there is one eval run. Add a prompt rather than
  rewording one that has results.
- `.github/workflows/agent-eval.yml`: the caller of the index's reusable eval workflow, copied verbatim. Do
  not edit it, and never add a trigger on `push` or `pull_request`.

## Editing conventions

- **Code blocks name their destination on the first line**: `// lib/supabase/paginate.ts`, `-- run read only`,
  `# run in the app repository`.
- **The TypeScript blocks compile** under `strict` and `noUncheckedIndexedAccess`; check them in a scratch
  project with `vitest` and `@types/node` after editing.
- **Identifiers are shared across files**: `fetchAllRows`, `PAGE_SIZE`, `MAX_TOTAL_ROWS`, the table names in
  the examples (`issue_reports`, `issue_report_attachments`, `audit_log`, `employee_services`), the `/reply`
  prefix and the option names of the user question. Change them in all or none.
- **Keep the three tables in sync** with `references/`: the reference directory and the quick start in
  `SKILL.md`, and the file table in `README.md`. Links are relative.
- **The hard rules and the non-negotiables move together**: the six hard rules in `SKILL.md`, the six
  non-negotiables in `README.md` and the restatement in `adaptation.md`, same order, never optional.
- **The odd-looking parts stay.** Asking before a blind fix, the file-scanning guard instead of a loader test,
  the scoped read instead of paging: each has an entry under *Kept deliberately* in `provenance.md`.
- **Mark additions as additions** under *Added* in `provenance.md`.
- **The host renames the vocabulary** in the table in `adaptation.md`; the step order, the matrix and the
  rules are the authoring contract.
- **Plain punctuation, wrapped at 110 columns.** No em-dashes, en-dashes, arrows, middle dots or smart quotes.
- Commits follow Conventional Commits with no AI attribution trailers; releases follow the index's
  STANDARD.md.
