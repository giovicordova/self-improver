---
name: observe
description: Observe execution, analyze patterns, propose improvements via parallel specialized observers
disable-model-invocation: true
---

# /si:observe — Self-Improvement Orchestrator

Spawn specialized observer subagents in parallel to analyze this project. Each observer gets fresh context for peak-quality analysis.

## Workflow

### 1. Load project context

Read these files to understand the project:
- `.claude/CLAUDE.md` or `~/.claude/CLAUDE.md` — project identity, conventions
- `.claude/project/state.md` — codebase snapshot (if exists)
- `.claude/project/reports/improvement-plan.md` — previous proposals (if exists, get next ID number)
- `.claude/project/reports/active-directives.md` — current directives (if exists)

### 2. Detect relevant observers

Check what the project contains and decide which observers to spawn:

**si-skill-observer** — Spawn if ANY of these exist:
- `.claude/skills/` or `~/.claude/skills/`
- `.claude/commands/` or `~/.claude/commands/`
- `.claude/agents/` or `~/.claude/agents/`
- `CLAUDE.md` with rules/instructions

**si-code-observer** — Spawn if source code exists:
- `src/`, `lib/`, `app/`, `pages/`, or any directory with `.ts`, `.tsx`, `.js`, `.py`, `.rs`, `.go` files
- Has a `package.json`, `Cargo.toml`, `go.mod`, `pyproject.toml`, or similar

**si-workflow-observer** — Always spawn. It analyzes the current session's execution patterns from conversation context.

**si-structure-observer** — Spawn if the project has multiple source files:
- More than 5 source files across 2+ directories

### 3. Spawn observers in parallel

Use the Task tool to spawn each relevant observer as a subagent. Launch them **in parallel** — this is the whole point of the architecture (fresh context per observer = peak quality).

For each observer, provide this context in the prompt:
- Project path and name
- What CLAUDE.md says about conventions
- Previous improvement-plan.md items (so observers don't duplicate)
- For workflow observer: summarize key session events (what ran, errors, user corrections)

Example spawn pattern:
```
Task(subagent_type="si-skill-observer", prompt="Analyze Claude Code meta-artifacts for project at [path]. Conventions: [from CLAUDE.md]. Previous proposals: [IDs and titles]. Return findings in standardized format.")

Task(subagent_type="si-code-observer", prompt="Analyze source code quality for project at [path]. Stack: [from state.md/CLAUDE.md]. Conventions: [key rules]. Return findings in standardized format.")

Task(subagent_type="si-workflow-observer", prompt="Analyze session execution. Session summary: [what happened]. Errors: [list]. User corrections: [list]. Active directives: [current]. Return findings in standardized format.")

Task(subagent_type="si-structure-observer", prompt="Analyze project structure at [path]. Stack: [from state.md]. Framework: [name]. Return findings in standardized format.")
```

### 4. Collect and merge results

After all observers return:

1. Collect all findings from all observers
2. Assign sequential IDs continuing from the last ID in existing `improvement-plan.md`
3. Deduplicate — if two observers flag the same issue, keep the more detailed one
4. Sort by category: skill, code, workflow, structure

### 5. Write reports

Run `mkdir -p .claude/project/reports` via Bash.

**improvement-plan.md** — Append new proposals to `## Pending`. Never overwrite existing pending/approved items. Each item gets the `**Category:**` field:

```markdown
### [ID]. [Title]
**Category:** skill | code | workflow | structure
**Evidence:** [1-line data reference]
**Current:** [What happens now]
**Proposed:** [What should change]
**Impact:** [Expected effect]
**Status:** pending
```

If the file doesn't exist, create it with this structure:
```markdown
# Improvement Plan

## Pending

[items here]

## Applied

## Rejected
```

**reasoning.md** — Rewrite with merged reasoning from all observers:

```markdown
# Reasoning
*Session: YYYY-MM-DD*

## Evidence
- [Merged evidence from all observers]

## Analysis
- [Merged analysis]

## Confidence
- [Per-finding confidence levels]
```

**system-observations.md** — Rewrite with workflow + skill findings:

```markdown
# System Observations
*Updated: YYYY-MM-DD*

## Skill Quality
- [From si-skill-observer]

## Workflow Efficiency
- [From si-workflow-observer]

## Code Opportunities
- [From si-code-observer, summarized]
```

**data-patterns.md** — Rewrite with code + structure findings:

```markdown
# Data Patterns
*Updated: YYYY-MM-DD*

## Code Patterns
- [From si-code-observer]

## Structure Patterns
- [From si-structure-observer]
```

### 6. Print summary

```
Observation complete.
- Observers spawned: [list of observer names]
- [N] new improvement proposals added ([breakdown by category])
- Key findings: [1-2 sentence summary of most important findings]
- Run /si:status to review, then /si:apply to apply approved items
```

## Constraints

- **Spawn observers in parallel.** Sequential spawning defeats the purpose.
- **Never modify source code or working files.** Only write to `reports/`.
- **Never touch active-directives.md.** That's `/si:apply`'s job.
- **Deduplicate across observers.** Two observers may flag the same issue from different angles.
- **Preserve existing improvement-plan.md items.** Append-only for pending section.
- **Keep reports concise.** Written for Claude to read at session start, not for humans to study.
