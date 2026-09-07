---
description: "Diagnose and fix a slow ABAP program, CDS view, or OData request — measure first, find the real bottleneck, fix the access path, prove the improvement."
agent: "agent"
argument-hint: "What is slow, e.g. 'ZRK_CL_ORDER_MANAGER->GET_OPEN_ITEMS' or 'ZRK_UI_TRAVEL_O4 list page'"
---
Diagnose the performance problem in: ${input:slowTarget:Program, CDS view, class/method, or OData request that is slow}

Apply [abap-performance.instructions.md](../instructions/abap-performance.instructions.md).
Read `system-info.md` first if it exists — HANA vs. AnyDB changes the right answer.

## 1. Get the number before the theory
Ask for, or establish, the actual measurement — do not start guessing from the source:
- **Database-side**: SQL trace (ST05) — which statement, how many executions, rows returned,
  time per execution, and whether an index was used.
- **ABAP-side**: runtime analysis (SAT / ADT Profiling) — gross vs. net time per call.
- **OData/RAP**: which request, list vs. object page, `$expand`/`$filter` in play, page size.

If no measurement exists, say so and name the one trace to run. A code review is not a diagnosis:
the statement you'd flag by eye is often not the one burning the time.

## 2. Classify the bottleneck
State which of these it is before proposing anything:
- **Too many executions** — a `SELECT` in a loop, EML inside a `LOOP`, an N+1 across an association.
- **Too many rows** — missing or non-selective `WHERE`, filtering in ABAP after the fact,
  an unguarded `FOR ALL ENTRIES` on an empty driver table pulling the whole table.
- **Too much per row** — `SELECT *`, expensive `CASE`/conversion in a deep CDS layer, per-row
  currency conversion.
- **Wrong access path** — no supporting index, a non-selective leading field, a function or
  concatenation on an indexed column defeating the index.
- **Not the database at all** — nested loops over internal tables, linear `READ TABLE` in a loop,
  quadratic string building.

## 3. Fix the cause
- Give the corrected code, and name which rule from the performance instructions it satisfies.
- Prefer the structural fix (set-based read, CDS association, hashed lookup) over a tweak.
- Where a new index is genuinely the answer, say so explicitly, with the field order and the
  reason — and note it as a DB change needing its own review, not a code fix.
- If the fix changes the result set (deduplication, sort order, null handling), call that out —
  a faster query with different results is a bug.

## 4. Prove it
Re-measure the same way you measured in step 1 and show before/after: executions, rows, time.
Run `abap_run_unit_tests` on the touched objects. An optimization with no after-number and no
green tests is a proposal, not a fix.

Rules:
- No unmeasured claims. "This will be faster" needs either a trace or a stated row count and
  selectivity — otherwise mark it `[CONFIRM in ADT]` and reason about both cases.
- Don't sacrifice Clean ABAP readability for a micro-optimization no measurement supports.
- Object changes and activation are gated: list what you'll modify and wait for confirmation.

---
**Chain**: `analyze-dump` → **debug-slow-sql** → `generate-abap-unit-tests` → `atc-fix`
