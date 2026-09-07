---
description: "Analyze an ABAP short dump, runtime error, or syntax/activation error down to root cause and a concrete fix."
agent: "agent"
argument-hint: "Paste the short dump / error text, or give the ST22 dump ID"
---
Analyze this ABAP error: ${input:dumpText:Paste the short dump, runtime error, or activation error}

## 1. Read the dump properly
Extract and state, before interpreting anything:
- **Exception / error category** (e.g. `CX_SY_ITAB_LINE_NOT_FOUND`, `TIME_OUT`,
  `DBSQL_DUPLICATE_KEY_ERROR`, `MESSAGE_TYPE_X`, `CALL_FUNCTION_NOT_FOUND`).
- **Terminating location** — program/class/include, method, line.
- **Trigger location** — where it was *called from*. The failing line is often a symptom of a
  caller passing bad data.
- **Context**: user, client, time, work-process type (dialog/batch/update/RFC), and any
  parameter/variable values the dump captured.

Read the actual source at the terminating line with the ADT tools rather than reasoning from the
dump text alone.

## 2. Root cause, not restatement
"Line 412 raised CX_SY_CONVERSION_NO_NUMBER" is the dump, not the cause. Say *why* the data got
there. Distinguish:
- **Data problem** — unexpected/empty/malformed input, missing customizing, a record deleted
  between read and use.
- **Logic problem** — unguarded `READ TABLE`, missing `sy-subrc` check, off-by-one, uninitialized
  reference, division by zero.
- **Environment problem** — missing authorization, locked object, timeout under volume,
  memory limit, transport/activation state, missing object in this client.
- **Concurrency problem** — duplicate key from a race, lost update, missing enqueue.

Timeouts (`TIME_OUT`) and memory dumps are usually performance defects wearing a different hat —
hand those to [debug-slow-sql](debug-slow-sql.prompt.md) once you've confirmed the pattern.

## 3. The fix
- The concrete code change, following [clean-abap.instructions.md](../instructions/clean-abap.instructions.md).
- Explicitly say whether this is the *fix* or a *guard*. Catching `CX_SY_ITAB_LINE_NOT_FOUND`
  around a read that should never fail hides the real defect — prefer fixing why the row is missing.
- Never "fix" a dump by widening a `CATCH` and swallowing it.
- A regression test that reproduces the failure: name the test method and what it asserts.

## 4. Blast radius
Where else does this pattern occur? Use where-used and search for the same construct — one
unguarded `READ TABLE` usually has siblings. List them; don't fix them silently.

Rules:
- Don't invent dump fields or values that weren't in the text you were given.
- If the dump is truncated or you need the caller's data to be sure, say exactly what's missing
  rather than producing a confident guess.
- Mark anything unverified `[CONFIRM in ADT]`.

---
**Chain**: **analyze-dump** → `debug-slow-sql` (if it's a timeout/memory dump) → `generate-abap-unit-tests` for the regression test
