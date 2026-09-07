---
description: "Explain an existing ABAP object — purpose, data flow, dependencies, side effects, and what breaks if you change it. Read-only."
agent: "agent"
argument-hint: "Object to explain, e.g. 'ZRK_CL_ORDER_MANAGER' or a pasted report/include"
---
Explain this existing ABAP object: ${input:explainTarget:Object name or pasted source to explain}

Read-only task: do not modify, create, or activate anything. Use the ADT read tools (object read,
where-used, metadata) to ground the explanation in the real system rather than inferring from the
name.

Work outside-in — the caller's view first, the implementation last.

## 1. Purpose (3 sentences max)
What business problem this solves, who or what triggers it, and how often. If you can't tell the
business purpose from the code, say that plainly rather than paraphrasing the method names back.

## 2. Contract
- Entry points (public methods, `START-OF-SELECTION`, RAP handlers, RFC-enabled FMs).
- Inputs: parameters, selection screen, the tables it reads.
- Outputs: returned data, tables written, files, messages, events raised.

## 3. Data flow
A numbered walkthrough of the main path — where data comes in, how it's transformed, where it
lands. Name the actual tables and CDS entities. Cover the main path first, then the notable
branches; don't narrate every `IF`.

## 4. Dependencies
| Dependency | Type | Released? | Notes |
| --- | --- | --- | --- |
Tables, CDS views, classes, function modules, BAdIs, external calls. Verify release state with the
ADT tools where possible; mark anything unverified `[CONFIRM in ADT]`.

## 5. Side effects — call these out loudly
`COMMIT WORK` / `ROLLBACK`, database updates, `ENQUEUE`/`DEQUEUE`, update-task calls, background
job scheduling, emails/IDocs/events, file or spool output, and any global/static state it mutates.
An object that looks like a read but commits is the single most dangerous thing to miss here.

## 6. Change risk map
| Area | Risk | Why |
| --- | --- | --- |
Where the landmines are: implicit ordering assumptions, hardcoded values, `sy-subrc` chains that
swallow errors, shared static state, `SELECT` in loops, missing authorization checks, silent
failure paths. Include where-used results — who calls this and would break.

## 7. Modernization notes (brief)
Only if relevant: what would need to change for ABAP Cloud compliance, and which Clean ABAP
violations are structural rather than cosmetic. Point at
[abap-cloud-readiness-check](abap-cloud-readiness-check.prompt.md) for the full assessment rather
than doing it here.

Rules:
- Don't guess what an SAP standard table, field, or FM does — verify it or mark it
  `[CONFIRM in ADT]`.
- If the object is large, say which parts you read and which you skipped. Never imply you read
  code you didn't.
- Flag dead code and commented-out blocks as observations, but don't refactor them here.
