<state>
<updated>2026-02-22 15:30</updated>

## What exists

<codebase>
No source code. This is a Claude Code meta-project — markdown files defining agents, commands, and documentation for the SI (Self-Improver) system.

**Commands** (4 SI orchestrators):
- `commands/si/observe.md` — spawns parallel observer agents, merges findings into improvement-plan.md
- `commands/si/apply.md` — user selects improvements, routes by category (skill/code/workflow/structure)
- `commands/si/status.md` — read-only status view of pending/applied/rejected items
- `commands/si/us.md` — rebuilds state.md from actual codebase

**Agents** (4 specialized observers):
- `agents/si-skill-observer.md` — analyzes skills, commands, agents, CLAUDE.md rules
- `agents/si-code-observer.md` — analyzes source code quality and anti-patterns
- `agents/si-workflow-observer.md` — analyzes session execution efficiency
- `agents/si-structure-observer.md` — analyzes project file organization

**Reports:**
- `project/reports/improvement-plan.md` — 18 items, all applied from first self-observation
- `project/reports/active-directives.md` — 3 directives from applied workflow items
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
│   ├── plans/
│   │   └── deep-knitting-clarke.md
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
</design>

<git>
Branch: main (2 commits + uncommitted changes from first /si:observe + /si:apply cycle)
Remote: origin → https://github.com/giovicordova/self-improver.git
</git>

## What's next

1. **Commit the /si:apply results** — 18 improvements applied, ready to commit
2. **Dogfood again** — run `/si:observe` on itself post-improvement to validate changes
3. **Build feedback loop** — track which applied improvements actually helped (Outcome field now exists but untested)
4. **Proactive observation** — currently manual (`/si:observe`), vision calls for automatic triggering
5. **Learning across projects** — accumulate transferable patterns (not started)

</state>
