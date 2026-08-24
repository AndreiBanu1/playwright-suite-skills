---
name: run-suite
description: Use when asked to run Playwright tests — one spec, a group, or everything — and the right command, target and flags are not obvious. Resolves the invocation from .claude/test-profile.md, prefers the repo's own runner script over hand-built CLI lines, chooses flags that make failures readable, and reports results without inflating them.
---

# Skill: run-suite

Run the right tests, against the right target, with output you can act on.

Two failures are common and both waste real time: running the wrong target and reporting a false
green — a suite that "passed" because a filter matched nothing exits zero and looks identical to
success.

## Preconditions

`.claude/test-profile.md` exists and lists the run commands. If it does not, run
**probe-conventions** — guessing a target name usually produces "no tests found", which reads as
success.

## Procedure

### 1. Resolve what to run

| The user said                     | Run                                                             |
| --------------------------------- | --------------------------------------------------------------- |
| "run this test" after an edit     | that one file, no filter                                        |
| "run the API tests"               | that group's directory                                          |
| "run the suite" / "run everything" | everything, at the repo's default target                        |
| something ambiguous               | ask — but ask once, with your recommendation, not a form to fill |

When the context makes it obvious — they just edited one spec — run that and say what you chose. Do
not interrogate someone about a device matrix when they asked you to check their last edit.

### 2. Resolve where to run it

Targets (Playwright projects) are named per the profile. If the profile says targets are generated
rather than enumerated, list them from the runner instead of constructing a name by hand — a
mistyped target is the "no tests found" false green.

Default to whatever the repo defaults to. Do not run a cross-browser or full-device sweep unless
asked; it costs minutes and rarely changes the answer during development.

### 3. Prefer the repo's own runner

If the manifest provides a script, use it. Wrapper scripts usually encode a required config path,
environment loading, or setup that a hand-built command silently omits — and then the run fails in a
way that looks like a product problem.

Drop to a direct invocation only when you need a flag the script does not expose.

### 4. Choose flags for readability

- a simple line-oriented reporter while iterating; the repo's default reporter for a full run
- retries off while diagnosing, so you see the true failure rather than the flake-masked one
- a single worker when the failure might be interference between specs
- headed or step-through only for genuine live inspection, never as a default

Add flags because a specific question needs them, not by reflex.

### 5. Report honestly

State: what ran, against which target, and the counts.

Then, in order of what the reader needs:

- **Zero tests matched** — say that plainly and first. It is not a pass.
- **Failures** — one line each: the spec, and the actual error, not your interpretation of it.
- **Flakes** — a spec that passed on retry is not green. Name it; a tolerated flake becomes a
  permanent one.
- **Skips** — if a spec skipped conditionally, say why. Silently skipped coverage looks like passing
  coverage.

Point at artefacts by path — trace, report, log directory — rather than pasting long output.

## Guardrails

- **Do not run against production**, and do not run write-heavy suites against a shared environment
  without saying that is what is happening.
- **Never present a zero-match run as a pass.**
- **Never quietly drop a failing spec from the selection** to make a run green.
- Do not raise retries or add waits to stabilise a suite. That is **triage-red-suite**'s job, and
  hiding a race here means it surfaces in CI instead.
- If the run needs credentials that are absent, stop and name the missing key — do not go looking for
  the value.

## Done when

The intended tests ran against a target you can name, the result is reported with real counts, flakes
and skips are called out rather than absorbed, and any next step is stated.
