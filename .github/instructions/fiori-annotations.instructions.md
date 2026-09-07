---
description: "Use when writing or reviewing @UI, @Search, @Consumption, @Semantics or @ObjectModel annotations for SAP Fiori Elements — list report, object page, value helps, and metadata extensions."
applyTo: "**/*.asddlx,**/*.asddls"
---
# CDS Annotations for Fiori Elements (OData V4)

Fiori Elements renders *only* what the annotations describe. There is no UI code to debug — a
missing field is almost always a missing or misplaced annotation. Reference implementation:
`SAP-samples/abap-platform-fiori-feature-showcase`.

## Where annotations belong
- `@UI` / `@Search` presentation annotations go in the **metadata extension** (`.ddlx`) for the
  consumption view, not inline in the CDS view. The view must carry
  `@Metadata.allowExtensions: true`.
- Set `@Metadata.layer: #CORE` for the developer layer; `#CUSTOMER` is for customer adaptation.
- Semantic annotations that describe the *data* (`@Semantics`, `@ObjectModel.text.element`,
  currency/UoM references) stay on the **CDS view**, not the extension — they are data meaning,
  not presentation.
- Never annotate an interface view (`ZRK_I_*`) with `@UI`.

## Element position numbering
Number `position` in steps of 10 (`10`, `20`, `30`). Inserting a field later then needs no
renumbering. Same for `@UI.facet` positions.

## List report
- `@UI.lineItem: [{ position: 10, label: '…' }]` — the table columns. Keep the default column
  set small; every column is fetched on every list load.
- `@UI.selectionField: [{ position: 10 }]` — the filter bar. Only fields users actually filter on,
  and they must be indexed or selective, or the list page will crawl.
- `@Search.searchable: true` on the view plus `@Search.defaultSearchElement: true` on the fields
  the free-text search should cover. Without both, the search box does nothing.
- `@UI.headerInfo` defines `typeName`, `typeNamePlural`, `title`, and `description` — this drives
  the page title and the object list text, and its absence is very visible.
- `@UI.presentationVariant` for the default sort order and visualization;
  `@UI.selectionVariant` for pre-set filters. Define a default sort — an unsorted list page
  returns rows in arbitrary DB order.

## Object page
- `@UI.facet` builds the page structure:
  - `#COLLECTION` groups facets; `#FIELDGROUP_REFERENCE` points at a `@UI.fieldGroup`;
    `#LINEITEM_REFERENCE` embeds a child entity's table via an association.
  - `purpose: #HEADER` puts a facet in the object-page header; `#STANDARD` in the body.
  - Every facet needs `id` when other facets or actions reference it.
- `@UI.identification: [{ position: 10 }]` — the general-information fields and the place where
  object-page actions appear.
- `@UI.fieldGroup: [{ qualifier: '…', position: 10 }]` — grouped sections, referenced by a facet's
  `targetQualifier`. A qualifier typo silently renders an empty section.
- `@UI.dataPoint` for header KPIs, ratings, and progress indicators.

## Value helps and texts
- `@Consumption.valueHelpDefinition: [{ entity: { name: 'ZRK_I_…', element: '…' } }]` — bind to a
  released or own CDS view, never a raw table. Add `additionalBinding` to filter the help by
  another field on the page.
- Text for a code field: `@ObjectModel.text.element: ['…']` on the code, or
  `@ObjectModel.text.association` where the text comes over an association. Without it the UI
  shows the raw key.
- Prefer data element labels; use `@EndUserText.label` only where the field needs a different
  label here. Labels must be translatable — never hardcode them into a UI-only literal.

## Semantics that change rendering
- Amounts **must** carry `@Semantics.amount.currencyCode: 'CURRENCY_FIELD'`, quantities
  `@Semantics.quantity.unitOfMeasure: 'UNIT_FIELD'`, and the referenced field must be exposed in
  the same projection. Missing this is the most common cause of wrong decimals and misaligned
  numbers.
- `@Semantics.user`, `@Semantics.systemDateTime.*` for created/changed-by fields.
- Criticality: `@UI.lineItem: [{ criticality: 'STATUS_CRITICALITY' }]` pointing at an integer
  element (1 red / 2 yellow / 3 green). Compute it in the CDS view — never in the UI layer.
- `@UI.hidden` (static or bound to a Boolean element) to hide conditionally;
  `@UI.masked` for sensitive values.

## Actions
- Expose a RAP action on the UI with
  `@UI.lineItem: [{ type: #FOR_ACTION, dataAction: 'ActionName', label: '…' }]`, and the same
  under `@UI.identification` for the object page.
- `invocationGrouping: #CHANGE_SET` when a mass action must succeed or fail as one unit.
- The action must be exposed in the **behavior projection** (`use action …`) or the button
  appears and fails at runtime.

## Side effects — the annotation people forget
When a RAP determination changes a field the user didn't type in, the UI does not know to refresh
it. Declare `@Common.SideEffects` on the triggering field, naming the `TargetProperties` or
`TargetEntities` to reload. Symptom of a missing one: "the value is right in the database but the
screen shows the old one until I refresh."

## Draft
- Draft-enabled UI services need draft actions and `@UI` annotations consistent across the draft
  and active instances.
- Don't annotate the draft administrative fields for display.

## Review checklist
- Every `@UI.lineItem`/`selectionField` field is actually exposed in the projection view **and**
  in the service definition — exposure is the usual missing link, not the annotation.
- Currency/UoM reference fields are exposed alongside their amount/quantity.
- Facet `targetQualifier` values match a real `fieldGroup` qualifier.
- Actions annotated in the UI exist in the behavior projection.
- No `@UI` on interface views; no business logic hidden in annotations.
