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

- [ ] Prefix on the first line; posted through the bridge's channel
- [ ] Reporter's language and form of address
- [ ] Cause in their terms, scope, when it is live, nothing to redo, a request to confirm
- [ ] No internals, no other tenant's data
- [ ] The user told the URL and what is still pending
