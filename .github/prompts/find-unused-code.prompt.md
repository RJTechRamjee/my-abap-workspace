---
description: "Find dead and unused custom code — unreferenced objects, unreachable branches, obsolete duplicates — and propose a safe, staged removal."
agent: "agent"
argument-hint: "Package or object range to scan, e.g. 'ZRK_CORE' or 'ZRK_CL_*'"
---
Scan for unused custom code in: ${input:scanScope:Package, namespace, or object range to scan}

Deleting the wrong thing is far more expensive than keeping it. Bias toward evidence.

## 1. Collect candidates
For each object in scope, gather usage evidence with the ADT tools — where-used, and any usage
statistics available:
- **Never referenced**: no where-used hits anywhere in the system.
- **Referenced only by other dead code**: reachable only from objects already on this list —
  resolve these as a graph, not one at a time.
- **Superseded**: a newer object does the same job (a RAP BO replacing a report, `_NEW`/`_V2`
  siblings, a copy that diverged).
- **Internally dead**: unreachable branches, unused private methods, unused parameters, commented
  blocks, variables written but never read.

## 2. Rank by confidence, and be honest about the blind spots
| Object | Type | Evidence | Confidence | Risk if wrong |
| --- | --- | --- | --- | --- |

Static where-used does **not** see: dynamic calls (`CALL METHOD (name)`, `CALL FUNCTION (fname)`,
`SUBMIT (prog)`), objects invoked from customizing/config tables, RFC or OData entry points called
from outside the system, batch jobs and job variants, BAdI/enhancement implementations, and
Fiori/UI5 apps calling a service. Explicitly check for these before calling anything unused —
and where you can't rule them out, say so and lower the confidence rather than hiding the doubt.

## 3. Propose a staged removal — never a bulk delete
1. **Deprecate**: mark obsolete, add a comment naming the replacement and the date.
2. **Observe**: leave it in place for at least one full business cycle (month-end, year-end —
   annual jobs are the classic thing that "wasn't used" until it was).
3. **Delete**: in its own transport, separate from functional change, so it can be reverted alone.

Group the deletions into transport-sized batches by dependency order.

## 4. Quick wins vs. investigate
Separate the obvious (a `ZZTEST_*` copy from 2019 with no references) from the genuinely uncertain
(a function module that might be called by a job variant). Give the uncertain ones a concrete
verification step, not a recommendation.

Rules:
- Read-only. Never delete or modify an object — this prompt produces a plan.
- Never recommend deleting something whose only evidence is "no where-used hits" if it is an RFC-
  enabled FM, an OData/service artifact, a BAdI implementation, or a report that could be in a
  job variant.
- Objects still referenced from a transport not yet imported everywhere are not unused.

---
**Chain**: `explain-abap` → **find-unused-code** → `pre-transport-check` (deletions travel in their own transport)
