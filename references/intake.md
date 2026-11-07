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
| Reporter and role | Whose actions to rebuild; what the role may see and do |
| Source | A form, the assistant, an email: the assistant path may carry a drafted description the form path lacks |
| Page | The screen, so the component and its data loader. Through the assistant it is where the chat was opened, which can be unrelated to the report: read the title first |
| App version | The commit that was live, so the code to read is that commit, not `HEAD` |
| Created at | The anchor of the timeline: the reporter's last writes before it usually name the record |
| Viewport, browser, time zone | Layout and date bugs; ignore otherwise |
| Report id or link | The key to the attachments and the report row in production |

Attachments are often links, not inline images, because a login-gated file cannot be proxied by the tracker.
No link in the body usually means no attachment, but confirm it against the report row in production.

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
| The labels, verbatim from source | The reply quotes buttons and menu items; a paraphrase sends them looking for a word that is not there |
| Which element does what: the name, the arrow beside it, a right click | "Click the name" and "click the arrow next to the name" open different things |

Then ask the user, with three options, the first recommended: reply with the how-to; reply and make a small
improvement the report points at; read production first to tell a capability nobody found from one that
failed when used. A report filed from an unrelated page through the assistant points to the first.

## Checklist

- [ ] Issue, comments and metadata read; app version noted
- [ ] Screen and its loader located in the code at that version
- [ ] Sufficiency matrix filled; every gap named
- [ ] Two or three hypotheses, each with the check that distinguishes it
- [ ] If gaps remain: the user asked, with the three options and a recommendation
- [ ] If the cause is behaviour: the user asked to pick a variant, the existing workaround among them
- [ ] If the capability exists: every entry point, every named edge and every record variant checked in
      the code before the user is asked
