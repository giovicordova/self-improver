# Improvement Plan

## Pending

## Applied

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
