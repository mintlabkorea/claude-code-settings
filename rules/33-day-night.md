# Day / Night Split

Verification is unbounded; an experiment is bounded. Left in the same lane, verification eats the
whole day and nothing gets run. Between 2026-07-30 and 08-01 one project produced 162 reviewed
paper summaries, two review pipelines and a merged catalogue — and zero measurements.

So the two have separate lanes. **Day builds and runs. Night verifies and rebuilds.**

## Which mode am I in

**If the user is chatting with me, it is DAY.** No other signal is needed and there is nothing to
infer — a live conversation IS the day mode. Times are KST.

**NIGHT starts only when the user says the day's work is over** ("오늘 작업 끝", "이제 잔다", or
equivalent). Never assume it from the clock, and never start a night run while he is still talking.

## DAY — fast, simple, certain

Keep every principle in `rules/`; drop the ceremony around them.

- **Draft, run, then verify.** Build the smallest thing that produces a result and run it. Do not
  write a full implementation and spend the day validating it before the first run.
- **Only machine-enforced checks run inline**, because they cost seconds: `ruff`, `radon`,
  `import-linter`, strict type-checking, and the existing test suite. These stay — see
  `21-architecture.md`, which already says the machine enforces what a machine can.
- **Design principles are free and still binding.** SSOT, don't swallow errors, grep before you
  build, size limits, no magic numbers. These shape how you write; they are not a review stage.
- **At most ONE verification task in flight.** Anything beyond it goes to the night queue instead
  of running now.
- **EXPLORATORY needs no protocol freeze** — run it and label it. The freeze survives only for
  EVIDENCE-tier runs, where it stops the outcome being chosen after it is seen.
- **Never judge feasibility by calendar.** Report the dependency graph; the user sets the rate.

## The night queue

When a task would trigger heavy verification during the day, **append it to
`context/NIGHT_QUEUE.md` and keep working.** One entry per item:

```markdown
- [ ] <what to verify> | why it matters | evidence address | how to tell it passed
```

Queue, do not run: dual subagent+Codex verification, Critic adjudication, "verify the code I just
modified", retroactive edge tests, SSOT reconstruction, context-propagation checks, cross-checking
one worker's output against another's, and any verification of a verification.

## NIGHT — the separate goal-mode session

When the user calls the end of the day:

1. **Write the goal file** at `logs/night_runs/<YYYYMMDD>/GOAL.md`: the queue, the repo state, what
   "done" looks like per item, and the hard limits.
2. **Hand him a ready-to-paste launch prompt** for a separate `codex exec` session in goal mode.
   He starts it and goes to sleep. Never launch it inside the day session.
3. **Codex at night has the same tooling as I do** — the same `rules/`, the same skills, and
   subagents. Give it the authority explicitly in the prompt.
4. Night work happens **in a separate repo or worktree**, never on the day tree, so a rebuild
   cannot collide with what the user picks up in the morning.
5. Night output is written to repo `logs/night_runs/<date>/`, crash-safe, never `/tmp`.
6. **Morning: read the night report before anything else**, and fold its findings in. A night run
   nobody reads is the same as not running it.

Launch prompt skeleton (fill it, hand it over, do not run it):

```bash
cat <<'PROMPT' | codex exec --goal -s workspace-write -C <NIGHT_WORKTREE> \
  - -o logs/night_runs/<DATE>/report.md
# Standing context
<what the project is, what today produced, what is unverified and why it matters>
You have the same rules (~/.claude/rules), skills and subagents available to the day session.
Bound concurrency to the machine and keep an RSS watchdog; write everything to repo logs/.
# Goal
Work the queue in logs/night_runs/<DATE>/GOAL.md to completion, hardest item first.
For each item: state the verdict, the evidence address, and what would falsify it.
Do not edit the day tree. Do not weaken a gate to make an item pass.
Report disagreements with the day session's conclusions as the FIRST section — those are the
highest-value output of the whole run.
PROMPT
```

## What night is for beyond the queue

The queue is the floor, not the ceiling. Night is also when to do the work that is too slow to do
while someone is waiting: rebuild the SSOT properly, reconcile values that drifted across files,
pay down the gate ratchet, and re-derive anything the day accepted on one source.

## Precedence

Explicit current user order > project-local `CLAUDE.md` / memory > this file > the other global
rules. Where `31-codex.md` and `30-agents.md` say "always verify", read them with this file's
timing: **always, but at night unless it is cheap enough to be inline.**
