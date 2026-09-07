---
description: "Decide the Clean Core extensibility approach for a requirement (standard config vs in-app vs side-by-side vs classic exception)."
agent: "agent"
argument-hint: "Requirement, code, or design to assess"
---
Assess the requirement/code/design below for Clean Core extensibility fit: ${input:requirement:Requirement, code, or design to assess}

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
