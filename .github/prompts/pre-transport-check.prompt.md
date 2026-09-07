---
description: "Gate a transport before release: review its contents and diff, confirm ATC and unit tests are clean, and catch $TMP leftovers and non-released dependencies."
agent: "agent"
argument-hint: "Transport request, e.g. 'A4HK900123'"
---
Run the pre-release gate for transport: ${input:transport:Transport request number}

Nothing here writes to the system. The outcome is a go/no-go with evidence.

## 1. Inventory
Use `abap_transport-get` to list every object in the request. Show:

| Object | Type | Package | New/Changed | Notes |
| --- | --- | --- | --- | --- |

Flag immediately:
- Objects in `$TMP` or a non-transportable package that were meant to move.
- Objects whose names break the `ZRK_` conventions in
  [copilot-instructions.md](../copilot-instructions.md#naming-convention).
- Deletions, and anything a downstream system may depend on.
- Objects that look unrelated to the stated purpose of the transport — scope creep is the most
  common cause of a failed import.

## 2. Diff review
Use `abap_transport-unifiedDifference` for the changes. Prefer the server-side diff and summary
over pulling full sources — keep the payload small and only read the full source of objects whose
diff actually needs context.

Review the diff for: leftover debug statements and `BREAK-POINT`s, commented-out code, hardcoded
values that belong in customizing, hardcoded clients/users/paths, and anything Clean Core
forbids ([copilot-instructions.md](../copilot-instructions.md)).

## 3. Quality gates
- **ATC**: run `abap_atc_run` scoped to the transport. Prio 1 and 2 findings block release. Any
  pseudo-comment suppression added in this transport must be justified in the summary — see
  [atc-fix](atc-fix.prompt.md).
- **Unit tests**: run `abap_run_unit_tests` over the touched objects and packages. Report
  pass/fail counts, not "tests exist".
- **Dependencies**: every SAP object referenced must be released. List them with API state; mark
  unverified ones `[CONFIRM in ADT]`.
- **Completeness**: does the request contain everything the change needs? A CDS view without its
  metadata extension, a BDEF without its behavior projection, or a service definition without its
  binding will activate in DEV and fail in QA.

## 4. Verdict
```
Verdict: GO / NO-GO
Blocking:   <list, or none>
Non-blocking follow-ups: <list>
Evidence:   ATC <before/after per prio> · Tests <passed/failed> · Objects <n>
```
Say NO-GO when a gate fails. Do not soften a blocking finding into a "follow-up" to get to GO.

Rules:
- Read-only. Never release, reassign, or modify the transport.
- Don't claim a gate passed without having run it — say "not run" and why.

---
**Chain**: `atc-fix` → **pre-transport-check** → release. Run **last**.
