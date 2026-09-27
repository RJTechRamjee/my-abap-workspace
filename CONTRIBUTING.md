# Working with this repository

This repository holds the team's shared GitHub Copilot setup for ABAP development: instructions,
prompts, skills and agents under `.github/`. The leads maintain it. You get their updates by
pulling, and you can keep your own personal skills and agents next to the shared ones without
losing them.

## How it works

```
  RJTechRamjee/my-abap-workspace         ← "upstream": the shared team repo
          │   (leads merge here)
          │ fork
          ▼
  <you>/my-abap-workspace                ← "origin": your own copy on GitHub
          │   (your personal local-* files are backed up here)
          │ clone
          ▼
  your laptop                            ← VS Code opens this folder
```

- **Upstream** is the team repo. Only leads can merge into its `main`.
- **Origin** is your fork. You own it, so you can push to it freely, and it backs up your personal
  files.
- Your personal files use the `local-` prefix. The team repo never contains files with that
  prefix, so syncing never touches yours.

## 1. One-time setup

1. Open https://github.com/RJTechRamjee/my-abap-workspace and click **Fork**.
2. Clone **your fork** and connect it to the team repo:

   ```bash
   git clone https://github.com/<your-user>/my-abap-workspace.git
   cd my-abap-workspace
   git remote add upstream https://github.com/RJTechRamjee/my-abap-workspace.git
   git remote -v
   ```

   `git remote -v` should list both `origin` (your fork) and `upstream` (the team repo).

3. Open the folder in VS Code.

### VS Code and ADT MCP setup

Most prompts and agents need the ADT tools (ATC, ABAP Unit, transports, generators). They come
from the **ADT MCP Server** that the SAP ABAP extension (`sapse.adt-vscode`) starts inside
VS Code. There is no separate MCP configuration file to maintain.

1. Install the SAP ABAP extension and connect it to your system (your own ADT connection).
2. Make sure `adt.mcpServer.enabled` is `true`. The repo's `.vscode/settings.json` sets it; if
   you work through a `.code-workspace` file, see "Multi-root workspace" below.
3. Run **MCP: List Servers**. **ADT MCP Server** must show as *Running*. If it doesn't, start it
   from there.
4. In Copilot Chat, open the tools picker and check that the ADT tools (`abap_atc_run`,
   `abap_run_unit_tests`, …) are ticked.

The extension generates its own token; you don't enter a password anywhere.

**Don't add the same server a second time** in your user `mcp.json` (for example as
`com.sap.adt/mcp` with a fixed port and password). Copilot would then see two copies of every ADT
tool, and one of them usually points at the wrong port. Other AI tools outside VS Code (Claude
Code, for example) do need their own entry: use `http://localhost:<adt.mcpServer.port>/mcp` with
the token from `adt.mcpServer.token`, and keep it in that tool's user settings, never in this repo.

**Multi-root workspace.** If you open this folder through a `.code-workspace` file that also
contains your ABAP system folders, VS Code may ignore some settings from
`.vscode/settings.json`. In that case copy its contents into the `"settings": { }` block of your
`.code-workspace` file. Keep that file outside the repo: it contains your personal ADT
connection name.

## 2. Adding your own skills, agents and prompts

Put them in the normal folders and start the name with `local-`:

| Type | Path |
|---|---|
| Skill | `.github/skills/local-<name>/SKILL.md` |
| Agent | `.github/agents/local-<name>.agent.md` |
| Prompt | `.github/prompts/local-<name>.prompt.md` |
| Instructions | `.github/instructions/local-<name>.instructions.md` |

Copilot picks them up like any other file. `.gitignore` excludes `local-*`, so they can't end up
in a team pull request by accident.

**Back them up to your fork.** Because they are ignored, add them with `-f` (force). You only need
`-f` the first time. After that, git tracks changes to the file normally.

```bash
git add -f .github/skills/local-my-helper/
git commit -m "Add my helper skill"
git push origin main
```

**Rules**
- Don't edit the shared files (anything without `local-`). If you want one to work differently,
  copy it to a `local-` name and change your copy, or propose the change to the team (section 4).
- Don't remove the `local-*` lines from `.gitignore` in your fork. Use `git add -f` instead.

## 3. Getting the latest team updates

Do this regularly, for example at the start of each week or when a lead announces an update:

```bash
git switch main
git fetch upstream
git merge upstream/main
git push origin main
```

Or, on GitHub, click **Sync fork** on your fork's page and then run `git pull` locally.

If you followed the rules in section 2, this never conflicts: the team only changes shared files,
and you only change `local-*` files.

**If you do get a conflict** (usually because a shared file was edited locally), keep the team's
version and move your change into a `local-` copy:

```bash
git checkout --theirs <file>
git add <file>
git commit
```

## 4. Proposing a change to the team setup

Built a skill everyone should have, or found a mistake in a shared prompt? Send it upstream.
**Branch from the team's `main`, not from your own `main`.** Your `main` contains your personal
files, and those would end up in the pull request.

```bash
git fetch upstream
git switch -c improve-rap-prompt upstream/main
# make your change; for a new shared skill, use a normal name (no local- prefix)
git add <files>
git commit -m "Clarify draft handling in create-rap-bo prompt"
git push origin improve-rap-prompt
```

Then open a pull request on GitHub from `<your-user>:improve-rap-prompt` to
`RJTechRamjee:main`. A lead reviews and merges it. Once it's merged, it reaches everyone,
including you, through the next sync (section 3).

Turning a personal skill into a shared one? Copy it to the branch under a name without `local-`,
and delete your `local-` version after the PR is merged.

## Quick reference

| I want to… | Command |
|---|---|
| Get team updates | `git fetch upstream && git merge upstream/main && git push origin main` |
| Back up my personal files | `git add -f <path>` (first time), then `git commit` and `git push origin main` |
| Propose a change to the team | `git switch -c <branch> upstream/main`, commit, `git push origin <branch>`, open a PR |
| See what's personal vs shared | `git status --ignored` |

## Using GitLab instead of GitHub

The workflow is identical. Only the names in the web interface differ:

| GitHub | GitLab |
|---|---|
| Pull request | Merge request |
| **Sync fork** button | **Update fork** button on your fork's project page |
| `https://github.com/...` | `https://gitlab.com/...` or your company's GitLab URL |

All `git` commands above work unchanged.
