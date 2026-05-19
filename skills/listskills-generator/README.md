# 📋 listskills-generator

> Works with any AI agent that supports the [agentskills.io](https://agentskills.io/specification) skill format — Claude Code, Gemini CLI, Copilot CLI, and others.

Generates and installs a PowerShell function — **`listskills`** (or a custom name) — that scans the user's global agent skills directory and displays all installed skills in a formatted, separator-delimited table with name and description columns that wrap to the terminal width.

---

## ⚡ Quick start

Install the skill first (see [How to install a skill](../../README.md#-how-to-install-a-skill)), then open your agent in any directory and invoke it:

| Agent | Command |
|-------|---------|
| **Claude Code** | `/listskills-generator` |
| **GitHub Copilot CLI** | `listskills-generator` |
| **Gemini CLI** | `@listskills-generator` |

---

## 🔧 Configure

The agent asks three questions before generating anything:

| # | Question | Default |
|---|----------|---------|
| 1 | **Function name** — what to call the shell function | `listskills` |
| 2 | **Skills directory** — full path to the folder containing skill subdirectories | `~/.agents/skills` |
| 3 | **Install target** — `profile`, `file`, or `show` | `profile` |

---

## 📥 Install options

| Target | What happens |
|--------|-------------|
| **`profile`** ⭐ | Appends the function to `$PROFILE` and reloads it — available immediately in every future session |
| **`file`** | Writes the function to a standalone `.ps1` file and dot-sources it from `$PROFILE` |
| **`show`** | Prints the generated code in chat only — paste it yourself wherever you like |

---

## ▶️ Use it

```powershell
listskills   # (or whatever name was chosen)
```

---

## 📊 Sample output

```
#    Name                       Description
---  -------------------------  -----------------------------------------------------------
1    refactor                   Surgical code refactoring to improve maintainability without
                                changing behavior. Covers extracting functions, renaming
                                variables, breaking down god functions...
---  -------------------------  -----------------------------------------------------------
2    sql-code-review            Universal SQL code review assistant that performs
                                comprehensive security, maintainability, and code quality
                                analysis across all SQL databases...
---  -------------------------  -----------------------------------------------------------
```

---

## 📐 Output columns

| Column | Meaning |
|--------|---------|
| `#` | Sequential number |
| `Name` | Skill folder name |
| `Description` | First `description:` value found in the skill's `SKILL.md` |

- Description column **wraps** to fill the remaining terminal width automatically.
- A separator line (`---`) is printed after each skill row for readability.

---

## 📁 What counts as a skill

Every **subdirectory** inside the skills directory that contains a `SKILL.md` file is treated as an installed skill. The `description` field is read from the YAML front matter of that file.

```
~/.agents/skills/
├── refactor/
│   └── SKILL.md
├── sql-code-review/
│   └── SKILL.md
└── azure-cost-optimization/
    └── SKILL.md
```
