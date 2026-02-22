# Self-Improver (SI)

A self-improvement system for Claude Code that observes execution patterns, analyzes quality across multiple dimensions, and proposes actionable improvements — all through parallel specialized agents.

## Bootstrap

Read `.claude/project/state.md` at the start of every session. It describes what exists and what's next.
If `.claude/project/vision.md` exists, read it too — the gap between vision and state is the work.
If `.claude/project/reports/active-directives.md` exists, read it too — these are learned directives from previous observations.

## Architecture

SI follows the **orchestrator + specialized subagent** pattern:

- **Commands** (`commands/si/`) are orchestrators that manage workflow and user interaction
- **Agents** (`agents/`) are specialized observers spawned in parallel for fresh-context analysis
- **Reports** (`.claude/project/reports/`) store observation results and improvement proposals

### Commands

| Command | Purpose |
|---------|---------|
| `/si:observe` | Spawn observers, collect findings, write improvement-plan.md |
| `/si:apply` | User selects items, routes by category, executes changes |
| `/si:status` | Quick view of pending/applied/rejected items |
| `/si:us` | Rebuild state.md from actual codebase |

### Agents

| Agent | Domain | Tools |
|-------|--------|-------|
| `si-skill-observer` | Skills, commands, agents, CLAUDE.md rules | Read, Glob, Grep |
| `si-code-observer` | Source code quality, anti-patterns | Read, Glob, Grep, Bash |
| `si-workflow-observer` | Session efficiency, error patterns | Read, Glob, Grep |
| `si-structure-observer` | File organization, naming, configs | Read, Glob, Grep, Bash |

## Conventions

- Observers return findings only — never modify files
- Improvement proposals use the standardized format with Category, Evidence, Current, Proposed, Impact, Status
- improvement-plan.md is append-only for pending items; items move between Pending/Applied/Rejected sections
- active-directives.md stays concise (read every session)
- All observers spawn in parallel — sequential spawning defeats the purpose

## Development

This repo is the canonical source for SI. Changes here should be synced back to `~/.claude/` for global availability.

When modifying agents or commands:
1. Edit files in this repo
2. Test with `/si:observe` on a real project
3. Copy working versions to `~/.claude/agents/` and `~/.claude/commands/si/`
