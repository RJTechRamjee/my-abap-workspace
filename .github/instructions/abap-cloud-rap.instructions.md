---
description: "Use when creating or modifying CDS views, behavior definitions, projections, metadata extensions, or RAP business objects for ABAP Cloud."
applyTo: "**/*.asddls,**/*.asddlx,**/*.asbdef,**/*.asdcls,**/*.srvdsrv,**/*.acds"
---
# ABAP Cloud / RAP & CDS Conventions

## CDS Naming
Full prefix table in [copilot-instructions.md](../copilot-instructions.md#naming-convention).
RAP-specific points:

- Interface views (basic/composite): `ZRK_I_<Name>`. Consumption/projection views: `ZRK_C_<Name>`.
- Always `DEFINE VIEW ENTITY` — view entities have **no** `@AbapCatalog.sqlViewName`. Only a
  legacy DDIC-based `DEFINE VIEW` needs one, and those are not created in this project.
- Metadata extension: same name as the projection view it annotates (`ZRK_C_<Name>`, `.ddlx`).
  The annotated view must carry `@Metadata.allowExtensions: true`.
- Behavior definition and behavior projection carry the exact name of the CDS view they define
  behavior for (`ZRK_I_<Name>` and `ZRK_C_<Name>`) — they get no name of their own.
- Behavior pool class: `ZRK_BP_I_<Name>`, declared via `implementation in class zrk_bp_i_<name> unique`.
- Draft table for a draft-enabled managed BO: `ZRK_D_<Name>`.
- Access control: `ZRK_DCL_<Name>`.

## Data Modeling
- Interface views: no UI annotations, expose technical associations, use `@AccessControl` only
  at the appropriate layer (usually not on interface views).
- Consumption views: add `@UI`, `@Search`, `@ObjectModel` annotations; expose only fields
  actually needed by the consumer.
- Always define primary key, and associations instead of ad-hoc joins where reusable.

## RAP Business Objects
- Prefer **managed** RAP BOs; use **unmanaged** only when persistence/logic can't be expressed
  declaratively (complex legacy integration, non-CDS-based persistence).
- Behavior definition: mark root entity `root`, use `strict ( 2 )`, declare `etag`/`last changed`
  where optimistic concurrency matters, use standard operations (`create`, `update`, `delete`)
  unless a custom action is genuinely needed.
- Name determinations `determine_<something>`, validations `validate_<something>`, actions as
  verbs (`release`, `reject`) — implement each in its own method.
- Draft: enable draft (`with draft`, plus a `ZRK_D_<Name>` draft table) for BOs consumed by a
  Fiori Elements UI service, since object-page editing depends on it. Leave draft off for
  API-only BOs and for read-only models. State which case applies rather than defaulting silently.

## Cloud Restrictions
- Use only released/public APIs (check the API State editor or the object's release contract)
  — never reference classic, non-released objects from Cloud-restricted code.
- No classic enhancement techniques (user-exits, implicit enhancement points, `SE38` reports)
  in ABAP Cloud contexts — use BAdIs/extension points designed for Cloud instead.
- No direct database access to standard tables outside of released CDS entities.

## Service Exposure
- Expose consumption views via a service definition (`ZRK_SD_<Name>`) and a service binding
  (`ZRK_UI_<Name>_O4` for UI OData V4, `ZRK_API_<Name>_O4` for inbound API OData V4) rather than
  binding interface views directly.
- Service definition `expose`s the root plus only the child/associated entities the consumer
  actually needs — don't over-expose.
- A binding is a published contract: once activated and released, entity/field removals are
  breaking. Get the exposed set right before publishing rather than trimming later.

## abapGit Object-Set Layout
When generating a complete RAP/CDS object set, produce each object as a separate, named source
block so it can be pasted into ADT or committed via abapGit, each headed by a one-line comment
with the object name, type, and the requirement it implements:
```
zrk_<entity>.tabl.xml             Persistence table (when the BO owns its data)
zrk_d_<entity>.tabl.xml           Draft table — draft-enabled managed BOs only
zrk_i_<entity>.ddls.asddls        CDS interface view(s) (root + children)
zrk_i_<entity>.bdef.asbdef        Behavior definition (managed/unmanaged) on the interface view
zrk_c_<entity>.ddls.asddls        CDS projection / consumption view
zrk_c_<entity>.bdef.asbdef        Behavior projection (`projection;`) on the consumption view
zrk_c_<entity>.ddlx.asddlx        Metadata extension (UI annotations)
zrk_bp_i_<entity>.clas.abap       Behavior pool class (+ local handler/saver classes)
zrk_sd_<entity>.srvd.srvdsrv      Service definition
zrk_ui_<entity>_o4.srvb.xml       Service binding (OData V4)
zrk_dcl_<entity>.dcls.asdcls      Access control (DCL) — when row-level auth applies
```
The behavior projection is not optional: a transactional service bound to `ZRK_C_<Name>` needs a
projection BDEF exposing the operations, not just the interface-level BDEF.

## Extensibility Tier Order
Always justify why the tier above wasn't sufficient before choosing the next one:
1. **Standard config** — IMG, BRFplus, condition technique. Default first stop.
2. **Key-user in-app** — custom fields/logic (RAP BAdI), custom CDS views, adaptation of Fiori.
3. **Developer extensibility (ABAP Cloud)** — new RAP BOs, released-API classes, released BAdI
   implementations, wrap-and-extend released CDS.
4. **Side-by-side (BTP)** — RAP/CAP on BTP talking to the system via released OData V4/events.
5. **Classic extensibility** — user-exit, enhancement, BAdI on a non-released object. Exception
   only: needs a stated, dated reason and an owner; never the default.
