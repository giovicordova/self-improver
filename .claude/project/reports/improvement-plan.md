# Improvement Plan

## Pending

## Applied

### 19. Orphaned supplementary report files contradict applied improvement #6
**Category:** structure
**Evidence:** `reasoning.md`, `system-observations.md`, `data-patterns.md` still exist in `.claude/project/reports/`. Applied item #6 removed them from the observe workflow, but the existing files were never deleted. They appear as untracked in `git status` and contain stale data from the first observation.
**Current:** Three stale files contradict the applied decision. If any future observer or session reads them, it gets outdated information.
**Proposed:** Delete the three files. Add a directive to active-directives.md: "When an improvement removes a file or output, delete existing instances as part of the apply step."
**Impact:** Completes cleanup started by item #6. Removes git status noise and stale data risk.
**Status:** applied (2026-02-22)
**Outcome:** pending-review

### 20. Phantom "approved" lifecycle state never produced
**Category:** skill
**Evidence:** `apply.md` line 16 collects items with `Status: pending or approved`. `status.md` line 32 lists "Approved" as a status. But nothing in the workflow ever sets an item to `approved` — items go directly from `pending` to `applied` or `rejected`.
**Current:** Dead lifecycle state adds confusion. `/si:status` always shows `Approved: 0` because nothing generates that state.
**Proposed:** Remove "approved" as a recognized status. Simplify apply.md to only collect `pending` items and status.md to not count approved. Lifecycle becomes: `pending → applied | rejected`.
**Impact:** Eliminates a phantom state that makes the system harder to understand.
**Status:** applied (2026-02-22)
**Outcome:** pending-review

### 21. apply.md doesn't write the Outcome field when moving items to Applied
**Category:** skill
**Evidence:** `observe.md` line 101 documents `**Outcome:** pending-review` for applied items. But `apply.md` step 5 (which actually moves items) has no instruction to add the Outcome field. Related: #22.
**Current:** The Outcome field from improvement #14 is defined in observe.md but never written by the command that applies items. The feedback loop is structurally broken.
**Proposed:** Add to apply.md step 5: "Add `**Outcome:** pending-review` to each applied item."
**Impact:** Without this, the Outcome field is orphaned. The feedback loop (improvement #14) remains non-functional.
**Status:** applied (2026-02-22)
**Outcome:** pending-review

### 22. Outcome field has no evaluation criteria — feedback loop is write-only
**Category:** workflow
**Evidence:** All 18 applied items have `Outcome: pending-review`. No component defines how to evaluate outcomes or transition them to `effective`/`ineffective`/`reverted`. The workflow observer's fallback mode mentions applied items needing follow-up but provides no rubric. Related: #21.
**Current:** The Outcome field is written but never transitions. The feedback loop exists structurally but has no evaluation logic — the #1 near-term goal in vision.md.
**Proposed:** Add outcome evaluation guidance to `si-workflow-observer.md`: "When assessing recently-applied items: (skill) does the modified file match the proposed change and is it internally consistent? (workflow) is the directive being followed? (structure) did the reorganization hold? Mark `effective` if change is in place with no new problems, `ineffective` if problem persists, `reverted` if change was undone."
**Impact:** Closes the feedback loop. Without evaluation criteria, Outcome stays `pending-review` forever.
**Status:** applied (2026-02-22)
**Outcome:** pending-review

### 23. .gitignore has no coverage for ephemeral report files
**Category:** structure
**Evidence:** `.gitignore` covers `.claude/plans/` but nothing in `.claude/project/reports/`. Only `improvement-plan.md` and `active-directives.md` have persistent value. Three untracked files (finding #19) confirm this gap. As SI evolves with new observers, each output would need manual .gitignore updates.
**Current:** No protection against accidental commits of generated report files.
**Proposed:** Add broad pattern with exceptions to `.gitignore`:
```
.claude/project/reports/*.md
!.claude/project/reports/improvement-plan.md
!.claude/project/reports/active-directives.md
```
**Impact:** Durable policy that scales with future observer additions.
**Status:** applied (2026-02-22)
**Outcome:** pending-review

### 24. observe.md skill-observer spawn checks global paths despite scoping guidance
**Category:** skill
**Evidence:** `observe.md` lines 31-35 check `~/.claude/skills/`, `~/.claude/commands/`, `~/.claude/agents/` to decide whether to spawn the skill observer. But improvement #7 added scoping guidance telling the observer to focus on project `.claude/` only. The orchestrator uses global directories to spawn the observer, then tells it not to look at them.
**Current:** Contradiction: spawn decision based on global state, but observer told to ignore global state.
**Proposed:** Remove global paths from spawn detection criteria. Spawn based on project-level `.claude/skills/`, `.claude/commands/`, `.claude/agents/`, or `CLAUDE.md` with rules. Global analysis should be an explicit opt-in flag.
**Impact:** Aligns spawn logic with scoping guidance from improvement #7.
**Status:** applied (2026-02-22)
**Outcome:** pending-review

### 25. structure-observer Bash constraints are implicit unlike code-observer
**Category:** skill
**Evidence:** `si-code-observer.md` has explicit Bash safety constraints from improvement #8. `si-structure-observer.md` also has Bash access but only says "Do not modify anything" — no explicit command-level constraints like the code observer.
**Current:** Both observers have Bash access but only one has explicit safety boundaries.
**Proposed:** Add to si-structure-observer.md constraints: "Only run read-only inspection commands with Bash: `find`, `ls`, `wc`, `tree`, `du`. Do not run any command that writes, moves, deletes, installs, or modifies files or system state."
**Impact:** Closes the same safety gap that improvement #8 addressed for the code observer.
**Status:** applied (2026-02-22)
**Outcome:** pending-review

### 26. observe.md Steps 2 and 3 have overlapping spawn instructions
**Category:** skill
**Evidence:** Step 2 describes detection criteria. Step 3 repeats spawning guidance with an "Example spawn pattern" section. The `subagent_type` guidance and context-to-include appear in two places that could diverge.
**Current:** Duplication between detection logic and spawn instructions creates risk of divergence during future edits.
**Proposed:** Consolidate Steps 2 and 3 into a single step. Remove the "Example spawn pattern" sub-section and fold its unique content into the main instruction.
**Impact:** Reduces ambiguity and eliminates divergence risk.
**Status:** applied (2026-02-22)
**Outcome:** pending-review

### 27. Tracked plan file defeats .gitignore coverage
**Category:** structure
**Evidence:** `.gitignore` has `.claude/plans/` but `deep-knitting-clarke.md` was committed before .gitignore was created. Git ignores only affect untracked files, so the plan file is permanently tracked despite the rule.
**Current:** False sense of .gitignore coverage. Implementation scaffolding persists in the repo.
**Proposed:** Run `git rm --cached .claude/plans/deep-knitting-clarke.md` to untrack while keeping on disk. Remove the `plans/` entry from state.md's structure tree.
**Impact:** Makes .gitignore effective for plans directory. Removes scaffolding from tracked repo.
**Status:** applied (2026-02-22)
**Outcome:** pending-review

### 28. us.md doesn't consult .gitignore — gitignored paths appear in structure tree
**Category:** skill
**Evidence:** `us.md` step 1 uses `find` with hardcoded exclusions but doesn't check `.gitignore`. This caused the plans directory (gitignored) to appear in state.md's structure tree.
**Current:** Any gitignored path not in the hardcoded list appears in the structure output.
**Proposed:** In us.md, prefer `git ls-files` when in a git repo (tracked + untracked-but-not-ignored), fall back to `find` for non-git repos. Add constraint: "Do not include gitignored paths in the structure tree."
**Impact:** State.md structure tree shows only meaningful project files. Prevents the issue found in #27 systemically.
**Status:** applied (2026-02-22)
**Outcome:** pending-review

### 29. Observers don't read active-directives.md (except workflow)
**Category:** skill
**Evidence:** Only `si-workflow-observer.md` references active-directives.md. The skill and structure observers don't read it, despite directives potentially containing conventions relevant to their domains (e.g., naming rules, namespace conventions).
**Current:** Active directives are a shared knowledge base consumed by only one of four observers.
**Proposed:** Add active-directives.md to the discovery step of skill and structure observers: "Read `.claude/project/reports/active-directives.md` if it exists — check whether artifacts/structure follow active directives."
**Impact:** Strengthens feedback loop — directives from one cycle get checked by the next across all domains.
**Status:** applied (2026-02-22)
**Outcome:** pending-review

### 30. us.md doesn't fall back to global CLAUDE.md like observe.md does
**Category:** skill
**Evidence:** `observe.md` reads `.claude/CLAUDE.md or ~/.claude/CLAUDE.md`. `us.md` only checks `.claude/CLAUDE.md`. Projects relying on global CLAUDE.md get incomplete state.md context.
**Current:** Inconsistency between the two commands' context-gathering behavior.
**Proposed:** Add the same fallback in us.md: "Read `.claude/CLAUDE.md` (or `~/.claude/CLAUDE.md` if no project-level one exists)."
**Impact:** Ensures state.md reflects conventions even when defined globally.
**Status:** applied (2026-02-22)
**Outcome:** pending-review

### 31. No sync-check between repo and global copies
**Category:** workflow
**Evidence:** CLAUDE.md says "Changes here should be synced back to `~/.claude/`" but there's no mechanism to detect or warn when copies drift. After applying improvements, the user must manually copy files.
**Current:** Sync is entirely manual with no reminder or verification.
**Proposed:** Add a step to `/si:apply`'s summary: "Files modified in this repo. Sync to global: `cp .claude/agents/si-*.md ~/.claude/agents/ && cp .claude/commands/si/*.md ~/.claude/commands/si/`". Add a directive about syncing after changes.
**Impact:** Prevents drift between canonical source and deployed copies.
**Status:** applied (2026-02-22)
**Outcome:** pending-review

### 32. observe.md structure observer detection has redundant criteria
**Category:** skill
**Evidence:** Detection lists three OR conditions: (1) "5+ source files across 2+ dirs", (2) ".claude/ with agents/commands", (3) "5+ files of any type across 2+ dirs". Condition 3 is a superset of condition 1 — condition 1 is redundant.
**Current:** Three conditions where two would suffice. Minor clarity issue.
**Proposed:** Remove condition 1. Keep "5+ files of any type across 2+ directories" and "has .claude/ with agents and commands."
**Impact:** Minor clarity improvement. Reduces ambiguity in edge cases.
**Status:** applied (2026-02-22)
**Outcome:** pending-review

### 33. state.md git section says "2 commits" — actual count is 4
**Category:** structure
**Evidence:** state.md line 68: "2 commits + uncommitted changes". Actual: 4 commits (`e49f1d2`, `ab61c3f`, `40dbb47`, `90e9998`), clean working tree.
**Current:** state.md was rebuilt as item #15 but before two subsequent commits. Claude reads stale git info at session start.
**Proposed:** Run `/si:us` to rebuild state.md. This is exactly what the command was built for.
**Impact:** Trivial. Keeps session-start context accurate.
**Status:** applied (2026-02-22)
**Outcome:** pending-review

### 1. Missing `/us` command file in repository
**Category:** skill
**Evidence:** state.md line 13, CLAUDE.md line 25, README.md line 41 all reference `commands/us.md`. File does not exist anywhere in the repo. README installation instructs `cp .claude/commands/us.md ~/.claude/commands/` — this will fail.
**Current:** The repo claims to be the canonical source for SI but is missing a documented command. Users following installation instructions get an error.
**Proposed:** Create `.claude/commands/si/us.md` (under si/ namespace — see #16) with state-rebuild logic. Update all references to use `/si:us` instead of `/us`.
**Impact:** Fixes broken installation, completes the canonical source claim, and maintains namespace consistency.
**Status:** applied (2026-02-22)
**Outcome:** pending-review

### 2. CLAUDE.md bootstrap paths are relative without anchor
**Category:** skill
**Evidence:** CLAUDE.md lines 6-8: `Read project/state.md` and `If project/vision.md exists`. These are relative paths but the anchor is ambiguous — could be repo root or `.claude/` directory.
**Current:** Claude must infer that `project/state.md` means `.claude/project/state.md`. On first read, this may cause a failed file lookup before self-correcting.
**Proposed:** Change to explicit paths: `Read .claude/project/state.md at the start of every session. If .claude/project/vision.md exists, read it too.`
**Impact:** Eliminates a failed file read at session start. Every session hits this.
**Status:** applied (2026-02-22)
**Outcome:** pending-review

### 3. CLAUDE.md bootstrap doesn't include active-directives.md
**Category:** skill
**Evidence:** CLAUDE.md line 41 says `active-directives.md stays concise (read every session)`. But the Bootstrap section (lines 6-8) only mentions state.md and vision.md — no mention of active-directives.md.
**Current:** The convention says directives should be read every session, but the bootstrap rule doesn't include them. Directives get written but never automatically consumed at session start.
**Proposed:** Add to Bootstrap: `If .claude/project/reports/active-directives.md exists, read it too — these are learned directives from previous observations.`
**Impact:** Closes the feedback loop. Without this, workflow improvements applied via `/si:apply` have no effect on future sessions.
**Status:** applied (2026-02-22)
**Outcome:** pending-review

### 4. observe.md and apply.md missing allowed-tools frontmatter
**Category:** skill
**Evidence:** status.md has `allowed-tools: Read, Glob, Grep` in frontmatter. observe.md and apply.md have `disable-model-invocation: true` but no `allowed-tools`.
**Current:** Inconsistent frontmatter pattern across the three SI commands. status.md correctly restricts tools; the others leave it implicit.
**Proposed:** Add `allowed-tools` to observe.md (`Read, Glob, Grep, Bash, Task, Write`) and apply.md (`Read, Glob, Grep, Bash, Write, Edit, AskUserQuestion, EnterPlanMode, ExitPlanMode`).
**Impact:** Consistency across commands. Makes tool requirements explicit and documentable.
**Status:** applied (2026-02-22)
**Outcome:** pending-review

### 5. observe.md Task spawn examples use pseudo-code syntax
**Category:** skill
**Evidence:** observe.md lines 52-58 show `Task(subagent_type="si-skill-observer", prompt="...")` — a Python function-call syntax that doesn't match actual Task tool invocation.
**Current:** The example uses pseudo-code that requires Claude to interpret intent and translate to actual tool calls. This works but adds an unnecessary interpretation layer.
**Proposed:** Replace with natural language: "Use the Task tool to spawn `si-skill-observer`, `si-code-observer`, `si-workflow-observer`, and `si-structure-observer` simultaneously. Each Task call should include subagent_type matching the agent name and a prompt with the context described above."
**Impact:** Reduces ambiguity between command prompt and actual tool invocation.
**Status:** applied (2026-02-22)
**Outcome:** pending-review

### 6. observe.md writes 4 report files — 3 are unnecessary
**Category:** skill
**Evidence:** observe.md Step 5 writes `improvement-plan.md`, `reasoning.md`, `system-observations.md`, and `data-patterns.md`. The latter 3 are overwritten each run, never read at session start, and not consumed by any automated process.
**Current:** 3 supplementary report files add write overhead without being part of any feedback loop. Only `improvement-plan.md` has persistent value.
**Proposed:** Remove `reasoning.md`, `system-observations.md`, and `data-patterns.md` from the observe workflow. If reasoning/context is valuable, add a `## Notes` section to each item in `improvement-plan.md`.
**Impact:** Simplifies observe from 4 file writes to 1. Follows the project's own simplicity principle.
**Status:** applied (2026-02-22)
**Outcome:** pending-review

### 7. Skill observer scope confusion — global vs project paths
**Category:** skill
**Evidence:** si-skill-observer.md lines 22-25 instruct scanning both `~/.claude/` and `.claude/`. When running on a non-SI project, findings about global GSD agents, frontend agents, etc. would mix with project findings.
**Current:** No instruction to distinguish global vs project scope. Observer may produce noise from unrelated global configs.
**Proposed:** Add scoping guidance: "Focus on the project's `.claude/` directory. Only scan `~/.claude/` if the orchestrator explicitly requests global analysis or if the project has no `.claude/` directory. Always note whether findings are project-level or global-level."
**Impact:** Prevents noise from global config analysis when observing specific projects.
**Status:** applied (2026-02-22)
**Outcome:** pending-review

### 8. Code observer Bash constraints are incomplete
**Category:** skill
**Evidence:** si-code-observer.md line 75: `No rm, no git, no file modifications.` Misses `mv`, `cp` (overwrite), `chmod`, `npm install`, `pip install`, `curl | bash`, etc.
**Current:** Constraint lists specific disallowed commands but the Bash tool gives full shell access. Gap between intent (read-only) and specification.
**Proposed:** Replace with: "Only run read-only analysis commands: linters (`eslint`, `tsc --noEmit`, `ruff check`), type-checkers, and `find`/`ls`/`wc` for inspection. Do not run any command that writes, moves, deletes, installs, or modifies files or system state."
**Impact:** Closes the safety gap. Prevents edge cases like running `npm install` to "check dependencies."
**Status:** applied (2026-02-22)
**Outcome:** pending-review

### 9. Structure observer `find` excludes are incomplete
**Category:** skill
**Evidence:** si-structure-observer.md line 24 excludes `node_modules`, `.git`, `dist`, `.next`, `__pycache__` but misses `.venv`, `venv`, `target` (Rust), `build`, `.cache`, `coverage`, `vendor` (Go/Ruby).
**Current:** On Python/Rust/Go projects, the 200-line `head` budget could be consumed by build artifacts and virtual environments.
**Proposed:** Add common build/dependency directories: `.venv`, `venv`, `target`, `build`, `.cache`, `coverage`, `vendor`.
**Impact:** Prevents structure observer from wasting its file listing on irrelevant directories.
**Status:** applied (2026-02-22)
**Outcome:** pending-review

### 10. Cross-observer deduplication criteria are vague
**Category:** skill
**Evidence:** observe.md line 67: `if two observers flag the same issue, keep the more detailed one`. No definition of "same issue" — same file? Same change? Same root cause?
**Current:** The orchestrator has no concrete criteria for recognizing duplicates vs related-but-different issues.
**Proposed:** Replace with: "If two proposals reference the same file AND propose the same type of change, keep the more specific one. If they describe different symptoms of the same root cause, keep both but add a cross-reference: 'Related: #[other-ID]'."
**Impact:** Prevents false deduplication and missed duplicates. Improves improvement-plan.md quality.
**Status:** applied (2026-02-22)
**Outcome:** pending-review

### 11. Workflow observer depends entirely on orchestrator summary quality
**Category:** workflow
**Evidence:** si-workflow-observer.md line 9 says it analyzes "what ran, what failed, what was inefficient." But subagents don't inherit conversation history — they only see what the orchestrator passes in the prompt.
**Current:** The observer's core capability depends on manual summary quality. Thin summaries produce empty/speculative findings. The agent prompt doesn't handle the sparse-context case.
**Proposed:** Two changes: (1) In the agent, add fallback: "If session context is sparse, focus on analyzing existing report files for patterns across observations." (2) In observe.md, specify minimum context: "List every error, every user correction, every retry loop, commands run with outcomes."
**Impact:** Makes workflow observer resilient to thin context instead of producing noise.
**Status:** applied (2026-02-22)
**Outcome:** pending-review

### 12. First-run should create reports directory before reading
**Category:** workflow
**Evidence:** observe.md Step 1 reads from reports/ directory. Step 5 runs `mkdir -p`. On first run, Step 1 produces file-not-found errors that are noise.
**Current:** Directory creation happens after read attempts, generating harmless but noisy errors on first run.
**Proposed:** Move `mkdir -p .claude/project/reports` to the beginning of Step 1.
**Impact:** Cleaner first-run experience. Eliminates unnecessary error messages.
**Status:** applied (2026-02-22)
**Outcome:** pending-review

### 13. Observer detection logic is hardcoded for source-code projects
**Category:** workflow
**Evidence:** observe.md lines 33-39: code observer triggers on `src/`, `lib/`, etc. Structure observer triggers on "more than 5 source files." Markdown-only projects like SI itself get only skill + workflow observers.
**Current:** Meta-projects, config-only projects, and documentation-heavy projects get incomplete analysis. SI can't fully dogfood on itself.
**Proposed:** Add fallback: "Also spawn structure observer if the project has 5+ files across 2+ directories (any type, not just source code), or has a `.claude/` directory with agents and commands."
**Impact:** Enables SI to meaningfully analyze itself and similar meta-projects.
**Status:** applied (2026-02-22)
**Outcome:** pending-review

### 14. No mechanism to track whether applied improvements helped
**Category:** workflow
**Evidence:** vision.md line 24 and state.md line 72 both identify this gap. Applied items get a date stamp and move to `## Applied` — end of lifecycle. No follow-up.
**Current:** SI has no learning signal about its own effectiveness. The central value proposition (evidence-based improvement) lacks a feedback loop.
**Proposed:** Add `**Outcome:** pending-review | effective | ineffective | reverted` field to applied items. In observe.md, include recently-applied items in workflow observer context so it can assess effectiveness.
**Impact:** First step toward the feedback loop described in vision.md.
**Status:** applied (2026-02-22)
**Outcome:** pending-review

### 15. state.md contains stale information
**Category:** structure
**Evidence:** state.md says "Not a git repository. Needs git init" but `git status` shows branch `main` with 2 commits and a remote. state.md also lists `commands/us.md` in the file tree but the file doesn't exist.
**Current:** state.md is read at session start per bootstrap instructions. Stale information misleads Claude into thinking foundational setup is still needed.
**Proposed:** Rebuild state.md from actual filesystem. Update git section, remove phantom `us.md` from tree, update "What's next" priorities.
**Impact:** Every session starts with accurate project understanding instead of outdated assumptions.
**Status:** applied (2026-02-22)
**Outcome:** pending-review

### 16. `/us` command should follow `si:*` namespace convention
**Category:** structure
**Evidence:** plans/deep-knitting-clarke.md line 137: "si: colon namespace (matches GSD pattern) — groups commands visually." All SI commands are at `commands/si/*.md` except `/us` which would be at `commands/us.md` (root level).
**Current:** When `/us` is created, placing it outside `si/` breaks the stated design principle that all SI commands live under one namespace.
**Proposed:** Create as `.claude/commands/si/us.md` → `/si:us`. Update all references.
**Impact:** Namespace consistency. All SI functionality under one prefix.
**Status:** applied (2026-02-22)
**Outcome:** pending-review

### 17. No .gitignore file
**Category:** structure
**Evidence:** No `.gitignore` exists. The repo is pushed to GitHub. Reports directory will contain generated files that get rewritten each observation run.
**Current:** No policy for which files should be tracked vs ignored. Running `/si:observe` on this repo (dogfooding) would commit generated report files.
**Proposed:** Create `.gitignore` with standard excludes. Track `improvement-plan.md` and `active-directives.md` (persistent value).
**Impact:** Prevents generated noise in git history while preserving improvement lifecycle data.
**Status:** applied (2026-02-22)
**Outcome:** pending-review

### 18. README installation instructions are incomplete
**Category:** structure
**Evidence:** README lines 36-41 copy agents and commands but don't explain that `.claude/CLAUDE.md` and `.claude/project/` are project-specific, not meant to be copied globally.
**Current:** Users might think they need to copy all `.claude/` contents. The README also references copying `us.md` which doesn't exist.
**Proposed:** Add a note: "The `.claude/CLAUDE.md` and `.claude/project/` files are specific to this repo's development — they're not needed for using SI in other projects." Fix the `us.md` reference (depends on #1).
**Impact:** Prevents confusion about which files are reusable tooling vs project-specific documentation.
**Status:** applied (2026-02-22)
**Outcome:** pending-review

## Rejected
