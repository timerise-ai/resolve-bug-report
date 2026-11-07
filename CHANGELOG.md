# Changelog

All notable changes to this skill are documented here. The format follows
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and the skill uses
[Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added

- A report that asks for something the app already has: the entry point, edge and record-variant checks in
  `references/intake.md`, and the how-to reply template.
- The claim check before any reply and the template for correcting a reply already sent, in
  `references/reply.md`; the reply hard rule now requires every claim to be checked against the code.
- The record nobody owns as a confirmed cause in `references/root-causes.md`: a tenant-wide row skipped by
  every path that matches busy time by owner.
- Proving a rule below the screen in `references/fix-and-verify.md`: a throwaway loader test, a database
  function compared in a rolled-back transaction, stashing only the fixed paths, and the shell-variable trap.
- Provenance entries 8 to 10, 16 and 19 from a third recorded session, and a fourth eval prompt.

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
