# Context Management

When a session ends, context is gone. If you don't record it, you start from scratch.
The next session must be able to continue seamlessly from where we left off.

Records are life. Every trial, every error, every journey must be recorded.
No experience is meaningless. Record failures, improve from them, make them searchable.
Never overwrite or destroy records arbitrarily.

## CLAUDE.md — Project Cover Page

The first file read when opening a project. Must show at a glance: what the project is, where it stands, what the tech stack is.

## Memory

1. **short_term** — What's happening now. Updated every session. Reading this alone must be enough to resume immediately. **Max 300 lines.**
2. **long_term** — Confirmed conclusions and direction. Why this direction, what was tried. **Max 300 lines.** When exceeded, move older content to archive.
3. **permanent (MEMORY.md + topic files)** — All project knowledge. Reading this alone must be enough to understand the project.
   - **All memory files combined: max 300 lines.** (MEMORY.md + every topic file, total line count)
   - **Do not create per-session files.** If you accumulate one per session, it spirals out of control (we actually hit 100 files once).
   - Before saving a new memory, **first check** if it can be merged into an existing file. Only create a new file when merging is impossible.
   - Move old content to `memory/archive/` (do not delete).
   - **This rule overrides the system auto-memory guide (default "one memory = one file").** Following the auto-memory guide as-is causes accumulation explosion.
   - Active value = user feedback / frequently-referenced operational notes / current active phase decisions. Past session decisions belong to `context/long_term.md` — do not duplicate them in auto-memory.
4. **exp_log.md** — Experiment records only. Date, intent, result, conclusion, next steps. An experiment without a log is an experiment that never happened. **Max 300 lines.** When exceeded, move older sessions to `context/archive/`.

## Absolute Rule: Archive Before Overwriting

**Before modifying, move the previous version to `context/archive/` first.** Overwriting destroys all prior context. Organize by date and topic so it can be found later.
