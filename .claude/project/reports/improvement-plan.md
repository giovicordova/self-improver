# Improvement Plan

## Pending

## Applied

### 1. state.md is stale -- missing 5 security agents and security-audit command
**Category:** structure
**Evidence:** `.claude/project/state.md` lists "4 specialized observers" and "4 SI orchestrators" but the project now has 9 agents and 5 commands. Structure tree omits all security files. `improvement-plan.md` is listed as existing but was deleted.
**Current:** Every session starts with Claude reading an inaccurate codebase map -- it doesn't know the security agents or security-audit command exist.
**Proposed:** Run `/si:us` to rebuild state.md from the actual filesystem.
**Impact:** Prevents every session from starting with a wrong mental model. Single most impactful fix.
**Status:** applied (2026-02-22)
**Outcome:** pending-review

### 2. README.md missing security-audit command and 5 security agents
**Category:** structure
**Evidence:** `README.md` lists only 4 observers, 4 commands. Grep for "security" returns zero matches. Architecture tree shows only original files.
**Current:** External users see no documentation about the security audit capability -- 5 agents and 1 command are invisible.
**Proposed:** Update README.md: add `si-security-observer` to Observers table, add "Security Audit" section for the 4 deep agents, add `/si:security-audit` to Usage, update Architecture tree.
**Impact:** Users discover and can use the security audit functionality.
**Status:** applied (2026-02-22)
**Outcome:** pending-review

### 3. /si:apply has no pathway to remediate security-audit.md findings
**Category:** skill
**Evidence:** `apply.md` reads only from `improvement-plan.md`. Security audit findings live in `security-audit.md` with a different format (`Status: open`, `Remediation:` instead of `Proposed:`).
**Current:** Running `/si:security-audit` then `/si:apply` shows "No pending items" -- security findings have no remediation pathway through SI. Related: #4
**Proposed:** Add to `/si:apply` Step 2: "Also read `security-audit.md` if it exists. Collect `Status: open` findings. Translate to improvement-plan format: map `Remediation` to `**Proposed:**`, `Risk` to `**Impact:**`, set `**Category:** security`. Present alongside regular pending items." Alternatively, `/si:security-audit` could auto-write Critical/High findings into improvement-plan.md.
**Impact:** Biggest functional gap in the security workflow. Without this, the security audit is purely informational.
**Status:** applied (2026-02-22)
**Outcome:** pending-review

### 4. observe.md doesn't pass security-audit.md context to security observer
**Category:** skill
**Evidence:** `observe.md` Step 2 tells orchestrator to pass "Previous improvement-plan.md items" to avoid duplication but doesn't mention `security-audit.md`.
**Current:** The quick security observer may flag issues already documented in a prior deep audit. Related: #3
**Proposed:** Add to observe.md Step 2 context: "For security observer: also include the Summary section of `security-audit.md` (if exists) so it can skip already-documented findings."
**Impact:** Makes the quick scan and deep audit complementary rather than overlapping.
**Status:** applied (2026-02-22)
**Outcome:** pending-review

### 5. /si:apply dependency update routing lacks explicit confirmation step
**Category:** skill
**Evidence:** `apply.md` line 259: "Dependency updates -> run appropriate package manager command, verify lockfile changed, run tests if available."
**Current:** A security item like "Update lodash to >= 4.17.21" would trigger `npm install` without confirming the exact command beyond batch approval. Package updates can introduce breaking changes.
**Proposed:** After the dependency update routing, add: "Before running any package install/update command, print the exact command and ask for confirmation. Package updates can introduce breaking changes."
**Impact:** Prevents unexpected breaking changes from automatic dependency updates.
**Status:** applied (2026-02-22)
**Outcome:** pending-review

### 6. Three deep security agents missing `.env` read constraint
**Category:** skill
**Evidence:** `si-security-secrets.md` has "NEVER read `.env` file contents." The agents `si-security-code.md`, `si-security-deps.md`, and `si-security-infra.md` lack this constraint. All 4 core observers have it.
**Current:** Three deep security agents could inadvertently read `.env` files during broad file scanning.
**Proposed:** Add to the Constraints section of all three: "NEVER read `.env` file contents. Only check existence and gitignore coverage."
**Impact:** Closes a defense-in-depth gap across all security agents.
**Status:** applied (2026-02-22)
**Outcome:** pending-review

### 7. si-security-observer spawns unnecessarily on meta-projects
**Category:** skill
**Evidence:** `observe.md` spawns the security observer if ANY config files (`*.json`, `*.yml`, `*.toml`) exist. This triggers on virtually all projects including markdown-only meta-projects with no security surface.
**Current:** Running `/si:observe` on SI itself spawns a security observer that will find nothing actionable -- no source code, no server, no dependencies.
**Proposed:** Tighten spawn criteria: spawn only if executable source code files exist (`.ts`, `.js`, `.py`, `.go`, `.rs`, `.rb`, `.java`) OR `.env` files OR Docker/CI files. Config files alone don't warrant a security scan.
**Impact:** Saves one unnecessary subagent spawn on meta-projects (~25% of observer spawns with zero coverage loss).
**Status:** applied (2026-02-22)
**Outcome:** pending-review

### 8. /si:security-audit overwrites previous report with no archival
**Category:** skill
**Evidence:** `security-audit.md` line 91: "This **replaces** any existing report (point-in-time snapshot)."
**Current:** Each audit overwrites the previous one. Users cannot track security posture improvement over time.
**Proposed:** Before overwriting, archive: "If `security-audit.md` exists, rename to `security-audit-YYYY-MM-DD.md` before writing." Update `.gitignore` accordingly.
**Impact:** Enables tracking security posture over time.
**Status:** applied (2026-02-22)
**Outcome:** pending-review

### 9. Report lifecycle models undocumented in CLAUDE.md
**Category:** workflow
**Evidence:** `improvement-plan.md` is append-only, `security-audit.md` is replace-on-run, `active-directives.md` is edit-in-place, `apply-queue.md` is ephemeral. These models are only discoverable by reading individual command files.
**Current:** A developer extending SI might create a new report type without knowing which lifecycle model to follow.
**Proposed:** Add a "Report Lifecycles" section to CLAUDE.md under Conventions documenting each report file's lifecycle model.
**Impact:** Makes report semantics explicit for anyone extending SI.
**Status:** applied (2026-02-22)
**Outcome:** pending-review

### 10. Sync directive missing `mkdir -p` for target directories
**Category:** workflow
**Evidence:** `active-directives.md` line 8: `cp .claude/agents/si-*.md ~/.claude/agents/ && cp .claude/commands/si/*.md ~/.claude/commands/si/`
**Current:** If `~/.claude/agents/` or `~/.claude/commands/si/` doesn't exist (first install or new machine), `cp` fails silently.
**Proposed:** Update to: `mkdir -p ~/.claude/agents ~/.claude/commands/si && cp .claude/agents/si-*.md ~/.claude/agents/ && cp .claude/commands/si/*.md ~/.claude/commands/si/` and add "(includes all observers and security agents)" for clarity.
**Impact:** Prevents sync failure on first installation.
**Status:** applied (2026-02-22)
**Outcome:** pending-review

### 11. No uncommitted changes tracking after apply cycles
**Category:** workflow
**Evidence:** Git status shows significant uncommitted work from cycle 2 apply plus security audit additions. `state.md` "What's next" says "Commit cycle 2 changes" but this was never acted on.
**Current:** Applied improvements accumulate as uncommitted changes across multiple cycles with no clean git boundaries.
**Proposed:** Add directive: "After `/si:apply` completes a full queue, prompt: 'These changes are uncommitted. Commit now with a descriptive message, or continue working?'"
**Impact:** Creates clean git boundaries between improvement cycles. Makes revert/cherry-pick possible per cycle.
**Status:** applied (2026-02-22)
**Outcome:** pending-review

### 12. Workflow observer needs second-level fallback when improvement-plan.md doesn't exist
**Category:** skill
**Evidence:** `si-workflow-observer.md` falls back to analyzing `improvement-plan.md` when session context is sparse, but that file can be deleted (as it is now).
**Current:** On a fresh project or after clearing history, the workflow observer's fallback has nothing to analyze and returns empty.
**Proposed:** Add a second-level fallback: "If improvement-plan.md doesn't exist, analyze active-directives.md for staleness, check CLAUDE.md conventions for consistency, and assess whether the observer/command architecture has process gaps (e.g., missing linkage between commands)."
**Impact:** Prevents empty findings on freshly initialized projects.
**Status:** applied (2026-02-22)
**Outcome:** pending-review

### 13. /si:observe workflow observer context instruction ambiguous when first command
**Category:** skill
**Evidence:** `observe.md` lines 54-55: "provide detailed session events -- list every error, user correction, retry loop, and commands run with outcomes."
**Current:** When `/si:observe` is the first command in a session, there are no prior session events. The orchestrator must infer what to do.
**Proposed:** Rephrase to: "Summarize all significant session events PRIOR to this /si:observe invocation. If /si:observe is the first or only command in this session, explicitly state 'No prior session events -- use fallback analysis mode.'"
**Impact:** Gives the orchestrator a clear decision rule instead of requiring inference.
**Status:** applied (2026-02-22)
**Outcome:** pending-review

### 14. /si:status and /si:security-audit use emoji character
**Category:** skill
**Evidence:** `status.md` line 92 and `security-audit.md` lines 170-171 use `⚠` character.
**Current:** System prompt specifies to avoid emojis. While `⚠` is technically a Unicode symbol, it creates inconsistency.
**Proposed:** Replace `⚠` with `** CRITICAL:` markers in both files.
**Impact:** Removes ambiguity about emoji usage in command output templates.
**Status:** applied (2026-02-22)
**Outcome:** pending-review

### 15. .gitignore allowlist pattern is fragile
**Category:** structure
**Evidence:** `.gitignore` uses `*.md` ignore-all + explicit `!` exceptions for each persistent report file.
**Current:** Adding a new persistent report file requires remembering to update .gitignore. A forgotten exception silently loses the file.
**Proposed:** Invert the pattern: ignore only known ephemeral files explicitly (`apply-queue.md`) instead of ignoring all and whitelisting. New report files tracked by default. Related: #16
**Impact:** Reduces maintenance burden of keeping .gitignore in sync with new reports.
**Status:** applied (2026-02-22)
**Outcome:** pending-review

### 16. .gitignore whitelists security-audit.md -- potential security concern
**Category:** structure
**Evidence:** `.gitignore` whitelists `security-audit.md`, meaning it will be committed to any repo where `/si:security-audit` runs.
**Current:** For SI itself (meta-project), this is harmless. For target projects, this could expose vulnerability details (file paths, CVEs, remediation gaps) in version history. Related: #15
**Proposed:** Remove the `!.claude/project/reports/security-audit.md` whitelist so security reports are generated but not tracked by default. Users can explicitly `git add` if they want to track them. Alternatively, add a warning in the security-audit command output.
**Impact:** Prevents accidental disclosure of vulnerability details in public repos.
**Status:** applied (2026-02-22)
**Outcome:** pending-review

### 17. vision.md doesn't mention security audit capability
**Category:** structure
**Evidence:** `vision.md` describes SI as a 4-observer system. "Observer specialization" is listed as medium-term, but security auditing already fulfills it.
**Current:** vision.md describes a future that has already partially arrived, creating confusion about project state.
**Proposed:** Update vision.md: acknowledge security auditing as a realized capability, update observer specialization bullet to reference it as the first example.
**Impact:** Aligns aspirational doc with reality.
**Status:** applied (2026-02-22)
**Outcome:** pending-review

### 18. Orphaned plan file in `.claude/plans/`
**Category:** structure
**Evidence:** `.claude/plans/deep-knitting-clarke.md` -- 142-line implementation plan from initial SI architecture. Fully executed, correctly gitignored.
**Current:** Historical plan file serves no purpose but appears in filesystem listings.
**Proposed:** Delete the file and empty `plans/` directory. Keep `.gitignore` entry for `.claude/plans/` (Claude Code recreates as needed).
**Impact:** Trivial cleanup. Removes noise from filesystem.
**Status:** applied (2026-02-22)
**Outcome:** pending-review

### 19. Agent naming convention doesn't scale
**Category:** structure
**Evidence:** `agents/` has 9 flat files with two naming patterns: `si-*-observer.md` (observation agents) and `si-security-*.md` (audit agents). `si-security-observer.md` bridges both patterns inconsistently.
**Current:** The only grouping signal is filename prefix. Vision.md implies 15+ agents coming (domain-specific observers). A flat unsorted directory with inconsistent naming will become hard to navigate.
**Proposed:** Standardize naming to `si-{role}-{domain}.md`: observation agents become `si-observe-skill.md`, `si-observe-code.md`, etc. Audit agents become `si-audit-secrets.md`, `si-audit-deps.md`, etc. Update all references in observe.md, security-audit.md, CLAUDE.md, README.md, state.md.
**Impact:** Prevents harder rename later. Moderate migration effort (file renames + reference updates across 6+ files + global sync).
**Status:** applied (2026-02-22)
**Outcome:** pending-review

### 20. Security agent colors all red -- no visual distinction
**Category:** structure
**Evidence:** All 5 security agents have `color: red` in frontmatter. Core observers each have distinct colors (magenta, blue, yellow, cyan).
**Current:** When 4 security agents run in parallel during `/si:security-audit`, their output is visually indistinguishable.
**Proposed:** Assign distinct colors: secrets=red, deps=yellow, code=magenta, infra=green.
**Impact:** Visual clarity during parallel execution. Trivial 4-word change, depends on whether Task tool renders agent colors.
**Status:** applied (2026-02-22)
**Outcome:** pending-review

## Rejected
