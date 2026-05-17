# 🛠️ agent-skills

Personal collection of AI agent skills (`SKILL.md`) for sharing with teammates. Works with any agent that supports the [agentskills.io](https://agentskills.io/specification) skill format — Claude Code, Gemini CLI, Copilot CLI, and others.

![Supported Agents](https://img.shields.io/badge/agents-Claude_Code_%C2%B7_Gemini_CLI_%C2%B7_Copilot_CLI-4f46e5?style=flat-square)
![Shell Support](https://img.shields.io/badge/shells-PowerShell_%C2%B7_Bash_%C2%B7_Zsh-2ea043?style=flat-square)
![Skill Format](https://img.shields.io/badge/format-agentskills.io-7c3aed?style=flat-square)

## 📦 How to install a skill

Check your agent's documentation for the skills directory path, then copy the skill folder there. The `SKILL.md` format is the same across all supported agents.

## 📋 Skills

| Skill | Description |
|-------|-------------|
| [scan-repos-generator](skills/scan-repos-generator/SKILL.md) | Generates and installs a workspace repos-scanner function for PowerShell, Bash, or Zsh, auto-discovering category folders from the workspace root. |

---

## 🔍 scan-repos-generator

Generates a shell function — **PowerShell**, **Bash**, or **Zsh** — that scans all Git repositories under your workspace root, grouped by subfolder, and reports branch, clean/dirty status, and remote sync state in a colour-coded table. Category subfolders are **auto-discovered** — no manual configuration needed.

### ⚡ Quick start

Open your AI agent in any project and type:

```
/scan-repos-generator
```

### 🔧 Step 1 — Configure

The agent asks four questions before generating anything:

| # | Question | Default |
|---|----------|---------|
| 1 | **Workspace root** — full path to the folder whose subfolders contain repos | _(required)_ |
| 2 | **Function name** — what to call the shell function | `repos` |
| 3 | **Shell** — `pwsh` (PowerShell), `bash`, or `zsh` | `pwsh` |
| 4 | **Install target** — `profile`, `file`, or `show` | `show` |

The agent scans the workspace root, shows you the discovered category folders for confirmation, then generates the tailored function.

### 📥 Step 2 — Install

| Target | What happens |
|--------|-------------|
| **`profile`** ⭐ | Appends the function to your shell profile (`$PROFILE` / `~/.bashrc` / `~/.zshrc`) and reloads it — available immediately in the current session and every future one |
| **`file`** | Writes the function to a standalone `.ps1` / `.sh` file and adds a source line to your profile |
| **`show`** | Prints the generated code in chat only — paste it yourself wherever you like |

### ▶️ Step 3 — Use it

**PowerShell**
```powershell
repos                 # scan all repos — branch, clean/dirty, last-known remote state
repos -Fetch          # fetch remotes first for accurate Behind/Ahead numbers, then scan
repos -Pull           # pull every repo and report what changed
repos -Help           # quick inline help
Get-Help repos -Full  # full PowerShell help
```

**Bash / Zsh**
```bash
repos           # scan all repos — branch, clean/dirty, last-known remote state
repos --fetch   # fetch remotes first for accurate Behind/Ahead numbers, then scan
repos --pull    # pull every repo and report what changed
repos --help    # quick inline help
```

---

### 📊 Sample output

> The colours below render exactly as they appear in the terminal.

#### `repos` — scan all repos

```ansi
[35m  [apps][0m
[36m  #    Name                                   Branch               Status         Remote[0m
[90m  --   --------------------------------------  --------------------  -------------- ------------------[0m
[32m    1  api-gateway                            main                 Clean           Up to date[0m
[33m    2  web-app                                feature/login        Dirty (3)       Ahead (2)[0m
[31m    3  mobile-app                             main                 Clean           Behind (1)[0m

[35m  [packages][0m
[36m  #    Name                                   Branch               Status         Remote[0m
[90m  --   --------------------------------------  --------------------  -------------- ------------------[0m
[32m    4  ui-components                          main                 Clean           Up to date[0m
[33m    5  shared-utils                           main                 Dirty (1)       Up to date[0m

[36mTotal: 5  |  Clean: 3  |  Dirty: 2  |  Behind: 1  (use -Fetch / --fetch to refresh remote state)[0m
```

#### `repos -Pull` / `repos --pull`

```ansi
[35m  [apps][0m
[36m  #    Name                                   Branch               Status         Pull[0m
[90m  --   --------------------------------------  --------------------  -------------- ------------------[0m
[32m    1  api-gateway                            main                 Clean           Up to date[0m
[33m    2  web-app                                feature/login        Dirty (3)       Up to date[0m
[36m    3  mobile-app                             main                 Clean           Pulled[0m

[35m  [packages][0m
[36m  #    Name                                   Branch               Status         Pull[0m
[90m  --   --------------------------------------  --------------------  -------------- ------------------[0m
[32m    4  ui-components                          main                 Clean           Up to date[0m
[33m    5  shared-utils                           main                 Dirty (1)       Up to date[0m

[36mTotal: 5  |  Pulled: 1  |  Up to date: 4  |  Errors: 0[0m
```

---

### 🎨 Color legend

| Color | Meaning |
|-------|---------|
| 🟢 **Green** | Clean and in sync with remote |
| 🟡 **Yellow** | Dirty — has uncommitted local changes |
| 🔴 **Red** | Behind remote — needs a pull |
| 🔵 **Cyan** | Successfully pulled (with `-Pull` / `--pull`) |
| 🟣 **Magenta** | Category group header |

### 📐 Output columns

| Column | Meaning |
|--------|---------|
| `#` | Sequential repo number across all categories |
| `Name` | Repository folder name |
| `Branch` | Current checked-out branch |
| `Status` | `Clean` or `Dirty (N)` — N = number of changed files |
| `Remote` | `Up to date`, `Behind (N)`, `Ahead (N)`, `Diverged`, or `No remote` |