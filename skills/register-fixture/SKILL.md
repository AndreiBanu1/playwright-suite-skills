---
name: register-fixture
description: Use when a newly written page object, API client, or helper needs to become available to specs — the Playwright fixture wiring step. Covers picking the right fixture file, scope choice, dependencies between fixtures, teardown for anything that holds a resource, and why a fixture is preferable to a beforeEach. Reads conventions from .claude/test-profile.md.
---

# Skill: register-fixture

Make something a spec can ask for by name.

This is the step most often skipped, and it fails loudly but misleadingly: the spec errors on an
undefined argument, so the investigation starts in the spec rather than at the missing registration.

## Preconditions

1. `.claude/test-profile.md` exists and names where fixtures are defined. If it says `none`, this
   repo passes dependencies some other way — follow that, and do not introduce a fixture layer
   without asking.
2. The thing being wired already exists and is importable.

## Procedure

### 1. Pick the file

The profile records whether this repo has one shared context file or several independent ones, one per
test group. Both designs are legitimate:

- **Independent per group** — a browser spec cannot accidentally acquire a database connection. Cheap
  isolation, some duplication.
- **One shared, composed** — no duplication, but a careless dependency makes every spec pay for a
  resource only some need.

Whichever it is, add to it. Do not migrate the repo to the other design as a side effect of wiring one
model; that is a separate, discussable change.

### 2. Register

Three edits, in the repo's own style:

1. import the thing;
2. declare it in whatever type or shape describes the available context;
3. provide it — construct, hand it to the test, and clean up afterwards.

Match the neighbours exactly, including ordering if the file is ordered. This file is read constantly;
an entry that reads differently from the rest slows everyone down.

### 3. Scope deliberately

Default to per-test scope. It is the reason a failing spec does not poison the next one, and reusing a
tainted object across a whole worker is one of the harder flake classes to diagnose because the
failure surfaces in a spec that is not the cause.

Widen scope only for something genuinely expensive and genuinely immutable — a fetched static
reference dataset, say. Write the reason in a comment next to it. If the object holds mutable state or
a live connection, it stays per-test even when that is slower. Correctness first.

### 4. Depend on other fixtures rather than reconstructing

A client that needs a transport takes the transport fixture, rather than building its own. Keep the
dependency graph acyclic and shallow: a fixture three levels deep means a browser launches for a spec
that never needed one.

Sequence matters where setup is ordered — authentication before anything that needs a token.

### 5. Tear down what you acquire

Anything holding a connection, a browser context, a temporary file, or a created record gets released
after the test, in the teardown half of the fixture. Teardown runs even when the test failed, which is
exactly why resource cleanup belongs here and not at the end of a spec body.

### 6. Prefer this over a hook

A fixture is better than a `beforeEach` for three reasons worth stating, because the hook is the more
familiar habit: only specs that ask for it pay for it; teardown is colocated with setup instead of
sitting in a separate hook; and the spec signature documents its own dependencies, so a reader knows
what the test touches without scrolling.

### 7. Verify

Run one spec that consumes the new context entry. Then run a second spec that does *not* ask for it,
and confirm nothing new happens for that one — the common wiring mistake is a dependency that makes
every spec in the file acquire the resource.

Run lint and typecheck if the profile lists them.

## Guardrails

- No secrets read into a fixture and then logged. Name the key; keep the value out of output.
- Do not make an expensive resource a dependency of a broadly used fixture. That is how a
  fifteen-second suite becomes a four-minute one, invisibly.
- Do not register the same object twice under two names.
- If two fixtures need to share mutable state, stop. That is a design question, not a wiring detail.

## Done when

The entry is registered, a consuming spec passes, a non-consuming spec is provably unaffected,
anything acquired is released, and you have reported the file and the name specs now use.
