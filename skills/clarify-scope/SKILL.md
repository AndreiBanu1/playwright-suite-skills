---
name: clarify-scope
description: Use before writing or planning any test when the ask is not yet precise — "add tests for checkout", "cover the new endpoint", "scope this first". Resolves the open decisions one question at a time, each with a recommended default, looking up in the repo anything that is a fact rather than a choice, and ends in a written brief the user confirms before any code is generated. Skip it when the request is already unambiguous.
---

# Skill: clarify-scope

Turn a vague testing request into a brief precise enough to build from.

"Add tests for checkout" contains at least eight unresolved decisions. Answering them silently means
building the wrong thing confidently; asking all eight at once is a form to fill in, which people
abandon. So: one question at a time, each carrying a recommended answer, and every question that is
really a lookup gets looked up instead of asked.

*The one-question-at-a-time interrogation pattern here is adapted from Matt Pocock's "grilling"
technique, narrowed to test-automation scoping.*

## When to use

- Real forks are open: which layer, which environment, whether negative paths are in scope.
- The request names a feature rather than a behaviour.
- The work is large enough that building the wrong thing costs more than the conversation.

**Skip it** when there is one obvious behaviour, one obvious layer, and an existing pattern to copy.
Interrogating someone who asked for a clearly specified test is its own kind of failure.

## The rules

1. **One question, then wait.** Not two, not a numbered list. Absorb the answer before forming the
   next question — later questions usually change shape based on earlier ones.
2. **Look up facts; ask only about decisions.** Whether a model already exists for that screen, what
   the environments are called, which data file holds the credentials, what the neighbouring specs
   do — all discoverable. Read the repo. Asking a user something the repo already says wastes their
   attention and reads as inattentive.
3. **Every question carries your recommendation, first.** "I'd default to a browser spec on the
   development environment, since that is what the neighbouring specs use — agree?" is answerable
   with one word. A bare open question makes the user do your thinking.
4. **Depth-first.** Resolve each answer's dependents before moving sideways. "Request layer only"
   deletes every browser, device and page-object question below it.
5. **Nothing gets built until the brief is confirmed.** No specs, no models, no files. The brief is
   the deliverable of this skill.
6. **Three questions is usually enough.** If you are past six, the ask is too big — propose splitting
   it rather than continuing.

## The decision tree

Adapt to the request; skip what is already settled.

1. **Restate the ask in one line** and confirm you have it right. Roughly a third of the time this
   alone surfaces the misunderstanding, and everything after it is cheaper.
2. **Behaviour, not feature.** What specifically should be true? "Checkout works" is a feature;
   "an order with an expired card is rejected and the cart is preserved" is a test. Push until you have
   the second kind. This is the highest-value question in the tree.
3. **Layer.** Browser, request, database, or component — from the groups the profile records. This
   gates everything below it. Recommend the cheapest layer that can actually observe the behaviour: a
   rule enforced by an endpoint does not need a browser to prove it.
4. **Environment**, and whether the target must be reachable now. Recommend the repo's default.
5. **Device or browser** (browser layer only). Recommend the repo's default; a matrix run is a
   deliberate ask, not an assumption.
6. **Paths.** Happy path only, or the failure cases too — and if so, which. Recommend including one
   negative case: it is usually where the actual bug lives, and it costs a fraction of the setup you
   are already writing.
7. **Data.** What state must exist first, where the values come from, and who cleans up. Look up how
   the repo does this before asking, then confirm the plan rather than requesting it.
8. **Reuse.** Which models, clients, or schemas already cover this — **look this up, never ask**.
   Report what you found and what will be net-new.

## The brief

State it back in this shape and wait for an explicit yes:

```
Brief
  Behaviour:    <the one thing that should be true>
  Layer:        <browser | request | database | component>
  Target:       <environment / project>
  Paths:        <happy: … · negative: …>
  Reuse:        <what exists and will be extended>
  New:          <what will be created>
  Data:         <source · setup · teardown>
  Out of scope: <what you are deliberately not covering>
  Sequence:     <which skills, in order>
```

`Out of scope` matters as much as the rest. It is what stops the same request coming back as "but it
doesn't cover…".

Then ask for confirmation, once. On yes, hand off to the sequence. On a correction, update the brief
and re-state it — do not start building from a half-agreed version.

## Guardrails

- Do not write code, files, or a plan document during this skill. Its output is the brief.
- Do not ask a question whose answer is in the repo.
- Do not accept "test everything" as scope. Offer the two or three highest-value behaviours and let
  the user pick — an unbounded scope produces a suite nobody maintains.
- Do not smuggle in a decision the user did not make. If they never chose an environment, the brief
  says so, and you ask.
- If the answers reveal the test cannot be written yet — no environment, no data, no hook — say that
  now rather than after the work.

## Done when

The brief is written, confirmed verbatim by the user, and the next skill in the sequence has been
named.
