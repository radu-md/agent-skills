# agent-skills

Personal collection of AI agent skills (`SKILL.md`) for sharing with teammates. Works with any agent that supports the [agentskills.io](https://agentskills.io/specification) skill format — Claude Code, Gemini CLI, Copilot CLI, and others.

## How to install a skill

Check your agent's documentation for the skills directory path, then copy the skill folder there. The `SKILL.md` format is the same across all supported agents.

## Skills

| Skill | Description |
|-------|-------------|
| [scan-repos-generator](skills/scan-repos-generator/SKILL.md) | Generates and installs a workspace repos-scanner function for PowerShell, Bash, or Zsh, auto-discovering category folders from the workspace root. |

## Usage

After copying a skill to the agent's skills directory, it becomes available immediately — no restart needed.

---

## scan-repos-generator

### What it does

Generates a shell function — **PowerShell, Bash, or Zsh** — that scans all Git repositories under your workspace root, grouped by subfolder, and reports branch, clean/dirty status, and remote sync state in a colour-coded table. Category subfolders are **auto-discovered** — no manual configuration needed.

### Step 1 — Generate the script

Open your AI agent in any project and type:

```
/scan-repos-generator
```

The agent will ask you four questions:

| Question | Default |
|----------|---------|
| Workspace root — full path to the folder whose subfolders contain repos | _(required)_ |
| Function name — what to call the shell function | `repos` |
| Shell — `pwsh` (PowerShell), `bash`, or `zsh` | `pwsh` |
| Install target — see options below | `show` |

The agent then scans the workspace root to discover category subfolders automatically, shows you the list for confirmation, and generates the tailored function.

### Step 2 — Install

Choose one of three install targets when the agent asks:

- **`profile`** _(recommended)_ — The agent appends the function directly to your shell profile (`$PROFILE` for PowerShell, `~/.bashrc` or `~/.zshrc` for Bash/Zsh) and reloads it. The function is available immediately in the current session and every future one.

- **`file`** — The agent writes the function to a standalone script file (`.ps1` for PowerShell, `.sh` for Bash/Zsh) and adds a source line to your shell profile. Useful if you prefer to keep your profile clean and manage scripts separately.

- **`show`** — The agent prints the generated function code in the chat without writing anything to disk. Use this if you want to review the code first, make manual edits, or paste it yourself into an existing script.

### Step 3 — Use it

Type the function name you chose (default: `repos`) and press Enter:

**PowerShell:**
```powershell
repos           # scan all repos — branch, clean/dirty, last-known remote state
repos -Fetch    # fetch remotes first for accurate Behind/Ahead numbers, then scan
repos -Pull     # pull every repo and report what changed
repos -Help     # quick inline help
Get-Help repos -Full  # full PowerShell help
```

**Bash / Zsh:**
```bash
repos           # scan all repos — branch, clean/dirty, last-known remote state
repos --fetch   # fetch remotes first for accurate Behind/Ahead numbers, then scan
repos --pull    # pull every repo and report what changed
repos --help    # quick inline help
```

### Sample output

#### `repos` (no flags)

```
  [apps]
  #   Name                                   Branch               Status         Remote
  --  --------------------------------------  --------------------  --------------  ------------------
    1  api-gateway                            main                 Clean           Up to date
    2  web-app                                feature/login        Dirty (3)       Ahead (2)
    3  mobile-app                             main                 Clean           Behind (1)

  [packages]
  #   Name                                   Branch               Status         Remote
  --  --------------------------------------  --------------------  --------------  ------------------
    4  ui-components                          main                 Clean           Up to date
    5  shared-utils                           main                 Dirty (1)       Up to date

Total: 5  |  Clean: 3  |  Dirty: 2  |  Behind: 1  (use -Fetch / --fetch to refresh remote state)
```

#### `repos -Pull` / `repos --pull`

```
  [apps]
  #   Name                                   Branch               Status         Pull
  --  --------------------------------------  --------------------  --------------  ------------------
    1  api-gateway                            main                 Clean           Up to date
    2  web-app                                feature/login        Dirty (3)       Up to date
    3  mobile-app                             main                 Clean           Pulled

  [packages]
  #   Name                                   Branch               Status         Pull
  --  --------------------------------------  --------------------  --------------  ------------------
    4  ui-components                          main                 Clean           Up to date
    5  shared-utils                           main                 Dirty (1)       Up to date

Total: 5  |  Pulled: 1  |  Up to date: 4  |  Errors: 0
```

**Output columns:**

| Column | Meaning |
|--------|---------|
| Name | Repository folder name |
| Branch | Current checked-out branch |
| Status | `Clean` or `Dirty (N)` — N = number of changed files |
| Remote | `Up to date`, `Behind (N)`, `Ahead (N)`, `Diverged`, or `No remote` |

**Row colors:**

| Color | Meaning |
|-------|---------|
| Green | Clean and in sync |
| Yellow | Dirty (uncommitted changes) |
| Red | Behind remote (needs pull) |
| Cyan | Successfully pulled (with `-Pull` / `--pull`) |
