---
description: "Generate the service definition and OData V4 binding to expose an existing CDS/RAP entity, with publish steps for UI vs inbound API."
agent: "agent"
argument-hint: "CDS/RAP entity name and consumer (Fiori UI or inbound integration)"
---
Generate the OData exposure for: ${input:entity:CDS/RAP entity name and consumer (Fiori UI or inbound integration)}

Follow [abap-cloud-rap.instructions.md](../instructions/abap-cloud-rap.instructions.md) naming
and service exposure conventions.

Produce:
1. Service definition — `ZRK_SD_<Name>` — `expose` the root plus only the child/associated
   entities the consumer actually needs (name them, don't over-expose).
2. Service binding — `ZRK_UI_<Name>_O4` (Fiori UI) or `ZRK_API_<Name>_O4` (inbound integration
   API), OData V4, binding type matching the consumer.
3. Publish steps:
   - UI service: activate binding, add to Fiori launchpad content / IAM app + business catalog.
   - Inbound API: create/extend a Communication Scenario, assign the service, note the
     Communication Arrangement + communication user + OAuth2 client credentials.
4. A note on which released authorizations/DCL apply, and any rate/size limits to set in API
   Management if it fronts the service.

Rules:
- Released APIs only. Don't invent catalog/scenario technical names — mark unknowns
  `[CONFIRM in ADT]`.
- Confirm target package/transport before creating objects.

---
**Chain**: `create-rap-bo` / `create-cds-view` → **expose-odata-service** → `atc-fix` → `pre-transport-check`
