---
description: "Troubleshoot a Fiori Elements app that renders wrong — missing field, empty section, dead filter, stale value, action that fails — down to the annotation or exposure that's at fault."
agent: "agent"
argument-hint: "What's wrong, e.g. 'Status column missing from ZRK_UI_TRAVEL_O4 list page'"
---
Diagnose this Fiori Elements problem: ${input:fioriBug:What the app does vs. what it should do}

Follow [fiori-annotations.instructions.md](../instructions/fiori-annotations.instructions.md).
Fiori Elements has no UI code — the defect is in the annotation, the projection, the exposure, or
the behavior projection. Work down the chain in order; the failure is usually further down than
it looks.

## The chain — check in this order
For a **field that doesn't appear**, the field must survive every one of these:
1. Present in the interface view (`ZRK_I_*`)?
2. Projected into the consumption view (`ZRK_C_*`)? — *most common failure*
3. Exposed in the service definition (`expose` covers the entity, and the entity carries the field)?
4. Annotated (`@UI.lineItem` / `identification` / `fieldGroup`) with a valid position?
5. If in a fieldGroup: does the facet's `targetQualifier` match the qualifier exactly?
6. Is `@Metadata.allowExtensions: true` set and the metadata extension **active**?
7. Is it hidden — `@UI.hidden`, a bound hidden element, or an authorization/DCL filter?
8. Is the service binding published and its metadata refreshed after your change?

State which step failed. Don't propose a fix before you've located the step.

## Symptom → likely cause
| Symptom | Look at |
| --- | --- |
| Column/field missing | Steps 2–4 above, in that order |
| Section renders empty | `targetQualifier` typo, or every field in the group is hidden |
| Filter bar field does nothing | `@UI.selectionField` present but field not filterable/exposed |
| Free-text search does nothing | `@Search.searchable` on the view *and* `@Search.defaultSearchElement` on fields |
| Amount shows wrong decimals / odd alignment | Missing `@Semantics.amount.currencyCode`, or currency field not exposed |
| Code shows raw key, no text | Missing `@ObjectModel.text.element` / `.association` |
| Value help empty or missing | `@Consumption.valueHelpDefinition` target, or `additionalBinding` over-filtering |
| Value correct in DB, stale on screen | Missing `@Common.SideEffects` on the triggering field |
| Button appears but action fails | Action not in the behavior projection (`use action …`) |
| Page title generic / wrong | `@UI.headerInfo` missing or incomplete |
| List order looks random | No `@UI.presentationVariant` sort order |
| Nothing changed after activation | Stale metadata — re-publish the binding, clear the browser cache |

## Verify at the source of truth
Inspect the service's `$metadata` / the service binding preview rather than trusting the CDS
source. If the annotation isn't in the metadata document, the UI was never going to show it — and
that narrows the problem to exposure, not rendering.

## Fix
Give the corrected source block, say which chain step it repairs, and name what to re-activate and
in what order (view → metadata extension → service binding). Then state how to confirm it's fixed.

Rules:
- Don't guess element names — read the view.
- One root cause, evidenced. If several things are wrong, rank them and say which one produces
  the reported symptom.
- Mark anything you couldn't verify live `[CONFIRM in ADT]`.

---
**Chain**: `annotate-fiori-app` → **debug-fiori-ui** → `atc-fix` → `pre-transport-check`
