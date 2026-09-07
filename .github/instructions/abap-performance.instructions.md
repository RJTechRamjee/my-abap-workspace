---
description: "Use when writing or reviewing database access, internal table processing, CDS views, or RAP behavior that runs over more than a handful of records."
applyTo: "**/*.abap,**/*.asddls"
---
# ABAP & CDS Performance

Code-to-data: let the database filter, join, aggregate and sort. Move only the result set into
ABAP. Most performance defects here are a loop doing work the DB should have done once.

## Open SQL
- `SELECT` only the fields you need into a strongly-typed target — never `SELECT *`. Narrow field
  lists also let the DB serve the query from an index alone.
- Never `SELECT` inside a `LOOP`. Use a `JOIN`, a CDS association, or one set-based read up front.
- Never `SELECT ... ENDSELECT` over mass data — read `INTO TABLE` in one call.
- Push `WHERE`, `ORDER BY`, `GROUP BY` and aggregates to the DB instead of filtering or `SORT`ing
  an oversized internal table afterwards.
- `SELECT SINGLE` is only deterministic with a **full key**. With a partial key it returns an
  arbitrary matching row — use `UP TO 1 ROWS` with an explicit `ORDER BY` when you need "the
  first" row by some rule.
- For genuinely large result sets, bound memory with `PACKAGE SIZE` and process per package.
- Check the `WHERE` clause against an existing index. If a frequent access path has no supporting
  index, say so — don't silently accept a full scan.

## FOR ALL ENTRIES — three traps
1. **Empty driver table means no restriction at all.** The `WHERE` condition is dropped and you
   select the entire table. Always guard: `IF lt_keys IS NOT INITIAL.`
2. **Duplicates are removed** from the result, like `DISTINCT`. Select the full key of the target
   table or you will silently lose rows.
3. It is split into several DB calls by the blocking factor, and it **bypasses the table buffer**
   — as does any `JOIN`. Prefer a `JOIN` or CDS association where the data model allows one.

## Internal tables
- Pick the type for the access pattern: `STANDARD` for append-and-iterate, `SORTED` for range
  reads and binary search, `HASHED` for repeated single-row lookups by unique key.
- `READ TABLE ... WITH KEY` on a `STANDARD` table is a linear scan. Use `WITH TABLE KEY` on a
  sorted/hashed table, or a secondary key, when the lookup sits inside a loop.
- Nested loops over two internal tables are O(n×m) — build a hashed lookup table instead.
- Use `FIELD-SYMBOL`s or `REFERENCE INTO` to modify rows in place rather than copying each row.
- `DELETE ADJACENT DUPLICATES` requires a prior `SORT` on the same fields — otherwise it silently
  under-deletes.

## CDS views
- Keep the view stack shallow. Every layer of nesting is re-evaluated; deep stacks of views that
  each add one field are the most common CDS performance problem.
- Filter as early (as deep) in the stack as possible so upper layers work on fewer rows.
- Associations are only resolved when a path expression actually requests them — prefer them over
  eager joins for optional data.
- Avoid expensive `CASE` logic, currency/unit conversion, and `UNION` in high-volume base views;
  push them to the consumption layer where the result set is already filtered.
- Annotate analytical models properly (`@Analytics.dataCategory`) — an OLTP-shaped view used for
  reporting will not perform.

## RAP
- EML is set-based. Pass all instances to one `MODIFY ENTITIES` / `READ ENTITIES` call — never
  call EML inside a `LOOP`.
- Determinations and validations receive a set of keys. Implement them set-based; a `SELECT` per
  key inside a determination multiplies with every mass update.
- Read what you need in one go in the handler rather than re-reading per instance.

## Before claiming something is faster
- Measure: SQL trace (ST05) for database questions, ABAP runtime analysis (SAT / ADT Profiling)
  for ABAP-side questions. State which one you used.
- Give the actual selectivity or row count you're reasoning about. "This table is large" is not an
  analysis — if you don't know the volume, say `[CONFIRM in ADT]` and reason about both cases.
- Don't trade readability for a micro-optimization that no measurement supports. Clean ABAP wins
  by default; performance wins when there is a number behind it.
