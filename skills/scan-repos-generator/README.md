# 🗂️ scan-repos-generator

> Works with any AI agent that supports the [agentskills.io](https://agentskills.io/specification) skill format — Claude Code, Gemini CLI, Copilot CLI, and others.

Interactively generates and installs a shell function — **PowerShell, Bash, or Zsh** — that scans all Git repositories under a workspace root, grouped by subfolder, reporting branch, clean/dirty status, and remote sync state in a colour-coded table. Categories are **auto-discovered** from the filesystem — no manual listing needed.

---

## ⚡ Quick start

Install the skill first (see [How to install a skill](../../README.md#-how-to-install-a-skill)), then open your agent in any directory and invoke it:

| Agent | Command |
|-------|---------|
| **Claude Code** | `/scan-repos-generator` |
| **GitHub Copilot CLI** | `scan-repos-generator` |
| **Gemini CLI** | `@scan-repos-generator` |

---

## 🔧 Step 1 — Configure

The agent asks four questions before generating anything:

| # | Question | Default |
|---|----------|---------|
| 1 | **Workspace root** — full path to the folder whose subfolders contain repos | _(required)_ |
| 2 | **Function name** — what to call the shell function | `repos` |
| 3 | **Shell** — `pwsh` (PowerShell), `bash`, or `zsh` | `pwsh` |
| 4 | **Install target** — `profile`, `file`, or `show` | `show` |

The agent scans the workspace root, shows you the discovered category folders for confirmation, then generates the tailored function.

---

## 📥 Step 2 — Install

| Target | What happens |
|--------|-------------|
| **`profile`** ⭐ | Appends the function to your shell profile (`$PROFILE` / `~/.bashrc` / `~/.zshrc`) and reloads it — available immediately in every future session |
| **`file`** | Writes the function to a standalone `.ps1` / `.sh` file and adds a source line to your profile |
| **`show`** | Prints the generated code in chat only — paste it yourself wherever you like |

---

## ▶️ Step 3 — Use it

**PowerShell**
```powershell
repos                 # scan all repos — branch, clean/dirty, last-known remote state
repos -Fetch          # fetch remotes first for accurate Behind/Ahead numbers, then scan
repos -Pull           # pull every repo and report what changed
repos -Help           # print quick usage reference
Get-Help repos -Full  # full PowerShell documentation
```

**Bash / Zsh**
```bash
repos           # scan all repos — branch, clean/dirty, last-known remote state
repos --fetch   # fetch remotes first for accurate Behind/Ahead numbers, then scan
repos --pull    # pull every repo and report what changed
repos --help    # print quick usage reference
```

---

## 📊 Sample output

### `repos` — scan all repos

```
  [apps]
  #    Name                                    Branch                Status         Remote
  --   --------------------------------------  --------------------  -------------- ------------------
    1  api-gateway                             main                  Clean          Up to date
    2  web-app                                 feature/login         Dirty (3)      Ahead (2)
    3  mobile-app                              main                  Clean          Behind (1)

  [packages]
  #    Name                                    Branch                Status         Remote
  --   --------------------------------------  --------------------  -------------- ------------------
    4  ui-components                           main                  Clean          Up to date
    5  shared-utils                            main                  Dirty (1)      Up to date

  Total: 5  |  Clean: 3  |  Dirty: 2  |  Behind: 1  (use -Fetch / --fetch to refresh remote state)
```

### `repos -Pull` / `repos --pull`

```
  [apps]
  #    Name                                    Branch                Status         Pull
  --   --------------------------------------  --------------------  -------------- ------------------
    1  api-gateway                             main                  Clean          Up to date
    2  web-app                                 feature/login         Dirty (3)      Up to date
    3  mobile-app                              main                  Clean          Pulled

  [packages]
  #    Name                                    Branch                Status         Pull
  --   --------------------------------------  --------------------  -------------- ------------------
    4  ui-components                           main                  Clean          Up to date
    5  shared-utils                            main                  Dirty (1)      Up to date

  Total: 5  |  Pulled: 1  |  Up to date: 4  |  Errors: 0
```

---

## 🎨 Color legend

| Color | Meaning |
|-------|---------|
| 🟢 Green | Clean and in sync with remote |
| 🟡 Yellow | Dirty — has uncommitted local changes |
| 🔴 Red | Behind remote — needs a pull |
| 🔵 Cyan | Successfully pulled (with `-Pull` / `--pull`) |
| 🟣 Magenta | Category group header |

## 📐 Output columns

| Column | Meaning |
|--------|---------|
| `#` | Sequential repo number across all categories |
| `Name` | Repository folder name |
| `Branch` | Current checked-out branch |
| `Status` | `Clean` or `Dirty (N)` — N = number of changed files |
| `Remote` | `Up to date`, `Behind (N)`, `Ahead (N)`, `Diverged`, or `No remote` |
