---
name: write-e2e-test
description: Use when adding a new browser-level Playwright spec — an end-to-end or journey test that drives a real page. Reads the repo's conventions from .claude/test-profile.md, mirrors the nearest existing spec, keeps interaction behind whatever model layer the repo uses, and verifies the spec actually runs before finishing. Not for API-only or component tests.
---

# Skill: write-e2e-test

Add one browser spec that a reviewer would not be able to tell from the specs already in the repo.

Matching the neighbours matters more than matching any external best practice. A spec that is
individually elegant and structurally foreign is a maintenance tax on everyone who touches it next.

## Preconditions

1. `.claude/test-profile.md` exists. If it does not, run **probe-conventions** first — without it
   you would be inventing an import path, and a spec that cannot resolve `test` is worse than no
   spec.
2. The profile lists a `browser` group. If it does not, this repo has no browser tests; ask before
   creating the first one, because it needs a fixture/context decision the user owns.
3. You know which flow is under test and where it starts. If not, use **clarify-scope**.

## Procedure

### 1. Read the neighbour before writing anything

Open the two specs closest to the feature you are adding. Note, concretely:

- the exact import block, in order;
- how the test is titled and whether titles carry tags;
- whether the body is flat or divided into steps, and how those step descriptions are phrased;
- how the test reaches its starting state — a navigation call, a stored session, a setup hook;
- what it asserts on, and whether assertions live in the spec or behind a model method.

You are going to copy all six decisions. Deviating from one of them needs a reason you can say out
loud.

### 2. Decide reuse before creating

Search the model layer named in the profile for the screens this flow crosses. Extend what exists.
Create a new model only when nothing covers the screen — and then via **write-page-object**,
followed by **register-fixture** so the spec can actually reach it.

Grep the existing specs for the same flow first. A duplicate journey test that nobody notices is the
most expensive kind of test: it doubles the maintenance and finds nothing new.

### 3. Write the spec

Structure it as arrange → act → assert, in the repo's own idiom:

```
<the profile's verbatim browser import line>
<the repo's own data import, if it has one>

test('<what the user can do, phrased as an outcome>', async ({ <only the fixtures you use> }) => {
  // arrange — reach the state the behaviour starts from
  // act     — the one interaction under test
  // assert  — the observable consequence that proves it worked
});
```

Rules that hold regardless of repo style:

- **One behaviour per spec.** A second `act` is a second test unless the steps genuinely cannot be
  reached independently.
- **No sleeps.** Wait on the condition you actually care about — a state, a response, an element —
  never on a duration. A fixed wait is either flake or wasted seconds, and usually both.
- **Assert something a user could observe.** Asserting on internal state proves the test ran, not
  that the feature works.
- **Independence by default.** Shared-state ordering (serial mode, or whatever the repo uses) is a
  deliberate choice with a comment explaining what state is shared — not a default.
- **Data from wherever the profile says data lives.** Values inline in a spec are fine only when the
  value *is* the test: a malformed input, a boundary, a deliberately absent record. Those are
  clearer inline than hidden in a fixture file.

### 4. Verify

Run the new spec alone, using the profile's `run one` command. Then run the lint and typecheck
commands if the profile lists them.

A spec you have not executed is a draft. Say "draft" if you are handing it over unrun — do not
report it as done.

### 5. Confirm it can fail

Once green, break the expectation deliberately — change the asserted value — and confirm it goes
red, then restore it. A test that passes against both the correct and the incorrect value asserts
nothing, and this ten-second check is the only way to notice.

## Guardrails

- Never widen an assertion to make a run pass. If the app disagrees with the test and the test is
  right, that is a product finding — report it rather than absorbing it.
- Never leave a generated or recorded spec in raw recorder output. The adapted spec is the
  deliverable.
- Do not introduce a helper, a wrapper, or a config field for a single call site.
- If the flow needs an auth or seeding step the repo has no pattern for, stop and ask. Inventing a
  second auth mechanism is a structural decision, not a test detail.

## Done when

The spec runs green in isolation, has been shown to fail when the expectation is wrong, reads like
its neighbours, and you have reported the file path, the target it was verified against, and any new
model or context wiring it required.
