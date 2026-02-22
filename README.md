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
| **si-skill-observer** | Claude Code skills, commands, agents, CLAUDE.md rules — prompt clarity, tool restrictions, consistency |
| **si-code-observer** | Source code — anti-patterns, dead code, performance issues, style drift |
| **si-workflow-observer** | Session execution — error patterns, wasted steps, missed parallelization, user corrections |
| **si-structure-observer** | Project organization — file layout, naming conventions, module boundaries, config health |

## Installation

Copy the agents and commands to your global Claude Code config:

```bash
# Clone the repo
git clone https://github.com/YOUR_USERNAME/self-improver.git
cd self-improver

# Copy agents
cp .claude/agents/si-*.md ~/.claude/agents/

# Copy commands
mkdir -p ~/.claude/commands/si
cp .claude/commands/si/*.md ~/.claude/commands/si/

# Copy the /us command (update state)
cp .claude/commands/us.md ~/.claude/commands/
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

# Update project state snapshot
/us
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
      observe.md    # Orchestrator — spawns observers
      apply.md      # Routes approved items by category
      status.md     # Quick status view
    us.md           # Update state.md snapshot
  agents/
    si-skill-observer.md
    si-code-observer.md
    si-workflow-observer.md
    si-structure-observer.md
```

## Design Principles

- **Fresh context per observer** — Monolithic analysis degrades as context fills. Parallel specialized agents don't.
- **Human in the loop** — SI proposes, you approve. No automatic source code changes.
- **Evidence-based** — Every proposal references specific files, lines, or session events.
- **Lightweight reports** — Written for Claude to consume, not for humans to study.

## License

MIT
