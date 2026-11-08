# Changelog

All notable changes to this skill are documented here. The format follows
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and the skill uses
[Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.2.0] - 2026-11-08

The flow now answers every kind of report the bridge delivers: a defect, behaviour the app was built with, a
capability it already has, a cause in the reporter's account, and a feature request, built or noted for later,
each with its reply template and a claim check before anything is posted.

### Added

- A report that asks for something the app already has: the entry point, edge and record-variant checks in
  `references/intake.md`, and the how-to reply template.
- The claim check before any reply and the template for correcting a reply already sent, in
  `references/reply.md`; the reply hard rule now requires every claim to be checked against the code.
- The record nobody owns as a confirmed cause in `references/root-causes.md`: a tenant-wide row skipped by
  every path that matches busy time by owner.
- Proving a rule below the screen in `references/fix-and-verify.md`: a throwaway loader test, a database
  function compared in a rolled-back transaction, stashing only the fixed paths, and the shell-variable trap.
- A report that asks for a feature the app does not have: confirm the absence, list every reader of the field,
  read the host's commercial terms and put the variants to the user, in `references/intake.md`; the template
  for a feature not built now, the claim-check row for a trade-off the user names, and reading earlier replies
  for the house tone, in `references/reply.md`; the commercial-terms seam in `references/adaptation.md`.
- Provenance entries 8 to 10, 31 and 48 from a third recorded session, entries 11, 32 and 49 from a fourth,
  entries 12 to 14 and 33 to 37 from a fifth, and entries 15 to 20, 38 to 41 and 50 from a sixth.
- A how-to the user proposes is checked like any other, and the capability must let the reporter finish the
  task they named, in `references/intake.md`; reading the screen a request names for a defect fixable under a
  freeze.
- Two confirmed causes in `references/root-causes.md`: a default-on view filter that hides a column with the
  records it holds, and a control (an export) that ignores the screen's period; the capped read's name-order
  signature and three new report signs.
- In `references/fix-and-verify.md`: a cap written on purpose and a look-alike control on a different field in
  the sweep table, the busy-table guard (comments stripped, wider window, narrowing shapes, a line-keyed set,
  one shared helper), changing tests that encoded the old behaviour, seeding a name-ordered list and a
  visibility fix, reading a download in two steps, restoring updated seed rows, `Refs #N` for a report that
  stays open, and asking before merging into the default branch.
- Three claim-check rows in `references/reply.md`: a label collapsed at the reporter's viewport, a toggle that
  is not saved, and a view that shows the record but cannot act on it.
- A report caused by the account's setup: the role in the bridge metadata read against the member's record
  now and its writes after the report, several reports from one member read as one timeline and answered one
  by one, and a capability with no records for the tenant, in `references/intake.md` and
  `references/evidence.md`; reading the live row level security policies.
- Permissions in two layers in `references/root-causes.md`: the wrong role, a per-member revoke, a grant the
  data layer refuses, a refusal shown as a generic failure, a write control shown to a role without it; the
  unordered capped read and the oversized id filter; four new report signs.
- In `references/fix-and-verify.md`: three sweep rows, `fetchAllByIds` for second reads by id, the three
  permission fixes with the policy probe, the read-only entry-point list and the two-role run, a shell
  function instead of a command in a variable, paths written out for `git stash push`, and listing migrations
  before a merge.
- The template for a cause in the account's setup, naming the tenant's own members to an admin, and four
  claim-check rows in `references/reply.md`.
- From a seventh recorded session, in `references/intake.md`: opening an attachment behind a login in the
  user's signed-in browser and reading the marks drawn on it; where the capability sits at the reporter's
  viewport as a how-to check; a variant the user changes mid-work replacing the old one everywhere; reading
  every path that would carry a requested field for a defect it already has; the full build's product
  questions asked in one batch before the migration.
- In `references/fix-and-verify.md`: printing a runtime value when the running app disagrees with a passing
  test (a dev server holding a stale module); proving a data migration that rewrites rows in a rolled-back
  transaction, then applying it without a reset, and the seed's changed section when a reset is refused; the
  whole suite before every push, and a failure that also fails on the old code named, not fixed.
- The template after a requested feature is built, its claim-check row and checklist line, in
  `references/reply.md`; critical fact 8 notes that the readers' map often finds a defect.
- Provenance entries 21 to 25, 42 to 45 and 51 from a seventh recorded session, renumbering the ledger to 51
  entries.

### Changed

- A single feature request filed through the bridge now stays in the flow up to the reply; only grooming a
  roadmap of requests is sent to the backlog in "When NOT to use".

## [0.1.0] - 2026-11-07

First release: the flow that takes one tenant bug report from a tracker issue to a fix and a reply, for a
corrected defect and for new behaviour the user chooses.

### Added

- `SKILL.md` with the flow diagram, eight critical facts, six hard rules and the reference directory.
- `references/intake.md`, `evidence.md`, `root-causes.md`, `fix-and-verify.md`, `reply.md`, `adaptation.md`
  and `provenance.md`.
- `evals/prompts.md` and the agent eval workflow caller.
- Behaviour causes: the product-decision step with its four variants, the "one action per record" and
  "integration drops a relation" causes, the pure-module pattern for new behaviour, the seed and cleanup
  traps of the repro, and the reply template after a behaviour change.
