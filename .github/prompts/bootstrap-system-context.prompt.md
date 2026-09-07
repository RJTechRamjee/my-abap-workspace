---
description: "Probe the connected SAP system once and write system-info.md — release, components, feature availability, and the RAP/CDS syntax ceiling every other prompt should respect."
agent: "agent"
argument-hint: "Optional: ADT destination to probe (defaults to the configured connection)"
---
Establish and record the system context for: ${input:destination:ADT destination to probe (blank = configured default)}

Run this **once per system**, and again after an upgrade. Every other prompt in this folder
assumes ABAP Cloud syntax; this is the file that proves the assumption holds.

## Why this exists
Generating `DEFINE VIEW ENTITY`, `strict ( 2 )`, or a managed BO with draft against a release
that doesn't support it produces code that looks right and won't activate. Ground first, generate
second.

## 1. Probe (read-only)
Use the connected ADT tools — `abap_list_destinations` for reachable systems, then object/metadata
reads. Collect:
- **Identity**: SID, client, release, kernel, ABAP language version support.
- **Components**: SAP_BASIS, SAP_ABA, SAP_GWFND, SAP_UI and their SP levels.
- **Feature availability**: HANA vs. AnyDB, RAP (managed/unmanaged, draft, `strict ( 2 )`),
  CDS view entities vs. DDIC-based views only, AMDP, abapGit, gCTS, Fiori launchpad,
  released-API contract support (`ABAP for Cloud Development` as a selectable language version).
- **Available generators**: call `abap_generators-list_generators` and record what the system
  offers, by **display name** — do not record or later match on an internal ID, they differ
  between releases.

Anything you cannot verify live is recorded as `[CONFIRM in ADT]`, never as a guess.

## 2. Derive the constraints that actually bite
Turn raw versions into rules a generator can follow. For the detected release, state explicitly:
- Is `DEFINE VIEW ENTITY` available, or must CDS use DDIC-based `DEFINE VIEW` with
  `@AbapCatalog.sqlViewName`? (This flips a core rule in
  [abap-cloud-rap.instructions.md](../instructions/abap-cloud-rap.instructions.md).)
- Which `strict` level the behavior definitions support.
- Whether managed draft, numbering, and `%control` semantics are available as this project assumes.
- Whether ABAP Cloud language version is selectable, or the system is classic-only — which
  changes the Clean Core rules from "enforced by the compiler" to "enforced by review".

## 3. Write `system-info.md` at the workspace root
Sections: Identity · Components · Feature Flags (✅/❌/`[CONFIRM in ADT]` per feature) ·
Available Generators · **Constraints Snapshot** · Probed-on date.

The Constraints Snapshot is the payload — a short list of "do / don't" lines that later prompts
can obey without re-probing. Write it as rules, not as version numbers.

## 4. Report the conflicts
If anything you found contradicts the conventions in this folder, say so directly and name the
file and rule that needs changing. A silent contradiction here poisons every later generation.

Rules:
- Read-only. Creating `system-info.md` is the only write, and it lives in the workspace, not the
  SAP system — no ADT writes, no transports.
- Don't infer a feature from a version number alone when you can probe for it.
- Keep it short enough that later prompts can read the whole file cheaply.

---
**Chain**: Run **first**, once per system — every other prompt reads its `system-info.md`.
