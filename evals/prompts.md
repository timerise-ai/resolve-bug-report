---
prompts:
  - prompt: "A customer filed a bug report from inside our app. The issue says only: title 'the booking form shows the wrong staff for a service', no description, filed from the page /console/reservations by a manager of one tenant, with the app version and a timestamp. Before anyone touches code, tell me whether we have enough to fix it, what you would look at to find out, and what you would need from me. You have no access to our production data from here."
  - prompt: "We fixed a tenant's report: our booking form read staff assignments in one request and the API cut it at 1000 rows, so some services showed everyone. Write the reply the reporter will get through our /reply bridge. They wrote the report in Polish and are not technical. The fix goes live with the next update."
  - prompt: "A member of a salon filed a report through our /reply bridge, with no description, titled: I work at reception and I cannot settle or finalize visits. The bridge metadata says the reporter's role is basic staff, and the same person filed two more reports within five minutes. Our app checks permissions in code and again in Supabase row level security, and owners can grant or revoke single permissions per employee. Tell me what you would read before deciding whether anything needs fixing, what you would ask me, and draft the reply for the case where the owner has already changed the role. The reporter writes in Polish."
---

# Prompts

What an operator types after installing this skill, in their own words. An agent eval installs the skill into
an empty Next.js app, gives the agent one of these prompts and no further help, then type-checks, builds and
tests the result; the first prompt runs before every release. This skill resolves a report in another app, so
each prompt carries the report it works on, and the checks only confirm the agent left the fixture app intact.
What a run shows is in its notes, scored on whether the agent followed the hard rules: deciding sufficiency
first, keeping production read only and asked for, fixing the pattern, proving fail then pass, replying
through the channel in the reporter's language, and acting outward only as asked. The results are the other
files in this folder. Section 10 of [STANDARD.md](https://github.com/timerise-ai/skills/blob/main/STANDARD.md)
says how a run is made.
