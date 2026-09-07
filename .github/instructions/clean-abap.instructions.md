---
description: "Use when writing, reviewing, or refactoring ABAP classes, methods, or programs. Covers Clean ABAP naming, structure, error handling, testing, and formatting."
applyTo: "**/*.abap"
---
# Clean ABAP

Condensed from SAP's Clean ABAP styleguide (`SAP/styleguides`). Where this file is silent, the
styleguide governs — cite the rule name when you apply one.

## Naming
- Repository object prefixes (`ZRK_CL_`, `ZRK_IF_`, `ZRK_CX_`, …) are in
  [copilot-instructions.md](../copilot-instructions.md#naming-convention).
- **Deliberate deviation from Clean ABAP § "Avoid encodings, esp. Hungarian notation and
  prefixes":** this project *keeps* the classic prefixes (`lv_`, `ls_`, `lt_`, `lo_`, `iv_`,
  `is_`, `it_`, `io_`, `ev_`, `cv_`, `rv_`, `rs_`, `rt_`, `ro_`), for consistency with existing
  ABAP. Apply them consistently; don't flag their presence as a finding. Everything else in
  § Names still applies.
- Descriptive, pronounceable names in snake_case; nouns for classes, verbs for methods.
- Avoid noise words (`data`, `info`, `object`, `_do_`) — the prefix already carries the type.
- One word per concept across the codebase (don't mix `create`/`generate`/`build` for the same operation).
- Booleans read as questions: `is_valid`, `has_entries`.

## Constants
- No magic numbers or literals in logic — name them.
- Prefer `ENUM` to constants interfaces; otherwise group related constants in a class, not a
  loose `CONSTANTS` list.

## Variables
- Prefer inline (`DATA(...)`, `FIELD-SYMBOL(...)`) to up-front declarations; declare at first use.
- Don't chain up-front declarations, and don't use a variable outside the block it belongs to.

## Internal tables
- Pick the table type deliberately: `STANDARD` to append and iterate, `SORTED` for range access,
  `HASHED` for unique-key lookups. See
  [abap-performance.instructions.md](abap-performance.instructions.md) for the cost side.
- **Avoid `DEFAULT KEY`** — it is rarely what you mean and quietly changes sort/read semantics.
  State the key explicitly, or use `EMPTY KEY` when there is genuinely none.
- Prefer `INSERT INTO TABLE` to `APPEND TO` (works for every table type).
- Prefer `line_exists( )` to a `READ TABLE` whose result you only test for `sy-subrc`.
- Prefer `LOOP AT ... WHERE` to a `LOOP` wrapping an `IF`.

## Booleans
- Use `abap_bool`, compare against `abap_true` / `abap_false`, set with `xsdbool( )`.
- A Boolean input parameter usually means the method does two things — split it instead.

## Conditions and control flow
- Make conditions positive; prefer `IS NOT` to `NOT ... IS`.
- Extract complex conditions into a well-named method returning `abap_bool`.
- Prefer `CASE` to long `ELSEIF` chains. No empty `IF` branches.
- Fail fast: validate and exit at the top. Use an explicit `IF ... RETURN. ENDIF.` guard clause —
  reserve `CHECK` for loop bodies, where its exit semantics are what readers expect.
- Keep nesting depth low; extract rather than indent.

## Classes and scope
- Prefer objects to static classes; prefer composition to inheritance.
- `FINAL` unless designed for inheritance; members `PRIVATE` by default, `PROTECTED` only when
  a subclass genuinely needs them.
- Prefer `NEW` to `CREATE OBJECT`.
- Prefer several well-named static creation methods to one constructor full of optional parameters.
- Public instance methods should be part of an interface — that's what makes them mockable.
- Don't mix stateful and stateless responsibilities in one class.

## Methods
- Do one thing, do it well, do it only. Keep methods small; descend one level of abstraction per method.
- A method handles the happy path *or* error handling, not both.
- Prefer functional calls: omit `RECEIVING`, the optional `EXPORTING` keyword, the parameter name
  in single-parameter calls, and the `me->` self-reference.
- Aim for fewer than three `IMPORTING` parameters; split the method rather than adding `OPTIONAL`.
- `RETURN`, `EXPORT`, or `CHANGE` exactly one parameter. Prefer `RETURNING`; returning large
  tables is fine. Don't combine `RETURNING` with `EXPORTING`/`CHANGING`.
- Name a generic returning parameter `rv_result` / `rt_result`.

## Error handling
- Prefer exceptions to return codes; never let a failure slip through silently.
- Class-based exceptions only, in a project superclass (`ZRK_CX_*`), with subclasses so callers
  can distinguish cases.
- Pick the right base: `CX_STATIC_CHECK` for errors the caller can reasonably handle,
  `CX_NO_CHECK` for usually unrecoverable ones, `CX_DYNAMIC_CHECK` where the caller can rule the
  error out. Dump for the truly unrecoverable.
- Prefer `RAISE EXCEPTION NEW` to `RAISE EXCEPTION TYPE`.
- Wrap foreign exceptions at your boundary instead of letting them leak through your API.
- Check `sy-subrc` immediately after the one statement that sets it — never a chain of them.
- Never swallow an exception: handle, log, or re-raise.

## Language
- Modern syntax: constructor expressions (`VALUE`, `NEW`, `COND`, `SWITCH`, `CORRESPONDING`,
  `FILTER`, `REDUCE`), string templates (`|...|`), backtick literals for character strings.
- Avoid obsolete elements: `MOVE`, `COMPUTE`, `ADD`/`SUBTRACT`, `OCCURS`, header lines,
  `PERFORM` without interfaces.
- Mind the legacy: match the surrounding style when editing old code rather than half-converting it.

## Comments
- Explain *why*, not *what*. Comments are not an excuse for a bad name — rename instead.
- Comment with `"` (not `*`), on the line before the statement they describe.
- Delete dead code instead of commenting it out; no manual versioning history in source.
- `TODO`/`FIXME`/`XXX` must carry an identifier and a reason.
- ABAP Doc for public APIs only — never a header that restates the signature.
- Prefer pragmas (`##PRAGMA`) to pseudo comments (`"#EC`) where both exist.

## Formatting
- Run the ABAP Formatter (pretty printer) with the team's settings before activating.
- One statement per line; reasonable line length; close brackets at line end.
- A single blank line separates things — never two.
- Don't align type clauses or chain assignments; align only assignments to the same object.

## Testing
- Local test class `ltcl_*` inside the class under test; `FOR TESTING DURATION SHORT
  RISK LEVEL HARMLESS` unless it genuinely needs more.
- Test publics against interfaces, not private internals. Write testable code: inject
  collaborators through the constructor so `cl_abap_testdouble` can replace them.
- Name the code under test `cut` (or something meaningful) and extract the call to it.
- Given/When/Then in every test; the "When" is **exactly one call**.
- Test method names state what's given and what's expected.
- Few, focused assertions with the right assert type; assert content, not quantity.
- Use `cl_abap_unit_assert=>fail( )` to check an expected exception was raised; forward
  unexpected exceptions rather than catching and failing.
- Constants for test data, so the meaningful values stand out. Don't obsess over coverage
  numbers — cover behaviour and boundaries.
