<state>
<updated>2026-02-22 15:34</updated>

## What exists

<codebase>
No source code. This is a Claude Code meta-project — markdown files defining agents, commands, and documentation for the SI (Self-Improver) system.

**Commands** (4 SI orchestrators):
- `commands/si/observe.md` — spawns parallel observer agents, merges findings into improvement-plan.md
- `commands/si/apply.md` — interactive category-by-category walkthrough with plain-language descriptions, routes by category
- `commands/si/status.md` — read-only status view of pending/applied/rejected items
- `commands/si/us.md` — rebuilds state.md from actual codebase

**Agents** (4 specialized observers):
- `agents/si-skill-observer.md` — analyzes skills, commands, agents, CLAUDE.md rules
- `agents/si-code-observer.md` — analyzes source code quality and anti-patterns
- `agents/si-workflow-observer.md` — analyzes session execution efficiency, evaluates applied item outcomes
- `agents/si-structure-observer.md` — analyzes project file organization

**Reports:**
- `project/reports/improvement-plan.md` — 33 items total, all applied across 2 dogfooding cycles
- `project/reports/active-directives.md` — 5 directives from applied workflow items
</codebase>

<structure>
self-improver/
├── .claude/
│   ├── CLAUDE.md
│   ├── agents/
│   │   ├── si-code-observer.md
│   │   ├── si-skill-observer.md
│   │   ├── si-structure-observer.md
│   │   └── si-workflow-observer.md
│   ├── commands/
│   │   └── si/
│   │       ├── apply.md
│   │       ├── observe.md
│   │       ├── status.md
│   │       └── us.md
│   └── project/
│       ├── reports/
│       │   ├── active-directives.md
│       │   └── improvement-plan.md
│       ├── state.md
│       └── vision.md
├── .gitignore
└── README.md
</structure>

<tests>
No tests. This is a prompt/config-only project — no test runner applicable.
</tests>

<design>
Orchestrator + specialized subagent pattern:
- Commands are orchestrators that manage user interaction and workflow
- Agents are specialized workers spawned in parallel with fresh context
- Reports directory (.claude/project/reports/) stores observation results
- improvement-plan.md is the central artifact: append-only pending, items move to Applied/Rejected
- active-directives.md captures learned workflow rules read at session start
- Each observer gets a clean context window for peak analysis quality
- Lifecycle: pending → applied | rejected (no intermediate "approved" state)
- Applied items track Outcome: pending-review → effective | ineffective | reverted
- All observers now read active-directives.md for cross-cycle feedback
</design>

<git>
Branch: main (4 commits, dirty — 12 files modified from cycle 2 apply)
Remote: origin → https://github.com/giovicordova/self-improver.git
Recent:
- 90e9998 feat(si): make /si:apply interactive with plain-language walkthrough
- 40dbb47 feat(si): apply 18 self-observed improvements from first dogfooding cycle
- ab61c3f docs: remove MIT license, add credit line, fix clone URL
- e49f1d2 feat: initial SI system — commands, agents, and project docs
</git>

## What's next

1. **Commit cycle 2 changes** — 15 improvements applied, uncommitted
2. **Dogfood /si:observe** — run on itself to validate cycle 2 changes and test the feedback loop (outcome evaluation)
3. **Proactive observation** — currently manual (`/si:observe`), vision calls for automatic triggering
4. **Learning across projects** — accumulate transferable patterns (not started)

</state>
