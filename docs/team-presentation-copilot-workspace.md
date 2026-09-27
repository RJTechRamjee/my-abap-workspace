# One Shared AI Workspace for ABAP Development

**Team walkthrough: what it is, how it's organised, and how it saves time day to day**

> Source document for two presentations given by the lead developer: first to **technical
> managers**, then to the **developer team**. Each `##` section maps to one or more slides.
> Speaker notes are marked **🗣 Say:**.

### Audience guide

| Section | Technical managers | Developer team |
|---|---|---|
| 0. Executive summary (managers) | ✅ Core | — |
| 1. The problem | ✅ | ✅ |
| 2. The approach | ✅ | ✅ |
| 3. How guidelines are enforced | ✅ Short version | ✅ Full |
| 4. Fork and sync | ✅ Diagram only | ✅ Full, with commands |
| 5. Workspace organisation | ✅ Repository structure, 5.0 overview, 5.0b "at a glance" table | ✅ Full, incl. 5.0b building blocks explained |
| 6. Theory | ✅ 6.1, 6.4, 6.6 | ✅ Full |
| 7. Day-to-day savings | ✅ Table and feature flow | ✅ Full |
| 8. Guardrails | ✅ | ✅ |
| 9. Getting-started checklist | — | ✅ |
| 10. Rollout plan and asks (managers) | ✅ Core | Briefly: timeline only |
| 11. Summary | ✅ | ✅ |

Suggested length: managers about 20–25 min (about 10 slides); developers about 45–60 min with a live demo (about 18–20 slides).

---

## 0. Executive summary (for technical managers)

**What:** a shared, version-controlled setup that makes GitHub Copilot follow *our* project guidelines (naming, ABAP Cloud, Clean Core, testing, transport hygiene) for every developer.

**Why it matters:**

| Management concern | What this setup delivers |
|---|---|
| **Consistency and quality** | The same rules and procedures for every developer. Guideline violations are prevented at generation time rather than caught in review |
| **Clean Core / upgrade risk** | Released APIs only. The extensibility tier order is enforced, and classic extensions need a documented exception (ADR) |
| **Governance and control** | Only leads merge changes. Every change is reviewed via PR, versioned and traceable in Git |
| **Security and compliance** | The AI cannot write to SAP without explicit developer confirmation. Architect and reviewer agents are read-only. No credentials or personal data in the repo |
| **Productivity** | Reusable prompts, skills and agents for daily tasks: build, test, ATC, dump analysis, transport checks, specs and designs |
| **Onboarding** | New joiners inherit the team's standards and know-how on day one |
| **Knowledge retention** | Expertise is captured in the repo instead of staying in individuals' heads |

**Cost to adopt:** about 10 minutes of setup per developer (fork, clone, connect). No new licences beyond GitHub Copilot and the SAP ABAP VS Code extension. *[Confirm the licence status for your organisation.]*

**What I'm asking for:** see [section 10](#10-rollout-plan-and-asks-for-technical-managers).

🗣 **Say:** "We already pay for AI assistance. This makes sure it works to our standards, under our control, for the whole team."

---

## 1. The problem we are solving

Today every developer uses GitHub Copilot (or another AI assistant) with their own habits:

- Everyone writes their own prompts, from scratch, every time.
- The AI doesn't know our project rules: the `ZRK_` prefix, ABAP Cloud first, released APIs only, the Clean Core tier order. So it guesses, and we correct it.
- Output quality depends on who is asking and how well they phrase it.
- Good prompts stay on one person's laptop, and nobody else benefits.
- When the project guidelines change, nothing tells the AI.

**The result:** inconsistent code, repeated review comments about the same issues, and time lost re-explaining the project to a tool that forgets.

🗣 **Say:** "The AI is only as good as the context we give it. Right now each of us gives it different context, or none."

---

## 2. The approach in one sentence

> **The project guidelines live in one Git repository, in a form the AI reads automatically. Every developer forks it once and receives every update the leads publish.**

What this gives the team:

| Benefit | How |
|---|---|
| **Guidelines are followed without anyone having to remember them** | `copilot-instructions.md` and the `instructions/` files are loaded into every chat automatically |
| **Same quality for everyone** | Everyone runs the same prompts, skills and agents, with the same guardrails |
| **One place to change a rule** | Change `sap-project-standards.md` once; every skill picks it up |
| **Updates reach everyone** | Leads merge into the team repo; developers sync their fork |
| **Personal freedom without drift** | Your own `local-*` files sit next to the shared ones and never conflict |
| **Improvements flow back** | Anyone can propose a better prompt through a pull request |

---

## 3. How the project guidelines are enforced

The guidelines are no longer a PDF people are asked to read. They are loaded into the AI's context, so the AI applies them while it writes code.

What the AI is told on **every** request (from [.github/copilot-instructions.md](../.github/copilot-instructions.md)):

- **Naming:** everything under `ZRK_` plus a type infix (`ZRK_CL_`, `ZRK_I_`, `ZRK_C_`, `ZRK_BP_I_`, `ZRK_SD_`, `ZRK_UI_…_O4`…). No invented prefixes.
- **ABAP Cloud first:** CDS view entities, managed RAP, released APIs only. Classic techniques only when explicitly requested, and the tradeoff must be stated.
- **Clean Core non-negotiables:** no access to non-released SAP objects, no modifications, and a list of forbidden statements (`CALL TRANSACTION`, `EXEC SQL`, `SUBMIT`, `EXPORT TO MEMORY`…).
- **Ground before you generate:** read `system-info.md` for the real system release, and check that an object doesn't already exist before creating a new one.
- **Never invent SAP names:** unverifiable items are marked `[CONFIRM in ADT]` instead of guessed.
- **Write safety:** reading the system is free; creating, changing or activating objects and transports needs explicit confirmation first.
- **Testing:** new logic needs ABAP Unit tests (`ltcl_*`, AAA pattern), and missing tests are flagged.
- **Authoritative sources:** SAP's Clean ABAP styleguide, ABAP cheat sheets, the Fiori feature showcase and the RAP flight reference scenario. The AI cites these instead of relying on memory.

The deeper rules sit in [.github/reference/sap-project-standards.md](../.github/reference/sap-project-standards.md), the **single source of truth** every skill cites:

- Platform baseline: S/4HANA Cloud Private Edition, release 2025, ABAP Cloud, DEV → QAS → PRD
- The six Clean Core dimensions
- **Extensibility tier order:** 0 Config/BRFplus → 1 Key-user → 2 Developer (on-stack) → 3 Side-by-side (BTP) → 4 Classic (a signed exception only)
- How to verify that an API is "Released"
- Placeholder conventions: `[CONFIRM]`, `[ASSUMPTION — confirm]`, `N/A`

🗣 **Say:** "When a rule changes, the lead edits one file. The next time you sync, your AI follows the new rule. Nobody has to send a memo."

---

## 4. Fork and sync: how updates reach you

```
  Team repo  (upstream)          ← leads review and merge here, then publish a release
        │
        │  fork (once)
        ▼
  Your fork  (origin)            ← your copy on GitHub; also backs up your local-* files
        │
        │  clone (once)
        ▼
  Your laptop / VS Code          ← Copilot reads .github/ from here
```

### One-time setup (about 10 minutes)
1. Fork the team repo on GitHub.
2. Clone your fork and add the team repo as `upstream`:
   ```bash
   git clone https://github.com/<you>/my-abap-workspace.git
   cd my-abap-workspace
   git remote add upstream https://github.com/RJTechRamjee/my-abap-workspace.git
   ```
3. Open the folder in VS Code, connect the SAP ABAP extension to your system, and check that the **ADT MCP Server** is running.

### Receiving a new version
When a lead announces a new version or release:
```bash
git fetch upstream
git merge upstream/main
git push origin main
```
Or click **Sync fork** on GitHub and then run `git pull`.

### Why this never conflicts
- Leads only change **shared** files.
- You only change **`local-*`** files (they are git-ignored, so they never end up in a team PR by accident).
- Want a shared file to behave differently? Copy it to `local-<name>` and edit the copy.

### Giving back
Found a better prompt or built a skill everyone should have?
Branch from `upstream/main` (not your own `main`), commit, and open a PR. A lead reviews it using the [PR template](../.github/pull_request_template.md) checklist, which checks for no personal data, `ZRK_` naming, working links, and that it was tested once against a real object. After the merge, everyone gets it on their next sync.

> **Suggested practice for leads:** tag each published update (`v1.0`, `v1.1`…) and create a GitHub Release with short notes ("What's new: debug-fiori-ui prompt, updated tier rules"). Developers then know when to sync and what changed.
> *[Note for review: the repo has no tags yet. Decide whether to start with v1.0.]*

🗣 **Say:** "Think of it like an SAP support package for our AI setup. The leads ship it, you import it, and your own customizations survive."

---

## 5. How the workspace is organised

**Repository structure** (as shown in the VS Code explorer):

```
my-abap-workspace/
├── .github/                      Everything Copilot reads
│   ├── agents/                   3 agents: architect, senior developer, reviewer
│   ├── instructions/             4 rule files, attached automatically by file type
│   ├── prompts/                  15 task recipes, run with /name
│   ├── reference/                6 ground-truth files (start: sap-project-standards.md)
│   ├── skills/                   12 skills, one folder with a SKILL.md each
│   ├── templates/                10 output templates: FS, TDD, SDD, ADR, reviews…
│   ├── copilot-instructions.md   Always-on project rules, loaded in every chat
│   └── pull_request_template.md  Checklist for team contributions
├── .vscode/settings.json         Turns on instructions, prompts, skills and ADT MCP
├── .gitignore                    Keeps local-* files and system-info.md personal
└── CONTRIBUTING.md               Fork / sync / contribute guide
```

*Personal and never shared: your `local-*` files and `system-info.md` (written by `/bootstrap-system-context`).*

### 5.0 Overview in one picture (manager slide)

| Folder | In plain words | Who maintains it |
|---|---|---|
| `copilot-instructions.md` + `instructions/` | **The rules**: what the AI must always follow | Lead team |
| `prompts/` | **The recipes**: step-by-step procedures for daily tasks | Lead team, with developer contributions |
| `skills/` | **The expertise**: specialist knowledge for specs, designs and reviews | Lead team / architects |
| `agents/` | **The roles**: architect, developer, reviewer, each with limited permissions | Lead team |
| `reference/` + `templates/` | **The facts and formats**: one source of truth and consistent deliverables | Lead team / architects |
| `local-*` files | **Personal additions**: never shared unless proposed via PR | Each developer |

### 5.0b What are instructions, prompts, skills and agents?

All four are plain Markdown files in `.github/` that Copilot reads. What sets them apart is **who starts them** and **how long they stay active**.

| | **Instruction** | **Prompt** | **Skill** | **Agent** |
|---|---|---|---|---|
| **In one line** | A rule the AI always follows | A saved task recipe you run | Specialist know-how the AI picks up when needed | An AI team member with a role |
| **File** | `copilot-instructions.md`, `instructions/*.instructions.md` | `prompts/<name>.prompt.md` | `skills/<name>/SKILL.md` (a folder) | `agents/<name>.agent.md` |
| **Who starts it** | Nobody: it is applied automatically | **You**, by typing `/name` | **The AI**, when your request matches its description | **You**, by picking it in the agent dropdown |
| **How long it's active** | Every chat, or every file that matches `applyTo` | One task | While the task needs it | The whole conversation |
| **Analogy** | Company coding standard | Checklist / runbook | Specialist's handbook | A colleague in a role |

#### Instruction: "the rules"
- **What:** standing rules added to the AI's context without you asking. `copilot-instructions.md` applies to every chat. Each `*.instructions.md` file has an `applyTo` pattern and attaches only when you work on a matching file type.
- **Why:** you never have to remind the AI about `ZRK_` naming, released APIs or ABAP Unit tests. The rule is simply always there.
- **Example:** open a `.abap` file and `clean-abap.instructions.md` applies. Open a `.asddls` file and the CDS/RAP, performance and Fiori annotation rules apply.
  ```yaml
  ---
  description: "Use when writing, reviewing, or refactoring ABAP classes..."
  applyTo: "**/*.abap"
  ---
  ```
- **You do:** nothing. Just work.

#### Prompt: "the recipe"
- **What:** a reusable, step-by-step task description you run on demand. It asks for its input (an object, a transport, a dump) and follows a fixed procedure.
- **Why:** the best way to do a task is written down once and tested, so nobody has to type a long prompt from memory. Everyone gets the same steps and the same output quality.
- **Example:** `/pre-transport-check A4HK900123` inventories the transport, runs ATC and unit tests, looks for `$TMP` leftovers and non-released dependencies, and returns a go/no-go.
  ```yaml
  ---
  description: "Gate a transport before release..."
  agent: "agent"
  argument-hint: "Transport request, e.g. 'A4HK900123'"
  ---
  ```
- **You do:** type `/` in Copilot Chat, pick the prompt, and give the input.

#### Skill: "the expertise"
- **What:** a folder with a `SKILL.md` holding in-depth instructions for one kind of work, often pointing to `reference/` files and filling a template in `templates/`. The AI first sees only the skill's **name and description**. It loads the full content only when your request matches (this is called *progressive disclosure*).
- **Why:** lots of deep know-how is available without cluttering every chat. Larger deliverables (FS reviews, technical designs, object sets, test classes) come out in the team's standard format.
- **Example:** ask *"Review this functional spec for the billing block requirement"* and the `functional-spec-reviewer` skill is picked up. It checks the FS against standard SAP and the S/4HANA red flags and fills `fs-review-report.md`.
  ```yaml
  ---
  name: functional-spec-reviewer
  description: Use to review a Functional Specification written by someone else...
  ---
  ```
- **You do:** describe the task in normal words. A skill whose description matches is used automatically.

#### Agent: "the team member"
- **What:** a custom chat mode with a **role**, a **restricted set of tools** and **handoffs** to other agents. Once you pick it, it stays in that role for the whole conversation.
- **Why:** it separates duties the same way our team does. The architect and the reviewer can only read. The developer can write to the system, but only after you confirm. Handoff buttons move the work to the next role with the context intact.
- **Example:** pick `abap-architect` to get a tier decision and object inventory. Click **Build this design** to switch to `abap-senior-developer`, then **Review my changes** to switch to `abap-clean-code-reviewer`.
  ```yaml
  ---
  description: "Use when a requirement needs a design decision before anyone builds..."
  tools: [read, search, 'com.sap.adt/mcp/abap_atc_run', ...]   # no write tools
  handoffs:
    - label: Build this design
      agent: abap-senior-developer
  ---
  ```
- **You do:** pick the agent from the agent dropdown in Copilot Chat.

#### How they work together
```
Agent (who is working)        abap-senior-developer
  └─ runs a Prompt (what)     /create-rap-bo Travel
       └─ can draw on a Skill (how)   e.g. abap-unit-test-writer for the test class
            └─ always under Instructions (rules)   ZRK_ naming, Clean Core, ABAP Unit
```

#### Which one should I write?
| You want to… | Write a… |
|---|---|
| Make the AI always follow a rule | **Instruction** |
| Save a task you repeat, with fixed steps | **Prompt** |
| Capture deep know-how for a type of deliverable | **Skill** |
| Create a role with its own permissions and handoffs | **Agent** |

Start with a `local-` prefix. If it proves useful, propose it to the team (section 4).

🗣 **Say:** "Instructions are always on. You run prompts. The AI picks up skills when they fit. You choose an agent. That's the whole model."

### 5.1 `copilot-instructions.md`: the constitution
Loaded into **every** Copilot chat in this workspace. Holds the non-negotiables: naming, ABAP Cloud first, Clean Core, write safety, testing, and authoritative sources. Kept short on purpose. The detail lives in the files below.

### 5.2 `instructions/`: rules by file type
Each file has an `applyTo` pattern, so it attaches automatically when you work on a matching file.

| File | Applies to | Covers |
|---|---|---|
| `clean-abap.instructions.md` | `*.abap` | Clean ABAP naming, structure, error handling, tests (condensed from SAP's styleguide) |
| `abap-performance.instructions.md` | `*.abap`, `*.asddls` | Code-to-data, Open SQL, internal tables, CDS performance |
| `abap-cloud-rap.instructions.md` | `*.asddls`, `*.asddlx`, `*.asbdef`, `*.asdcls`, `*.srvdsrv`, `*.acds` | RAP/CDS conventions, extensibility tiers, abapGit object-set layout |
| `fiori-annotations.instructions.md` | `*.asddls`, `*.asddlx` | `@UI` annotations for list report / object page |

### 5.3 `prompts/`: reusable task recipes
Start one with `/` in Copilot Chat. Each prompt asks for its input (an object name, transport, dump text) and runs a tested, step-by-step procedure. They are organised by lifecycle phase:

| Phase | Prompts |
|---|---|
| **Ground** (once per system) | `bootstrap-system-context` → writes `system-info.md` |
| **Understand** | `explain-abap`, `abap-cloud-readiness-check`, `clean-core-extensibility-check` |
| **Build** | `create-cds-view`, `create-rap-bo`, `expose-odata-service` |
| **Fiori UI** | `annotate-fiori-app`, `debug-fiori-ui` |
| **Verify** | `generate-abap-unit-tests`, `atc-fix` |
| **Troubleshoot** | `analyze-dump`, `debug-slow-sql`, `debug-fiori-ui` |
| **Clean up & ship** | `find-unused-code`, `pre-transport-check` |

### 5.4 `skills/`: packaged expertise for larger deliverables
A skill is a folder with a `SKILL.md`. Copilot reads only its `description` until the task matches, and then loads the full instructions. The skills cover the architect's and lead's document-heavy work:

| Area | Skills |
|---|---|
| Requirements | `requirement-workshop-facilitator`, `functional-spec-reviewer`, `functional-spec-writer` |
| Design | `solution-architect`, `technical-design-writer`, `clean-core-extensibility-advisor` |
| Build & quality | `abap-object-generator`, `abap-unit-test-writer`, `clean-abap-code-reviewer`, `abap-cloud-readiness-checker` |
| Knowledge sharing | `coe-session-planner`, `sap-innovation-radar` |

### 5.5 `agents/`: role-based assistants
An agent is a persona with a **defined job, a restricted tool set, and handoffs** to the next role.

| Agent | Role | Writes to system? | Hands off to |
|---|---|---|---|
| `abap-architect` | Tier decision, solution shape, object inventory, ADR | **Never** (read-only tools) | Senior developer: "Build this design" |
| `abap-senior-developer` | Reads existing code, reuses, builds, writes tests, runs ATC and ABAP Unit | **Only after explicit confirmation** | Reviewer: "Review my changes"; back to architect |
| `abap-clean-code-reviewer` | Clean ABAP, naming, RAP/CDS review | **Never** | Senior developer: "Fix these findings" |

This mirrors our real process: **design → build → review → fix**, with the same separation of duties.

### 5.6 `reference/`: the ground truth
Facts the skills cite instead of re-deriving (or hallucinating) them:
`sap-project-standards.md` (start here), `adt-mcp-usage.md`, `fit-to-standard-patterns.md`, `l2c-standard-process-reference.md`, `pricing-condition-technique-reference.md`, `s4hana-simplification-redflags.md`.

### 5.7 `templates/`: consistent outputs
Every document the skills produce follows the same template, so an FS review from one person looks like an FS review from another:
functional spec, FS review report, technical design, solution design, interface spec, Clean Core ADR, Clean ABAP review checklist, workshop notes, KM session and deck outline.

### 5.8 Supporting files
- **`.vscode/settings.json`** enables instruction files, prompt files, skills and the ADT MCP server, and suggests the entry prompts in an empty chat.
- **`.gitignore`** keeps `local-*` files and the per-system `system-info.md` out of the team repo.
- **`pull_request_template.md`** is the contribution checklist.
- **`CONTRIBUTING.md`** is the full fork / sync / contribute guide.

---

## 6. The theory: why these building blocks work

### 6.1 Context engineering
A language model knows general ABAP but nothing about **our** project. The quality of its answer depends on the context it gets. Instead of each developer re-typing that context, we **engineer it once** and version it in Git, just like code.

### 6.2 Four layers, from always-on to on-demand

| Layer | Loaded when | Analogy | Purpose |
|---|---|---|---|
| **Instructions** | Always / by file type | Company coding standard | Rules that must never be forgotten |
| **Prompts** | When you type `/name` | A checklist or runbook | A repeatable procedure for one task |
| **Skills** | When the task matches the description | A specialist's handbook | Deep expertise without bloating every chat |
| **Agents** | When you pick the agent | A team member with a role | A persona with its own tools, limits and handoffs |

**Progressive disclosure:** the AI's context window is limited. Always-on rules stay short. Detailed knowledge (skills, references) loads only when needed. This keeps answers focused and accurate.

### 6.3 Grounding through MCP (Model Context Protocol)
MCP is an open standard that lets the AI call tools. The SAP ABAP extension ships an **ADT MCP Server**, so the AI can:
- read objects, run where-used and check release state (**grounding**: facts come from the system, not from memory),
- run ATC and ABAP Unit and read the results (**verification**),
- create and activate objects and transports, but **only after you confirm** (**human in the loop**).

`bootstrap-system-context` records the connected system's release and syntax ceiling in `system-info.md`, so generated code compiles on *our* system rather than on a generic one.

### 6.4 Least privilege and separation of duties
Agents only get the tools their role needs. The architect and the reviewer **cannot** write to the system. The developer can, but only after an explicit "yes" for a described batch. This is the same four-eyes principle we apply to people.

### 6.5 Single source of truth
Naming lives in one table. Tier rules live in one file. Prompts and skills **link** to them instead of copying them. Change a rule once and it changes everywhere, which is DRY applied to guidelines.

### 6.6 Guidelines as code
Because it's all Markdown in Git, we get history, review, diffs, rollback, releases and contribution for free. We are applying our normal engineering discipline to the AI setup.

---

## 7. Day-to-day: where it saves time

| Situation | Before | With the workspace |
|---|---|---|
| Joining the project / new system | Read wiki pages, ask colleagues about naming and rules | Fork, clone, run `/bootstrap-system-context`. The rules are already active |
| Understanding legacy code | Manually trace callers and dependencies | `/explain-abap ZRK_CL_…`: purpose, data flow, where-used, risk of change |
| New RAP BO | Copy an old BO, rename by hand, fix naming | `/create-rap-bo`: full abapGit object set, `ZRK_` names, tests, "verify before activation" list |
| Fiori field not showing | Trial and error on annotations | `/debug-fiori-ui`: works down annotation → projection → exposure → behavior |
| Short dump | Read ST22, search SCN | `/analyze-dump`: root cause and a concrete fix |
| Slow report / OData | Guess, then rewrite | `/debug-slow-sql`: measure first, fix the access path, prove the gain |
| Unit tests | Often skipped | `/generate-abap-unit-tests`: AAA pattern, test doubles, RAP test environment |
| ATC findings | Fix one by one | `/atc-fix`: triage, deterministic quickfixes, justified exemptions |
| Before releasing a transport | Manual checklist, often incomplete | `/pre-transport-check`: go/no-go with evidence (ATC, tests, `$TMP` leftovers, non-released dependencies) |
| FS arrives from a functional consultant | Hours of reading and challenging | `functional-spec-reviewer` skill: findings report plus a design position |
| Technical design | Blank Word document | `technical-design-writer` skill fills the TDD template |
| End-to-end feature | Hand-offs by chat and email | `abap-architect` → `abap-senior-developer` → `abap-clean-code-reviewer` via handoff buttons |

### A typical feature flow
```
Requirement
   │  clean-core-extensibility-check / abap-architect   → tier decision + object inventory
   ▼
Build
   │  abap-senior-developer (create-rap-bo, annotate-fiori-app)   → objects + tests, you confirm writes
   ▼
Verify
   │  generate-abap-unit-tests, atc-fix
   ▼
Review
   │  abap-clean-code-reviewer   → severity-tagged findings → "Fix these findings"
   ▼
Ship
      pre-transport-check   → go / no-go
```

### Where the savings come from
- **No prompt writing:** tested prompts are one `/` away.
- **Fewer review loops:** naming, Clean Core and test rules are applied while generating, not caught afterwards.
- **Less rework from hallucinations:** SAP facts are verified in ADT or flagged `[CONFIRM]`.
- **Faster onboarding:** new joiners inherit the team's accumulated know-how on day one.
- **Reuse compounds:** each improvement one person contributes benefits everyone after the next sync.

> *[Note for review: add real numbers from a pilot if you have them, e.g. "RAP BO scaffold: ~X h → ~Y min". The document deliberately doesn't invent figures.]*

---

## 8. Guardrails and responsibilities

- **The AI proposes; you decide.** All system writes need explicit confirmation, and you remain the author of the code.
- **Review still applies.** AI output goes through ATC, unit tests and code review like any other code.
- **No personal data in the shared repo:** no connection names, users, passwords or system URLs. `system-info.md` stays local.
- **Don't edit shared files locally.** Use `local-*` copies or a PR.
- **`$TMP` is for experiments only.** Real work goes into a transportable package with a transport request.

---

## 9. Getting started checklist

- [ ] Fork the team repo and clone your fork
- [ ] `git remote add upstream …`
- [ ] Install the SAP ABAP extension, connect to your system, check that **ADT MCP Server** is *Running*
- [ ] In Copilot Chat, open the tools picker and confirm the ADT tools are enabled
- [ ] Run `/bootstrap-system-context` once
- [ ] Try `/explain-abap` on an object you know well
- [ ] Sync weekly or when a lead announces a release
- [ ] Put your own experiments in `local-*` files; propose the good ones upstream

---

## 10. Rollout plan and asks (for technical managers)

### Proposed rollout
*[Adjust the dates and durations to your project plan.]*

| Phase | When | What | Success signal |
|---|---|---|---|
| **1. Pilot** | Weeks 1–2 | 2–3 developers use the workspace on real tickets | Setup works for everyone; first feedback captured |
| **2. Team rollout** | Weeks 3–4 | Developer walkthrough session; everyone forks and runs `bootstrap-system-context` | All developers synced; prompts used in daily work |
| **3. Steady state** | Ongoing | The lead publishes a versioned release (e.g. every 2–4 weeks); developers sync; contributions come in via PR | Regular releases; developer PRs merged |
| **4. Review** | After ~2 months | Compare review findings, ATC results and rework before and after | Measurable improvement to report back |

### How we'll measure it
- Number of recurring review comments (naming, Clean Core, missing tests), which should go down
- ATC findings per transport at the `pre-transport-check` gate
- Time to scaffold a standard object (e.g. a RAP BO) and time for new joiners to become productive
- Developer feedback and the number of contributions to the shared repo

### Governance model
- **Owner:** lead developer (me), with a named backup lead. *[Name the backup.]*
- **Change process:** every change to shared files goes through a PR and is reviewed against the PR template checklist.
- **Releases:** tagged versions with short release notes, announced to the team.
- **Hosting:** *[Decide: personal GitHub account today → move to the company GitHub/GitLab organisation?]*

### Risks and mitigations

| Risk | Mitigation |
|---|---|
| AI generates wrong or non-compliant code | Rules are enforced at generation time; ATC, ABAP Unit and code review remain mandatory; the AI flags `[CONFIRM]` instead of guessing |
| Unintended changes in the SAP system | Writes need explicit confirmation; architect and reviewer agents are read-only; `$TMP` only for experiments |
| Sensitive data leaves the company | No credentials or connection data in the repo (enforced by `.gitignore` and the PR checklist). *[Confirm the Copilot data policy with IT/security.]* |
| Setup drifts between developers | Fork and sync model; personal changes are isolated in `local-*` files |
| Dependence on one person | Everything is documented in the repo; add a backup maintainer |

### What I need from you
1. **Approval** to roll this out to the developer team.
2. **Time**: a 1-hour team session plus about 2 hours per sprint for me to maintain releases.
3. **A decision on hosting**: keep the repo where it is, or move it into the company Git organisation.
4. **Confirmation** from IT/security on the Copilot and MCP usage policy. *[If not already covered.]*

🗣 **Say:** "Small investment, low risk because humans stay in control, and it scales our standards to the whole team."

---

## 11. Summary

1. **Guidelines as code:** project rules live in Git and are applied by the AI automatically.
2. **Fork once, sync always:** leads publish; everyone receives; personal files are never touched.
3. **A clear structure:** instructions (rules), prompts (recipes), skills (expertise), agents (roles), reference (truth), templates (format).
4. **Grounded and safe:** MCP connects the AI to the real system, and writes need your confirmation.
5. **Reusable and compounding:** every improvement reaches the whole team.

**Q&A**
