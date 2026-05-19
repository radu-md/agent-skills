# workspace-indexer-generator

Generates a **zero-scan workspace index** so AI agents can look up local repo paths from a pre-built JSON file instead of scanning the filesystem every session.

## What it produces

| Output | Purpose |
|--------|---------|
| `repos.json` | Pre-built index: every repo listed with `scope`, `name`, `path` |
| `refreshworkspace` PowerShell function | Regenerates the index — run once, then again when repos change |
| `CLAUDE.md` / `GEMINI.md` snippet | Tells the AI to read the index instead of scanning |

## Quick start

Tell the agent:

> "Use the workspace-indexer-generator skill to set up my workspace index."

The agent will ask:

| # | Question | Default |
|---|----------|---------|
| 1 | **Workspace root** — the folder whose subfolders hold repos | _(required)_ |
| 2 | **Scope subfolder** — intermediate folder name (e.g. `Git`) | `Git` |
| 3 | **Index file path** — where to write `repos.json` | `<workspace_root>\repos.json` |
| 4 | **Install target** — `profile`, `file`, or `show` | `show` |

## Install options

| Mode | What happens |
|------|-------------|
| `profile` | Appends `refreshworkspace` to `$PROFILE` and dot-sources it |
| `file` | Writes to a `.ps1` file and adds a dot-source line to `$PROFILE` |
| `show` | Prints the function — you paste it yourself |

## Sample output

After running `refreshworkspace`:

```
Workspace index updated: 89 repos → D:\Workspace\repos.json
```

Generated `repos.json` shape:

```json
[
  { "scope": "Faigo",    "name": "faigo",        "path": "D:\\Workspace\\Faigo\\Git\\faigo" },
  { "scope": "Learnings","name": "BlazorApp",     "path": "D:\\Workspace\\Learnings\\Git\\BlazorApp" },
  { "scope": "RaduMD",   "name": "agent-skills",  "path": "D:\\Workspace\\RaduMD\\Git\\agent-skills" }
]
```

## `refreshworkspace` usage

```powershell
refreshworkspace                                     # rebuild with defaults
refreshworkspace -WorkspaceRoot E:\OtherWorkspace    # different root
refreshworkspace -OutputFile D:\my-repos.json        # custom output path
```

## The CLAUDE.md snippet

The agent adds this to your `CLAUDE.md` (or `GEMINI.md`):

```markdown
## Workspace Index

A pre-built repo index lives at `D:\Workspace\repos.json`.
**Always read this file to resolve repo paths — never scan the workspace folder tree.**

Format: `[{ "scope": "...", "name": "...", "path": "..." }]`

To find a repo: read the index, filter by `name` or `scope`, use `path` directly.
If the file is missing, ask the user to run `refreshworkspace`.
```

## How to install this skill

```bash
# Universal (Claude Code, Copilot CLI, Gemini CLI)
cp -r skills/workspace-indexer-generator ~/.agents/skills/
```

```powershell
# Windows
Copy-Item -Recurse skills\workspace-indexer-generator ~\.agents\skills\
```
