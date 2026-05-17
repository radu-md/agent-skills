---
name: scan-repos-generator
description: Use when a teammate wants to install a workspace repos-scanner PowerShell function, or when adapting the repos scanner to a different workspace root or folder structure.
---

# Workspace Repos Scanner – Generator

## Overview

Interactively generates and installs a PowerShell function that scans all Git repositories under a workspace root, grouped by subfolder, reporting branch, clean/dirty status, and remote sync state. Categories are **auto-discovered** from the filesystem — no manual listing needed.

## Setup Flow

Ask these questions **before** generating any code. If the user skips a question, use the default.

| # | Question | Default |
|---|----------|---------|
| 1 | **Workspace root** — full path to the folder whose subfolders contain repos | _(required — no default)_ |
| 2 | **Function name** — what to call the PowerShell function | `repos` |
| 3 | **Install target** — `profile` (append to `$PROFILE`), `file` (standalone `.ps1`), or `show` (print only) | `show` |

After collecting answers, **discover categories automatically**:

```powershell
# Run this to preview which subfolders contain git repos
Get-ChildItem -Directory '<WORKSPACE_ROOT>' |
    Where-Object { Get-ChildItem -Directory $_.FullName | Where-Object { Test-Path "$($_.FullName)\.git" } } |
    Select-Object -ExpandProperty Name
```

Show the discovered list to the user and ask: **"Are these the right category folders? Any to exclude?"**

Then confirm all choices before generating.

## Generated Output

Replace placeholders with the user's answers and emit the function.

```powershell
function <FUNCTION_NAME> {
<#
.SYNOPSIS
    Scans all Git repos under the workspace root and reports their status.
.DESCRIPTION
    Walks category subfolders under the workspace root, finds all Git repositories,
    and displays branch, clean/dirty status, and remote sync state in a colour-coded table.
.PARAMETER Fetch
    Fetch remote state for all repos before reporting. Gives accurate Behind/Ahead numbers.
.PARAMETER Pull
    Pull all repos and report what changed (Pulled / Up to date / Error).
.PARAMETER Help
    Print a quick usage reference and exit.
.EXAMPLE
    <FUNCTION_NAME>
    Scan all repos with last-known remote state.
.EXAMPLE
    <FUNCTION_NAME> -Fetch
    Fetch remotes first, then scan.
.EXAMPLE
    <FUNCTION_NAME> -Pull
    Pull every repo and show results.
#>
    param(
        [string]$Base = '<WORKSPACE_ROOT>',
        [switch]$Fetch,
        [switch]$Pull,
        [switch]$Help
    )

    if ($Help) {
        Write-Host ""
        Write-Host "  Usage: <FUNCTION_NAME> [-Fetch] [-Pull] [-Help]" -ForegroundColor Cyan
        Write-Host ""
        Write-Host "  (no flags)   Show branch, status, and last-known remote state"
        Write-Host "  -Fetch       Fetch all remotes first for accurate Behind/Ahead counts"
        Write-Host "  -Pull        Pull every repo and report Pulled / Up to date / Error"
        Write-Host "  -Help        Show this help"
        Write-Host ""
        Write-Host "  Tip: run 'Get-Help <FUNCTION_NAME> -Full' for full documentation." -ForegroundColor DarkGray
        Write-Host ""
        return
    }

    # Auto-discover category subfolders that contain at least one git repo
    $Categories = Get-ChildItem -Directory $Base |
        Where-Object { Get-ChildItem -Directory $_.FullName -ErrorAction SilentlyContinue |
                       Where-Object { Test-Path "$($_.FullName)\.git" } } |
        Select-Object -ExpandProperty Name

    $allDirs = @(foreach ($cat in $Categories) {
        $catPath = Join-Path $Base $cat
        Get-ChildItem -Directory $catPath |
            Where-Object { Test-Path "$($_.FullName)\.git" } |
            ForEach-Object { [PSCustomObject]@{ Dir = $_; Category = $cat } }
    })

    $total = $allDirs.Count
    $i     = 0
    $ProgressPreference = 'Continue'

    $results = foreach ($item in $allDirs) {
        $i++
        $repoPath = $item.Dir.FullName
        Write-Progress -Activity 'Scanning repos' -Status "$i/$total  $($item.Dir.Name)" -PercentComplete (($i / $total) * 100)

        $branch = git -C $repoPath rev-parse --abbrev-ref HEAD 2>$null
        if (-not $branch) { $branch = 'N/A' }

        $raw    = git -C $repoPath status --porcelain 2>$null
        $count  = if ($raw) { ($raw -split "`n" | Where-Object { $_ -ne '' }).Count } else { 0 }
        $status = if ($count -eq 0) { 'Clean' } else { "Dirty ($count)" }

        $pullResult = $null
        if ($Pull) {
            $pullOut = git -C $repoPath pull 2>&1
            $pullResult = if ($LASTEXITCODE -ne 0) { 'Error' }
                          elseif ($pullOut -match 'Already up to date') { 'Up to date' }
                          else { 'Pulled' }
        } elseif ($Fetch) {
            git -C $repoPath fetch --quiet 2>$null
        }

        $hasUpstream = git -C $repoPath rev-parse --abbrev-ref '@{u}' 2>$null
        $remote = if (-not $hasUpstream) { 'No remote' }
                  else {
                      $behind = [int](git -C $repoPath rev-list --count "HEAD..@{u}" 2>$null)
                      $ahead  = [int](git -C $repoPath rev-list --count "@{u}..HEAD" 2>$null)
                      if     ($behind -gt 0 -and $ahead -gt 0) { "Diverged (+$ahead/-$behind)" }
                      elseif ($behind -gt 0)                   { "Behind ($behind)" }
                      elseif ($ahead  -gt 0)                   { "Ahead ($ahead)" }
                      else                                     { 'Up to date' }
                  }

        [PSCustomObject]@{
            '#'        = $i
            Category   = $item.Category
            Name       = $item.Dir.Name
            Branch     = $branch
            Status     = $status
            Remote     = $remote
            PullResult = $pullResult
        }
    }

    Write-Progress -Activity 'Scanning repos' -Completed

    $clean  = ($results | Where-Object { $_.Status -eq 'Clean' }).Count
    $dirty  = $results.Count - $clean
    $behind = ($results | Where-Object { $_.Remote -like 'Behind*' }).Count
    $pulled = ($results | Where-Object { $_.PullResult -eq 'Pulled' }).Count

    $remoteHeader = if ($Pull) { 'Pull' } else { 'Remote' }
    $header = "{0,3}  {1,-38} {2,-20} {3,-14} {4}" -f '#', 'Name', 'Branch', 'Status', $remoteHeader
    $sep    = "{0,3}  {1,-38} {2,-20} {3,-14} {4}" -f '--', ('-'*38), ('-'*20), ('-'*14), ('-'*18)

    foreach ($cat in $Categories) {
        $catResults = @($results | Where-Object { $_.Category -eq $cat })
        if ($catResults.Count -eq 0) { continue }

        Write-Host ""
        Write-Host "  [$cat]" -ForegroundColor Magenta
        Write-Host $header    -ForegroundColor Cyan
        Write-Host $sep       -ForegroundColor DarkGray

        foreach ($r in $catResults) {
            $displayRemote = if ($Pull) { $r.PullResult } else { $r.Remote }
            $color = switch ($true) {
                { $Pull -and $r.PullResult -eq 'Error'  } { 'Red';    break }
                { $Pull -and $r.PullResult -eq 'Pulled' } { 'Cyan';   break }
                { $r.Remote -like 'Behind*'             } { 'Red';    break }
                { $r.Status -ne 'Clean'                 } { 'Yellow'; break }
                default                                    { 'Green' }
            }
            $line = "{0,3}  {1,-38} {2,-20} {3,-14} {4}" -f $r.'#', $r.Name, $r.Branch, $r.Status, $displayRemote
            Write-Host $line -ForegroundColor $color
        }
    }

    Write-Host ""
    if ($Pull) {
        $errors = ($results | Where-Object { $_.PullResult -eq 'Error' }).Count
        Write-Host ("Total: {0}  |  Pulled: {1}  |  Up to date: {2}  |  Errors: {3}" -f
            $results.Count, $pulled, ($results.Count - $pulled - $errors), $errors) -ForegroundColor Cyan
    } else {
        $fetchNote = if ($Fetch) { '' } else { '  (use -Fetch to refresh remote state)' }
        Write-Host ("Total: {0}  |  Clean: {1}  |  Dirty: {2}  |  Behind: {3}{4}" -f
            $results.Count, $clean, $dirty, $behind, $fetchNote) -ForegroundColor Cyan
    }
}
```

## Install Steps

**`profile`** — Append to the user's PowerShell profile:
```powershell
Add-Content -Path $PROFILE -Value "`n<generated function>"
. $PROFILE
```
Then type the function name (e.g. `repos`) and press Enter — that's all.

**`file`** — Write to a `.ps1` file and add a dot-source line to `$PROFILE`:
```powershell
# Add to $PROFILE:
. "C:\path\to\repos-scanner.ps1"
```
Then run `. $PROFILE` and type the function name to use it.

**`show`** — Print the generated function only; the user copies it manually into their profile or a script file, then runs `. $PROFILE`.

---

## How to Use the Installed Function

After installation, reload the profile with `. $PROFILE` and use:

```powershell
# Scan all repos — branch, clean/dirty, last-known remote state
<function-name>

# Fetch remotes first for accurate Behind/Ahead numbers, then scan
<function-name> -Fetch

# Pull every repo and report what changed
<function-name> -Pull

# Quick inline help
<function-name> -Help

# Full PowerShell help (synopsis, parameters, examples)
Get-Help <function-name> -Full
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
