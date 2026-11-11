# Intake: reading the report and deciding whether it is enough

The first question is not "what is the bug" but "do I have enough to know". Answer it from everything the
report carries, not from the description alone, and answer it before reading code.

## Read everything the tracker holds

```bash
# run in the app repository
gh issue view <number> -R <owner>/<repo> --json title,body,comments,labels,state,author
```

`gh issue view --comments` can print nothing for an issue authored by a GitHub App; the `--json` form always
returns the fields. Read the comments too: a developer may already have asked something, and the reporter's
answer arrives there through the bridge.

A bridge typically appends a metadata table under the description. These are the fields that matter and what
each one tells you:

| Field | What it gives you |
|---|---|
| Tenant | Which data to read; the scale of that tenant is itself a clue (see [root-causes.md](root-causes.md)) |
| Reporter and role | Whose actions to rebuild; what the role may see and do. The bridge reads the role when it files the issue, so a role that contradicts the title ("I am reception" from a basic staff account) is the first cause to check |
| Source | A form, the assistant, an email, the app's error screen: the assistant path may carry a drafted description the form path lacks; the error screen carries the message and an error reference, the message is often the action's generic fallback, and the reference finds the server's log line |
| Page | The screen, so the component and its data loader. Through the assistant it is where the chat was opened, which can be unrelated to the report: read the title first |
| App version | The commit that was live, so the code to read is that commit, not `HEAD` |
| Created at | The anchor of the timeline: the reporter's last writes before it usually name the record |
| Viewport, browser, time zone | Layout and date bugs; ignore otherwise |
| Report id or link | The key to the attachments and the report row in production |

Attachments are often links, not inline images, because a login-gated file cannot be proxied by the tracker.
No link in the body usually means no attachment, but confirm it against the report row in production. When the
user says the agent may use their signed-in browser, open the link there: it redirects to a short-lived signed
URL, and the screenshot is evidence, often more than the description. Marks drawn on it (a box where a tab
should be, a line under a header) show the layout the reporter wants. The signed URL stays out of the reply
and the commit.

An image pasted into an issue in a private GitHub repository is a `user-attachments` URL that answers 404 to
a plain download. Fetch it with the CLI's token and read the file:

```bash
# run anywhere; saves one screenshot of a private-repository issue
curl -sL -H "Authorization: token $(gh auth token)" -o shot.png \
  https://github.com/user-attachments/assets/<asset id>
```

## The sufficiency matrix

Fill one row per item. Every "no" is a gap to close before writing code.

| Need | Enough when | Where it usually comes from |
|---|---|---|
| Where | You can name the screen and the action | Page field, title, source |
| What was expected | You can state the right result | Title, description, the product's rules |
| What happened | You can state the wrong result | Description, screenshot, or the timeline |
| Which record | You can name the entity: a service, a client, an invoice | Description, or the reporter's writes just before filing |
| When it started | You know whether it ever worked | App version, recent commits to the loader, recent data changes |
| Reproducible | You can make it happen on demand, locally or from data | The cause, once known |

A title such as "the booking form shows the wrong staff for a service" fills "where" and "what kind", and
leaves "which record", "expected" and "what happened" empty. That is the common case, and it is not a dead
end: the timeline in [evidence.md](evidence.md) fills "which record" more often than not.

## Closing the gaps: ask the user, with the choice laid out

When the matrix has gaps that only production data or the reporter can close, stop and ask the user. Do not
read production unasked, and do not post a question to the reporter unasked. Offer three options, the
recommended one first:

| Option | When it is right | Cost |
|---|---|---|
| Read production, read only | The tenant's data and the audit trail can name the record and the sequence | The user's approval; the permission layer may need a rule |
| Ask the reporter | The bug is in what they saw, not in the data: layout, wording, a step they took | A round trip of hours or days |
| Fix the likely causes | Each hypothesis is cheap to fix and harmless if wrong | A fix that may miss, and a reply that may have to be retracted |

Before asking, do the code reading you can do without data: find the screen's loader and the rule that
produces the wrong-looking result, and write down two or three hypotheses, each with the data check that
would confirm or kill it. The user decides better with those in front of them, and the question carries
them.

A run that may not ask (unattended, or told not to ask questions) still owes this choice. Write the three
options with the recommended one first into the report, in place of the question, and stop there: a list of
what you would need does not let the user decide, and "do not ask" never means picking an option yourself.

## When the cause is behaviour, not a defect

Sometimes the evidence shows the app doing exactly what it was built to do, and the tenant expecting
something else: a payment screen that settles one visit while the tenant thinks of several treatments in one
sitting as one visit. Nothing is broken, so there is no single correct fix, and choosing the design is the
user's call, not yours. Ask once the cause is confirmed, with the variants laid out:

| Variant | Typical shape | Cost |
|---|---|---|
| Change the behaviour where the reporter was | The screen in the report does what they expected | The biggest change, on the path people use |
| Route to a screen that already does it | A notice and a link that opens the other screen prefilled | Smaller; one more step for the user |
| Reply with the existing workaround | No code; the reply explains where the app already does it | Cheapest; the friction stays |
| Fix it upstream | The integration or import carries the missing relation | Often the cleanest data, often the slowest to ship |

Look for the workaround before asking. A capability that exists on another screen, unreachable from the one
in the report, is both a variant and evidence for the recommendation. Recommend one variant and say why; the
user picks.

## When the app already does what the reporter asks

A report can ask for something the app already has: a "missing" block in a calendar that is one menu away,
a tooltip that already shows the full note. Nothing is broken, nothing needs building, and the reply is a
how-to. Do not say so until the code says so, at the app version in the metadata:

| Check | Why |
|---|---|
| Every entry point to the capability: toolbar, row or column menu, shortcut, quick action, a dialog that offers it | The reply names the ones the reporter will find first; a single path is often the one they missed |
| Every edge the reporter named: "after hours too", "the whole salon", "without clicking" | The capability can exist and still fail on exactly the edge they care about |
| Every variant of the record: per person and for everyone, assigned and unassigned | A variant the app draws but another path ignores is a defect hiding inside a how-to (see [root-causes.md](root-causes.md)) |
| The task they named, to its end: settle, send, print, not only see | A view that lists the record but cannot act on it is not the answer; a list whose rows only link elsewhere sends them to a dead end |
| The labels, verbatim from source | The reply quotes buttons and menu items; a paraphrase sends them looking for a word that is not there |
| The tenant's own records on that screen | A capability with nothing in it for this tenant (the records live in another system that does not sync them) is answered with why it is empty and how records get there, not only with where it is |
| Which element does what: the name, the arrow beside it, a right click | "Click the name" and "click the arrow next to the name" open different things |
| Where it sits at the reporter's viewport (the metadata has the size) | A card at the bottom of a panel can sit below the fold on a short laptop screen and read as missing; that is the cause, and moving it up or into its own tab is the small improvement |

Then ask the user, with three options, the first recommended: reply with the how-to; reply and make a small
improvement the report points at; read production first to tell a capability nobody found from one that
failed when used. A report filed from an unrelated page through the assistant points to the first.

A how-to the user proposes ("explain where the list mode is") is a hypothesis like any other. Run the same
checks on it before drafting. When the place the user named shows the record but cannot finish the task, say
so before writing a word to the reporter, name the path that does work, and look for why the reporter could
not reach it: a default-on filter that hides the record together with its column is a defect, and the user
then picks between fixing it with the how-to and the how-to alone.

The user may change the variant while you work ("after all, give it its own tab"). The new choice replaces the
old one everywhere the old one reached: the code, the tests, the help text and the draft reply. Say in the
report back which variant shipped.

## When the cause is the account's setup

A member writes "I am at reception and cannot settle visits" and the bridge says their role is basic staff.
Nothing is broken: the account was created with the wrong role, or a per-member switch took a permission
away. Read the role and the per-member overrides at the time of the report and now, and the audit trail of
the member's record between the two ([evidence.md](evidence.md)): the tenant's admin often fixes it minutes
after the report, and sometimes changes something else that contradicts what was asked.

| Check | What it settles |
|---|---|
| The role at filing (the bridge metadata) against the role that holds the permission | Whether the role alone explains it |
| The per-member overrides, at filing and now | A revoke the admin set, a grant the data layer refuses ([root-causes.md](root-causes.md)) |
| Writes to the member's record after the report | Already fixed; or a new setting that undoes what the report asks for |
| The screen the reporter used, as their role | A control offered to a role that cannot use it, and the message it gave: a generic failure hides the permission |

The reply is then a how-to: what the role allows, what has changed, what the reporter does now (often only a
refresh), and where the admin changes it ([reply.md](reply.md)). The permission system itself is a screen to
read for defects, as in a request: an override the data layer refuses, a label that hides what a permission
covers, a write control shown to a role without it. Each goes to the user as a variant beside the how-to.

Several reports from one reporter within minutes usually share this cause. Read the tenant's timeline once,
across all of them, and still answer each report in its own thread: each reporter waits there, and each
report can lead to a different sibling defect.

## When the report asks for a feature the app does not have

A report can be a request: "can one service sit in two categories?" Nothing went wrong and there is nothing
to point at, yet the reporter filed it where bugs go and is waiting for an answer. Do not drop it into the
backlog without a word: the flow runs up to the reply, even if the code is never written.

| Step | What it settles |
|---|---|
| Confirm the absence at the app version: the field, its form control, every entry point a how-to would use | That the answer is not a how-to; a single-choice select over a single foreign key is the usual proof |
| List every reader of the field the feature would change: screens, public pages, rules, reports, integrations | The real cost and the trade-offs: a rule that picks one value per record turns ambiguous when a record may hold several |
| Read the host's commercial terms: a feature freeze, a clause that work outside the contract is quoted first, the labels for requests | Whether "build it" is on the table now; these live in the host's notes, not in the code |
| Write the variants, the narrow one among them | A narrow variant often keeps the rules that pay or bill on one value and widens only what is displayed |
| Read the screen the request names for a defect: its export, totals and filters against what it shows | A control that ignores the screen's state (an export that ignores the period on screen) is a defect a freeze allows fixing; it goes in as a variant beside "noted for later" |
| Read every path that would carry the new field for a defect it already has | The readers' map often finds money already miscounted: an amount folded into a visit's total that the payout never reads. It is fixed whichever variant ships, and told to the reporter |

Then ask the user, the recommended variant first:

| Variant | When it is right |
|---|---|
| Reply that it is noted for later | A freeze or a quote clause is in force, or the list of readers is long; label the issue and keep it open |
| Build the narrow variant | The request is cheap without touching the rules that pay or bill: one primary value, extra values for display |
| Build it fully | The tenant needs the rules to follow too, and the user accepts deciding which value wins in each |
| Read production first | Their data would change the answer, such as how their category tree is actually shaped |

When the user picks the full build, the readers' map raises product questions the code cannot answer: what a
rule falls back to when the new target has none of its own, what happens to existing data that now overlaps
(copies to merge, or to keep apart), where in the app the new choice is offered. Ask them in one question,
each with a recommendation, before writing the migration. A full build under a freeze or outside the contract
is the user's call; the quote clause goes into the report back to the user, not into the reply.

The user may add a consequence while you work, such as "it would mean removing the per-category commission
rules". That is their product decision, not a claim the code can prove. Carry it into the reply in the
screen's words, check that the screen and its label exist, and say in the report back to the user that the
reasoning was theirs. See [reply.md](reply.md).

## When a teammate has built it and asks for a review

A member of the product team may get to a request first: their comment on the report names a commit on a
preview branch, lists what they checked and asks a few questions before anyone answers the reporter. The run
becomes a review, and the flow still ends in the reply.

| Step | What it settles |
|---|---|
| Read the commit at its hash, not the branch tip, and every file it touches | The review covers what they built, not what landed after |
| Answer each question they asked with the line of code that settles it | They asked those because they were unsure; an answer without the line is another opinion |
| Re-run each check they say passed, and list the ones they say they did not run | A claim in a comment is a claim; "not opened in Google or Outlook" stays open until somebody does |
| Follow everything that leaves the app (a file, a feed, an email, a push) back in through every inbound path that could read it again: a sync, an import, deduplication, echo suppression | The identifiers the new output mints must be ones the inbound path recognises ([root-causes.md](root-causes.md)) |
| Read the integration next door for the defects the sweep looks for anyway | Checking one question often walks through code with a capped read of its own |

A defect the review finds is fixed on the same branch in its own commit after theirs, with its own guard and
`Refs #N`, never folded into their commit. The answers go on the thread as an ordinary comment to the
teammate, in the language they wrote in, without the reply prefix: they may name files and commits, the
reporter never sees them. The reply to the reporter is separate and waits for the deploy
([reply.md](reply.md)). When the user asks for the review only, post nothing and report the answers.

## When a teammate has already answered it

The thread may already hold a reply from the team: a request answered as noted for later, with a question to
the reporter. The run is then a claim check on that reply, not a second answer.

| Check | What it settles |
|---|---|
| Every label, menu and role the reply names, against the code at the current version | Whether a correction is owed ([reply.md](reply.md)) |
| The layer that holds the rule the reply states ("only the owner or a manager"): the server action and the database function or policy | Whether the rule holds when the screen is bypassed; a rule held only in the screen is a defect |
| Whether the reporter answered the reply's question | Whether there is anything to act on yet |
| Whether a per-member grant could give the reporter what they ask | A workaround the reply could have named |

When the reply holds and waits on the reporter, recommend waiting and post nothing; the reporter's answer
arrives on the thread through the bridge.

## When a tester filed it, without a bridge

A report can come from the product's own QA: an issue opened by hand on the tracker, a title, a sentence and
two screenshots, and no metadata table. Nothing names the tenant, the role or the app version, but three
things stand in for them:

| Bridge field | What stands in |
|---|---|
| Created at | The issue's own `createdAt`; the tester filed it minutes after the attempts |
| Page and action | The screenshot: the dialog, its title and the error under the button name the screen and its API route |
| What happened | The platform's request log on that route in the window before filing: the status of every attempt ([evidence.md](evidence.md)) |

The request log often settles the cause before any database read: the same 4xx on every attempt is a rule
refusing, not a save failing, and the code names the rule. The reporter is a colleague, so the reply is a
short memo on the thread, not a `/reply`, and the issue usually stays open for the tester's own check after
the deploy: commit with `Refs #N` and leave the closing to them ([reply.md](reply.md),
[adaptation.md](adaptation.md)). Read the user's instructions for that policy; "do not close it, QA
reviews" is common and overrides the closing keyword.

## One report, several defects

A tester's report often lists two unrelated things on one screen: an error on save and a wrong label in a
picker. Treat each as its own report inside the issue: a sufficiency row each, a cause each, a guard each.
They rarely share a cause, and a fix found for the first does not explain the second. One change may carry
both when both are small; the commit body and the reply number them in the order the reporter wrote them, so
each is answered.

## Checklist

- [ ] Issue, comments and metadata read; app version noted
- [ ] Screen and its loader located in the code at that version
- [ ] Sufficiency matrix filled; every gap named
- [ ] Two or three hypotheses, each with the check that distinguishes it
- [ ] If gaps remain: the user asked, with the three options and a recommendation; in a run that may not
      ask, the same three in the report
- [ ] If the cause is behaviour: the user asked to pick a variant, the existing workaround among them
- [ ] If the capability exists: every entry point, every named edge and every record variant checked in
      the code before the user is asked
- [ ] If it asks for something new: the absence confirmed, every reader of the field listed, the host's
      commercial terms read, and the user asked to pick a variant
- [ ] If it is built in full: the open product questions asked in one batch, and every reader of the new
      field checked for a defect it already has
- [ ] If the account's setup may be the cause: role and overrides at filing and now, the member's record's
      writes after the report, and the reporter's screen as their role
- [ ] If a tester filed it without a bridge: `createdAt`, the screenshot's screen and the request log stand
      in for the metadata; the closing policy read from the user's instructions
- [ ] Several defects in one report: a row, a cause and a guard each, answered in the reporter's order
- [ ] If a teammate already answered: every claim of their reply checked, the rule's enforcing layer read,
      and nothing posted while it waits on the reporter
- [ ] If a teammate built it: the commit read at its hash, each question answered with its line, their checks
      re-run, every output followed back through the inbound paths, and the answers kept off the reply prefix
