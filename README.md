# 🛠️ agent-skills

Personal collection of AI agent skills (`SKILL.md`) for sharing with teammates. Works with any agent that supports the [agentskills.io](https://agentskills.io/specification) skill format — Claude Code, Gemini CLI, Copilot CLI, and others.

![Supported Agents](https://img.shields.io/badge/agents-Claude_Code_%C2%B7_Gemini_CLI_%C2%B7_Copilot_CLI-4f46e5?style=flat-square)
![Shell Support](https://img.shields.io/badge/shells-PowerShell_%C2%B7_Bash_%C2%B7_Zsh-2ea043?style=flat-square)
![Skill Format](https://img.shields.io/badge/format-agentskills.io-7c3aed?style=flat-square)

## 📦 How to install a skill

Copy the skill folder into your agent's personal skills directory. The `SKILL.md` format is the same across all supported agents.

| Agent | Personal skills directory |
|-------|--------------------------|
| **Any agent** ⭐ | `~/.agents/skills/` — universal path, works across all supporting agents |
| **Claude Code** | `~/.claude/skills/` |
| **GitHub Copilot CLI** | `~/.copilot/skills/` |
| **Gemini CLI** | `~/.gemini/skills/` |

**Example (all platforms):**
```bash
# Universal — works with Claude Code, Copilot CLI, Gemini CLI, and others
cp -r skills/scan-repos-generator ~/.agents/skills/
cp -r skills/listskills-generator ~/.agents/skills/

# Or agent-specific:
cp -r skills/scan-repos-generator ~/.claude/skills/     # Claude Code
cp -r skills/scan-repos-generator ~/.copilot/skills/    # GitHub Copilot CLI
```

On Windows use `Copy-Item` instead:
```powershell
# Universal
Copy-Item -Recurse skills\scan-repos-generator ~\.agents\skills\
Copy-Item -Recurse skills\listskills-generator ~\.agents\skills\

# Or agent-specific:
Copy-Item -Recurse skills\scan-repos-generator ~\.claude\skills\    # Claude Code
Copy-Item -Recurse skills\scan-repos-generator ~\.copilot\skills\   # GitHub Copilot CLI
```

## 📋 Skills

| Skill | Description |
|-------|-------------|
| [scan-repos-generator](skills/scan-repos-generator/README.md) | Generates and installs a workspace repos-scanner function for PowerShell, Bash, or Zsh, auto-discovering category folders from the workspace root. |
| [listskills-generator](skills/listskills-generator/README.md) | Generates and installs a PowerShell function that lists all globally installed agent skills in a formatted, terminal-width-aware table. |
