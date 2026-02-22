# Vision

SI (Self-Improver) is a meta-cognitive layer for Claude Code — it makes Claude observe its own execution, find patterns of inefficiency or quality loss, and systematically improve itself across sessions.

## Core Idea

Every Claude Code session generates signal: what worked, what failed, what the user corrected, what was slow. Today that signal is lost when the session ends. SI captures it, analyzes it through specialized observers, and converts it into concrete, trackable improvements.

## Design Principles

**Fresh context per observer.** Each observer gets a clean context window focused on one analysis domain. This is the key architectural insight — monolithic analysis degrades as context fills up. Parallel specialized agents don't.

**Human in the loop.** SI proposes, humans approve. No automatic changes to source code or critical configs. The `/si:apply` command is deliberately interactive.

**Evidence-based only.** Every proposal must reference specific files, lines, patterns, or session events. "This could be better" is noise. "This function at path:line has N+1 queries evidenced by X" is signal.

**Lightweight reports.** Reports are written for Claude to read at session start, not for humans to study. Concise, structured, machine-friendly.

## Where It's Going

### Near-term
- Polish the four observers (skill, code, workflow, structure) for higher signal-to-noise
- Add cross-observer deduplication and conflict resolution
- Build a feedback loop: track which applied improvements actually helped

### Medium-term
- **Learning across projects**: SI currently analyzes one project at a time. It should accumulate patterns that transfer — "this anti-pattern appears in 3 of your projects"
- **Proactive observation**: Instead of explicit `/si:observe`, trigger observation automatically when a session has enough signal (errors, corrections, retries)
- **Observer specialization**: Domain-specific observers (React patterns, API design, database queries) that activate based on the project's stack

### Long-term
- **Self-improving observers**: The observers themselves should improve over time. If an observer consistently produces low-value proposals, that's a signal to improve its prompts.
- **Community patterns**: A shared library of high-value improvement patterns that any SI instance can draw from
- **Quantified improvement**: Measure whether applied improvements actually reduce errors, corrections, and wasted context in subsequent sessions
