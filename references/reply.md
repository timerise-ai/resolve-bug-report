# Reply: answering the reporter

The reporter is a customer's staff member, not a developer. The reply tells them what was wrong in terms of
their screen, what is fixed, when they will see it, and asks them to confirm. It travels only through the
bridge's channel.

## The channel

With an issue bridge such as the one in `in-app-bug-reports`, a tracker comment reaches the reporter only
when its first line starts with the reply prefix (`/reply`) and its author is on the team. Anything else stays
a developer note. Post with the CLI:

```bash
# run in the app repository
gh issue comment <number> -R <owner>/<repo> --body-file - <<'EOF'
/reply <first line of the reply>

<the rest>
EOF
```

The prefix goes on the first line, followed by a space and the text. A comment with the prefix on a later line
is not mirrored.

Read two or three earlier replies on the tracker before writing one (`gh issue list --label <bridge label>
--state all`, then their comments): the greeting, the form of address and the length the team uses are
the house tone, and the reporter has seen them before.

## What goes in, and what stays out

| In | Out |
|---|---|
| A thank-you, one line | Table names, file names, commit hashes, stack traces |
| The cause, in the words of their screen: "the form loaded only part of the assignments" | Row counts of other tenants, any other tenant's data |
| What was not wrong: "the assignments you saved were correct" | Blame, of them or of a colleague |
| Every place the fix covers that they use: the form, the public booking page | Speculation about causes you did not confirm |
| When it is live: "after the next update", not a date you do not control | Promises about other reports |
| Whether they must do anything: usually "refresh, nothing to redo" | Internal ticket numbers other than their own |
| A request to check, on the record they reported if you know it | |
| What to send if it is still wrong: the record's name and the expected result | |
| To an admin, the members of their own tenant whose cards need a change, by name | To a member, which colleague changed their account: "the salon's owner changed your role" is enough |

## Every claim is checked before it is posted

A reply is a list of assertions about the product, and the reporter acts on each one. Before posting, list
them and trace each to the code or the running app. A claim that has not been traced stays out of the reply.

| Claim in the reply | Check |
|---|---|
| "Click X, then Y" | The handler on X opens Y; quote both labels verbatim from source |
| "It works after hours too", "for the whole salon" | The edge, run or read, not inferred from the main case |
| "Nobody can book this time" | Every path that books: the console, the public page, a cart, an integration, for every variant of the record |
| "Nothing to set up again" | The fix applies to existing rows, not only to new ones |
| "After the next update" | What makes the commit live in the host's branch policy |
| "This would require removing X", from the user | The user's product decision: check that X exists under that label, and tell the user the reasoning went out as theirs |
| "Today it allows only one" | The form control and the column: a single-choice select over a single foreign key |
| "Click Filters" | The label at the reporter's viewport (the metadata has it): a word that collapses to an icon on a narrow toolbar is named with the icon too |
| "Untick X to see everyone" | Whether the choice is saved: a toggle held only in page state resets on reload, and the reply says so |
| "You can do it in view X" | The view lets them finish the task, not only see the record: a list whose rows only link elsewhere cannot settle anything |
| "Your role is now X", "the owner changed it" | The member's row now and the write in their audit trail; never inferred from the report or from what the admin said they would do |
| "A refresh is enough" | The role and overrides are read on every request, not held in the session token until the next sign-in |
| "Switch X back to Default" | The card's control and its option labels as they read in production today; a label renamed in an unreleased commit is named as the new one coming with the update |
| "The screen is empty because" | The tenant's own count on that screen, and that the other system does not sync those records |
| "Your existing records move over", "identical ones are merged into one" | The migration's step run on local data, and the rule it merges by, named in the screen's fields |
| "Resolved via #<pull request>, merged" | `gh pr view <number> --json state,mergeCommit`: merged, and into the branch the memo names |
| "The limit is reached", "your plan includes one" | The plan's setting the refusal read and the tenant's count, never the label on the plan's marketing page |

The cheapest moment to find a defect is while checking a reply. A scope claim that fails its check is often a
sibling defect in a path nobody looked at: stop, tell the user, and fix it before the correction goes out.

Write in the reporter's language and their form of address. A report written in Polish is answered in Polish,
with the formal "Pani" or "Pan" if the tenant's product uses it.

## Template, after a fix

```text
/reply Hello, thank you for the report!

We found the cause. <One or two sentences: what the screen did wrong, in the reporter's terms, and why,
without internals.> <One sentence on what was not wrong, if they may have doubted their own setup.>

The fix covers <the screens they use>. It will work after the next app update; refresh the page then, there
is nothing to set up again.

Please check after the update, for example on <the record they reported>, and confirm that <the expected
result>. If something is still not right, send us <the record's name and what should appear>.
```

## Template, after a behaviour change

When the fix changed how a screen works, the reporter has to recognise the new behaviour and know how to step
out of it. Describe it in the order they will meet it, and say that past records need no correction when that
is true.

```text
/reply Hello, thank you for the report!

We checked <what they did, on which day>. <What the screen did, in their terms, and why it led to what they
saw.>

We changed it. When you <the action they take>, the screen now <the new behaviour, in the order they meet
it>. If <the case where they want the old behaviour>, <how to opt out, for example untick it>.

It will work after the next app update. <Past records do not need correcting, when true.>

Please check after the update, on <the next case like theirs>, and confirm it works the way you expected. If
it should work differently, tell us briefly how.
```

## Template, when the app already does it

The capability exists ([intake.md](intake.md)). Name the places to find it, the one in plain sight first, with
labels exactly as the screen shows them; answer each edge the reporter named; ask them to try.

```text
/reply Hello, thank you for the report!

Good news: <the capability> is already in <the screen>, and it works the way you describe. You can add it:

1. <The most visible entry point, with the exact labels.>
2. <A second entry point, with what it fills in for them.>

<One sentence per edge they named: "after hours too", "for the whole team", with what they will see.>

Please try it and let us know whether it does what you need. If something is missing, tell us briefly what.
```

## Template, when the feature is not built now

The app lacks what they ask and the user chose to note it for later ([intake.md](intake.md)). Say what the
app does today in the words of their screen, that this is a new feature, what it would change if a
consequence was named, why not now, the workaround until then, and ask for what would shape the later
decision.

```text
/reply Hello, thank you for the report!

Today <what the app does, with the screen's label: "the Category field takes one choice">, so <their example
cannot happen without the workaround they wanted to avoid>.

<The request> is a new feature. <If the user named a consequence: it would mean <the change, with the label
the reporter knows>, because <the reason in their terms>. <What would keep working.>>

<Why not now, in the terms agreed with the tenant: "as agreed, until launch we are not adding new features">,
so we have noted it and will <what happens next: a quote, a review after launch>. Until then,
<the workaround>.

If <the consequence> would be acceptable to you, please let us know, and we will take it into account.
```

## Template, after a requested feature is built

The user chose to build what the reporter asked ([intake.md](intake.md)). Say what the app did before in the
screen's words, where the feature now lives with every label as the screen shows it, the rules they asked
about with what each falls back to, what happened to their existing data, and any defect fixed on the way.

```text
/reply Hello, thank you for the report!

Until now <what the app did, in the screen's words>, so <the friction they described>.

We changed it. After the next app update <where the feature lives: the button, the screen>. <What they set
there, with the labels as the screen shows them.> <Each rule they asked about: who it applies to, and what
it falls back to when nothing is set.>

<Existing data, when the change moved it: "Your existing <records> move over by themselves; identical ones
are merged into one." Nothing to enter again, when true.>

<A defect found on the way, when it touched them: "While building this we also fixed <what was wrong, in
their terms>.">

Please open <the screen> after the update, <the setup step they need to take>, and confirm it works the way
you expected. If something is missing, tell us briefly what.
```

## Template, correcting a reply already sent

When a posted reply turns out wrong, post a new reply through the channel; an edited comment may never reach
the reporter, so check the bridge before relying on an edit. Say what was wrong and what is right, give a
workaround that works until a fix is live, and ask for confirmation again.

```text
/reply Hello, two corrections to our previous answer.

1. <What we said, and what is right instead.>

2. <What checking it turned up, in the reporter's terms.> We fixed it: after the next app update <the new
   result>. <Nothing to set up again, when true.>

Until the update, <the workaround that gives them the right result today>.

Please confirm after the update that <the expected result>.
```

## Template, when the account's setup is the cause

The app works; the member's role or a per-member switch stopped them ([intake.md](intake.md)). Say what the
role allows in the words of the console, what has changed since the report (only what the audit trail shows),
what they do now, and, to an admin, which cards to change and how.

```text
/reply Hello, thank you for the report!

The cause was the account's role. When you reported it, your account had the role <role>, which <does not
include the action, in their words>. <The roles that do.> Nothing was wrong with your data or your steps.

<What changed, when the audit trail shows it: "Today your salon's owner changed your role to <role>">, so
<the action> works after a page refresh; there is nothing to set up again.

<To an admin, per member: "<name>: <permission label> is switched off, which also blocks <what it covers>.">
<How to undo it: <section> > <card> > <panel>, set <label> to <option> and press <button>.>

Please check <the action, on the record they named if known> and confirm here. If it still does not work,
tell us which <record> and what message appears.
```

When the fix also changes the console (labels that now name what a permission covers, controls hidden from
roles without them, a refusal that names the permission), add one paragraph saying so and that it arrives with
the next update.

## Template, when the evidence did not settle it

Send this only when the user chose to ask the reporter.

```text
/reply Hello, thank you for the report!

To find the cause we need a few details:
1. Which <record kind> were you booking or editing when it happened?
2. What did you expect to see, and what did you see instead? For example, which <member> was missing or
   should not have been there.
3. If you can, a screenshot of the screen, attached to this report.

We will get back to you as soon as we have them.
```

## Template, a memo for a tester

When the reporter is the product's own tester ([intake.md](intake.md)), the answer is an ordinary comment
on the thread, without the reply prefix: they may read pull request numbers and branch names, they will
check the fix by hand, and the issue stays open for that check unless the user says otherwise. Keep it to a
memo: what was wrong per item, what changed, how to check it.

```bash
# run in the app repository; a plain comment, no reply prefix
gh issue comment <number> -R <owner>/<repo> --body-file - <<'EOF'
Resolved via #<pull request> (merged to `<branch>`, <what makes it live>). Leaving this open for QA review.

**1. <the first thing they reported, in their words>.** <The cause in the screen's terms: "the app refused a
second location because the plan includes one, and the dialog said try again instead of why">. It now
<what the screen does>. <What the tenant can do: archive one, upgrade, an add-on>.

**2. <the second thing>.** <Cause and fix, one or two sentences>.

**To check on staging:**
- <screen> -> <action> -> <what they should see>
- <screen> -> <action> -> <what they should see>
EOF
```

Number the items in the order they reported them, one per defect, and give each check as a path through the
screens they will click. The limit itself, when the cause is a refusal, is stated as the product's rule with
the ways out, not as a defect.

## When the merge is part of the same run

The user may ask for everything at once: review, reply, merge, push, close. The reply says when the fix is
live, so the order is merge and push, wait for the production deploy to report success, post the reply, then
close. Once the deploy has succeeded the reply can say "available now, refresh the page if you do not see it"
instead of "after the next update".

```bash
# run in the app repository; prints the production deploy's state for the merge commit
sha=$(git rev-parse <merge commit>)
url=$(gh api "repos/<owner>/<repo>/deployments?sha=$sha" \
  --jq '[.[] | select(.environment | test("production"; "i"))][0].statuses_url')
gh api "$url" --jq '.[0].state'   # success, failure, error, or pending while it builds
```

A host that does not report deploys to the tracker has its own status command in the branch-and-deploy seam
([adaptation.md](adaptation.md)); with neither, the reply keeps "after the next update".

Closing by hand before the reporter confirms is the user's call. The reply still asks them to confirm, the
bridge mirrors the close and later comments, and the report back to the user says the issue may need
reopening if the answer is "still wrong".

## Closing several resolved issues on request

Asked to close "the resolved ones", decide each open report on what its thread and the history show, not on
its age:

| State | Close |
|---|---|
| A commit that references it reached the default branch, the production deploy succeeded, the reply was sent | Yes, even when the reply said "after the next update": that update is now live |
| A how-to or setup reply with nothing left on the team's side | Yes |
| The reply asked the reporter for something (a code, an example) and no answer came | No |
| Noted for later, or no reply at all | No |
| A report outside the bridge (a website task, a tester's issue) | Leave it to its owner |

Report what was closed and why each other one stays open.

## After posting

Tell the user the comment's URL, what the reply said in one line, and what is still pending on their side:
the deploy that makes the fix live, and the issue closing when the commit reaches the default branch.

## Checklist

- [ ] Every claim listed and traced to the code or the running app; labels quoted verbatim
- [ ] Prefix on the first line; posted through the bridge's channel
- [ ] Reporter's language and form of address
- [ ] Cause in their terms, scope, when it is live, nothing to redo, a request to confirm
- [ ] For a behaviour change: the new behaviour in the order they meet it, and how to opt out
- [ ] For a feature not built now: today's behaviour, the consequence the user named, why not now, the
      workaround, and the question that shapes the later decision
- [ ] For a feature built: where it lives, what to set, the fallback rules, what happened to existing data,
      and a defect fixed on the way
- [ ] For the account's setup: the role then and now from the data, what to do now, and to an admin the cards
      to change with labels as production shows them
- [ ] No internals, no other tenant's data
- [ ] "Available now" only after the production deploy reported success; otherwise "after the next update"
- [ ] A review for a teammate posted as a plain comment, never under the reply prefix
- [ ] A memo for a tester: a plain comment, one numbered item per defect, a check per item, the issue left
      open unless the user said to close it
- [ ] Issues closed on request only when their fix is live or nothing is left on the team's side
- [ ] The user told the URL and what is still pending
