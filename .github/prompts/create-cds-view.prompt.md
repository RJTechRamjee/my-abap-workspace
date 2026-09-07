---
description: "Generate a CDS interface or consumption view following the project's naming and annotation conventions."
agent: "agent"
argument-hint: "View purpose and base table/view, e.g. 'consumption view for Product based on ZRK_I_PRODUCT'"
---
Generate a read-only CDS model for: ${input:viewDescription:View purpose and base table or interface view}

Follow [abap-cloud-rap.instructions.md](../instructions/abap-cloud-rap.instructions.md) for naming
(`ZRK_I_<Name>` for interface views, `ZRK_C_<Name>` for consumption views) and annotation placement.

Produce as separate named source blocks:
1. CDS interface view(s) `ZRK_I_<Name>` — `DEFINE VIEW ENTITY` (no `sqlViewName`), released SAP
   CDS/tables as data sources only,
   associations exposed, `@Semantics` where relevant, no `@UI`.
2. CDS consumption view `ZRK_C_<Name>` — projected/calculated fields,
   `@Metadata.allowExtensions: true`, `@AccessControl.authorizationCheck: #CHECK`.
3. For an analytical model: add `@Analytics.dataCategory`, `@ObjectModel` on the interface view,
   annotate measures/dimensions.
4. Metadata extension `ZRK_C_<Name>` (.ddlx, same name as the view it annotates) if it backs a
   Fiori elements list/analytical page — otherwise annotate inline.
5. DCL `ZRK_DCL_<Name>` — base row-level restrictions on a released authorization CDS/aspect,
   only if row-level auth applies.

Rules:
- Released APIs only; no direct SAP table `SELECT` outside released CDS; language version
  *ABAP for Cloud Development*.
- No behavior — read only.
- Don't invent field names — mark unknowns `[CONFIRM in ADT]`.
- Show the generated DDL source before creating the object, confirm target package/transport,
  and end with the released-dependency list and their assumed API state.
