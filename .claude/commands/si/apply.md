---
name: apply
description: Apply human-approved improvements from observers, routed by category
disable-model-invocation: true
allowed-tools: Read, Glob, Grep, Bash, Write, Edit, AskUserQuestion, EnterPlanMode, ExitPlanMode, Task
---

# /si:apply — Apply Improvements

Read improvement-plan.md, walk the user through selection, organize into batches, and execute with per-batch control. Supports resume across sessions via a queue file.

## Workflow

### 1. Check for existing queue

Read `.claude/project/reports/apply-queue.md`. If it exists:

1. Parse progress (completed/skipped/remaining counts)
2. Check that referenced items still exist as `pending` in improvement-plan.md — drop any that were applied or rejected externally
3. Present status and ask:

```
AskUserQuestion:
  question: "You have an apply queue in progress ([N] batches remaining, [M] items). What would you like to do?"
  header: "Queue"
  options:
    - label: "Resume"
      description: "Continue from where you left off"
    - label: "Start fresh"
      description: "Discard the queue and pick new items"
    - label: "View queue"
      description: "Show remaining batches before deciding"
```

- **Resume**: Skip to Step 6 with the next pending batch
- **Start fresh**: Delete `apply-queue.md`, fall through to Step 2
- **View queue**: Print remaining batches with item titles, then re-ask Resume/Start fresh

If no queue file exists, fall through to Step 2.

### 2. Read the plan

Read `.claude/project/reports/improvement-plan.md`. Collect all items with `**Status:** pending`.

If no pending items exist, report and exit:
```
No pending items in improvement-plan.md. Run /si:observe first.
```

Group items by category (`skill`, `code`, `workflow`, `structure`, `security`) and count each group.

### 3. Interactive selection

This step has three phases: orient, choose, and handle unselected.

#### Phase 1: Orient

Print a plain overview. No technical jargon. Use these category labels:

| Category | Label | Explanation |
|----------|-------|-------------|
| skill | Skill fixes | how your Claude commands and agents are set up |
| code | Code fixes | source code cleanup and bug prevention |
| workflow | Workflow | habits for smoother sessions |
| structure | Structure | file organization and tidiness |
| security | Security fixes | vulnerabilities and security hardening |

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

If zero items were selected across all categories, report and exit:
```
Nothing selected. Items stay pending for next time.
```

### 4. Generate batches

Organize selected items into batches:

1. **Group by category** — never mix categories in a batch (routing rules differ per category)
2. **Max 3 items per batch**
3. **Within a category**, group items that touch the same files or the same concern together
4. **Order categories**: skill → workflow → structure → code → security

Write the queue to `.claude/project/reports/apply-queue.md`:

```markdown
# Apply Queue
*Created: YYYY-MM-DD | Items: [N] selected | Batches: [M]*

## Progress
Completed: 0 | Skipped: 0 | Remaining: [M]

## Batches

### Batch 1 — [Category label] ([short theme]) [pending]
- [ ] #[ID] — [Item title]
- [ ] #[ID] — [Item title]

### Batch 2 — [Category label] ([short theme]) [pending]
- [ ] #[ID] — [Item title]
- [ ] #[ID] — [Item title]
- [ ] #[ID] — [Item title]

...
```

### 5. Choose execution mode

```
AskUserQuestion:
  question: "How do you want to apply these [M] batches?"
  header: "Mode"
  options:
    - label: "One by one"
      description: "I'll show each batch and you decide — execute, skip, or stop for now"
    - label: "Run all"
      description: "I'll work through every batch automatically using fresh context per batch"
    - label: "Save for later"
      description: "Queue is saved — run /si:apply anytime to resume"
```

- **Save for later**: Print queue location and exit
- **One by one**: Proceed to Step 6a
- **Run all**: Proceed to Step 6b

### 6a. Interactive execution

For each batch with status `[pending]`:

```
AskUserQuestion:
  question: "Batch [N]/[M] — [Category]: [item titles]. Execute this batch?"
  header: "Batch [N]"
  options:
    - label: "Execute"
      description: "[count] items: [brief list of what changes]"
    - label: "Skip"
      description: "Move to the next batch — these items stay pending"
    - label: "Done for now"
      description: "Save progress and exit — resume with /si:apply"
```

**Execute**: Apply items using category routing (Step 7), mark batch `[completed]` in queue, update improvement-plan.md.

**Skip**: Mark batch `[skipped]` in queue. Items remain `pending` in improvement-plan.md. Move to next batch.

**Done for now**: Update queue progress counts, save, print summary of what was done so far, exit.

After all batches processed, proceed to Step 8.

### 6b. Auto execution

For each batch with status `[pending]`:

1. Build a subagent prompt containing:
   - The full text of each item in the batch (from improvement-plan.md)
   - The category routing rules for that batch's category (from Step 7)
   - Target file paths extracted from Evidence/Proposed fields
   - Instruction: "Apply these improvements. For each item, update its status in `.claude/project/reports/improvement-plan.md` from `pending` to `applied (YYYY-MM-DD)` and add `**Outcome:** pending-review`. Move applied items from `## Pending` to `## Applied`."
   - The conventions from CLAUDE.md that apply

2. Spawn via `Task(subagent_type="general-purpose")` with a descriptive name like `"apply-batch-1-skill"`

3. Wait for return. Check improvement-plan.md to verify items were marked applied.

4. Update queue: mark batch `[completed]` or `[failed]` based on verification.

5. Print one-line progress: `Batch [N]/[M] done — [item titles] applied`

6. If the subagent fails or items aren't marked applied: mark batch `[failed]` in queue, continue to next batch.

After all batches, list any failed batches and proceed to Step 8.

### 7. Category routing

Each item routes to the appropriate action based on its category.

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
- For items that require rebuilding state.md (#47 and similar): execute the work directly — read actual codebase state and rewrite state.md. Do NOT invoke `/si:us` as a subcommand.

**Security items** (`**Category:** security`):
- Route by the nature of the fix:
  - Source code fixes (injection, auth, XSS) → same as code items (plan mode required)
  - Configuration fixes (headers, CORS, Docker) → same as structure items (show changes, apply directly)
  - Dependency updates → run appropriate package manager command, verify lockfile changed, run tests if available. Show before/after versions before executing.
  - Process changes (secret rotation, dependency updates) → same as workflow items (update active-directives.md)
- **Secret rotation detection:** When a security finding's title contains "hardcoded", "exposed credential", or "leaked secret", print a warning:
  "This item requires rotating a credential. You must do this manually:
   1. [specific rotation steps]
   2. Update the reference in [file]
   3. Verify the old credential is revoked"
  Then mark the item as `applied` with `**Outcome:** manual-action-required`

**For all categories — after applying each item:**
- Change `**Status:** pending` to `**Status:** applied (YYYY-MM-DD)`
- Add `**Outcome:** pending-review` to the item (unless manual-action-required)
- Move the item from `## Pending` to `## Applied`

**For code items (after plan approval):**
- Apply edits using Edit tool
- Run available checks (type-check, lint, test) if the project has them
- Fix any issues before moving on

### 8. Cleanup and summary

If all batches are completed or skipped (none remaining):
- Delete `apply-queue.md`
- Print full summary

If some batches remain (failed or unprocessed):
- Keep `apply-queue.md` with updated progress
- Note: "Run /si:apply to resume remaining batches"

Summary format:
```
Applied [N] improvements across [B] batches:
- [category] [Title]: [1-line what changed]
- [category] [Title]: [1-line what changed]
...

[If any skipped:]
Skipped [S] batches ([X] items — still pending)

[If any failed:]
Failed [F] batches — run /si:apply to retry:
- Batch [N]: [item titles]

Active directives updated: [yes/no]
Code changes applied: [N]
Skill files modified: [N]
Structure changes: [N]

[If repo files modified:]
Sync to global: cp .claude/agents/si-*.md ~/.claude/agents/ && cp .claude/commands/si/*.md ~/.claude/commands/si/
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
- **Never mix categories in a batch.** Routing rules differ per category.
- **Max 3 items per batch.** Keeps each execution focused and context-light.
- **Queue file is ephemeral.** Delete it when all batches are done. It's working state, not history.
- **New observations don't modify the queue.** If `/si:observe` runs mid-queue, complete the queue first, then re-select from new items.
