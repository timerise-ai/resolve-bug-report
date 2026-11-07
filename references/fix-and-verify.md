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

## Commit and push

- Stage the changed files by path; one commit for the fix, its siblings and the guard.
- Subject in the host's convention, for example `fix(booking): read every staff assignment past the
  1000-row cap`; the body says what the tenant saw, why, and what the guard holds; the last line is
  `Fixes #<number>`.
- `Fixes #N` closes the issue only when the commit reaches the default branch. On a preview or release
  branch the issue stays open until the merge; tell the user.
- No tool attribution in the message. Check `git log -1 --format=%B` before pushing.
- Push only when the user asked for the issue to be resolved end to end, and only to the branch the host's
  policy names.

## Checklist

- [ ] Every sibling found and classified; every defect fixed in one change
- [ ] Guard test fails on the old code with exactly the fixed lines, passes on the new
- [ ] Full suite and linter green
- [ ] Running app: wrong on the old code, right on the new; seeded data removed
- [ ] Commit with `Fixes #N`, no attribution, pushed per the branch policy
