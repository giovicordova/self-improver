---
name: us
description: Rebuild state.md from actual codebase
disable-model-invocation: true
allowed-tools: Read, Glob, Grep, Bash, Write
---

# /si:us — Update State

Rebuild `.claude/project/state.md` from the actual filesystem. This gives Claude an accurate snapshot at session start.

## Workflow

### 1. Gather filesystem state

Use Glob and Bash to map what actually exists:

```bash
find . -type f -not -path '*/.git/*' -not -path '*/node_modules/*' -not -path '*/dist/*' -not -path '*/.next/*' -not -path '*/__pycache__/*' -not -path '*/.venv/*' -not -path '*/venv/*' -not -path '*/target/*' -not -path '*/build/*' -not -path '*/.cache/*' | head -200
```

### 2. Detect tech stack

Look for:
- `package.json`, `tsconfig.json` → Node/TypeScript
- `pyproject.toml`, `requirements.txt` → Python
- `Cargo.toml` → Rust
- `go.mod` → Go
- No source code → meta-project (commands/agents/skills only)

### 3. Check git state

```bash
git log --oneline -5 2>/dev/null || echo "Not a git repo"
git remote -v 2>/dev/null
git status --short 2>/dev/null
```

### 4. Read existing context

Read `.claude/CLAUDE.md` and `.claude/project/vision.md` (if they exist) to understand what the project considers important.

### 5. Write state.md

Write `.claude/project/state.md` with this structure:

```markdown
<state>
<updated>YYYY-MM-DD HH:MM</updated>

## What exists

<codebase>
[Brief description of what this project is and what tech it uses]

**Key directories/files:**
- [List the important parts, not every file]
</codebase>

<structure>
[ASCII tree of the project — focus on meaningful structure, skip obvious generated dirs]
</structure>

<tests>
[Test framework, how to run, rough coverage — or "No tests" if none]
</tests>

<design>
[Key architectural patterns and decisions]
</design>

<git>
[Branch, recent commits, remote, clean/dirty state]
</git>

## What's next

[Based on vision.md and current state — what are the priorities?]

</state>
```

### 6. Confirm

```
State updated: .claude/project/state.md
```

## Constraints

- **Describe what IS, not what should be.** state.md is a snapshot, not a wishlist.
- **Keep it scannable.** Claude reads this every session — brevity matters.
- **Don't invent.** If you can't determine something from the filesystem, leave it out.
