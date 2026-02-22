---
name: apply
description: Apply human-approved improvements from observers, routed by category
disable-model-invocation: true
---

# /si:apply — Apply Improvements

Read improvement-plan.md, let the user select items, route each by category, then execute on approval. Human review gate at every stage.

## Workflow

### 1. Read the plan

Read `.claude/project/reports/improvement-plan.md`. Collect all items with `**Status:** pending` or `**Status:** approved`.

If no pending or approved items exist, report and exit:
```
No pending or approved items in improvement-plan.md. Run /si:observe first.
```

### 2. Present items for selection

Show all pending/approved items grouped by category. Use AskUserQuestion with multiSelect to let the user pick which items to apply now.

Group by category in the options:
- **[skill]** "3. Fix vague role in si-code-observer" — Rewrite role section for clarity
- **[code]** "5. Remove dead export in utils.ts" — Delete unused `formatDate` export
- **[workflow]** "7. Add directive for reading CLAUDE.md first" — Prevent repeated context misses
- **[structure]** "9. Move orphaned helper to utils/" — Relocate `src/helpers/format.ts`

Mark selected pending items as `approved` in improvement-plan.md immediately.

### 3. Route by category

Each selected item routes to the appropriate action:

**Skill items** (`**Category:** skill`):
- Direct edits to skill/command/agent files in `.claude/` or `~/.claude/`
- Read the target file, apply the proposed change
- These are typically prompt edits — no plan mode needed unless the change is complex

**Code items** (`**Category:** code`):
- Source code changes require plan mode
- Call EnterPlanMode
- Write a concrete plan for each code change (file path, what changes, why)
- Call ExitPlanMode for user review
- After approval, apply edits

**Workflow items** (`**Category:** workflow`):
- Update `.claude/project/reports/active-directives.md`
- Create the file if it doesn't exist
- Keep directives concise and actionable (1-2 sentences each)
- Format:
```markdown
# Active Directives
*Updated: YYYY-MM-DD*

[Concise, actionable instructions derived from approved improvements]
```

**Structure items** (`**Category:** structure`):
- File moves, renames, directory reorganization
- Show the user what will move/rename before executing
- Update any imports that reference moved files (Ripple Rule)
- Use Bash for `mv` operations, Edit for import updates

### 4. Execute changes

Apply each selected item according to its category routing above.

For code items (after plan approval):
- Apply edits using Edit tool
- Run available checks (type-check, lint, test) if the project has them
- Fix any issues before moving on

For skill items:
- Read the target file
- Apply the specific change proposed
- Verify the file is still well-formed

For workflow items:
- Read current active-directives.md (or create new)
- Add/modify directives
- Keep the file short — it's read every session

For structure items:
- Execute file moves
- Update all imports referencing moved files
- Verify no broken references

### 5. Update improvement-plan.md

For each applied item:
- Change `**Status:** approved` to `**Status:** applied (YYYY-MM-DD)`
- Move the item from `## Pending` to `## Applied`

For any items the user explicitly rejected during selection:
- Change status to `**Status:** rejected (YYYY-MM-DD)`
- Move from `## Pending` to `## Rejected`

Items not selected are left as `pending` — not rejected.

### 6. Print summary

```
Applied [N] improvements:
- [skill] [Title]: [1-line what changed]
- [code] [Title]: [1-line what changed]
- [workflow] [Title]: [1-line what changed]
- [structure] [Title]: [1-line what changed]

Active directives updated: [yes/no]
Code changes applied: [N]
Skill files modified: [N]
Structure changes: [N]
```

## Constraints

- **Never apply without user selection.** User picks items first.
- **Code changes require plan mode.** No direct source edits without user reviewing the plan.
- **Skill/workflow changes can be applied directly** — they're prompt text, not production code.
- **Preserve improvement-plan.md history.** Never delete entries — move between sections.
- **active-directives.md must stay concise.** It's read every session. No bloat.
- **Follow the Ripple Rule for structure changes.** Every moved file must have its imports updated.
- **Unselected items stay pending.** Only explicitly rejected items get moved to Rejected.
