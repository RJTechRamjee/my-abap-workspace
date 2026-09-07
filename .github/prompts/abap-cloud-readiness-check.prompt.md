---
description: "Check whether ABAP code or a design is ABAP Cloud-compliant (released APIs only, portable language subset) ahead of a migration or new build."
agent: "agent"
argument-hint: "Code, object name, or design to assess"
---
Assess ABAP Cloud readiness for: ${input:target:Code, object name, or design to assess}

Follow [abap-cloud-rap.instructions.md](../instructions/abap-cloud-rap.instructions.md) and
[copilot-instructions.md](../copilot-instructions.md) Clean Core rules.

Check for:
- Restricted language scope violations: `EXPORT/IMPORT TO MEMORY`, dynamic subroutine pools,
  classic dynpro (`CALL SCREEN`), `NATIVE SQL`/`EXEC SQL`, `CALL TRANSACTION`/`DIALOG`.
- Non-released table/CDS/function-module/class/BAdI usage (anything without a confirmed
  "Released" API state — verify via the connected ADT tools where possible).
- Object type/pattern: favor RAP Business Objects and released CDS over classic BAPI/report patterns.
- Namespace/packaging fit for ABAP Cloud development scope.

Output a verdict table (Object | Verdict ✅/⚠️/❌ | Blocking issues | Effort S/M/L), then Quick
Wins vs Redesign Candidates, then a suggested migration sequence.

Don't call something "Ready" just because it compiles/runs today on a classic system — Ready
means release-compliant. Mark anything you can't verify live `[CONFIRM in ADT]`.
