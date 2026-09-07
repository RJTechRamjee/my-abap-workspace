---
description: "Use when writing, reviewing, or refactoring ABAP classes, methods, or programs. Covers Clean ABAP naming, structure, error handling, and formatting rules."
applyTo: "**/*.abap"
---
# Clean ABAP

## Naming
- Repository object prefixes (`ZRK_CL_`, `ZRK_IF_`, `ZRK_CX_`, …) are defined in
  [copilot-instructions.md](../copilot-instructions.md#naming-convention) — follow that table, and
  don't introduce a second prefix family.
- Intention-revealing names; no abbreviations that aren't standard ABAP idiom
  (`lv_`, `ls_`, `lt_`, `lo_`, `iv_`, `is_`, `it_`, `io_`, `rv_`, `rs_`, `rt_`, `ro_`).
- Booleans read as questions: `is_valid`, `has_entries`.
- One word per concept across the codebase (don't mix `create`/`generate`/`build` for the same operation).

## Methods
- Small, single-responsibility methods — extract until each method does one thing.
- Prefer functional style: return a value (`RETURNING`) over `EXPORTING`/`CHANGING` when there's one result.
- Max ~3 parameters where practical; group related data into a structure otherwise.
- Avoid deep nesting: use guard clauses (early `RETURN`) instead of nested `IF`. Prefer an explicit
  `IF … RETURN. ENDIF.` over `CHECK` outside of loops — `CHECK`'s exit behaviour is easy to misread.
- Avoid `EXPORTING`-only side-effect methods when a function would do — but never fake a
  side effect as a return value either.

## Error Handling
- Use class-based exceptions instead of `sy-subrc` chains where feasible; check `sy-subrc`
  immediately after the one statement that sets it.
- Prefer specific exception classes over generic `cx_root`/`cx_static_check` catches.
- Never silently swallow exceptions — log or re-raise.

## Language & Syntax
- Use modern ABAP syntax: inline declarations (`DATA(...)`, `FIELD-SYMBOL(...)`), constructor
  expressions (`VALUE`, `NEW`, `COND`, `SWITCH`, `REDUCE`), string templates (`|...|`).
- Avoid obsolete statements: `MOVE`, `COMPUTE`, `ADD`/`SUBTRACT` (use `+=`/`-=`), chained `PERFORM`
  without interfaces, `OCCURS`/header lines.
- Avoid `SELECT *`; select only needed fields into a strongly-typed structure/table.
- Never `SELECT` inside a `LOOP` — use `FOR ALL ENTRIES`, a `JOIN`, or a CDS association instead.

## Comments
- Comments explain *why*, not *what* — the code should be self-explanatory for *what*.
- Delete commented-out code; rely on version control history instead.
- No redundant method-header doc comments that just restate the signature.

## Formatting
- One statement per line; keep lines short enough to read without horizontal scrolling.
- Consistent indentation via pretty printer defaults; don't hand-align columns.

## Testing
- Unit tests live in a local test class `ltcl_<name>` inside the class under test — never a
  separate global test class.
- `FOR TESTING DURATION SHORT RISK LEVEL HARMLESS` unless the code genuinely needs otherwise.
- Arrange/Act/Assert per test method; one business rule per method; deterministic (inject
  date/time/random, no real DB, no `WAIT`, no RFC).
- Depend on interfaces (`ZRK_IF_*`) and inject via constructor so collaborators can be doubled
  with `cl_abap_testdouble`.
