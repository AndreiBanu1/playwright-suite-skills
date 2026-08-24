---
name: triage-red-suite
description: Use when a suite has multiple failures and needs to be sorted out — "run it and fix what's broken", "get the suite green". Classifies every failure by cause before touching code, fixes only the test-side ones, escalates product bugs instead of absorbing them, and reports a before/after that separates the two. For a single failing test, use the healer agents instead.
---

# Skill: triage-red-suite

Take a suite with many failures and return an honest account of it.

The instruction "make it green" and the goal "make it trustworthy" come apart precisely here, and the
gap is where suites go to die. Every red test is one of two things: a defect in the test, or a message
about the product. Fixing the first is the job. Silencing the second destroys the reason the suite
exists — and it is invisible afterwards, because a suppressed signal looks exactly like no signal.

Nine of ten fixes to a red suite are legitimate. It is the tenth that determines whether anyone trusts
the green.

## Preconditions

`.claude/test-profile.md` exists — you need the run commands and the repo's conventions before
changing test code. For one failing test, dispatch **playwright-test-healer** (browser) or
**playwright-api-test-healer** (request) instead; this skill is for volume.

## Procedure

### 1. Get a clean, complete failure list

Run the suite with retries off and a line-oriented reporter. Retries on will hide flakes as passes and
you will fix the wrong things.

Capture every failure with its actual error. Do not start fixing while the run is in progress —
triage-before-fix is what stops you from patching six symptoms of one cause.

### 2. Look for the single cause first

Before classifying individually, check whether the failures share one root. Common shapes:

- **All of them, immediately** — environment down, wrong target, missing credentials, expired auth.
  Fix that; do not touch a single spec.
- **All specs on one screen or endpoint** — one selector or contract changed. One fix, many greens.
- **Everything after the first failure** — the first spec left state behind, or serial ordering broke.
  Fix the leak, not the followers.
- **Only in parallel, passing alone** — shared state between specs. This is a real defect in the
  suite and deserves a fix, not a serial-mode workaround that hides it.

Twenty failures with one cause is a common and welcome outcome. Twenty individual fixes to the same
cause is the expensive version of the same afternoon.

### 3. Classify every remaining failure before editing

| Class            | What you saw                                            | Where the fix belongs                         |
| ---------------- | ------------------------------------------------------- | --------------------------------------------- |
| **Locator**      | not found, not visible, or several matches               | the model's locator — **harden-locators**     |
| **Contract**     | response shape or field changed                          | the schema or assertion, *if* intended        |
| **Data**         | expected record absent or stale                          | setup and teardown, or resilience to state    |
| **Timing**       | passes alone, fails under load, passes on retry          | wait on the real condition, never a duration  |
| **Auth**         | unauthorised, expired token                              | credential handling or the auth fixture       |
| **Interference** | one spec broke because another ran                       | isolation and cleanup                         |
| **Obstruction**  | a banner, dialog or overlay intercepted an interaction   | dismiss it in setup, not with a sleep         |
| **Product**      | the app genuinely does something else now                | **nowhere in the test** — escalate            |
| **Unknown**      | you cannot reproduce or explain it                       | stays unknown; do not guess a fix             |

Write the classification down before editing. The class dictates the fix; skipping the step is how a
product bug becomes a "fixed assertion".

### 4. Fix one at a time, verifying each

Reproduce the individual failure on demand, change one thing, re-run that spec. A batch of
simultaneous fixes cannot be attributed when the suite is still red afterwards, and you lose the
information about which change helped.

For a stubborn single case, dispatch the healer agent and move on to the next class rather than
sinking the whole session into it.

### 5. Escalate what is not yours

For each **Product** failure: leave the test red, and write up what the test expected, what the app
did, and the shortest reproduction. That is a bug report, and it is the highest-value output of the
whole exercise — it is the thing the suite was built to produce.

If a test is confirmed correct and cannot pass because the product is wrong, mark it as a known
expected-failure using whatever mechanism the repo already uses, with a comment naming the actual
behaviour and a pointer to the report. Never delete it, never comment it out, and never quietly skip
it — a deleted test is coverage nobody knows they lost.

### 6. Re-run everything

Fixed specs individually green is not the same as the suite green. Run the whole thing again to catch
regressions your fixes introduced, then run it once more if anything smelled like interference.

### 7. Report in two columns

```
Suite: <what ran> on <target>
Before: <n> passed / <n> failed        After: <n> passed / <n> failed

Fixed — test defects
  <spec> · <class> · <what changed>

Not fixed — product findings
  <spec> · what the test expected vs what the app did · repro

Not fixed — environment
  <spec> · what is missing or down

Unresolved
  <spec> · what you ruled out, what you would try next

Suite health
  <flakes seen, specs still marked expected-failure, coverage skipped>
```

The two columns are the deliverable. A single "now green" number tells the reader nothing about
whether to trust it.

## Guardrails

- **Never loosen an assertion or a schema to clear a failure.** This is the one rule that makes the
  rest of the suite worth running.
- **Never mark a test skipped or expected-failing to reach a green number.** Only ever to record a
  confirmed product bug, with a comment and a report.
- **Never raise retries as a fix.** Retries reveal flakes; they do not resolve them.
- **Never add a fixed wait.** Wait on the condition.
- **Never delete a failing test** without the user explicitly deciding to drop that coverage.
- If more than a third of the suite is red, stop fixing and report. That is usually one environmental
  or structural cause, and grinding through it spec by spec is the wrong shape of work.

## Done when

Every failure is classified, test defects are fixed and individually verified, product findings are
written up rather than absorbed, the full suite has been re-run, and the report separates what you
fixed from what you found.
