<state>
<updated>2026-02-22</updated>

## What exists

<codebase>
No source code. This is a Claude Code meta-project — markdown files defining agents, commands, and documentation for the SI (Self-Improver) system.

**Commands** (5 SI orchestrators):
- `commands/si/observe.md` — spawns parallel observer agents, merges findings into improvement-plan.md
- `commands/si/apply.md` — interactive category-by-category walkthrough with batched execution, also reads security-audit.md for Critical/High findings
- `commands/si/status.md` — read-only status view of pending/applied/rejected items
- `commands/si/security-audit.md` — deep security audit, spawns 4 specialized audit agents in parallel
- `commands/si/us.md` — rebuilds state.md from actual codebase

**Agents** (9 total — 5 observers + 4 audit agents):

Observers (spawned by `/si:observe`):
- `agents/si-observe-skill.md` — analyzes skills, commands, agents, CLAUDE.md rules
- `agents/si-observe-code.md` — analyzes source code quality and anti-patterns
- `agents/si-observe-workflow.md` — analyzes session execution efficiency, evaluates applied item outcomes
- `agents/si-observe-structure.md` — analyzes project file organization
- `agents/si-observe-security.md` — quick security surface scan (secrets, injection, config gaps)

Audit agents (spawned by `/si:security-audit`):
- `agents/si-audit-secrets.md` — deep audit: hardcoded creds, .env leaks, git history
- `agents/si-audit-deps.md` — deep audit: CVEs, outdated packages, lockfile integrity
- `agents/si-audit-code.md` — deep audit: OWASP Top 10 code vulnerabilities
- `agents/si-audit-infra.md` — deep audit: Docker, CI/CD, IaC, security headers

**Reports:**
- `project/reports/improvement-plan.md` — 20 items pending (cycle 3), 33 prior items applied across 2 cycles
- `project/reports/active-directives.md` — directives from applied workflow items
</codebase>

<structure>
self-improver/
├── .claude/
│   ├── CLAUDE.md
│   ├── agents/
│   │   ├── si-observe-skill.md
│   │   ├── si-observe-code.md
│   │   ├── si-observe-workflow.md
│   │   ├── si-observe-structure.md
│   │   ├── si-observe-security.md
│   │   ├── si-audit-secrets.md
│   │   ├── si-audit-deps.md
│   │   ├── si-audit-code.md
│   │   └── si-audit-infra.md
│   ├── commands/
│   │   └── si/
│   │       ├── apply.md
│   │       ├── observe.md
│   │       ├── security-audit.md
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
- Agent naming: si-{role}-{domain}.md (observe-* for observers, audit-* for security agents)
- /si:apply reads both improvement-plan.md and security-audit.md (Critical/High findings)
- Security audit archives previous reports with date stamp before overwriting
- Report lifecycles: improvement-plan.md (append-only), security-audit.md (replace-on-run with archive), active-directives.md (edit-in-place), apply-queue.md (ephemeral)
</design>

<git>
Branch: main (6 commits, dirty — cycle 3 apply in progress)
Remote: origin → https://github.com/giovicordova/self-improver.git
Recent:
- 394bbc1 feat(si): add batched execution and progress tracking to /si:apply
- 0517a11 feat(si): apply 15 self-observed improvements from second dogfooding cycle
- 90e9998 feat(si): make /si:apply interactive with plain-language walkthrough
- 40dbb47 feat(si): apply 18 self-observed improvements from first dogfooding cycle
</git>

## What's next

1. **Commit cycle 3 changes** — 20 improvements applied, uncommitted
2. **Sync to global** — copy updated agents and commands to ~/.claude/
3. **Dogfood /si:observe** — run on itself to validate cycle 3 changes
4. **Proactive observation** — currently manual (`/si:observe`), vision calls for automatic triggering
5. **Learning across projects** — accumulate transferable patterns (not started)

</state>
