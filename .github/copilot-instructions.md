# ABAP Project Guidelines

You are assisting an experienced ABAP developer. Write production-quality ABAP
following Clean ABAP and, where applicable, ABAP Cloud (RAP/CDS) principles.

## System & Package
- Target system: `A4H` (client-local development system, ADT connection `A4H_001_RAMJEE_EN`).
- `$TMP` is for temporary, never-transported local objects only — use it for
  throwaway spikes/experiments. All real development must go into a proper
  transportable package, and a transport request must be attached (never assume
  `$TMP` for anything meant to persist or move to other systems).

## Naming Convention
Everything this project owns lives under the single custom prefix `ZRK_`, followed
by a short object-type infix. There is no `ZCL_`/`ZIF_`/`ZBP_` family here.

| Object | Pattern | Example |
| --- | --- | --- |
| Class | `ZRK_CL_<name>` | `ZRK_CL_ORDER_MANAGER` |
| Interface | `ZRK_IF_<name>` | `ZRK_IF_ORDER_SOURCE` |
| Exception class | `ZRK_CX_<name>` | `ZRK_CX_ORDER_INVALID` |
| CDS interface view | `ZRK_I_<Name>` | `ZRK_I_TRAVEL` |
| CDS consumption view | `ZRK_C_<Name>` | `ZRK_C_TRAVEL` |
| Behavior pool class | `ZRK_BP_I_<Name>` | `ZRK_BP_I_TRAVEL` |
| Service definition | `ZRK_SD_<Name>` | `ZRK_SD_TRAVEL` |
| Service binding | `ZRK_UI_<Name>_O4` / `ZRK_API_<Name>_O4` | `ZRK_UI_TRAVEL_O4` |
| Access control (DCL) | `ZRK_DCL_<Name>` | `ZRK_DCL_TRAVEL` |
| Test class | local `ltcl_*` inside the class under test | `ltcl_order_manager` |

- Behavior definitions and behavior projections carry the exact name of the CDS
  view they define behavior for — no separate name of their own.
- Do not invent a different prefix or namespace unless the user explicitly asks.

## Ground Before You Generate
- If `system-info.md` exists at the workspace root, read it before generating or reviewing
  anything. It records the connected system's release, feature availability, and the RAP/CDS
  syntax ceiling — it outranks the defaults in this file where the two disagree.
- If it doesn't exist, run the `bootstrap-system-context` prompt once. Until then, treat the
  ABAP Cloud assumptions below as unverified and say so when they matter.
- Before creating a new object, check whether one already exists that does the job (where-used,
  object search on `ZRK_*`). Reuse or extend beats a near-duplicate.

## Classic vs. ABAP Cloud
- Default to ABAP Cloud patterns for new development: CDS view entities, RAP
  (managed first, unmanaged only when managed can't fit the requirement),
  released/public APIs only.
- Only use classic techniques (dynpro, function modules, direct table access,
  enhancement points) when explicitly requested or when extending existing
  classic objects — and call out the tradeoff when you do.
- Never drop a tier in the extensibility order without justifying why the tier
  above didn't fit. The full tier definitions live in
  [abap-cloud-rap.instructions.md](instructions/abap-cloud-rap.instructions.md#extensibility-tier-order).

## Clean Core Non-negotiables
- Never suggest code that accesses a non-released SAP table, function module,
  class, or BAdI directly. Check for a released API/CDS equivalent first, and
  say so if substituting one.
- Never suggest modifying SAP standard objects or classic user-exits when a
  released BAdI, RAP extension point, or in-app extensibility tool exists.
- Forbidden in ABAP Cloud code: `EXPORT/IMPORT ... TO MEMORY`, dynamic
  subroutine pool generation, classic dynpro (`CALL SCREEN`/`CALL DIALOG`),
  `NATIVE SQL`/`EXEC SQL`, `CALL TRANSACTION`, `SUBMIT` of classic reports,
  `PERFORM`/VOFM-style routines, and `AUTHORITY-CHECK` against a non-released
  authorization object. If the target system is confirmed on-prem/classic and
  this is explicitly requested, it's allowed but call out that it isn't ABAP
  Cloud portable.

## Verifying Released APIs (use the connected ADT MCP tools)
- Before using any SAP table, CDS view, class, function module, or BAdI in
  generated code, verify its release/API state rather than assuming — use the
  ADT tools (e.g. object read/search) available in this session.
- If the release state can't be verified live, mark the assumption
  `[CONFIRM in ADT]` instead of guessing — never invent SAP field, table, or
  BAPI/API names.

## ADT MCP Write Safety
- Reading objects, metadata, where-used, syntax checks, ATC results, and unit
  test results is free — do it to ground a review or generation in real system
  state instead of guessing.
- Creating/changing/activating objects, or creating/assigning transports, is
  gated: show exactly what will be created or changed (object names, types,
  target package, target transport) and wait for explicit confirmation in the
  same turn before calling a write tool. One confirmation covers the described
  batch only, not future writes.
- Review-only tasks (Clean ABAP review, ABAP Cloud readiness check) never write.

## Style & Testing
- Full conventions live in [clean-abap.instructions.md](instructions/clean-abap.instructions.md),
  [abap-cloud-rap.instructions.md](instructions/abap-cloud-rap.instructions.md),
  [abap-performance.instructions.md](instructions/abap-performance.instructions.md), and
  [fiori-annotations.instructions.md](instructions/fiori-annotations.instructions.md).
  All four attach automatically to the file types they cover (`applyTo`); don't
  duplicate their rules here. When working on a behavior pool or RAP handler
  class, read the RAP file explicitly — it keys off CDS/BDEF extensions.
- New or changed logic needs ABAP Unit tests (`ltcl_*`, AAA pattern, test
  doubles for dependencies) — flag when tests are missing instead of skipping them silently.
- When generating a new object (not a snippet), prefer the full abapGit-style
  object set (see abap-cloud-rap.instructions.md § abapGit Object-Set Layout),
  each block headed by a one-line comment: object name, type, and the
  requirement it implements — end with a "verify before activation" list of
  released dependencies used.

## What NOT to Do
- Don't invent SAP standard field names, table names, or BAPI/API signatures —
  say so and suggest how to verify (ADT where-used, API Hub) instead of
  guessing a plausible-looking name.
- Don't hardcode business-relevant values that belong in customizing or constants.
- Don't silently pick classic extensibility as a shortcut because it's
  simpler to generate — always name it as an exception when used.

## Available Prompts
Run them roughly in this order; each names its own chain.

| Phase | Prompt |
| --- | --- |
| Ground (once per system) | `bootstrap-system-context` |
| Understand | `explain-abap`, `abap-cloud-readiness-check`, `clean-core-extensibility-check` |
| Build | `create-cds-view`, `create-rap-bo`, `expose-odata-service` |
| Fiori UI | `annotate-fiori-app`, `debug-fiori-ui` |
| Verify | `generate-abap-unit-tests`, `atc-fix` |
| Troubleshoot | `analyze-dump`, `debug-slow-sql`, `debug-fiori-ui` |
| Clean up & ship | `find-unused-code`, `pre-transport-check` |

Reviews use the `abap-clean-code-reviewer` agent (read-only).

## Authoritative Sources
Cite these rather than inventing a rule, and prefer them over memory when they disagree with this
folder. Say which one you used.

| Topic | Source |
| --- | --- |
| Clean ABAP rules | `SAP/styleguides` → `clean-abap/CleanABAP.md` |
| ABAP language syntax, RAP EML, CDS view entities, BDEF, performance notes | `SAP-samples/abap-cheat-sheets` |
| Fiori Elements UI annotations (OData V4, RAP) | `SAP-samples/abap-platform-fiori-feature-showcase` |
| RAP end-to-end reference scenario | `SAP-samples/abap-platform-refscen-flight` |

The cheat sheets cover ABAP language and RAP, **not** Fiori Elements — for UI annotations use the
feature showcase and [fiori-annotations.instructions.md](instructions/fiori-annotations.instructions.md).
