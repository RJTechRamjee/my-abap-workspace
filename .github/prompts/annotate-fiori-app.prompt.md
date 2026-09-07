---
description: "Generate or refine the @UI metadata extension for a Fiori Elements list report and object page over an existing CDS consumption view."
agent: "agent"
argument-hint: "Consumption view and app shape, e.g. 'ZRK_C_TRAVEL, list report + object page'"
---
Build the Fiori Elements annotations for: ${input:fioriTarget:Consumption view and the app shape (list report, object page, analytical)}

Follow [fiori-annotations.instructions.md](../instructions/fiori-annotations.instructions.md) and
[abap-cloud-rap.instructions.md](../instructions/abap-cloud-rap.instructions.md).

## 0. Read the model first
Read the consumption view and its interface view before annotating. You cannot annotate a field
that isn't projected. List the available elements, their types, and any associations — then
annotate against that list, never against assumed field names.

Confirm the view has `@Metadata.allowExtensions: true`; if not, that change comes first.

## 1. Ask what the user actually needs to see
Don't annotate every field. Establish, or ask:
- Which columns matter in the list, and the default sort.
- Which fields users filter on.
- What identifies an object (header title/description).
- Which fields are editable vs. display-only.
If the requirement doesn't say, propose a minimal set and mark it as a proposal.

## 2. Produce the metadata extension
One `.ddlx` source block, `@Metadata.layer: #CORE`, in this order:
1. `@UI.headerInfo` — typeName, typeNamePlural, title, description.
2. `@UI.presentationVariant` — default sort order (always set one).
3. Per-element: `@UI.lineItem`, `@UI.selectionField`, `@UI.identification`, `@UI.fieldGroup`,
   `@UI.dataPoint` as needed, positions in steps of 10.
4. `@UI.facet` — the object-page structure, header facets first.

## 3. Data-level annotations go on the view, not the extension
Call out separately any `@Semantics`, `@ObjectModel.text.element`,
`@Consumption.valueHelpDefinition`, or criticality element that belongs in the CDS view — and
give that source block too. Amounts without a currency reference and codes without a text
association are defects, not style preferences.

## 4. Wire up the rest
- Actions shown in the UI must be exposed in the behavior projection (`use action …`).
- Declare `@Common.SideEffects` for any field a determination changes, or the UI will show stale
  values until refresh.
- Confirm every annotated field is exposed in the service definition, not just the view.

## 5. Hand back a check
End with the review checklist from the instructions file, answered — not just restated. Then say
how to preview: activate the service binding and use its preview, and what to look for.

Rules:
- Don't invent element names — read them, or mark `[CONFIRM in ADT]`.
- Labels come from data elements where possible; never hardcode a label that should be translatable.
- Keep the default column and filter set small; every column costs list-load time.
- Object changes and activation are gated: show what you'll change and wait for confirmation.

---
**Chain**: `create-rap-bo` → **annotate-fiori-app** → `expose-odata-service` → `debug-fiori-ui`
