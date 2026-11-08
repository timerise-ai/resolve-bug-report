# Root causes: naming the cause the data distinguishes

A report names a symptom. Several causes produce the same symptom, and each leaves a different trace in the
data. Write the candidates down with the check that tells them apart, run the checks, and keep the one the
data confirms. Do not fix the first plausible candidate.

## The table

The first row, the two rows after the filter row, the null-owner row and the rows after it up to the cache
row are causes the flow confirmed on real reports. The rows between them were competing hypotheses in the
first run, each eliminated by the check in its row; the last two are additions, marked as such.

| Cause | Symptom | Check that confirms it | Check that kills it |
|---|---|---|---|
| Capped read | A list is wrong only for a large tenant: too short, or too wide where empty means everyone; in name order, only the first letters; the newest rows missing, totals too low | Rows for the tenant on the table exceed the API cap, and the loader reads it in one request without `range` (a `.limit()` with no order, sorted afterwards, keeps an arbitrary subset) | Row count under the cap, or the loader pages |
| Oversized id filter | Columns derived from a second read (who is credited, a document number, a cost) are blank on long lists only | The second read sends every id of the first in one `.in(...)`, which past a few hundred ids outgrows the request line or the cap | The ids go in chunks, each paged |
| Sync overwrites manual edits | A setting the reporter changed reverts or never takes effect | The audit trail shows a sync job writing the same rows after the manual write | No sync write after the manual one |
| Narrower rule takes precedence | The broad setting is right, the screen still shows otherwise | A narrower scope (a variant, a location, a per-member override) has rows of its own | The narrower scope has no rows for the record |
| Filter hides the record | The configured person or item never appears | The record fails a filter the screen applies: archived, inactive, lacking a role or function | The record passes every filter |
| One action per record, the tenant thinks in groups | Several things that belong together are handled one by one: separate receipts, separate messages | The action's code takes one id; the timeline shows the same write repeated for records of one client or day | The action already takes a group, and the group was set |
| An integration drops a relation | Records that belong together arrive unrelated; grouping logic never fires | The rows from the integration have an empty relation column (a group id) that native rows fill | The relation is filled on those rows |
| A record for everyone is skipped by an owner match | One screen shows an entry that applies to all (a closure, a default, an unassigned slot), another path acts as if it were not there | The record's owner columns are null, and the other path matches `owner = <id>`, which null never equals | The path handles the null owner, or the record has an owner |
| A view filter hides the container with what it holds | "X is not among the people, so I cannot settle Y": records assigned to someone vanish from a grid | A default-on filter on columns (people by job function, rooms by kind) drops a column without asking whether it holds records that day | The column stays whenever it holds a record |
| A control ignores the screen's state | An export, total or print does not match what the screen shows | The control sends none of the screen's period or filter, and the server falls back to its own default window | The control passes the state and the server honours it for that dataset |
| The account has the wrong role | "I am at reception and cannot settle visits"; several reports from one member in minutes | The role in the bridge metadata (read when the issue was filed) lacks the permission, and the member's audit trail shows the admin changing it afterwards | The role holds the permission, and no override takes it away |
| A per-member switch takes a permission away | An admin asks for access the role already gives | A revoke in the member's overrides, set by the admin, often after the report; the label of the permission does not name what it covers | No override on the permission |
| A per-member grant the data layer refuses | The buttons are there, every save fails | The permission matrix lets an override grant it, the write goes through the user's session, and the table's policy enumerates roles without this one | The write path uses a service client after the guard, or the policy is tenant-wide |
| A refusal shown as a generic failure | The error screen says only "could not move the visit" and gives a reference | The guard throws an error class the action wrapper does not treat as meant for the user, so it returns the fallback and logs a crash under the reference | The wrapper returns the guard's message |
| A write control shown to a role without it | A member can drag, edit or confirm, and every save fails | The screen takes no prop for the write permission, while each action behind it checks it | The screen hides the controls for that role |
| Stale cache (added) | A change shows after a reload or a deploy, not before | The screen's data comes from a cache the write does not invalidate | The screen reads fresh on every request |
| Time zone (added) | Wrong only near midnight, or for one tenant in another zone | The stored instant is right and the rendered day is not | Times are right at noon |

## The capped read, in detail

PostgREST, which Supabase's REST API is, answers any request with at most `max_rows` rows, 1000 by default,
and returns them with status 200 and no error. A loader that reads a whole tenant's table in one request
works until the tenant crosses the cap, then reads a prefix of it.

Two things make it hard to see:

- **The prefix is not "the oldest rows".** Under row level security the planner often answers with an index
  scan on the tenant and another column, so which rows survive depends on that index, not on insertion order.
  The same data can be complete for one record and empty for another.
- **A fallback amplifies it.** Where "no rows for this record" means "no restriction", a record whose rows
  all fell past the cap shows everyone. Where it means "nobody", it shows no one. Either looks like a
  configuration error to the tenant.

Confirm it with three facts: the tenant's row count on the table, the loader's query (one request, no
`range`, no filter that keeps it under the cap), and, ideally, the record from the report missing from the
first page under the reporter's role.

```sql
-- run read only, or locally with the reporter's claims: is the record in the first page?
begin;
set local role authenticated;
select set_config('request.jwt.claims', '{"sub":"<auth user id>","role":"authenticated"}', true);
select count(*) filter (where service_id = '<record id>')
from (select service_id from employee_services limit 1000) first_page;
explain select service_id, employee_id, variant_id from employee_services limit 1000;
rollback;
```

The `explain` shows which index orders the page, which is what you need to build a local reproduction in
[fix-and-verify.md](fix-and-verify.md).

## Signs that point to a cause before any query

| In the report | Look first at |
|---|---|
| "wrong", "not all", "everyone" in a list | Capped read, then filters |
| "I changed it and it is still wrong" | Narrower rule, sync overwrite, cache |
| "since yesterday" | Commits to the loader at the app version, recent data jobs |
| Only one tenant reports it | Scale (capped read), that tenant's integrations |
| A date or hour off by one | Time zone |
| "separately instead of together" | One action per record, then the integration's relations |
| "the whole salon", "for everyone", "nobody should be able to" | Records with no owner, and every path that matches by owner |
| "only names starting with A", "cannot pick anyone else" | Capped read over a list ordered by name, then every picker on that table |
| "X is missing from the people, so I cannot do Y" | Default-on view filters that hide a column, before the person's own settings |
| "the export does not match", "the file has other dates" | The control's request: does it carry the screen's period and filters |
| "I am <role> and cannot <action>"; a role in the metadata that contradicts the title | The member's role and overrides at filing and now, before any code |
| Filed from the error screen, with a generic message and a reference | The guard on that action, then the server log under the reference |
| A manager asks for access a role already has | Per-member revokes on the cards of that role, and the permission's label |
| "where can I see X", and the screen exists | The tenant's own count of X on that screen |

## The record nobody owns, in detail

A tenant-wide entry is often stored as a row whose owner columns (person, room) are null: a closure for the
whole business, a default, a hold. The screen that draws it fans it across every column, so it looks right.
Every other path that asks "is this person busy" matches on the owner and silently skips it: the public
booking engine, the overlap check, a cart that books several people at once, an assistant's free-slot tool.

Sweep for it by the question, not the table: every function that decides availability or conflict, and
whether it handles a row with no owner. Two traps:

- **A database exclusion constraint does not cover it.** An overlap constraint keyed on the owner treats null
  as distinct from everything, so the row collides with nothing.
- **Not every ownerless row means everyone.** An unassigned visit is waiting for an owner, not closing the
  business. Name the kinds that mean "everyone" in one pure function and test both sides of it.

## Permissions in two layers, in detail

Many apps check a permission twice: a matrix in application code that drives menus and guards, and row level
security that enumerates roles. A per-member override (a grant or a revoke stored on the member) lives only in
the first. SQL cannot read it, so a grant to a role the policy does not list passes the matrix and fails the
write. The member sees the button, presses it, and gets whatever the action says about an unexpected error.

Three places to read, in this order:

| Layer | What to read | The defect it hides |
|---|---|---|
| The permission matrix and its override list | Which permissions an override may grant, and to whom | A grant the data layer will refuse |
| The live policies ([evidence.md](evidence.md)) | Which roles each table's insert, update and select admit | The list the matrix must agree with |
| Each write path | User session or service client after the guard | A service client makes the grant work; the user session makes it bounce |

Two siblings come with it. A guard that checks the role alone ignores overrides both ways: a grant never
works there, and a revoke never holds. A wrapper that folds the guard's error into the action's fallback hides
which permission was missing, so every such refusal reads like a crash.

## Checklist

- [ ] Two or more candidates written down, each with its confirming and killing check
- [ ] Every check run; the cause kept is the one the data confirms
- [ ] For a capped read: row count, the loader's query, and the record's absence from the first page
- [ ] For a record nobody owns: every availability and conflict path checked for the null owner
- [ ] For a permission report: the role and overrides read, the matrix checked against the live policies,
      and each write path's client named
- [ ] If the cause is behaviour the app was built with: the user asked to pick a variant
      ([intake.md](intake.md))
