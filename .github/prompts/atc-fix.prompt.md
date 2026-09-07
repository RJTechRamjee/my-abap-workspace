---
description: "Run ATC on an object, package, or transport, triage findings by priority, apply deterministic quickfixes, and hand-fix the rest with justified exemptions."
agent: "agent"
argument-hint: "Object, package, or transport to check, e.g. 'ZRK_CL_ORDER_MANAGER' or 'A4HK900123'"
---
Run and remediate ATC findings for: ${input:atcTarget:Object, package, or transport to check}

Follow [clean-abap.instructions.md](../instructions/clean-abap.instructions.md),
[abap-performance.instructions.md](../instructions/abap-performance.instructions.md), and the
Clean Core rules in [copilot-instructions.md](../copilot-instructions.md).

## 1. Run
- Use the connected ADT ATC tools (`abap_atc_run`, then `abap_atc_get_result`) against the target.
- State which check variant you ran. If the variant isn't given, ask rather than guessing — a
  Clean Core/cloud-readiness variant and a default syntax variant produce very different lists.
  Mark an unconfirmed variant `[CONFIRM in ADT]`.

## 2. Triage before touching anything
Group the findings and show this table before proposing a single edit:

| Priority | Check | Object/Line | Finding | Proposed action |
| --- | --- | --- | --- | --- |

- **Prio 1** — must fix. Never exempt.
- **Prio 2** — fix unless there is a concrete technical reason not to.
- **Prio 3** — fix when cheap; batch the rest into a follow-up note.
- Findings that are Clean Core / ABAP Cloud violations (non-released object access, forbidden
  classic statements) are treated as Prio 1 regardless of how ATC ranked them.

## 3. Fix
1. Apply `abap_atc_execute_deterministic_quickfixes` first — these are mechanical and safe.
   Report exactly what it changed.
2. For the remainder, propose the fix yourself and explain the *cause*, not just the edit. Use
   `abap_atc_apply_ai_fix` / `abap_atc_get_ai_fix_result` where it genuinely fits, but review its
   output against Clean ABAP before accepting — don't forward an AI fix unread.
3. A finding that reveals a design problem (a `SELECT` in a loop that should be a CDS association,
   a non-released table read that needs a released API) gets the design fix, not a local patch.
   Say so explicitly instead of silencing the symptom.

## 4. Suppressions — the part that matters
- A pseudo-comment (`"#EC <CHECK_ID>`) or an ATC exemption request is a last resort, never a way
  to clear the list. Each one needs, in the code and in your summary: what the check flagged, why
  the code is correct anyway, and who decided.
- Never add a blanket suppression at include/class level to clear multiple findings.
- If you cannot justify a suppression in one sentence, it isn't justified — fix the code.

## 5. Verify
- Re-run ATC on the same target and show before/after counts per priority.
- Run `abap_run_unit_tests` on the touched objects; ATC-clean but test-red is not done.
- Anything still open goes in a explicit "remaining findings" list with an owner suggestion.

Rules:
- Object changes and activation are gated: show every object you intend to modify (name, type,
  package, transport) and wait for explicit confirmation in the same turn before any write tool.
  One confirmation covers that batch only.
- Never change an object outside the stated target scope to make a finding go away.
