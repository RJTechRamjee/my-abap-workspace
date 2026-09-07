---
description: "Decide the Clean Core extensibility approach for a requirement (standard config vs in-app vs side-by-side vs classic exception)."
agent: "ask"
argument-hint: "Requirement, code, or design to assess"
---
Assess the requirement/code/design below for Clean Core extensibility fit: ${input:requirement:Requirement, code, or design to assess}

This is an advisory decision, not a system inspection: it runs in **ask** mode with no ADT
access. Reason from the requirement and the tier rules. If a step genuinely needs live release
state, say which object to verify and hand off to `abap-cloud-readiness-check` rather than
guessing.

Follow the extensibility tier order in
[abap-cloud-rap.instructions.md](../instructions/abap-cloud-rap.instructions.md).

1. Challenge whether standard config or key-user in-app tools (custom fields/logic, custom CDS
   views) already cover it before reaching for developer extensibility.
2. Classify the needed approach, in preference order: standard config → in-app key-user →
   in-app developer (ABAP Cloud/RAP) → side-by-side (BTP) → classic extensibility (exception
   only, needs justification).
3. Name concrete SAP mechanisms, not generic advice (e.g. "Custom Logic BAdI via Extensibility
   app", "SAP Integration Suite iFlow with released OData V4 API").
4. If classic extensibility is the answer, require a dated, signed-off exception: state the
   owner, the reason no released alternative exists, and a remediation target.

Output:
- Recommended tier + concrete mechanism.
- One-paragraph justification for why higher tiers don't fit.
- Clean Core checklist across all six dimensions (software stack, extensions, data,
  integrations, processes, operations) — state a position on each dimension the requirement
  touches, not only "extensions".

---
**Chain**: `abap-cloud-readiness-check` → **clean-core-extensibility-check** → `create-rap-bo` / `create-cds-view`
