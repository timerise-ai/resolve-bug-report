# Provenance

The flow was written by the engineer who runs the reporting bridge of a multi-tenant booking product, from
three recorded sessions on tenant reports. The first two arrived as an issue with a title and no description;
the first ended in a corrected read, the second in new behaviour the user chose. The third asked for a calendar
capability the app already had, was answered with a how-to, and ended in a defect found while checking that
answer, a fix, and a correction to the reporter. Each fix ended in a reproduction on the old and new code.
This ledger has 19 entries: what the sessions corrected in the flow, what they kept that looks odd, and what
the skill adds that no session ran.

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

## Kept deliberately

### 11. Ask before fixing the likely causes

The session had three plausible hypotheses before any data, and each was cheap to change. None was the cause.
Fixing them blind would have shipped three changes and left the bug. The skill keeps the question to the user
even when a blind fix looks harmless.

### 12. A source guard rather than a loader test

The guard reads source files and matches a pattern, which looks crude next to a unit test of the loader. A
loader test needs a mocked client chain and covers one loader; the guard covers every current and future read
on the table and failed on all four defective reads at once. It stays.

### 13. One read scoped instead of paged

The assistant tool needed one service's assignments and read the whole table. It was narrowed to that service
rather than paged: fewer rows, one request, and the guard accepts a read filtered to one record.

### 14. The user chose the design of a behaviour fix

In the second session the evidence showed a payment screen working as built, one visit per receipt, and a
tenant settling two back-to-back treatments on two receipts. Four variants were put to the user: change the
screen, route to the till that already settled several visits, reply with that workaround, or carry the
missing relation through the integration. The user picked the first. The skill keeps the choice with the user
even when one variant is clearly recommended. **Shipped:** "When the cause is behaviour, not a defect" in
[intake.md](intake.md) and the two causes it confirmed in [root-causes.md](root-causes.md).

### 15. A pure module instead of a guard for new behaviour

The second fix had no repeated pattern for a source guard to hold. The rules (which visits join, which may)
went into a pure module with unit tests, and the server action calls it. A test of a module that did not exist
fails on the old code trivially, so the running app carries the fail-then-pass proof. **Shipped:** "When the
fix is new behaviour" in [fix-and-verify.md](fix-and-verify.md).

### 16. A how-to went out without reading production

In the third session the code at the reported version already had every entry point and edge the reporter
asked for, and the report was filed through the assistant from an unrelated page. The user chose a how-to
reply over a production read. The skill keeps that option first when the capability exists: the audit trail
would have told whether anyone had tried the feature, not whether it existed. **Shipped:** "When the app
already does what the reporter asks" in [intake.md](intake.md).

## Added

### 17. Not run in any recorded session

The root-cause rows for stale cache and time zone, the alternatives in the seam table (Linear, Jira, `psql` on
a replica, an admin API), the reply template for asking the reporter (every session read production or the
code instead), the "route to another screen" and "fix it upstream" variants, and the vocabulary table were
written for this skill and have not run as part of the flow.

### 18. The reply template after a behaviour change is a generalisation

The second session's reply described the new behaviour and how to opt out of it; the template in
[reply.md](reply.md) generalises that one reply and has not been used as a template yet.

### 19. The how-to and correction templates, and the loader repro, are generalisations

The how-to and correction templates in [reply.md](reply.md) each generalise one reply from the third session.
The throwaway loader test in [fix-and-verify.md](fix-and-verify.md) uses placeholder names for the host's
loader and seed helpers; the run it describes used the host's own.
