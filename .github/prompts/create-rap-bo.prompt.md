---
description: "Scaffold a managed RAP Business Object: interface view, behavior definition, projection view, behavior projection, behavior pool, and service binding."
agent: "agent"
argument-hint: "Business object name and base table, e.g. 'Travel based on ZRK_TRAVEL'"
---
Scaffold a complete managed RAP Business Object for: ${input:boDescription:Business object name and base table}

Follow [abap-cloud-rap.instructions.md](../instructions/abap-cloud-rap.instructions.md) and
[clean-abap.instructions.md](../instructions/clean-abap.instructions.md) naming, layout, and
conventions.

First state the consumer (Fiori Elements UI service vs. API-only), because that decides draft.
If it isn't clear from the request, ask before generating.

Produce every artifact as a separate, named source block (abapGit layout), each headed by a
one-line comment with the object name, type, and the requirement it implements:
1. CDS interface view(s) `ZRK_I_<Name>` — `DEFINE ROOT VIEW ENTITY` for the root plus child
   entities, associations to released SAP entities only, no `@UI`, no `sqlViewName`.
2. Behavior definition `ZRK_I_<Name>` — `managed`, `strict ( 2 )`,
   `implementation in class zrk_bp_i_<name> unique`, persistent table mapping, `etag master` on
   the last-changed field. Declare determinations/validations/actions traced to numbered
   business rules (R1, R2…) if given.
3. CDS projection view `ZRK_C_<Name>` — `provider contract transactional_query`,
   `@Metadata.allowExtensions: true`, `@AccessControl.authorizationCheck: #CHECK`.
4. Behavior projection `ZRK_C_<Name>` — `projection;`, `use create/update/delete`, `use action …`
   for each exposed action. Required for a transactional service; don't stop at the interface BDEF.
5. Metadata extension `ZRK_C_<Name>` (.ddlx) — all `@UI`/`@Search` annotations here, not inline.
6. Behavior pool class `zrk_bp_i_<name>` — local handler/saver classes, one method per rule,
   `READ`/`MODIFY ENTITIES` only, results via `MAPPED`/`REPORTED`/`FAILED`, `TODO(Rn)` where the
   requirement isn't explicit.
7. Draft table `ZRK_D_<Name>` and `with draft` on both BDEFs — UI-service BOs only; omit both for
   API-only BOs and say why.
8. Service definition `ZRK_SD_<Name>` + service binding `ZRK_UI_<Name>_O4` (UI) or
   `ZRK_API_<Name>_O4` (inbound API), OData V4.
9. DCL `ZRK_DCL_<Name>` only if row-level auth applies.

Rules:
- Language version *ABAP for Cloud Development*; released APIs only; Clean ABAP style; no
  `SELECT` in loops; exception classes, not `sy-subrc` swallowing.
- Don't invent SAP field/table names — mark unknowns `[CONFIRM in ADT]`.
- End with a "verify before activation" list of every released API/CDS/table used with its
  assumed API state, plus the activation order (table → interface view → BDEF → projection view →
  projection BDEF → behavior pool → service definition → binding).
- Note to run the `generate-abap-unit-tests` prompt next.
- Ask for the target package and transport request before creating objects if not already known —
  never assume `$TMP` unless the user explicitly says this is a throwaway/experiment.
