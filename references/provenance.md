# Provenance

The flow was written by the engineer who runs the reporting bridge of a multi-tenant booking product, from
ten recorded sessions on tenant reports. The first two arrived as an issue with a title and no description;
the first ended in a corrected read, the second in new behaviour the user chose. The third asked for a
calendar capability the app already had, was answered with a how-to, and ended in a defect found while
checking that answer, a fix, and a correction to the reporter. The fourth asked for a feature the app did not
have and was answered that it was noted for after a feature freeze, with a trade-off the user named. The fifth
resolved three reports in a row: a request noted for later with a defect fixed inside it, a picker cut off by
a capped read in name order, and a how-to the user proposed that the code showed could not finish the task.
The sixth resolved four reports from one tenant that shared one cause in the reporting members' accounts, and
found a defect in the permission system behind each. The seventh resolved three requests from one member: a
capability below the fold, a screen split into tabs, and a feature built in full during a freeze, whose map of
readers found commission never paid on add-ons. The eighth reviewed a request a teammate had built before
anyone answered the reporter: its export would have come back through calendar sync as a foreign booking,
and the code it walked through had capped reads of its own. The ninth answered a report the product's own
tester filed by hand, with no bridge: two defects on one screen, a plan limit shown as "try again" and an id
shown where a name belonged, settled from the platform's request log, merged, and answered with a memo that
left the issue open for the tester. The tenth answered five requests from one member: a reply a teammate
had already given was checked and left to wait, three features were built from what the timeline showed the
member doing by hand, and the readers of one of them found a row that had never shown anything. Each fix ended
in a reproduction on the old and new code.
This ledger has 77 entries: what the sessions corrected in the flow, what they kept that looks odd, and what
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

### 15. The role in the metadata was the cause

In the sixth session a member filed "I am at reception and cannot settle visits" with no description. The
bridge metadata gave their role as basic staff, and the member's audit trail showed the tenant's admin
switching it to reception twenty minutes after the report. Nothing needed fixing for them; the flow had no
place for a cause in the account's setup. The same member filed two more reports in the same five minutes, and
the admin a fourth, all from that one setup. **Shipped:** the role note in the metadata table and "When the
cause is the account's setup" in [intake.md](intake.md), the member reads in [evidence.md](evidence.md), the
setup rows in [root-causes.md](root-causes.md), and the template in [reply.md](reply.md).

### 16. A permission refusal read as a crash

One report came from the app's error screen: "could not move the visit" and a reference. The move needed a
permission the member lacked, the guard threw an error class the action wrapper did not hand back, and the
wrapper answered with its generic fallback and logged a crash. Booking a visit was worse: it skipped the app's
check and returned an unrelated refusal from the database. **Shipped:** the error-screen source in
[intake.md](intake.md), the refusal row in [root-causes.md](root-causes.md), and the refusal fix with its unit
test in [fix-and-verify.md](fix-and-verify.md).

### 17. Switching drag off removed the click

The read-only calendar first reused the touch-device flag to stop tiles from dragging. On a desktop the click
that opened a tile travelled through the drag handler, so tiles stopped opening at all. Code reading and the
tests missed it; driving the app as the role found it, and the run as a role with the permission confirmed
nothing else moved. **Shipped:** the two-role run in [fix-and-verify.md](fix-and-verify.md).

### 18. A list of paths in one variable

Stashing the old UI for the comparison used a variable holding seven paths. Zsh passed them as one path, git
refused it, and nothing was stashed; the run stopped only because git rejected the path. A tolerant command
would have run the "old" side on the new code. **Shipped:** the extended shell trap in
[fix-and-verify.md](fix-and-verify.md).

### 19. A how-to would have pointed at an empty screen

A manager asked where to see the history of retail sales with seller and commission. The screen existed with
every column they named. The tenant's data had no product sale at all: products were sold in another system
whose sync carries stock levels, not sales. A plain how-to would have sent them to an empty list. **Shipped:**
the tenant-records check in [intake.md](intake.md) and the count before a how-to in
[evidence.md](evidence.md).

### 20. A grant the database refuses passed the matrix

Checking the first report's setup found per-member grants the app's matrix offered and row level security
refused: the policies enumerate roles and cannot read the overrides column. Two receptionists had been handed
the task board that way; every save would have failed. Guards that checked the role alone ignored overrides
in both directions on seven more paths. **Shipped:** "Permissions in two layers" in
[root-causes.md](root-causes.md), the policy read in [evidence.md](evidence.md), and the policy probe in
[fix-and-verify.md](fix-and-verify.md).

### 21. An attachment behind a login was nearly left unread

The first report's attachment was a link that needs the operator console's login. The user said the agent was
signed in to that console in their browser; opened there, the link redirected to a signed URL and showed the
visit panel with a third tab drawn in. The drawing named the layout the description only hinted at.
**Shipped:** the signed-in browser paragraph in [intake.md](intake.md).

### 22. A card below the fold read as missing

The reporter asked for notes on a visit. The app had them: a card at the bottom of the visit tab, below the
fold at the reporter's 607-pixel-high window. The same script showed the card off screen on the old code and
on screen on the new one. **Shipped:** the viewport row in the how-to checks in [intake.md](intake.md).

### 23. A help text broke a guard found one fix later

The first fix added two sentences to a help article, and only the touched folders were tested. The next
report's full run failed a word-limit guard over that article, by then on a pushed commit.
**Shipped:** the whole-suite line in [fix-and-verify.md](fix-and-verify.md).

### 24. The running app disagreed with a passing integration test

After a rule list gained a column, the add-on card still showed no own rate, while an integration test on
the same database showed it. Printing the constant from the running page showed the old column list: the dev
server kept a module edited in place, and touching the file did not wake it. A restart, asked for and given,
fixed the screen. **Shipped:** the runtime-value paragraph in [fix-and-verify.md](fix-and-verify.md).

### 25. The readers of a feature hid a money defect

Building a shared add-on list meant mapping every reader of an add-on. The map found that add-ons booked
online were folded into the visit's price and dropped from the commission lines at finalization, while a
finalization correction paid them inside the service line. The test on the old code paid 20 where the new
pays 35. It was fixed in the same change and told to the reporter. **Shipped:** the readers-for-defects row in
[intake.md](intake.md) and the feature-built template in [reply.md](reply.md).

### 26. A teammate's build was reviewed before the reporter heard anything

The eighth session opened on a request a team member had already built on a preview branch, with four
questions and a note that no reply had gone out. The flow had no shape for it: the questions were answered
from the commit at its hash, one each with the line that settled it, the checks the comment named were run
again, and the answers went on the thread as a plain comment, apart from the reply. **Shipped:** the review
section in [intake.md](intake.md) and the review-comment row in the [reply.md](reply.md) checklist.

### 27. An exported file came back as somebody else's booking

The build exported visits as an iCalendar file with `UID:<visit id>@<domain>`. Calendar sync suppressed its
own events by link row, provider marker and the provider's `iCalUID`, and the providers mint their own, so
no stored UID ever had that shape. A file imported into a calendar the sync reads would have blocked the
visit's own slot and queued it for triage as a booking nobody knew. A fourth suppression layer that
recognises the export's UID fixed it, with a guard that failed on the old code. **Shipped:** the
output-comes-back row in [root-causes.md](root-causes.md) and the follow-it-back-in step in
[intake.md](intake.md).

### 28. The echo check walked into capped reads next door

Reading the suppression code for that question showed it loaded the tenant's link rows in one request, and
three other readers of the same table did too: a count on the settings screen, and a disconnect that offers
to delete our events. Past 1000 rows our older events would read as foreign, the count stopped at 1000 and
the rest stayed in the provider. Seeded with 1,200 links, the screen showed 1000 on the old code and 1200 on
the new. **Shipped:** the read-next-door row in the review section of [intake.md](intake.md); the paging
and the source guard were already in [fix-and-verify.md](fix-and-verify.md).

### 29. Another project's dev server answered on the port, and a seed half-landed

The verify procedure named port 3000; the server there belonged to a different repository and answered the
login with a redirect and a 404. Its working directory gave it away, and the host's server ran on a free
port. The seed then failed on a generated column after `psql` had already committed the account and the
connection it created first. **Shipped:** the right-server and independent-parser paragraphs and the seeding
paragraph in [fix-and-verify.md](fix-and-verify.md).

### 30. A report with no bridge was read from the request log

A tester opened the issue by hand: a title, two sentences, two screenshots behind the repository's login,
and no metadata table. The screenshots came down with the CLI's token, the dialog in them named the screen
and its route, and the issue's own creation time anchored the window. The platform's request log on that
route showed ten attempts in five minutes before filing, every one a `402`, so no database read was needed
to know a rule had refused each write. **Shipped:** the tester section and the private-attachment command in
[intake.md](intake.md), and the request-log section in [evidence.md](evidence.md).

### 31. A plan limit read as "try again"

The tenant's plan included one active location and it had one. The dialog's error mapping knew two codes
and showed "Couldn't save the location. Try again." for every other, so the tester retried ten times. The
same code could arrive on restoring an archived location, which got the same message. **Shipped:** the
refusal-code row, the sign and the detail section in [root-causes.md](root-causes.md).

### 32. A shared picker showed ids, and one caller wanted codes

The service's location picker showed the selected location's uuid: the shared multi-select rendered each
selected option's value. Its history showed why: it was built for a language picker, whose values are
compact codes. The first fix added a prop to choose between value and label; the component's option type
already had a short form for exactly that, used by the single select, and the prop was undone in favour of
it. **Shipped:** the shared-component paragraph in [fix-and-verify.md](fix-and-verify.md) and the
value-instead-of-label row in [root-causes.md](root-causes.md).

### 33. One report held two defects

The two items shared a screen family and nothing else: a refusal on save and a label in another screen's
picker. Each got its own cause, guard and numbered paragraph in the memo; one change carried both.
**Shipped:** the several-defects section in [intake.md](intake.md) and the numbered memo in
[reply.md](reply.md).

### 34. The limit was reproduced by setting it locally

Seeded plans have no location limit, so the old code never refused. The run saved the plan's features, set
the limit to the tenant's count, saw the `402` and the old message, then the new one, removed the key and
compared with the saved copy. **Shipped:** the configured-limit repro in
[fix-and-verify.md](fix-and-verify.md).

### 35. A "Generated with" line landed in the pull request body

The commit check passed, and the harness appended its line to the pull request description, which the
user's instructions forbid as much as a trailer. It was edited out after reading the body back.
**Shipped:** the body check in [fix-and-verify.md](fix-and-verify.md), and pull requests and comments
named in the sixth hard rule, the README's non-negotiables and [adaptation.md](adaptation.md).

### 36. The reporter's detour was in the timeline

A title asked to edit a visit "from its finalization". It did not say which field. The timeline did: the
visit's service changed twice in the calendar editor, the visit was paid ten seconds later, the report came
fifty seconds after that. The request became a service field in the payment window, which the user chose
among three variants. **Shipped:** the detour paragraph in [evidence.md](evidence.md) and its row among the
signs in [root-causes.md](root-causes.md).

### 37. A row read a column nothing ever wrote

Mapping the readers for a voucher request found the visit drawer's voucher row reading a column that no code,
migration or database function had ever written, so it showed "-" on every visit, voucher-paid ones included.
It was replaced by a row read from the voucher ledger, with a source guard that keeps the column unread.
**Shipped:** the nothing-writes row in [root-causes.md](root-causes.md).

### 38. The repro through the screen found a join the tests missed

The new voucher row passed its checks and came back empty after a real payment: the payment window had ticked
the client's other visit of the day, and a joint receipt links its visits only through a column on each
visit, which the loader did not read. The loader was fixed before commit; the same gap in an older correction
path was named to the user, not fixed. **Shipped:** the one-path-link row in [root-causes.md](root-causes.md)
and the money-path paragraph in [fix-and-verify.md](fix-and-verify.md).

### 39. Two requests in one working tree went in two commits

Two requests resolved one after the other shared two files. With no interactive staging, the first commit's
version of each shared file was staged as a blob, the commit was type-checked and tested alone with the second
change stashed, and both were committed with their own closing keyword. **Shipped:** the split bullet in
[fix-and-verify.md](fix-and-verify.md).

### 40. Guards from earlier reports caught two slips

A source guard written for an earlier report expected a modal's body mounted on one literal line, and a help
article's word limit was passed by a new sentence. The full suite caught both before commit; the line went
back to the form the guard scans for, and the sentence was cut. **Shipped:** the guard paragraph in
[fix-and-verify.md](fix-and-verify.md).

## Kept deliberately

### 41. Ask before fixing the likely causes

The session had three plausible hypotheses before any data, and each was cheap to change. None was the cause.
Fixing them blind would have shipped three changes and left the bug. The skill keeps the question to the user
even when a blind fix looks harmless.

### 42. A source guard rather than a loader test

The guard reads source files and matches a pattern, which looks crude next to a unit test of the loader. A
loader test needs a mocked client chain and covers one loader; the guard covers every current and future read
on the table and failed on all four defective reads at once. It stays.

### 43. One read scoped instead of paged

The assistant tool needed one service's assignments and read the whole table. It was narrowed to that service
rather than paged: fewer rows, one request, and the guard accepts a read filtered to one record.

### 44. The user chose the design of a behaviour fix

In the second session the evidence showed a payment screen working as built, one visit per receipt, and a
tenant settling two back-to-back treatments on two receipts. Four variants were put to the user: change the
screen, route to the till that already settled several visits, reply with that workaround, or carry the
missing relation through the integration. The user picked the first. The skill keeps the choice with the user
even when one variant is clearly recommended. **Shipped:** "When the cause is behaviour, not a defect" in
[intake.md](intake.md) and the two causes it confirmed in [root-causes.md](root-causes.md).

### 45. A pure module instead of a guard for new behaviour

The second fix had no repeated pattern for a source guard to hold. The rules (which visits join, which may)
went into a pure module with unit tests, and the server action calls it. A test of a module that did not exist
fails on the old code trivially, so the running app carries the fail-then-pass proof. **Shipped:** "When the
fix is new behaviour" in [fix-and-verify.md](fix-and-verify.md).

### 46. A how-to went out without reading production

In the third session the code at the reported version already had every entry point and edge the reporter
asked for, and the report was filed through the assistant from an unrelated page. The user chose a how-to
reply over a production read. The skill keeps that option first when the capability exists: the audit trail
would have told whether anyone had tried the feature, not whether it existed. **Shipped:** "When the app
already does what the reporter asks" in [intake.md](intake.md).

### 47. The user's trade-off went out as the user's

While the reply was being drafted the user added that the feature would mean removing the commission rules
set per category. The code cannot prove a product decision. The reply carried it in the words of the screen,
with the rule type's label checked in the source, and the report back to the user said the reasoning was
theirs. The skill keeps that split rather than tracing a consequence the code does not decide.
**Shipped:** the claim-check row in [reply.md](reply.md).

### 48. A defect inside a request was fixed under `Refs`

A request for finer date filters in the reports was noted for after the freeze. Reading its screen found the
export sending none of the period on screen, so most datasets exported the last 30 days whatever month was
shown. The user chose to fix that and answer the request as noted for later; the commit said `Refs #N`, the
issue was labelled and stayed open. **Shipped:** the screen-defect step in [intake.md](intake.md), the
control-ignores-state row in [root-causes.md](root-causes.md), and the `Refs #N` line in
[fix-and-verify.md](fix-and-verify.md).

### 49. A sibling that looked the same was left out

The vouchers list had a date filter and an export that ignored it too. The list filtered by expiry date and
the export bounded the issue date, so passing the filter through would have exported a different set than the
one on screen. It stayed out of the change and the user was asked which date should win. **Shipped:** the
different-field row in the sweep table in [fix-and-verify.md](fix-and-verify.md).

### 50. A cap written on purpose was treated as the defect

One picker read clients with `.limit(500)` in name order, with a comment calling it a convenience. Cut in name
order, it hid exactly the people the screen was for, so it was paged with the rest and the comment rewritten.
**Shipped:** the cap-on-purpose row in the sweep table in [fix-and-verify.md](fix-and-verify.md).

### 51. Tests that encoded the old behaviour were rewritten

The column fix made two existing grid tests fail: their fixture had a non-treatment person holding a visit and
expected the column hidden. Their expectations were changed, not the fixture, and the commit and the report
named them. **Shipped:** the paragraph and the checklist line in [fix-and-verify.md](fix-and-verify.md).

### 52. A bare "push" was asked about before merging

After both fixes were pushed to the preview branch the user typed "push". Nothing was left to push; the only
step left was merging into the default branch, which deploys to production. The session listed the two commits
it would ship and asked; the user then asked for the merge. **Shipped:** the merge line in
[fix-and-verify.md](fix-and-verify.md).

### 53. A refused grant was hidden, not made to work

The user was offered three shapes: hide the grants the database refuses, the same plus a migration letting
one table's policies read the overrides, or teach every policy the overrides. The first shipped. The skill
keeps the choice with the user and the smallest shape first: the third touches money tables during a freeze.
**Shipped:** the matrix row in [fix-and-verify.md](fix-and-verify.md).

### 54. The admin's own switch was answered, not reverted

The admin asked that reception get full access to vouchers, then switched the till permissions off on two
reception cards, which took vouchers away with them. The reply named the two cards and how to switch them
back, and the labels were renamed to say what they cover. A separate voucher permission would be a new
feature under the freeze. **Shipped:** the setup template and the naming rule in [reply.md](reply.md).

### 55. One cause, one reply per report

Four reports shared one setup. Each got its own reply in its own thread, and each led to a different defect:
refused grants, a misleading label, an uncapped list, a calendar with no read-only mode. The skill keeps one
reply per report even when one answer would cover them. **Shipped:** the paragraph in
[intake.md](intake.md).

### 56. The live policies were read, not reconstructed

Which roles may write each table came from `pg_policies` on production, read only, rather than from the
migration history. The guard test then holds the same list against the local database. **Shipped:** the
policy read in [evidence.md](evidence.md).

### 57. The variant changed mid-work, and everything followed

The user picked a how-to with a small improvement, then wrote "after all, give it its own tab" while the repro
ran. The tab replaced the move everywhere: code, help text, the repro script and the reply. **Shipped:** the
mid-work paragraph in [intake.md](intake.md).

### 58. The full build's product questions went in one batch

The user chose the full build of an add-on library during a freeze. Three questions only the user could
answer (the commission fallback, merging identical add-ons, where the console offers them) went in one
question with recommendations, before the migration was written. **Shipped:** the full-build paragraph in
[intake.md](intake.md).

### 59. A failing test that also failed on the old code was left alone

A list-scroll test failed after the tab change. Run on the old code it failed the same way; the report named
it and the fix did not touch it. **Shipped:** the whole-suite line in [fix-and-verify.md](fix-and-verify.md).

### 60. A data migration and the seed were proven in rolled-back transactions

The library migration merges identical add-ons and repoints visit lines. Its merge step ran on synthetic
copies inside a transaction that was rolled back; the whole file then ran with the non-destructive command
over local data. A database reset that would have run the new seed was refused by the permission layer, so
the seed's changed section ran in a rolled-back transaction instead. **Shipped:** the data-migration paragraph
in [fix-and-verify.md](fix-and-verify.md).

### 61. The reply waited for the deploy, even when asked for all at once

Asked to post the review and the reply, merge, push and close in one message, the run merged and pushed
first, waited for the production deploy to report success, and only then wrote "available now". A reply
posted in the order the user listed would have told the reporter about a button that was not there yet.
**Shipped:** the merge-in-the-same-run section in [reply.md](reply.md).

### 62. The issue was closed before the reporter confirmed

The reply asked the reporter to confirm, and the user closed the issue anyway. That was kept as the user's
call: the bridge mirrors later comments, and the report back said the issue may need reopening.

### 63. An independent parser stood in for the consumer apps

The exported file was parsed with a separate iCalendar library: lines within 75 octets, no fold inside a
multibyte letter, escaped commas and semicolons round-tripping. That proves the format and nothing about
Google, Apple or Outlook, which nobody had opened; the report and the review comment both said so.

### 64. The plan limit itself was left alone

The refusal was right: the plan includes one location. The fix changed what the dialog says and nothing
about the limit, and the memo told the tester the ways out the product already offers (archive one, upgrade,
an add-on). Raising a limit is a commercial setting the operator owns.

### 65. A teammate's reply was checked, not replaced

A request already had a teammate's "noted for later" reply with a question to the reporter. Every claim in it
was checked against the code, and the rule it stated was found held in the database function as well as on
the screen. Nothing was posted: the run recommended waiting for the reporter, and the user agreed.
**Shipped:** the already-answered section in [intake.md](intake.md).

### 66. Only issues whose fix was live were closed

Asked to close the resolved issues, the run closed a setup how-to and a request whose `Refs` commit had since
reached production, and left open the ones waiting on the reporter or never answered. **Shipped:** the closing
table in [reply.md](reply.md).

### 67. The cleanup undid a payment by listing what it wrote

Two repros took real payments on the local database. Each was undone by listing every table that gained rows
since a noted time and deleting them in one transaction, with updated rows restored from a snapshot and from
the audit rows' old values, rather than by a reset that would also wipe other local data. **Shipped:** the
cleanup query in [fix-and-verify.md](fix-and-verify.md).

## Added

### 68. Not run in any recorded session

The root-cause rows for stale cache and time zone, the alternatives in the seam table (Linear, Jira, `psql` on
a replica, an admin API), the reply template for asking the reporter (every session read production or the
code instead), the "route to another screen" and "fix it upstream" variants, and the vocabulary table were
written for this skill and have not run as part of the flow.

### 69. The reply template after a behaviour change is a generalisation

The second session's reply described the new behaviour and how to opt out of it; the template in
[reply.md](reply.md) generalises that one reply and has not been used as a template yet.

### 70. The how-to and correction templates, and the loader repro, are generalisations

The how-to and correction templates in [reply.md](reply.md) each generalise one reply from the third session.
The throwaway loader test in [fix-and-verify.md](fix-and-verify.md) uses placeholder names for the host's
loader and seed helpers; the run it describes used the host's own.

### 71. The feature-request variants and template are a generalisation

The template "when the feature is not built now" in [reply.md](reply.md) generalises one reply from the fourth
session. The narrow and full build variants in [intake.md](intake.md) were put to the user and not built; the
commercial-terms seam has values from one host only. Reading earlier replies for the house tone ran once.

### 72. The sixth session's shapes are generalisations

The setup template in [reply.md](reply.md) generalises three replies from one session. `ID_CHUNK` of 200 is
one host's value. The policy probe assumes a tenant column on every probed table and a no-op update with no
trigger side effects. The list of read-only entry points comes from one calendar.

### 73. The feature-built template is a generalisation

The template "after a requested feature is built" in [reply.md](reply.md) generalises one reply from the
seventh session. The fallback rule it shows (an add-on follows a percentage service rate, not a fixed amount)
was one user's decision.

### 74. The choice still goes in the report when the run may not ask

The paragraph in [intake.md](intake.md) and the clause in the quick start come from the agent eval of 0.2.0,
not from a recorded session. Told to work unattended, one agent listed what it would need and never laid out
the three options. No session has run the unattended case with a real report.

### 75. The review table and the deploy status command are generalisations

The review table in [intake.md](intake.md) generalises one review of one build. The deploy status command in
[reply.md](reply.md) is written for GitHub deployments; the eighth session queried the same API, and other
hosts report deploys elsewhere.

### 76. The request-log patterns beyond the refusal are a generalisation

The ninth session saw one pattern in the request log, the same `402` on every attempt. The rows for a `5xx`,
for no request at all and for one success among failures, and the other platforms' logs in the seam table,
were written for this skill and have not run.

### 77. The tenth session's shapes are generalisations

The cleanup query assumes every table carries `created_at`; a table without one is missed and has to be read
by hand. The blob-staging split ran once, on two files. The closing table generalises one request to close a
batch, on one tracker.
