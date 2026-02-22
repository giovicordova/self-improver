---
name: observe
description: Observe execution, analyze patterns, propose improvements via parallel specialized observers
disable-model-invocation: true
allowed-tools: Read, Glob, Grep, Bash, Task, Write
---

# /si:observe — Self-Improvement Orchestrator

Spawn specialized observer subagents in parallel to analyze this project. Each observer gets fresh context for peak-quality analysis.

## Workflow

### 1. Load project context

First, ensure the reports directory exists:
```bash
mkdir -p .claude/project/reports
```

Then read these files to understand the project:
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

**si-structure-observer** — Spawn if the project has structural complexity:
- More than 5 source files across 2+ directories
- OR has a `.claude/` directory with agents and commands (meta-projects like SI itself)
- OR has 5+ files of any type across 2+ directories

### 3. Spawn observers in parallel

Use the Task tool to spawn each relevant observer as a subagent. Launch them **in parallel** — this is the whole point of the architecture (fresh context per observer = peak quality).

For each observer, provide this context in the prompt:
- Project path and name
- What CLAUDE.md says about conventions
- Previous improvement-plan.md items (so observers don't duplicate)
- For workflow observer: provide detailed session events — list every error, user correction, retry loop, and commands run with outcomes. If no significant session events occurred, note that explicitly so the observer can use its fallback analysis mode.
- For workflow observer: also include any recently-applied items (from improvement-plan.md ## Applied section) so the observer can assess whether they had the intended effect.

Example spawn pattern:

Use the Task tool to spawn all relevant observers **simultaneously in a single message**. Each Task call needs:
- `subagent_type` matching the agent name (e.g., `si-skill-observer`)
- A prompt containing: project path, CLAUDE.md conventions, previous proposal IDs/titles (to avoid duplicates), and for workflow observer: session events summary (errors, user corrections, retries)

### 4. Collect and merge results

After all observers return:

1. Collect all findings from all observers
2. Assign sequential IDs continuing from the last ID in existing `improvement-plan.md`
3. Deduplicate — if two proposals reference the same file AND propose the same type of change, keep the more specific one. If they describe different symptoms of the same root cause, keep both but add a cross-reference: "Related: #[other-ID]"
4. Sort by category: skill, code, workflow, structure

### 5. Write reports

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

When items move to `## Applied`, add: `**Outcome:** pending-review`
Valid outcome values: `pending-review` | `effective` | `ineffective` | `reverted`

Only write `improvement-plan.md`. Do not create supplementary report files — the improvement plan is the single source of truth.

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
