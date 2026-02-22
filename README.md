# Self-Improver (SI)

A self-improvement system for [Claude Code](https://claude.com/claude-code) that observes execution patterns, analyzes quality, and proposes actionable improvements through parallel specialized agents.

## What It Does

SI adds a meta-cognitive loop to Claude Code:

1. **Observe** (`/si:observe`) — Spawns specialized observer agents in parallel to analyze your project across four dimensions: skill quality, source code, workflow efficiency, and project structure
2. **Review** (`/si:status`) — Shows pending improvement proposals grouped by category
3. **Apply** (`/si:apply`) — Lets you select which improvements to apply, with human approval at every step

Each observer runs in a fresh context window, giving it full attention on its domain instead of competing for tokens in a monolithic analysis.

## Observers

| Observer | What It Analyzes |
|----------|-----------------|
| **si-observe-skill** | Claude Code skills, commands, agents, CLAUDE.md rules — prompt clarity, tool restrictions, consistency |
| **si-observe-code** | Source code — anti-patterns, dead code, performance issues, style drift |
| **si-observe-workflow** | Session execution — error patterns, wasted steps, missed parallelization, user corrections |
| **si-observe-structure** | Project organization — file layout, naming conventions, module boundaries, config health |
| **si-observe-security** | Quick security surface scan — secrets, injection patterns, configuration gaps |

## Security Audit

For deep security analysis, run `/si:security-audit`. This spawns 4 specialized audit agents in parallel:

| Agent | What It Audits |
|-------|---------------|
| **si-audit-secrets** | Hardcoded credentials, .env exposure, git history leaks, .gitignore gaps |
| **si-audit-deps** | CVEs, outdated packages, lockfile integrity, typosquatting risks |
| **si-audit-code** | OWASP Top 10 code vulnerabilities — injection, auth bypass, SSRF, XSS |
| **si-audit-infra** | Docker, CI/CD, IaC misconfigurations, security headers |

## Installation

Copy the agents and commands to your global Claude Code config:

```bash
# Clone the repo
git clone https://github.com/giovicordova/self-improver.git
cd self-improver

# Copy agents
cp .claude/agents/si-*.md ~/.claude/agents/

# Copy commands
mkdir -p ~/.claude/commands/si
cp .claude/commands/si/*.md ~/.claude/commands/si/

# The /si:us command is included in the si/ commands directory above
```

## Usage

In any Claude Code session:

```
# Run observation on the current project
/si:observe

# Check what was found
/si:status

# Apply selected improvements
/si:apply

# Deep security audit
/si:security-audit

# Update project state snapshot
/si:us
```

## How It Works

SI follows the **orchestrator + specialized subagent** pattern:

- `/si:observe` is the orchestrator — it reads project context, determines which observers are relevant, spawns them in parallel, then merges results into `improvement-plan.md`
- Each observer is a subagent with fresh context and domain-specific analysis logic
- `/si:apply` routes approved improvements by category: skill edits go directly, code changes go through plan mode, workflow items update directives, structure items handle file moves with import updates

Reports live in `.claude/project/reports/` and are designed to be read by Claude at session start — concise and structured, not prose.

## Architecture

```
.claude/
  commands/
    si/
      observe.md         # Orchestrator — spawns observers
      apply.md           # Routes approved items by category
      status.md          # Quick status view
      security-audit.md  # Deep security audit orchestrator
      us.md              # Update state.md snapshot
  agents/
    si-observe-skill.md       # Skill/command/agent quality
    si-observe-code.md        # Source code analysis
    si-observe-workflow.md    # Session efficiency
    si-observe-structure.md   # File organization
    si-observe-security.md    # Quick security scan
    si-audit-secrets.md       # Deep: credentials & leaks
    si-audit-deps.md          # Deep: dependency CVEs
    si-audit-code.md          # Deep: OWASP Top 10
    si-audit-infra.md         # Deep: infrastructure config
```

> **Note:** The `.claude/CLAUDE.md` and `.claude/project/` files in this repo are specific to SI's own development. You don't need them to use SI in your projects.

## Design Principles

- **Fresh context per observer** — Monolithic analysis degrades as context fills. Parallel specialized agents don't.
- **Human in the loop** — SI proposes, you approve. No automatic source code changes.
- **Evidence-based** — Every proposal references specific files, lines, or session events.
- **Lightweight reports** — Written for Claude to consume, not for humans to study.

---

Built by [@giovicordova](https://github.com/giovicordova) and Claude Code.
