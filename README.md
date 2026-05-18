# 🛠️ agent-skills

Personal collection of AI agent skills (`SKILL.md`) for sharing with teammates. Works with any agent that supports the [agentskills.io](https://agentskills.io/specification) skill format — Claude Code, Gemini CLI, Copilot CLI, and others.

![Supported Agents](https://img.shields.io/badge/agents-Claude_Code_%C2%B7_Gemini_CLI_%C2%B7_Copilot_CLI-4f46e5?style=flat-square)
![Shell Support](https://img.shields.io/badge/shells-PowerShell_%C2%B7_Bash_%C2%B7_Zsh-2ea043?style=flat-square)
![Skill Format](https://img.shields.io/badge/format-agentskills.io-7c3aed?style=flat-square)

## 📦 How to install a skill

Copy the skill folder into your agent's personal skills directory. The `SKILL.md` format is the same across all supported agents.

| Agent | Personal skills directory |
|-------|--------------------------|
| **Claude Code** | `~/.claude/skills/` |
| **GitHub Copilot CLI** | `~/.copilot/skills/` |
| **Gemini CLI** | `~/.gemini/skills/` |

**Example (all platforms):**
```bash
# Claude Code
cp -r skills/scan-repos-generator ~/.claude/skills/

# GitHub Copilot CLI
cp -r skills/scan-repos-generator ~/.copilot/skills/
```

On Windows use `Copy-Item` instead:
```powershell
# Claude Code
Copy-Item -Recurse skills\scan-repos-generator ~\.claude\skills\

# GitHub Copilot CLI
Copy-Item -Recurse skills\scan-repos-generator ~\.copilot\skills\
```

## 📋 Skills

| Skill | Description |
|-------|-------------|
| [scan-repos-generator](skills/scan-repos-generator/SKILL.md) | Generates and installs a workspace repos-scanner function for PowerShell, Bash, or Zsh, auto-discovering category folders from the workspace root. |

---

## 🔍 scan-repos-generator

Generates a shell function — **PowerShell**, **Bash**, or **Zsh** — that scans all Git repositories under your workspace root, grouped by subfolder, and reports branch, clean/dirty status, and remote sync state in a colour-coded table. Category subfolders are **auto-discovered** — no manual configuration needed.

### ⚡ Quick start

First, install the skill (see [How to install a skill](#-how-to-install-a-skill) above), then open your agent in any directory and invoke it:

**Claude Code** — type in chat:
```
/scan-repos-generator
```

**GitHub Copilot CLI** — type in chat:
```
scan-repos-generator
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
```

**Bash / Zsh**
```bash
repos           # scan all repos — branch, clean/dirty, last-known remote state
repos --fetch   # fetch remotes first for accurate Behind/Ahead numbers, then scan
repos --pull    # pull every repo and report what changed
```

---

### 📊 Sample output

#### `repos` — scan all repos

```
  [apps]
  #    Name                                   Branch               Status         Remote
  --   --------------------------------------  --------------------  -------------- ------------------
    1  api-gateway                            main                 Clean           Up to date
    2  web-app                                feature/login        Dirty (3)       Ahead (2)
    3  mobile-app                             main                 Clean           Behind (1)

  [packages]
  #    Name                                   Branch               Status         Remote
  --   --------------------------------------  --------------------  -------------- ------------------
    4  ui-components                          main                 Clean           Up to date
    5  shared-utils                           main                 Dirty (1)       Up to date

  Total: 5  |  Clean: 3  |  Dirty: 2  |  Behind: 1  (use -Fetch / --fetch to refresh remote state)
```

#### `repos -Pull` / `repos --pull`

```
  [apps]
  #    Name                                   Branch               Status         Pull
  --   --------------------------------------  --------------------  -------------- ------------------
    1  api-gateway                            main                 Clean           Up to date
    2  web-app                                feature/login        Dirty (3)       Up to date
    3  mobile-app                             main                 Clean           Pulled

  [packages]
  #    Name                                   Branch               Status         Pull
  --   --------------------------------------  --------------------  -------------- ------------------
    4  ui-components                          main                 Clean           Up to date
    5  shared-utils                           main                 Dirty (1)       Up to date

  Total: 5  |  Pulled: 1  |  Up to date: 4  |  Errors: 0
```

---

### 🎨 Color legend

Row colours in the actual terminal output:

| Color | Meaning |
|-------|---------|
| 🟢 Green | Clean and in sync with remote |
| 🟡 Yellow | Dirty — has uncommitted local changes |
| 🔴 Red | Behind remote — needs a pull |
| 🔵 Cyan | Successfully pulled (with `-Pull` / `--pull`) |
| 🟣 Magenta | Category group header |

### 📐 Output columns

| Column | Meaning |
|--------|---------|
| `#` | Sequential repo number across all categories |
| `Name` | Repository folder name |
| `Branch` | Current checked-out branch |
| `Status` | `Clean` or `Dirty (N)` — N = number of changed files |
| `Remote` | `Up to date`, `Behind (N)`, `Ahead (N)`, `Diverged`, or `No remote` |
