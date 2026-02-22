# SI (Self-Improvement) System — Implementation Plan

## Context

The current `/observe` and `/apply` commands work but are monolithic — one observer tries to analyze everything (skills, code, workflow, data) in a single context window, limiting depth. The goal is to build a hierarchical system where a main orchestrator spawns specialized observer subagents in parallel, each with fresh context for peak-quality analysis.

## Architecture

Following the proven GSD pattern: **Commands** (orchestrators) + **Subagents** (specialized workers).

No SKILL.md needed — SI is user-invoked, not auto-triggered.

```
~/.claude/
├── commands/
│   ├── si/
│   │   ├── observe.md      # /si:observe — orchestrator, spawns observers
│   │   ├── apply.md        # /si:apply — orchestrator, routes by category
│   │   └── status.md       # /si:status — view reports and pending items
│   ├── observe.md          # DELETE (replaced)
│   └── apply.md            # DELETE (replaced)
├── skills/
│   ├── observe.md          # DELETE (logic moves to agents)
│   └── apply.md            # DELETE (logic moves to agents)
├── agents/
│   ├── si-skill-observer.md      # Claude skill/command/agent quality
│   ├── si-code-observer.md       # Source code quality & patterns
│   ├── si-workflow-observer.md   # Session execution & efficiency
│   └── si-structure-observer.md  # File organization & project health
```

## What Each Piece Does

### Commands (Orchestrators)

**`/si:observe`** — Main entry point for observation
1. Reads project context (CLAUDE.md, state.md, previous reports)
2. Detects which categories are relevant:
   - Has `.claude/skills/`, `commands/`, or `agents/`? → spawn **si-skill-observer**
   - Has source code? → spawn **si-code-observer**
   - Always → spawn **si-workflow-observer** (analyzes current session)
   - Has multi-file project? → spawn **si-structure-observer**
3. Spawns relevant observers **in parallel** (each gets fresh context = peak quality)
4. Collects results and merges into unified `improvement-plan.md` with category tags
5. Prints summary

**`/si:apply`** — Apply approved improvements (enhanced from current)
1. Reads `improvement-plan.md`, groups items by category
2. User selects items via AskUserQuestion (multiSelect)
3. Routes each item to appropriate action:
   - **Skill items** → edit skill/command/agent files
   - **Code items** → edit source code with plan mode gate
   - **Workflow items** → update active-directives.md
   - **Structure items** → reorganize files/directories
4. Updates improvement-plan.md status

**`/si:status`** — Quick view of current SI state
1. Shows pending improvement count by category
2. Shows recent observation date
3. Shows applied vs pending vs rejected ratio

### Subagents (Specialized Observers)

Each observer gets **only** the context relevant to its domain — no wasted tokens.

**si-skill-observer** — Claude Code meta-quality
- Analyzes: skill files, command definitions, agent prompts, CLAUDE.md rules
- Looks for: unclear prompts, missing tool restrictions, inconsistent patterns, prompt engineering opportunities
- Tools: Read, Glob, Grep

**si-code-observer** — Source code quality
- Analyzes: source files touched in session + overall codebase patterns
- Looks for: anti-patterns, dead code, performance issues, missing validation, style drift
- Tools: Read, Glob, Grep, Bash (for linters/type-checkers)

**si-workflow-observer** — Process efficiency
- Analyzes: current conversation context (what ran, errors, retries, user corrections)
- Looks for: redundant steps, bottlenecks, tool misuse, error patterns
- Tools: Read, Glob, Grep

**si-structure-observer** — Project organization
- Analyzes: directory tree, naming conventions, module boundaries, config files
- Looks for: inconsistent naming, poor grouping, missing configs, documentation gaps
- Tools: Read, Glob, Grep, Bash (for tree/ls)

### Output Format

All observers write to a standardized format that the orchestrator merges:

```markdown
### [ID]. [Title]
**Category:** skill | code | workflow | structure
**Evidence:** [1-line data reference]
**Current:** [What happens now]
**Proposed:** [What should change]
**Impact:** [Expected effect]
**Status:** pending
```

### Reports Directory

Same location as current: `.claude/project/reports/`
- `improvement-plan.md` — unified, categorized (backward compatible)
- `reasoning.md` — merged reasoning from all observers
- `system-observations.md` — workflow + skill observations
- `data-patterns.md` — code + structure patterns

## Implementation Steps

### Step 1: Create observer subagents
Write 4 agent files in `~/.claude/agents/`:
- `si-skill-observer.md`
- `si-code-observer.md`
- `si-workflow-observer.md`
- `si-structure-observer.md`

### Step 2: Create SI commands
Write 3 command files in `~/.claude/commands/si/`:
- `observe.md` — orchestrator with parallel agent spawning
- `apply.md` — enhanced with category routing
- `status.md` — quick report viewer

### Step 3: Remove old files
- Delete `~/.claude/commands/observe.md`
- Delete `~/.claude/commands/apply.md`
- Delete `~/.claude/skills/observe.md`
- Delete `~/.claude/skills/apply.md`

### Step 4: Verify
- Run `/si:observe` in a test project
- Confirm observers spawn in parallel
- Confirm improvement-plan.md is correctly populated
- Run `/si:apply` and confirm category routing works

## Key Design Decisions

1. **`si:` colon namespace** (matches GSD pattern `gsd:*`) — groups commands visually
2. **No applier subagent** — apply flow is interactive (user selection + plan mode) which doesn't suit subagent isolation
3. **Observer-per-category** — each gets fresh context (~0% usage = peak quality) vs current monolithic approach (~50%+ context = degraded quality on later analysis)
4. **Backward compatible reports** — same `improvement-plan.md` format, just adds `**Category:**` field
5. **Smart spawning** — orchestrator only spawns observers relevant to the project (no wasted agents for projects without skills, etc.)
