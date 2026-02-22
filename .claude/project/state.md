<state>
<updated>2026-02-22 14:07</updated>

## What exists

<codebase>
No source code. This is a Claude Code meta-project — it contains only markdown files defining agents, commands, and documentation for the SI (Self-Improver) system.

**Commands** (3 SI orchestrators + 1 utility):
- `commands/si/observe.md` — spawns parallel observer agents, merges findings into improvement-plan.md
- `commands/si/apply.md` — user selects improvements, routes by category (skill/code/workflow/structure)
- `commands/si/status.md` — read-only status view of pending/applied/rejected items
- `commands/us.md` — rebuilds state.md from actual codebase

**Agents** (4 specialized observers):
- `agents/si-skill-observer.md` — analyzes skills, commands, agents, CLAUDE.md rules
- `agents/si-code-observer.md` — analyzes source code quality and anti-patterns
- `agents/si-workflow-observer.md` — analyzes session execution efficiency
- `agents/si-structure-observer.md` — analyzes project file organization

**Plans:**
- `plans/deep-knitting-clarke.md` — original architectural blueprint for the SI system
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
│   │   ├── si/
│   │   │   ├── apply.md
│   │   │   ├── observe.md
│   │   │   └── status.md
│   │   └── us.md
│   ├── plans/
│   │   └── deep-knitting-clarke.md
│   └── project/
│       ├── state.md
│       └── vision.md
└── README.md
</structure>

<tests>
No tests. This is a prompt/config-only project — no test runner applicable.
</tests>

<design>
Orchestrator + specialized subagent pattern (borrowed from GSD):
- Commands are orchestrators that manage user interaction and workflow
- Agents are specialized workers spawned in parallel with fresh context
- Reports directory (.claude/project/reports/) stores observation results
- improvement-plan.md is the central artifact: append-only pending, items move to Applied/Rejected
- Each observer gets a clean context window for peak analysis quality
</design>

<git>
Not a git repository. Needs `git init`.
</git>

## What's next

Based on vision.md, the gap between current state and vision:

1. **Initialize git repo** — no version control yet
2. **Polish observers for higher signal-to-noise** — the four observers exist but haven't been battle-tested from this repo as canonical source
3. **Add cross-observer deduplication** — currently noted in observe.md but could be more robust
4. **Build feedback loop** — track which applied improvements actually helped (no mechanism for this yet)
5. **Proactive observation** — currently manual (`/si:observe`), vision calls for automatic triggering on sufficient session signal
6. **Learning across projects** — accumulate transferable patterns (not started)

Immediate priority: git init, then test the observe/apply cycle on this repo itself (dogfooding).

</state>
