---
name: apply
description: Apply human-approved improvements from observers, routed by category
disable-model-invocation: true
allowed-tools: Read, Glob, Grep, Bash, Write, Edit, AskUserQuestion, EnterPlanMode, ExitPlanMode
---

# /si:apply — Apply Improvements

Read improvement-plan.md, walk the user through each category conversationally, route approved items by category, then execute. Human review gate at every stage.

## Workflow

### 1. Read the plan

Read `.claude/project/reports/improvement-plan.md`. Collect all items with `**Status:** pending`.

If no pending items exist, report and exit:
```
No pending items in improvement-plan.md. Run /si:observe first.
```

Group items by category (`skill`, `code`, `workflow`, `structure`) and count each group.

### 2. Interactive selection

This step has three phases: orient, choose, and handle unselected.

#### Phase 1: Orient

Print a plain overview. No technical jargon. Use these category labels:

| Category | Label | Explanation |
|----------|-------|-------------|
| skill | Skill fixes | how your Claude commands and agents are set up |
| code | Code fixes | source code cleanup and bug prevention |
| workflow | Workflow | habits for smoother sessions |
| structure | Structure | file organization and tidiness |

Format:
```
I found [N] improvements from the last observation:

  [Label] ([count]) — [explanation]
  [Label] ([count]) — [explanation]
  ...

Let's walk through each group so you can pick what to apply.
```

Only list categories that have pending items.

#### Quick path: 1-2 total items

If there are only 1-2 pending items total, skip the category walk-through. Present a single `AskUserQuestion` with `multiSelect: true` containing all items. Note the category in each description (e.g., "This is a skill fix that...").

Then skip directly to handling unselected items.

#### Phase 2: Choose (category by category)

For each category that has pending items, present one `AskUserQuestion` with `multiSelect: true`:

- **header**: The category label (e.g., `Skill fixes`)
- **Up to 4 items per question.** If a category has more than 4, split into batches with numbered headers: `Skill fixes 1/2`, `Skill fixes 2/2`
- Each option:
  - **label**: Short action phrase, 1-5 words, no jargon (e.g., "Fix startup paths", "Remove dead reports")
  - **description**: 2-3 sentences in plain language following the Plain-Language Translation Rule below

#### Phase 3: Handle unselected items

After each category (or batch), if ANY items were NOT selected, ask a follow-up:

```
AskUserQuestion:
  question: "What should I do with the [N] items you skipped in [Category Label]?"
  header: "Skipped"
  options:
    - label: "Keep for later"
      description: "They'll stay pending for next time you run /si:apply"
    - label: "Reject them"
      description: "Removed permanently — they won't come back"
```

**Skip this follow-up if the user selected everything in that category.**

Mark rejected items immediately: set status to `**Status:** rejected (YYYY-MM-DD)`.

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
- Change `**Status:** pending` to `**Status:** applied (YYYY-MM-DD)`
- Add `**Outcome:** pending-review` to the item
- Move the item from `## Pending` to `## Applied`

For any items rejected during the unselected follow-up:
- Change status to `**Status:** rejected (YYYY-MM-DD)`
- Move from `## Pending` to `## Rejected`

Items not selected and not rejected stay as `pending`.

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

## Plain-Language Translation Rule

When presenting items in AskUserQuestion (Phase 2), translate the technical improvement-plan.md format into human-friendly language.

**Rules:**
- Use "you" and "your" — address the user directly
- First sentence: what's wrong and why should they care
- Second sentence: what this fix does
- Third (optional): what gets better
- **Never mention file paths, line numbers, or technical identifiers in the selection UI**
- No code formatting, no backticks, no raw evidence

Example translation:

```
BAD (raw from report):
  "observe.md Step 5 writes reasoning.md, system-observations.md, and
   data-patterns.md which are overwritten each run"

GOOD (translated):
  "The observe command creates extra report files that nobody reads.
   This removes them so observations run faster."
```

## Constraints

- **Never apply without user selection.** User picks items first.
- **Code changes require plan mode.** No direct source edits without user reviewing the plan.
- **Skill/workflow changes can be applied directly** — they're prompt text, not production code.
- **Preserve improvement-plan.md history.** Never delete entries — move between sections.
- **active-directives.md must stay concise.** It's read every session. No bloat.
- **Follow the Ripple Rule for structure changes.** Every moved file must have its imports updated.
- **Unselected items stay pending unless explicitly rejected.** The follow-up question determines fate.
- **4 items max per AskUserQuestion.** Split larger categories into numbered batches.
- **No technical jargon in selection UI.** All descriptions must follow the Plain-Language Translation Rule.
