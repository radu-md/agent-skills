---
name: listskills-generator
description: Use when someone wants to install a `listskills` shell function that lists all globally installed Copilot CLI / agent skills from the ~/.agents/skills directory in a formatted, readable table.
---

# listskills Generator

## Overview

Works with any AI agent that supports the [agentskills.io](https://agentskills.io/specification) skill format (Claude Code, Gemini CLI, Copilot CLI, and others).

Generates and installs a PowerShell function — **`listskills`** (or a custom name) — that scans the user's global agent skills directory and displays all installed skills in a colour-coded, separator-delimited table with name and description columns that wrap to the terminal width.

## Setup Flow

Ask these questions **before** generating any code. If the user skips a question, use the default.

| # | Question | Default |
|---|----------|---------|
| 1 | **Function name** — what to call the shell function | `listskills` |
| 2 | **Skills directory** — full path to the folder containing skill subdirectories | `~/.agents/skills` (i.e. `$HOME\.agents\skills` on Windows) |
| 3 | **Install target** — `profile` (append to `$PROFILE`), `file` (write a `.ps1`), or `show` (print only) | `profile` |

After collecting answers, confirm all choices before generating.

## Generated Output

Replace `<FUNCTION_NAME>` and `<SKILLS_DIR>` with the user's answers and emit the function below.

### PowerShell

```powershell
function <FUNCTION_NAME> {
    $skillsDir = '<SKILLS_DIR>'
    if (-not (Test-Path $skillsDir)) {
        Write-Warning "Skills directory not found: $skillsDir"
        return
    }
    $i = 0
    $skills = Get-ChildItem $skillsDir -Directory | ForEach-Object {
        $i++
        $skillFile = Join-Path $_.FullName "SKILL.md"
        $description = ""
        if (Test-Path $skillFile) {
            $descLine = Get-Content $skillFile | Select-String "^description:" | Select-Object -First 1
            if ($descLine) {
                $description = $descLine.Line -replace "^description:\s*'?", "" -replace "'?$", ""
            }
        }
        [PSCustomObject]@{ '#' = $i; Name = $_.Name; Description = $description }
    }

    $numW  = 3
    $nameW = ($skills | ForEach-Object { $_.Name.Length } | Measure-Object -Maximum).Maximum
    $descW = [Math]::Max(20, ($Host.UI.RawUI.WindowSize.Width - $numW - $nameW - 6))
    $sep   = "{0}  {1}  {2}" -f ('-' * $numW), ('-' * $nameW), ('-' * $descW)

    Write-Host ("{0,-$numW}  {1,-$nameW}  {2}" -f '#', 'Name', 'Description') -ForegroundColor Cyan
    Write-Host $sep -ForegroundColor DarkGray

    foreach ($s in $skills) {
        $words = $s.Description -split ' '
        $lines = @()
        $line  = ''
        foreach ($word in $words) {
            if (($line + ' ' + $word).TrimStart().Length -le $descW) {
                $line = ($line + ' ' + $word).TrimStart()
            } else {
                if ($line) { $lines += $line }
                $line = $word
            }
        }
        if ($line) { $lines += $line }

        Write-Host ("{0,-$numW}  {1,-$nameW}  {2}" -f $s.'#', $s.Name, $lines[0])
        for ($j = 1; $j -lt $lines.Count; $j++) {
            Write-Host ("{0,-$numW}  {1,-$nameW}  {2}" -f '', '', $lines[$j])
        }
        Write-Host $sep -ForegroundColor DarkGray
    }
}
```

## Install Steps

### `profile` — Append to PowerShell profile

```powershell
Add-Content -Path $PROFILE -Value "`n<generated function>"
. $PROFILE
```

Then type the function name (e.g. `listskills`) and press Enter to verify it works.

### `file` — Write to a `.ps1` file and dot-source from profile

1. Save the generated function to a file, e.g. `C:\scripts\listskills.ps1`
2. Add this line to `$PROFILE`:
   ```powershell
   . "C:\scripts\listskills.ps1"
   ```
3. Reload: `. $PROFILE`

### `show` — Print only

Print the generated function; the user pastes it into their profile or script file, then runs `. $PROFILE`.

---

## How to Use the Installed Function

```powershell
listskills   # (or whatever name was chosen)
```

**Output columns:**

| Column | Meaning |
|--------|---------|
| # | Sequential number |
| Name | Skill folder name |
| Description | First `description:` value found in the skill's `SKILL.md` |

**Layout behaviour:**
- Description column wraps to fill remaining terminal width automatically.
- A separator line (`---`) is printed after each skill row for readability.

## What Counts as a Skill

The function treats every **subdirectory** inside the skills directory that contains a `SKILL.md` file as an installed skill. The `description` field is read from the YAML front matter of that file.

Example skills directory layout:
```
~/.agents/skills/
├── refactor/
│   └── SKILL.md
├── sql-code-review/
│   └── SKILL.md
└── azure-cost-optimization/
    └── SKILL.md
```
