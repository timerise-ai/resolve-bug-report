# Provenance

The flow was written by the engineer who runs the reporting bridge of a multi-tenant booking product, from
five recorded sessions on tenant reports. The first two arrived as an issue with a title and no description;
the first ended in a corrected read, the second in new behaviour the user chose. The third asked for a
calendar capability the app already had, was answered with a how-to, and ended in a defect found while
checking that answer, a fix, and a correction to the reporter. The fourth asked for a feature the app did not
have and was answered that it was noted for after a feature freeze, with a trade-off the user named. The fifth
resolved three reports in a row: a request noted for later with a defect fixed inside it, a picker cut off by
a capped read in name order, and a how-to the user proposed that the code showed could not finish the task.
Each fix ended in a reproduction on the old and new code. This ledger has 30 entries: what the sessions
corrected in the flow, what they kept that looks odd, and what the skill adds that no session ran.

## Fixed in the flow

### 1. The description was treated as the whole report

The first reading found an empty body and stopped at "not enough information". The bridge metadata (page,
timestamp, tenant) and the audit trail then named the screen, the sequence and the record class.
**Shipped:** the metadata table and the sufficiency matrix in [intake.md](intake.md), and the timeline query
in [evidence.md](evidence.md).

### 2. Production was read without asking

A first attempt to read production was refused by the agent's permission layer, and so was an attempt to
fetch a service key into a scratch file. Asking the user with three options resolved it in one turn.
**Shipped:** the "ask the user" step in [intake.md](intake.md) and the protocol table in
[evidence.md](evidence.md).

### 3. Column names were guessed

Three queries failed on columns that did not exist (`body` was `body_md`, `entity_type` was `entity`, a
soft-delete column existed on one table and not another). **Shipped:** introspection first, `to_jsonb(row)`,
in [evidence.md](evidence.md).

### 4. The actor filter compared the wrong ids

The audit trail's `actor_id` is the auth user id; comparing it with the reporter's member id returned false
for every row. **Shipped:** the join through the member table's user id in [evidence.md](evidence.md).

### 5. The first reproduction passed on the old code

The local reproduction showed the right list on both the old and the new code. The read under row level
security followed an index, the seeded test row sorted into the first page, and the cap never cut it. Moving
the test row to the member with the largest id made the old code show everyone and the new code one member.
**Shipped:** step 2 and the "if it shows the right one" rule in [fix-and-verify.md](fix-and-verify.md), and
the first-page check in [root-causes.md](root-causes.md).

### 6. A seed insert failed and the comparison still ran

In the second session a seeded booking was refused by the overlap constraint on one member's bookings. The old
and new code then showed the same screen, which looked like a failed fix. Reading the insert's output and
seeding a free slot gave the real comparison. **Shipped:** "check that every seed insert succeeded" in
[fix-and-verify.md](fix-and-verify.md).

### 7. A submitted repro left rows in many tables

Proving one receipt with two lines meant submitting on the new code, which wrote the sale, its lines, payment
splits, journal rows, a fiscal job and commissions. **Shipped:** prefer showing over submitting on the old
code, and list the referencing tables by column name before deleting, in
[fix-and-verify.md](fix-and-verify.md).

### 8. A reply asserted what nobody had read

In the third session the first reply went out naming two things the code had not been read for: that clicking
an employee's name opens the column menu (it opens the profile; the menu is the arrow beside it), and that a
blocked time cannot be booked online (true for a block on one person, not for one on the whole business). A
second reply had to correct both. **Shipped:** the claim check and the correction template in
[reply.md](reply.md), and "with every claim checked against the code" in the reply hard rule.

### 9. Checking a claim found the defect

Tracing "nobody can book this time" through every booking path showed the public booking engine and the
console's overlap check matching busy time by owner, so a block owned by nobody, drawn across every column,
closed nothing. The fix named the kinds that mean "everyone" in one pure function and covered the single and
the multi-service booking paths and the database function together. **Shipped:** the null-owner row and its
section in [root-causes.md](root-causes.md), and the record-variant check in [intake.md](intake.md).

### 10. A seed command in a shell variable never ran

The running-app comparison first showed the same slot list on the old and the new code. The database command
had been kept in a shell variable, which zsh does not split into words, so the seed insert never executed.
Calling the tool directly gave the real comparison. **Shipped:** the trap and the loader and database-function
repros in [fix-and-verify.md](fix-and-verify.md).

### 11. A feature request was routed out of the flow

The flow sent a pure feature request to the backlog. The fourth report asked whether one service could sit in
several categories; nothing had gone wrong, but the reporter had filed it through the bridge and was waiting
for an answer. The session confirmed the single-choice field at the reported version, listed every reader of
the category (public catalog, commission rules, a price-list screen, the integration sync), read the
operator's note of a feature freeze and a quote-first clause, and put four variants to the user, who chose a
reply that it was noted for later. **Shipped:** "When the report asks for a feature the app does not have" in
[intake.md](intake.md), the template in [reply.md](reply.md), the commercial-terms seam in
[adaptation.md](adaptation.md), and the narrowed "When NOT to use" line in `SKILL.md`.

### 12. The how-to the user proposed could not finish the task

In the fifth session the user asked for a reply explaining where the list mode was, for a receptionist who
could not settle sauna visits. The list showed the visits, but its rows only linked to the client, the service
and the employee, so nothing could be settled from it. Tracing why the visits were out of reach found a
default-on filter that hid every non-treatment person's column even on days they held visits. The user chose
the fix with a how-to. **Shipped:** the task check and the proposed how-to paragraph in
[intake.md](intake.md), the view-filter row in [root-causes.md](root-causes.md), and the "you can do it in
view X" claim row in [reply.md](reply.md).

### 13. A download read through the page came back empty

The export repro waited for the page's response to the export route and read its body, which was empty: the
page had consumed it to save the file. An empty file looked like a filter that matched nothing. Fetching the
same URL with the page's request context returned the rows. **Shipped:** the two-step download read in
[fix-and-verify.md](fix-and-verify.md).

### 14. A comment cut the guard's statement short

A guard for a busy table split each read at its first `;` to keep a chain apart from the next statement. A
comment inside one chain held a `;`, the statement ended before its narrowing filter, and a single-card lookup
was flagged. Stripping comments first, a wider window and a line-keyed set for the one read whose filter sat
in a variable fixed it. **Shipped:** the busy-table notes in [fix-and-verify.md](fix-and-verify.md).

## Kept deliberately

### 15. Ask before fixing the likely causes

The session had three plausible hypotheses before any data, and each was cheap to change. None was the cause.
Fixing them blind would have shipped three changes and left the bug. The skill keeps the question to the user
even when a blind fix looks harmless.

### 16. A source guard rather than a loader test

The guard reads source files and matches a pattern, which looks crude next to a unit test of the loader. A
loader test needs a mocked client chain and covers one loader; the guard covers every current and future read
on the table and failed on all four defective reads at once. It stays.

### 17. One read scoped instead of paged

The assistant tool needed one service's assignments and read the whole table. It was narrowed to that service
rather than paged: fewer rows, one request, and the guard accepts a read filtered to one record.

### 18. The user chose the design of a behaviour fix

In the second session the evidence showed a payment screen working as built, one visit per receipt, and a
tenant settling two back-to-back treatments on two receipts. Four variants were put to the user: change the
screen, route to the till that already settled several visits, reply with that workaround, or carry the
missing relation through the integration. The user picked the first. The skill keeps the choice with the user
even when one variant is clearly recommended. **Shipped:** "When the cause is behaviour, not a defect" in
[intake.md](intake.md) and the two causes it confirmed in [root-causes.md](root-causes.md).

### 19. A pure module instead of a guard for new behaviour

The second fix had no repeated pattern for a source guard to hold. The rules (which visits join, which may)
went into a pure module with unit tests, and the server action calls it. A test of a module that did not exist
fails on the old code trivially, so the running app carries the fail-then-pass proof. **Shipped:** "When the
fix is new behaviour" in [fix-and-verify.md](fix-and-verify.md).

### 20. A how-to went out without reading production

In the third session the code at the reported version already had every entry point and edge the reporter
asked for, and the report was filed through the assistant from an unrelated page. The user chose a how-to
reply over a production read. The skill keeps that option first when the capability exists: the audit trail
would have told whether anyone had tried the feature, not whether it existed. **Shipped:** "When the app
already does what the reporter asks" in [intake.md](intake.md).

### 21. The user's trade-off went out as the user's

While the reply was being drafted the user added that the feature would mean removing the commission rules
set per category. The code cannot prove a product decision. The reply carried it in the words of the screen,
with the rule type's label checked in the source, and the report back to the user said the reasoning was
theirs. The skill keeps that split rather than tracing a consequence the code does not decide.
**Shipped:** the claim-check row in [reply.md](reply.md).

### 22. A defect inside a request was fixed under `Refs`

A request for finer date filters in the reports was noted for after the freeze. Reading its screen found the
export sending none of the period on screen, so most datasets exported the last 30 days whatever month was
shown. The user chose to fix that and answer the request as noted for later; the commit said `Refs #N`, the
issue was labelled and stayed open. **Shipped:** the screen-defect step in [intake.md](intake.md), the
control-ignores-state row in [root-causes.md](root-causes.md), and the `Refs #N` line in
[fix-and-verify.md](fix-and-verify.md).

### 23. A sibling that looked the same was left out

The vouchers list had a date filter and an export that ignored it too. The list filtered by expiry date and
the export bounded the issue date, so passing the filter through would have exported a different set than the
one on screen. It stayed out of the change and the user was asked which date should win. **Shipped:** the
different-field row in the sweep table in [fix-and-verify.md](fix-and-verify.md).

### 24. A cap written on purpose was treated as the defect

One picker read clients with `.limit(500)` in name order, with a comment calling it a convenience. Cut in name
order, it hid exactly the people the screen was for, so it was paged with the rest and the comment rewritten.
**Shipped:** the cap-on-purpose row in the sweep table in [fix-and-verify.md](fix-and-verify.md).

### 25. Tests that encoded the old behaviour were rewritten

The column fix made two existing grid tests fail: their fixture had a non-treatment person holding a visit and
expected the column hidden. Their expectations were changed, not the fixture, and the commit and the report
named them. **Shipped:** the paragraph and the checklist line in [fix-and-verify.md](fix-and-verify.md).

### 26. A bare "push" was asked about before merging

After both fixes were pushed to the preview branch the user typed "push". Nothing was left to push; the only
step left was merging into the default branch, which deploys to production. The session listed the two commits
it would ship and asked; the user then asked for the merge. **Shipped:** the merge line in
[fix-and-verify.md](fix-and-verify.md).

## Added

### 27. Not run in any recorded session

The root-cause rows for stale cache and time zone, the alternatives in the seam table (Linear, Jira, `psql` on
a replica, an admin API), the reply template for asking the reporter (every session read production or the
code instead), the "route to another screen" and "fix it upstream" variants, and the vocabulary table were
written for this skill and have not run as part of the flow.

### 28. The reply template after a behaviour change is a generalisation

The second session's reply described the new behaviour and how to opt out of it; the template in
[reply.md](reply.md) generalises that one reply and has not been used as a template yet.

### 29. The how-to and correction templates, and the loader repro, are generalisations

The how-to and correction templates in [reply.md](reply.md) each generalise one reply from the third session.
The throwaway loader test in [fix-and-verify.md](fix-and-verify.md) uses placeholder names for the host's
loader and seed helpers; the run it describes used the host's own.

### 30. The feature-request variants and template are a generalisation

The template "when the feature is not built now" in [reply.md](reply.md) generalises one reply from the fourth
session. The narrow and full build variants in [intake.md](intake.md) were put to the user and not built; the
commercial-terms seam has values from one host only. Reading earlier replies for the house tone ran once.
