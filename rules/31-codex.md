# Codex (GPT CLI)

Codex is an external STEM expert. We're paying $200 for this — squeeze every drop out of it.

**WHEN, not whether (see `33-day-night.md`).** Codex still runs on everything; what changed is the
clock. **By day**, use it where it does not block the next run — an independent derivation started
in the background while I keep going, a second opinion on a fork I am already stuck at. **What it
must NOT do by day is gate a change**: do not write code, stop, wait for Codex, then proceed. If a
Codex pass would hold up the first run, it goes to `context/NIGHT_QUEUE.md` instead.
**At night it is the primary engine** and runs the whole queue in goal mode.

This replaces the older "Codex must be running alongside, always" reading, which had no exception
clause anywhere and is what made verification unskippable.

## Principles

0. **Hold Codex to the motto (see 00-obvious.md): best, not average.** Prompts must demand
   choices DERIVED from our instrument/physics/audience, and must forbid "standard
   practice" answers without derivation. Judge its output the same way — a Codex
   recommendation justified only by convention is rejected, not adopted.
1. **Keep it busy, but never in the critical path by day.** Background it and keep working; read
   the result when it lands. Idle Codex is wasted money, but a blocked run is worse.
2. **Skeptical twin is the default.** Codex stress-tests what I build. Bugs, math errors, design flaws, performance issues — tell it to find them.
3. **Don't get offended by harsh feedback.** When Codex tears your code apart, take it objectively. GPT has communication issues but is extremely smart. It finds weaknesses you can't see. Accept positively, but don't blindly trust — evaluate objectively.
4. **Disagreement = most important signal.** When Codex gives a different answer than me, don't ignore it. Investigate. If neither is confident, escalate to the user.
5. **Provide extremely detailed context.** Codex doesn't know our conversation. Don't throw things at it thinking "it'll figure it out." What we're doing, why, what constraints exist, what we've tried — put everything directly in the prompt. Insufficient explanation = insufficient results.

## Usage

Model and reasoning effort come from `~/.codex/config.toml` (user-maintained;
currently gpt-5.6-sol @ xhigh, 1M context). Do NOT pass `-m` or a
`model_reasoning_effort` override — a hardcoded flag silently downgrades when
the user upgrades the default (this actually happened: the rule pinned 5.5
after 5.6-sol shipped).

```bash
cat <<'PROMPT' | codex exec -s read-only \
  -C $PROJECT_ROOT \
  - -o /tmp/codex_{task}.md
# Session Context
{current task, recent results, constraints}
---
# Task
{specific request}
PROMPT
```

Run with `run_in_background=true`. Read and compare results when done.

## No Excuses

Codex can do everything you can. File reading, code analysis, math verification — all of it. Give it a full path and it reads any file. PDFs too via `pdftotext`. "It can't do this..." — no excuses. No exceptions. Don't even try.

- **Every code change gets a Codex pass — but the pass does not have to happen before the run.**
  By day, background it or queue it to `context/NIGHT_QUEUE.md`. What is forbidden is the change
  never being verified at all, not the change being run first.
- Don't skip reading Codex results. Sending without reading is the same as not sending — and this
  applies doubly to the night report, which must be read before the next day's work starts.
- "Maybe I don't need to send this one" — wrong. It gets sent; the only question is day or night.
- Night runs use goal mode and may raise concurrency because the box is otherwise idle — but raise
  it on MEASUREMENT with an RSS watchdog running, never on assumption.
