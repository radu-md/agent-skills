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
| Install target — `profile`, `file`, or `show` | `show` |

Claude then scans the workspace root to discover category subfolders automatically, shows you the list for confirmation, and generates the tailored function.

### Step 2 — Install

**`profile`** (recommended) — Claude appends the function to your PowerShell profile and reloads it:

```powershell
. $PROFILE
```

**`file`** — Claude writes a standalone `.ps1` and adds a dot-source line to your `$PROFILE`, then reloads:

```powershell
. $PROFILE
```

**`show`** — Claude prints the function so you can copy it manually into your profile or a script file, then run `. $PROFILE`.

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
