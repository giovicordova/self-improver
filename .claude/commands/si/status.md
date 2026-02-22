---
name: status
description: View SI reports and pending improvement items
disable-model-invocation: true
allowed-tools: Read, Glob, Grep
---

# /si:status — Self-Improvement Status

Quick view of the current SI state for this project.

## Workflow

### 1. Check for reports directory

Check if `.claude/project/reports/` exists. If not:
```
No SI reports found. Run /si:observe to start.
```

### 2. Read improvement-plan.md

Read `.claude/project/reports/improvement-plan.md`. Count items by status and category:

- **Pending** — items awaiting review
- **Applied** — items that have been implemented
- **Rejected** — items explicitly rejected

Also count by category:
- **skill** — Claude meta-artifact improvements
- **code** — source code quality
- **workflow** — process and directive improvements
- **structure** — file organization
- **security** — vulnerability and security hardening

### 3. Check apply queue

Read `.claude/project/reports/apply-queue.md`. If it exists, parse the Progress line and batch statuses. Include in output:

```
Apply queue in progress:
  [completed]/[total] batches ([X] items applied, [Y] items remaining)
  Run /si:apply to resume
```

If the file doesn't exist, skip this section.

### 4. Check observation date

Read the most recent item in `.claude/project/reports/improvement-plan.md` for approximate last observation date.

### 5. Print status

```
SI Status
=========

Last observation: YYYY-MM-DD
Reports directory: .claude/project/reports/

Improvements by status:
  Pending:  [N]
  Applied:  [N]
  Rejected: [N]

Pending by category:
  skill:     [N] items
  code:      [N] items
  workflow:  [N] items
  structure: [N] items
  security:  [N] items

[If pending > 0:]
Next step: Review pending items, then run /si:apply

[If pending == 0 and applied > 0:]
All items processed. Run /si:observe for a fresh analysis.

[If no reports at all:]
No observations yet. Run /si:observe to start.
```

### 6. Check security audit report

If `.claude/project/reports/security-audit.md` exists, read it and include a security section:

```
Security Audit:
  Last audit: YYYY-MM-DD
  Critical: [N]  High: [N]  Medium: [N]  Low: [N]  Info: [N]
  [If Critical > 0:] ⚠ Critical findings require attention
  Run /si:security-audit for a fresh scan
```

If the file doesn't exist, skip this section.

### 7. Optionally show pending items

If there are pending items, list them briefly:

```
Pending items:
  [ID]. [Category] — [Title]
  [ID]. [Category] — [Title]
  ...
```

## Constraints

- **Read-only.** This command never modifies any files.
- **Keep output concise.** Status should be scannable in 5 seconds.
- **Show actionable next step.** Always tell the user what to do next.
