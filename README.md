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
