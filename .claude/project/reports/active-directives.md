# Active Directives
*Updated: 2026-02-22*

- Read `.claude/project/state.md`, `.claude/project/vision.md`, and this file at session start.
- When running `/si:observe`, provide detailed session context to the workflow observer — errors, corrections, retries, outcomes.
- All SI commands live under the `si:` namespace. Use `/si:us` not `/us`.
- When an improvement removes a file or output, delete existing instances as part of the apply step.
- After applying changes in the SI repo, sync to global: `cp .claude/agents/si-*.md ~/.claude/agents/ && cp .claude/commands/si/*.md ~/.claude/commands/si/`
