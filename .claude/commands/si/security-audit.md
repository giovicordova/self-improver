---
name: security-audit
description: Deep security audit — spawns specialized security agents in parallel for comprehensive vulnerability analysis
disable-model-invocation: true
allowed-tools: Read, Glob, Grep, Bash, Task, Write
---

# /si:security-audit — Deep Security Audit

Perform a comprehensive security audit by spawning specialized security sub-agents in parallel. Each agent targets a specific attack surface. Results are deduplicated, severity-ranked, and written to a single report.

## Workflow

### 1. Load project context

First, ensure the reports directory exists:
```bash
mkdir -p .claude/project/reports
```

Then read these files to understand the project:
- `.claude/CLAUDE.md` or `~/.claude/CLAUDE.md` — project identity, conventions
- `.claude/project/state.md` — codebase snapshot (if exists)
- `.claude/project/reports/security-audit.md` — previous audit (if exists, will be replaced)

### 2. Detect project stack

Run these checks to build a stack descriptor. Pass this to all agents.

**Languages and frameworks:**
- Glob for manifest files: `package.json`, `pyproject.toml`, `Cargo.toml`, `go.mod`, `Gemfile`, `composer.json`
- Read manifest(s) to identify frameworks (Express, Django, Flask, Rails, Spring, etc.)
- Glob for source files to confirm languages: `**/*.ts`, `**/*.py`, `**/*.go`, `**/*.rs`, `**/*.rb`, `**/*.java`

**Infrastructure:**
- Glob for: `Dockerfile*`, `docker-compose*.yml`
- Glob for: `.github/workflows/*.yml`, `.gitlab-ci.yml`, `Jenkinsfile`, `.circleci/config.yml`
- Glob for: `*.tf`, `cloudformation*.yml`, `k8s/*.yml`, `helm/**/*.yml`
- Glob for: `nginx*.conf`, `Caddyfile`, `.htaccess`

**Environment:**
- Check for `.env*` files, `.gitignore` coverage
- Check for config files with potential secrets

Build a stack descriptor string, e.g.:
```
Stack: Node.js/TypeScript + Express | Docker + GitHub Actions | No IaC
Manifests: package.json, package-lock.json
Frameworks: express, passport, prisma
Infra: Dockerfile, docker-compose.yml, .github/workflows/ci.yml
```

### 3. Spawn security agents

Spawn relevant agents **in parallel** using the Task tool. Launch them simultaneously.

**si-audit-secrets** — **Always spawn.** Every project can have accidental credential commits.
- Provide: stack descriptor, .gitignore contents, list of config/env files found

**si-audit-deps** — Spawn if **any manifest or lockfile exists.**
- Provide: stack descriptor, list of manifest/lockfile paths, ecosystems detected

**si-audit-code** — Spawn if **any source code files exist.**
- Provide: stack descriptor, frameworks detected, entry points (routes/handlers) if identifiable, list of source directories

**si-audit-infra** — Spawn if **any of these exist:** Dockerfile, CI config, IaC files, web server config, OR if a web framework is detected (for security header checks).
- Provide: stack descriptor, list of infrastructure files found, web framework info

For each agent, include in the prompt:
- The full stack descriptor
- Project path and name
- Relevant file lists for their domain
- Any project conventions from CLAUDE.md that affect security (e.g., "we use Vault for secrets")

### 4. Collect and deduplicate results

After all agents return:

1. **Collect** all findings from all agents
2. **Assign sequential finding IDs** starting from F-001
3. **Deduplicate** using these rules:
   - Same file + same line → keep the domain-authority agent's finding (e.g., secrets agent wins for credential findings)
   - Same file + different lines + same CWE → keep both, add cross-reference note
   - Same root cause across files → merge into single finding, list all affected locations
   - Different instances of same vulnerability class → keep all (each needs individual remediation)
4. **Sort by severity:** Critical → High → Medium → Low → Informational
5. **Within same severity:** sort by CWE (groups related vulnerabilities)

### 5. Write security audit report

If `.claude/project/reports/security-audit.md` already exists, archive it first:
```bash
mv .claude/project/reports/security-audit.md .claude/project/reports/security-audit-$(date +%Y-%m-%d).md
```

Write to `.claude/project/reports/security-audit.md` (point-in-time snapshot).

```markdown
# Security Audit Report

*Generated: YYYY-MM-DD*
*Project: [name]*
*Stack: [stack descriptor]*

## Summary

| Severity | Count |
|----------|-------|
| Critical | N |
| High | N |
| Medium | N |
| Low | N |
| Informational | N |
| **Total** | **N** |

External tools used: [list or "none available — Claude-native analysis only"]
Agents spawned: [list]

## Critical Findings

### F-001. [Title]
Severity: Critical
CWE: CWE-XXX (Name)
OWASP: A0X (Category)
Location: file/path:line
Evidence: [description — NEVER actual secret values]
Risk: [concrete attack scenario]
Remediation:
  1. [immediate fix]
  2. [prevention]
Detected by: [tool or claude-native]
Status: open

## High Findings

[same format]

## Medium Findings

[same format]

## Low Findings

[same format]

## Informational

[same format]

## Methodology

- Secrets scanning: [tools used + claude-native patterns]
- Dependency audit: [tools used + manual analysis]
- Code analysis: [tools used + OWASP categories checked]
- Infrastructure review: [tools used + configs checked]
```

### 6. Print summary

```
Security audit complete.

  Stack: [stack descriptor]
  Agents spawned: [list]
  External tools: [list or "none available"]

  Findings:
    Critical:      [N]
    High:          [N]
    Medium:        [N]
    Low:           [N]
    Informational: [N]

  [If Critical > 0:]
  ** CRITICAL findings require immediate attention. Review .claude/project/reports/security-audit.md

  [If Critical == 0 and High > 0:]
  High-severity findings should be addressed before deployment.

  [If Critical == 0 and High == 0:]
  No critical or high-severity issues found. Review medium/low findings at your convenience.

  Top findings:
  - F-001: [title] (Critical)
  - F-002: [title] (High)
  - F-003: [title] (High)
  [List up to 5 most severe]

  Full report: .claude/project/reports/security-audit.md
```

## Constraints

- **Spawn agents in parallel.** Sequential spawning defeats the purpose.
- **Replace the report on each run.** Security audits are point-in-time snapshots, not accumulating proposals.
- **Always spawn the secrets agent.** Every project can have accidental credential commits.
- **NEVER include actual secret values in the report.** Use `[REDACTED]` and describe patterns only.
- **Tag findings with `Detected by`.** Enables reproducibility — external tool findings can be re-run.
- **Never modify source code or configs.** Only write to `reports/`.
- **Deduplicate across agents.** Multiple agents may flag the same issue from different angles.
- **Keep the summary actionable.** Tell the user what to do first.
