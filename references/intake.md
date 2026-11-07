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
| Page | The screen, so the component and its data loader |
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

## Checklist

- [ ] Issue, comments and metadata read; app version noted
- [ ] Screen and its loader located in the code at that version
- [ ] Sufficiency matrix filled; every gap named
- [ ] Two or three hypotheses, each with the check that distinguishes it
- [ ] If gaps remain: the user asked, with the three options and a recommendation
