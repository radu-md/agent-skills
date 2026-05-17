# claude-skills

Personal collection of Claude Code skills (`SKILL.md`) for sharing with teammates.

## How to install a skill

Copy the skill folder into your personal Claude skills directory:

```powershell
Copy-Item -Recurse skills\scan-repos-generator "$env:USERPROFILE\.claude\skills\scan-repos-generator"
```

Or clone the whole repo and copy any skill you need.

## Skills

| Skill | Description |
|-------|-------------|
| [scan-repos-generator](skills/scan-repos-generator/SKILL.md) | Generates and installs a PowerShell workspace repos-scanner function, auto-discovering category folders from the workspace root. |

## Usage

After copying a skill to `~/.claude/skills/`, it becomes available immediately in Claude Code — no restart needed.

---

## scan-repos-generator

### What it does

Generates a PowerShell function that scans all Git repositories under your workspace root, grouped by subfolder, and reports branch, clean/dirty status, and remote sync state in a colour-coded table. Category subfolders are **auto-discovered** — no manual configuration needed.

### Step 1 — Generate the script with Claude Code

Open Claude Code in any project and type:

```
/scan-repos-generator
```

Claude will ask you three questions:

| Question | Default |
|----------|---------|
| Workspace root — full path to the folder whose subfolders contain repos | _(required)_ |
| Function name — what to call the PowerShell function | `repos` |
| Install target — see options below | `show` |

Claude then scans the workspace root to discover category subfolders automatically, shows you the list for confirmation, and generates the tailored function.

### Step 2 — Install

Choose one of three install targets when Claude asks:

- **`profile`** _(recommended)_ — Claude appends the function directly to your PowerShell profile (`$PROFILE`) and reloads it with `. $PROFILE`. The function is available immediately in the current session and every future one.

- **`file`** — Claude writes the function to a standalone `.ps1` file of your choice and adds a dot-source line to your `$PROFILE`. Useful if you prefer to keep your profile clean and manage scripts separately.

- **`show`** — Claude prints the generated function code in the chat without writing anything to disk. Use this if you want to review the code first, make manual edits, or paste it yourself into an existing script.

### Step 3 — Use it

Type the function name you chose (default: `repos`) and press Enter:

```powershell
# Scan all repos — branch, clean/dirty, last-known remote state
repos

# Fetch remotes first for accurate Behind/Ahead numbers, then scan
repos -Fetch

# Pull every repo and report what changed
repos -Pull

# Quick inline help
repos -Help

# Full PowerShell help
Get-Help repos -Full
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
| Cyan | Successfully pulled (with `-Pull`) |
