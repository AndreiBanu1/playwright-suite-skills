---
name: audit-changeset
description: Use when reviewing test-automation changes — a pull request, the current branch, or a set of edited spec and model files. Applies a review order that puts trust-destroying changes first (weakened assertions, disabled tests, committed secrets) before style, and reports blocking issues separately from advisory ones. Reads the repo's conventions from .claude/test-profile.md rather than imposing a house style.
---

# Skill: audit-changeset

Review changes to a test suite in the order that matters.

Most review checklists are alphabetical or structural, so naming lands next to secrets and everything
looks equally important. This one is ordered by consequence, because a review that flags a filename
and misses a widened assertion has done harm: it produced an approval.

The question is not "is this code tidy". It is **"after this merges, will a broken product still turn
this suite red?"**

## Preconditions

`.claude/test-profile.md` exists. Without it you will review against remembered conventions rather
than this repo's, and most of your findings will be noise the author is right to ignore.

## Procedure

### 1. Get the diff and the intent

Establish the changed files, the full diff against the base branch, and the commit messages. If a PR
description or linked ticket exists, read it — a review without knowing the intent can only check
style.

Read every changed file **in full**, not just the hunks. A hunk cannot show you that an assertion was
removed three lines above it, or that a spec now has no assertions at all.

### 2. Tier 1 — trust (any one of these blocks the merge)

Check these before anything else, and check them on every hunk:

- **Weakened assertions.** An expectation replaced with something laxer — a specific value becoming a
  truthiness check, an exact match becoming a substring, a status range replacing a status. Ask why. If
  the answer is "it was failing", that is a product finding wearing a test change as a disguise.
- **Weakened schemas or contracts.** A required field made optional, a type widened, a validation
  removed. Same question.
- **Tests disabled, skipped, or deleted.** Every one needs a stated reason and, for a product bug, a
  reference. Coverage removed silently is the most expensive line in any test diff, because nobody can
  see what is no longer being checked.
- **Secrets.** A token, password, key, connection string, or bearer value in a spec, a config, a
  fixture, or a snapshot. **Blocking, and it does not stop at the diff:** once pushed, the secret is in
  history and needs rotating, not just deleting. Say that explicitly in the review.
- **Real personal data** used as test data. Same treatment.
- **Retries or timeouts raised** to stabilise something. That converts a diagnosable flake into a
  permanent slow one.
- **A production or shared-environment target** written into a spec or config.

### 3. Tier 2 — correctness

- Does each test assert on something a user could observe, rather than on the mechanics of its own
  setup?
- Can it actually fail? A spec whose assertion is satisfied by any application state is decoration.
- Is the coverage matched to the intent — does the negative path in the ticket exist in the diff?
- Do writes clean up, in teardown rather than at the end of the body, idempotently?
- Is anything shared between tests without ordering or isolation? That is tomorrow's flake, and it is
  cheapest to catch here.
- Do fixed waits appear anywhere? Each one is either flake or wasted time.
- Are database statements parameterised throughout?
- Are new locators tied to what the user perceives, rather than to position or generated class names?
  Any new XPath is blocking — point at **harden-locators**.

### 4. Tier 3 — structure

Judged against the profile's *observed* conventions, not your preferences:

- Do the new files live where their kind lives, named the way their siblings are named?
- Does the spec obtain `test` from the same place its neighbours do?
- Are new models and clients registered so specs can reach them?
- Is interaction behind whatever layer this repo uses, rather than reaching around it?
- Are locators held where this repo holds locators?
- Are environment values sourced the way this repo sources them?
- Does the change mirror the nearest existing sibling, or invent a parallel structure beside it?

### 5. Tier 4 — advisory

Say these once, and do not block on them:

- abstraction introduced for a single call site;
- an option or config field added for one caller;
- redundant assertions, or a step that asserts nothing;
- a test name that does not say what breaks when it fails;
- duplication that a later change will have to fix in two places.

### 6. Run the checks the repo already has

Run lint and typecheck from the profile. A failure there is blocking and is not worth writing prose
about — it just needs to pass.

Where practical, run the changed specs. A review that verifies is worth several that infer.

## Reporting

```
Review: <branch or PR>

What this changes
  <one or two sentences, in the author's terms>

Blocking
  <file:line> — <what> — <why it blocks> — <the fix>

Advisory
  <file:line> — <suggestion> — <why>

Verified
  <what you ran and its result>

Not reviewed
  <anything you could not assess, and what you would need>
```

Order blocking items by consequence, not by file. Attribute nothing to intent — describe the effect of
the code and let the author explain.

If there are no blocking issues, say so plainly. A review that manufactures a finding to look thorough
teaches the author to discount the next one.

## Guardrails

- **Never approve a diff that weakens a check without an explanation you find convincing.** That is the
  whole job.
- Do not fix the code while reviewing it unless asked. Review and rewrite are different requests.
- Do not require the author to adopt a convention this repo does not use.
- Do not reproduce a secret's value in the review, in a comment, or in a commit message. Name the file
  and the key.
- Distinguish "I could not verify this" from "this is fine". They are not the same, and only one of
  them is an approval.

## Done when

Every changed file has been read in full, all four tiers have been applied, blocking and advisory are
separated, the repo's own checks have been run, and anything you could not assess is stated as such.
