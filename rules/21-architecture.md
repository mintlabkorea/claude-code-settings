# Architecture & AI-Code Hygiene

The hard part of AI-generated code isn't writing it — it's that it rots.
Memory resets every session, so the same mistakes recur: duplicated helpers, hidden coupling, swallowed errors, god files, a flat dump of files no one can navigate.
This file is the constitution against that rot. Every rule here has a real violation in our own codebase (cited). Don't treat it as theory.
Where a machine can enforce it, the machine enforces it — a rule that lives only in prose gets forgotten by the next session.

`20-code-style.md` = micro (naming, types, function size). This file = macro (boundaries, coupling, errors, structure, tests).

## The 10 failure modes (what goes wrong)

1. **Re-implementation** — memory is not continuous, so similar functions/helpers/shapes get rebuilt each session. *(us: 5 trackers, 3 sizers, 4 detectors, `demo_negatives_v1~v9`.)*
2. **Coupling ignored** — implicit contracts, hidden imports, init-order dependencies. One edit breaks something elsewhere. *(us: `um_per_px` sim 2.63 / live 6.33, "MUST match" only in comments.)*
3. **Fake tests** — assert a name exists (string check), or over-mock until test and prod are coupled.
4. **No edge cases** — empty/null/boundary/failure paths unhandled.
5. **Nesting spam** — deep nesting inflates complexity.
6. **God function / god file** — one function or one file does everything. *(us: `lab_event_logger.py` 1905 lines, `detect_loop` 397, `main` 520.)*
7. **No boundaries** — file relationships ignored → circular deps make every change harder.
8. **Re-export rot** — re-exports / deprecated aliases left lying around. *(us: `blob_sizer` re-exports static_sizer, `--size-stride` ignored.)*
9. **Silent errors + defense spam** — errors swallowed; fallbacks and per-call-site guards everywhere → duplicated defense, bloated LOC. *(us: 31 `except` in one file, `_broken` self-disables logging while UI still says CAPTURING.)*
10. **Flat hierarchy** — files not organized by layer/dependency. *(us: 195 flat files in `scripts/`.)*

## Design time (before writing)

- **Boundaries first.** Write the module/layer boundaries before code. Know what depends on what.
- **SSOT.** Shared constants / types / shapes / geometry / calibration get exactly one source. No re-declaring a value in a second file. *(us: outlet geometry lived in 4 files with conflicting values.)*
- **Hierarchy by domain/layer.** Directories show dependency direction at a glance. Dependencies point one way only (top → down); the config/SSOT layer imports nothing.
- **Dependency rules are design-time.** Define the allowed import graph in lint/import-linter contracts *before* coding — don't wait for CI to discover boundary drift. The layer contract is part of the design, not an afterthought.

## Instruction time (while writing)

- **Don't swallow errors.** No bare/blanket `except` that continues. No meaningless fallback. No per-call-site defensive guards. Handle errors only at the boundary/handler. A `fail-closed` path MUST surface its state (alert / metric / flag) — silent self-disable is forbidden.
- **Flatten nesting.** 3+ levels: prove the nesting is needed, else early-return / guard-clause it out.
- **Size limits.** A function over 50 lines or a file over 600 lines must be split by responsibility (see `20-code-style.md`).
- **Edge tests before code.** Before writing the production code, write tests for empty input / null / boundary / concurrency / failure path. Always verify observable side-effects — what the function mutates, writes, or emits, not just its return value.
- **Grep before you build.** Before adding any new function/helper/shape, search for an existing one and reuse it. This is the direct countermeasure to memory reset — most duplication is a forgotten existing function.
- **Contracts in code, not comments.** Parity / "MUST equal" / "MUST match" must be enforced by a shared import (SSOT) + a parity test, never by a comment alone.
- **Minimal production hot path.** Experimental features ship off-default and isolated. Do not build a feature elaborately and then leave it sitting off in the tree — off-default and unused means archive it. *(us: touching_split / FWHM / crescent / cv_rl_live_sort all built then OFF.)*

## Review time (checklist before commit)

- Did a duplicate appear where existing code should have been reused? Any dead code / commented-out code left?
- Any silently swallowed error?
- Any hidden coupling — implicit contract, init-order dependency, shared global state, side-effect-linked modules?
- Are re-exports cleaned up? Deprecated aliases / ignored args removed?
- Is fan-in / fan-out over-concentrated in one place?

## Enforce (machine, not memory)

- **Strict types.** All public function args/returns are typed; strict type-checking must make implicit contracts visible. Do not launder a contract with `typing.cast`, a bare `Any`, or a blanket `# type: ignore` (these are the Python equivalents of `as any` / `as unknown as`) — make the type real instead. Untyped languages document contracts with docstrings.
- **CI fails on circular deps / boundary violation.** Tool: `import-linter` (layer contracts).
- **Lint rejects god functions / deep nesting / long files.** Tools: `ruff` (complexity, max nesting, line count) + `radon` (god functions). `lumin-repo-lens` is TS/JS-only → we use the Python equivalents above.

## Test time

- **Verify behavior, not strings.** Assert output/behavior, not that a name exists. Coverage % alone is meaningless.
- **Mock only at external boundaries.** Never mock internal implementation detail — that couples test to prod.
- **Pin edge cases as tests.** Empty input, boundary, null, failure path become permanent test cases.
- **No test-passing defense code.** Check whether defensive code was added to prod just to make a test pass — if so, fix the test or the design, not the prod path.
