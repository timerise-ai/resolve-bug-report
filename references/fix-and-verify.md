# Fix and verify: the pattern, the guard, the repro, the commit

A fix is done when every sibling of the defect is fixed, a test fails on the old code and passes on the new,
and the running app shows the wrong result before and the right one after.

## Sweep for siblings

The line the report led you to is one instance. Find every other read or write with the same shape before
fixing any of them:

```bash
# run in the app repository: every request on the table the bug lives in
grep -rn 'from("employee_services")' src | grep -v test
```

Classify each hit. For a capped read the classes are:

| Hit | Verdict | Fix |
|---|---|---|
| Whole-tenant read feeding a list or a rule | Defect | Page it |
| Read that needs one record's rows | Defect if unfiltered | Filter by the record instead of paging everything |
| Read filtered to one parent (one service, one member) | Safe, far below the cap | None |
| Write (insert, delete) | Not a read | None |
| A cap written on purpose (`.limit(500)` and a comment calling it a convenience) over a whole table in name order | Defect: it hides the same names | Page it, and rewrite the comment |
| A control on the same screen that bounds a different field (the list filters by expiry, the export by issue date) | Not the same defect: wiring the filter through exports a different set than the one shown | None in this change; tell the user which field should win |
| A `.limit(n)` with no `.order(...)`, sorted in code afterwards | Defect: past the cap it keeps an arbitrary subset, often without the newest rows | Page it with a stable order, sort after |
| A second read that sends every id of the first in one `.in(...)` | Defect past a few hundred ids: the request line outgrows the gateway or the cap cuts it | Chunk the ids and page each chunk |
| A permission guard that checks the role alone where the others check role and overrides | Defect: a grant never works there, a revoke never holds | The override-aware check |

A storefront or public path that uses the same rule as the screen in the report usually has the same defect,
and the reporter never saw it. Fix it in the same change.

## Page the read

Use the host's paging helper if it has one. If it does not, this one pages until a short page proves the end,
and refuses to truncate silently past a hard ceiling:

```ts
// lib/supabase/paginate.ts
/** Must match the API's max_rows (PostgREST default 1000). */
export const PAGE_SIZE = 1000;

/** Refuse to hold more than this in memory for one read. */
export const MAX_TOTAL_ROWS = 100_000;

type PageResult = { data: unknown[] | null; error: { message: string } | null };

/**
 * Reads every row of a query by pages. The factory must build the same query for each window with a stable
 * order, or a row can appear on two pages.
 */
export async function fetchAllRows<T>(
  page: (from: number, to: number) => PromiseLike<PageResult>,
  label: string,
): Promise<T[]> {
  const rows: T[] = [];
  for (let from = 0; ; from += PAGE_SIZE) {
    const { data, error } = await page(from, from + PAGE_SIZE - 1);
    if (error) throw new Error(`${label}: ${error.message}`);
    const batch = (data ?? []) as T[];
    rows.push(...batch);
    // A short page is the only reliable end signal: the API reports no "has more".
    if (batch.length < PAGE_SIZE) return rows;
    if (rows.length >= MAX_TOTAL_ROWS) throw new Error(`${label}: more than ${MAX_TOTAL_ROWS} rows`);
  }
}
```

At the call site, order by the primary key and keep the `{ data }` shape the surrounding code destructures, so
the change stays local:

```ts
// server/reservations.ts, inside the loader's Promise.all
fetchAllRows<StaffAssignmentRow>(
  (from, to) =>
    supabase
      .from("employee_services")
      .select("service_id, employee_id, variant_id")
      .order("id", { ascending: true })
      .range(from, to),
  "getNewAppointmentOptions(employee_services)",
).then((data) => ({ data })),
```

A second read keyed by the first read's ids needs the same care. Chunk the ids so the request line stays
short, and page each chunk, since one id can match many rows:

```ts
// lib/supabase/paginate.ts, after fetchAllRows
/** Ids per `.in(...)` filter: past a few hundred uuids the request line outgrows what gateways accept. */
export const ID_CHUNK = 200;

/** `fetchAllRows` over a list of ids, in chunks; the factory builds the same ordered query for each. */
export async function fetchAllByIds<T>(
  ids: readonly string[],
  page: (chunk: string[], from: number, to: number) => PromiseLike<PageResult>,
  label: string,
): Promise<T[]> {
  const rows: T[] = [];
  for (let i = 0; i < ids.length; i += ID_CHUNK) {
    const chunk = ids.slice(i, i + ID_CHUNK);
    rows.push(...(await fetchAllRows<T>((from, to) => page(chunk, from, to), label)));
  }
  return rows;
}
```

## The guard test

A unit test of the loader needs a mocked client chain and proves little. A source guard is cheaper and holds
the pattern for every future read: it fails when a read on the table neither pages nor filters to one record.

```ts
// lib/service-staff-reads.test.ts
import { readdirSync, readFileSync, statSync } from "node:fs";
import { join, relative } from "node:path";
import { describe, expect, it } from "vitest";

/** Every read of the assignment table must page, or be scoped to one service or one employee. */
const ROOT = join(__dirname, "..");

function sourceFiles(dir: string): string[] {
  return readdirSync(dir).flatMap((name) => {
    const path = join(dir, name);
    if (statSync(path).isDirectory()) return sourceFiles(path);
    return /\.tsx?$/.test(name) && !/\.test\.tsx?$/.test(name) ? [path] : [];
  });
}

describe("employee_services reads", () => {
  it("page or stay scoped to one service or employee", () => {
    const offenders: string[] = [];
    for (const file of sourceFiles(ROOT)) {
      const lines = readFileSync(file, "utf8").split("\n");
      lines.forEach((line, i) => {
        if (!line.includes('from("employee_services")')) return;
        const chain = lines.slice(i, i + 8).join(" ");
        // Writes (delete, insert) are not reads.
        if (!/\.select\(/.test(chain)) return;
        if (/\.range\(|\.eq\("(service_id|employee_id)"/.test(chain)) return;
        offenders.push(`${relative(ROOT, file)}:${i + 1}`);
      });
    }
    expect(offenders).toEqual([]);
  });
});
```

**Prove it fails on the old code.** `git stash` stashes tracked changes and leaves a new, untracked test file
in place, which is exactly what you need: stash, run the test, read the offenders it lists (they must be the
lines you fixed), `git stash pop`, run it again green. Then run the whole suite and the linter.

When the new tests sit in a file that already existed, a bare `git stash` takes them away with the fix. Stash
the fixed source paths only, `git stash push <fixed file> ...`, and check `git stash list` is empty again
afterwards.

A busy table (clients, products) has many single-row lookups the guard must not flag. Three changes keep it
precise without an allowlist that grows with every file:

| Change | Why |
|---|---|
| Strip `//` comments from the window, then cut it at the first `;` | A `;` inside a comment ends the statement early, and the lookup's narrowing filter falls outside it |
| Widen the window to about 16 lines | A multi-line select with a comment per column pushes `.range(` past 8 lines |
| Accept the narrowing shapes: `.range(`, `.in(`, `.eq("id"`, an exact key (`phone`, `email`), `.limit(1)`, `.maybeSingle(`, `.single(`, `head: true` | Each returns a handful of rows at most |

What stays unmatched (an `.or()` built in a variable from two exact keys) goes into a set keyed by
`file:line`, each with a comment saying why it stays small. Keyed by line on purpose: moving the read makes
the test name it again. When several pickers read the same whole table, give them one helper that pages in the
order they display, with the primary key as a tie-break so namesakes never repeat or drop between pages.

When the fix changes what the screen shows, an existing test may encode the old behaviour (a fixture where a
column holding a visit stays hidden). Change its expectations rather than the fixture, and name each changed
test in the commit body and in the report to the user.

## When the fix is new behaviour

A product decision usually means new behaviour rather than a corrected line, and a source guard has nothing to
hold. Put the rule in a pure module and test that instead:

| Piece | Where | Why |
|---|---|---|
| Which records join the action (same client, same day) | A pure function, unit tested | The browser and the server choose the same set |
| Whether the extra records may join (same payer, not paid, not cancelled, priced) | A pure function returning a refusal message or null, unit tested | The rule is tested without mocking the database client |
| Reading the rows and refusing | The server action, calling the function | Ids arrive from the browser and are never trusted |

The server holds each extra record to the same rules as the main one, plus the rule that joins them (one
receipt, one payer), and refuses a requested id that did not come back from the read: under row level security
another tenant's id simply returns nothing. A test file for a module that did not exist fails on the old code
by construction; the running app is where fail then pass is shown.

Commit new behaviour as `feat`, a corrected one as `fix`; both carry `Fixes #<number>`.

## When the defect is in permissions

A permission defect is rarely one line. Three fixes recur, often together, and each has its own proof.

| Fix | Change | Proof |
|---|---|---|
| The refusal says which permission | The guard's error becomes the error class the action wrapper hands back to the form, with the permission's label as its message | A unit test: the wrapper given that error returns the label and logs nothing; it fails on the old code with the generic fallback |
| The matrix agrees with the data layer | A list per permission of the roles the table's policy admits; an override may grant only to those, the effective check ignores a stored grant outside it, and the save drops it | An integration test that probes each policy as every seeded role and compares with the list |
| The screen is read-only for a role without the write | One prop for the write permission on the screen; every entry point to a write reads it | The running app as that role and as one that has it |

The policy probe, with a client signed in as a seeded member of each role and one seeded row per table:

```ts
// tests/integration/grant-probe.ts
import type { SupabaseClient } from "@supabase/supabase-js";

export type Probe = { table: string; op: "update" | "select"; id: string };

/** The roles whose session gets through the table's policy on one seeded row, sorted. */
export async function admittedRoles<R extends string>(
  clients: Record<R, SupabaseClient>,
  probe: Probe,
  tenantId: string,
): Promise<R[]> {
  const admitted: R[] = [];
  for (const role of Object.keys(clients) as R[]) {
    const db = clients[role];
    // A no-op update: a role outside the policy's USING sees zero rows, and no error.
    const { data } =
      probe.op === "update"
        ? await db.from(probe.table).update({ tenant_id: tenantId }).eq("id", probe.id).select("id")
        : await db.from(probe.table).select("id").eq("id", probe.id);
    if ((data ?? []).length === 1) admitted.push(role);
  }
  return admitted.sort();
}
```

One test per listed permission asserts `admittedRoles` equals the list, and one asserts every listed
permission has a probe. Two unit tests keep the matrix honest without a database: a role holding a permission
by default is always one the list admits, and a revoke works for every role. Existing tests that used a grant
the database refuses as their example (a basic member granted the till) encoded the defect; move them to a
grant that works and add the refused one as a negative case.

For the read-only screen, list every entry point before hiding any: dragging, the context menu, the column
menu, the toolbar's add button, keyboard shortcuts, a click on an empty cell, the detail drawer's status menu
and edit link and primary button, inline editors, and list rows with their own buttons. Keep what the role may
still do (their own time block), and check the server: an action that only asks for a session reaches the
database and comes back with a misleading refusal, so give it the permission check too.

Drive the running app as the role without the permission, then as one with it. The second run catches a
regression the first cannot see, and the first catches what the code reading missed: switching drag off by
passing the touch-device flag also removed the click that opened the tile, because on a desktop that click
travelled through the drag handler.

## Reproduce in the running app

The guard proves the code; the repro proves the reporter's screen. Use the host's verify procedure (a project
skill, Playwright, a dev login) and this order:

1. **Seed past the cap locally.** Insert filler records with enough rows to exceed the cap, then one test
   record with a single, distinctive row.
2. **Make the test row land past the first page under the reporter's role.** Check with the `explain` and the
   first-page count from [root-causes.md](root-causes.md), or with a client signed in as a seeded user. Seeded
   ids are often small and sort first; point the test row at the member with the largest id.
3. **Read the screen on the new code**: the list for the test record shows only its member.
4. **Read it on the old code**: `git stash`, touch the changed file, wait for the dev server to recompile,
   read the screen again. It must show the wrong list. If it shows the right one, the repro is not
   reproducing: go back to step 2, do not conclude the old code was fine.
5. `git stash pop`, then delete the seeded records so the local dataset stays neutral.

When the running app disagrees with a test that passes against the same database, print the runtime value
before debugging the logic. A dev server can keep a stale module after edits made in place (`sed -i` swaps the
file under the watcher), and touching the file may not wake it. Render the suspect constant into the page for
one request, compare it with the source, and when they differ ask the user before restarting their dev
server. Remove the probe afterwards.

For a list ordered by name, seed filler names that sort first (`Aaa Repro 0001` onwards, past the cap) and one
target that sorts last, then search the picker for the target as the reporter's role. For a visibility fix,
change a seeded member's setting locally (a job function, an archived flag) and give it one record on the day
on screen.

A screen that downloads through `fetch` and saves a blob is read in two steps: wait for the response to learn
the request's URL and query, then fetch that URL again with the page's own request context
(`page.request.get(url)`, same cookies) to read the body. The body of the response the page consumed can come
back empty, and an empty file looks like a filter that matched nothing.

Two traps in seeding and cleaning up:

- **Check that every seed insert succeeded before comparing.** An insert refused by a constraint (an
  exclusion constraint on overlapping bookings of one member is common) leaves the record missing, the old and
  new code then show the same screen, and the comparison proves nothing. Read the insert's output; pick a free
  slot and insert again.
- **Prefer a repro that does not submit on the old code.** Showing the old screen is enough; submitting
  creates rows in several tables. When the new code must submit to prove the result (one receipt with two
  lines), list what references the record before deleting it:

```sql
-- run locally: every table with a column pointing at a sale
select table_name from information_schema.columns
where table_schema = 'public' and column_name = 'sale_id';
```

Delete those rows, then the record, then restore the seeded visits' status, in one transaction. A seeded row
you updated rather than inserted is cleanup too: put back the value you read before changing it.

## Proving a fix below the screen

Some fixes change a rule the reporter never sees directly: an availability engine, a database function. Show
it on the screen where you can (the public slot list for a day, before and after), and prove the rule itself
against local data as well.

**A server loader**, run with the host's test runner so path aliases and server-only guards resolve: a
throwaway test that seeds one record, calls the loader, records the result and deletes the record in
`finally`. Test runners often swallow console output, so write the result to a file in the scratch directory.

```ts
// src/server/zz-repro.test.ts, deleted after the run
import { appendFileSync } from "node:fs";
import { it } from "vitest";

const OUT = process.env.REPRO_OUT ?? "/tmp/repro.out";
const log = (...parts: unknown[]) => appendFileSync(OUT, parts.join(" ") + "\n");

it("repro", async () => {
  // Import inside the test so the local environment variables are set first.
  const { loadDayAvailability, pickSlot } = await import("@/server/public-booking");
  const { seedClosure, removeRecord } = await import("./repro-seed");
  const id = await seedClosure("2031-03-04T11:00:00Z", "2031-03-04T13:00:00Z");
  try {
    const day = await loadDayAvailability("2031-03-04");
    log("under the record:", JSON.stringify(pickSlot(day, 12 * 60 + 30, 60)));
    log("after it:", JSON.stringify(pickSlot(day, 14 * 60, 60)));
  } finally {
    await removeRecord(id);
  }
}, 60_000);
```

Run it on the new code, `git stash push` the fixed file, run it again, pop. The two outputs side by side are
the proof: a slot under the record on the old code, none on the new, and the slot after it on both.

**A database function**, in one transaction that is rolled back: set the claims the function reads, seed,
call the live function, load the migration file inside the same transaction, call it again.

```sql
-- run locally with psql -v ON_ERROR_STOP=1 -f repro.sql
begin;
select set_config('request.jwt.claims', '{"sub":"<seeded auth user id>","role":"authenticated"}', true);
insert into appointments (tenant_id, kind, employee_id, resource_id, title, starts_at, ends_at)
values ('<tenant id>', 'block', null, null, 'repro', '2031-03-03 12:00+00', '2031-03-03 13:00+00');
select count(*)
from find_appointment_conflicts('<member id>', null, '[2031-03-03 12:15+00,2031-03-03 12:45+00)')
where title = 'repro';
\i supabase/migrations/<new migration>.sql
select count(*)
from find_appointment_conflicts('<member id>', null, '[2031-03-03 12:15+00,2031-03-03 12:45+00)')
where title = 'repro';
rollback;
```

**A data migration** that rewrites existing rows (merging copies, moving a column into a link table) is proven
the same way: in a rolled-back transaction, seed the shapes it must handle (two identical rows, an inactive
copy, a near copy that must stay apart, a visit line pointing at the row that goes away), run the migration's
step, read the result, roll back. Then apply the whole migration to the local database with the host's
non-destructive command (`supabase migration up`, not a reset) to run it over real local data. When a reset
that would also run the seed is refused, run the seed's changed section inside a rolled-back transaction.

Call the database tool directly in shell commands. A command held in a variable (`DB="docker exec ... -c"`,
then `$DB "insert ..."`) is not split into words under zsh; the insert never runs, both sides of the
comparison read the same data, and the repro looks like a fix that changed nothing. A shell function
(`D() { docker exec ... -c "$1"; }`) splits as expected. The same holds for a list of paths kept in one
variable and passed to `git stash push`: zsh hands it over as one path, nothing is stashed, and the "old
code" run is the new code. Write the paths out.

## Commit and push

- Stage the changed files by path; one commit for the fix, its siblings and the guard.
- Subject in the host's convention, for example `fix(booking): read every staff assignment past the
  1000-row cap`; the body says what the tenant saw, why, and what the guard holds; the last line is
  `Fixes #<number>`.
- `Fixes #N` closes the issue only when the commit reaches the default branch. On a preview or release
  branch the issue stays open until the merge; tell the user.
- A defect fixed inside a report that stays open (a request noted for later) is committed with `Refs #N`, not
  `Fixes #N`, so the merge does not close the request; label the issue as the host's request label.
- No tool attribution in the message. Check `git log -1 --format=%B` before pushing.
- Run the whole suite before every push, not the folders you touched: a guard over the help text or the docs
  (a word limit, a label check) lives in another folder and fails only in a full run, by then on a pushed
  commit. A test that fails on the old code as well is not yours: run it there, name it in the report, and
  leave it.
- Push only when the user asked for the issue to be resolved end to end, and only to the branch the host's
  policy names.
- Merging into the default branch deploys to production: do it only when the user asks for that merge. A bare
  "push" when the branch is already pushed most likely means it; say what is already pushed, list the commits
  the merge would ship, and ask. Before the merge, list the database migrations it carries when the host
  applies them on push, and follow the host's merge convention (a merge commit when its history has them).

## Checklist

- [ ] Every sibling found and classified; every defect fixed in one change
- [ ] Guard test fails on the old code with exactly the fixed lines, passes on the new; for new behaviour,
      the rule in a pure module with unit tests, and the server re-checking every client-supplied id
- [ ] Whole suite and linter green before every push; a failure that also fails on the old code named, not
      fixed
- [ ] A migration that rewrites rows proven in a rolled-back transaction, then applied without a reset
- [ ] Every seed insert confirmed; running app wrong on the old code, right on the new; seeded data removed
- [ ] A rule below the screen proven against local data on both codes; a database function in a rolled-back
      transaction
- [ ] Existing tests that encoded the old behaviour changed and named
- [ ] For a permission defect: the refusal names the permission, the matrix is held against the live policies,
      and the screen is checked as a role without the write and as one with it
- [ ] Commit with `Fixes #N` (or `Refs #N` when the report stays open), no attribution, pushed per the branch
      policy; the default branch merged only when asked
