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

The cheapest moment to find a defect is while checking a reply. A scope claim that fails its check is often a
sibling defect in a path nobody looked at: stop, tell the user, and fix it before the correction goes out.

Write in the reporter's language and their form of address. A report written in Polish is answered in Polish,
with the formal "Pani" or "Pan" if the tenant's product uses it.

## Template, after a fix

```text
/reply Hello, thank you for the report!

We found the cause. <One or two sentences: what the screen did wrong, in the reporter's terms, and why, without
internals.> <One sentence on what was not wrong, if they may have doubted their own setup.>

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

## After posting

Tell the user the comment's URL, what the reply said in one line, and what is still pending on their side:
the deploy that makes the fix live, and the issue closing when the commit reaches the default branch.

## Checklist

- [ ] Every claim listed and traced to the code or the running app; labels quoted verbatim
- [ ] Prefix on the first line; posted through the bridge's channel
- [ ] Reporter's language and form of address
- [ ] Cause in their terms, scope, when it is live, nothing to redo, a request to confirm
- [ ] For a behaviour change: the new behaviour in the order they meet it, and how to opt out
- [ ] No internals, no other tenant's data
- [ ] The user told the URL and what is still pending
