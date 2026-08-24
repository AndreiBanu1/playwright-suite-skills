---
name: write-db-test
description: Use when adding a Playwright test that queries a database directly — verifying persistence, migrations, or that a write through the app landed correctly. Covers parameterised statements only, ordering for shared-row tests, teardown that survives failure, and keeping connection details out of the spec. Reads conventions from .claude/test-profile.md.
---

# Skill: write-db-test

Add one spec that talks to a database.

First, a question worth asking out loud: is a database test the right instrument here? These specs
couple to schema rather than behaviour, so they break on refactors that changed nothing a user can
see. They earn their keep for persistence that has no API surface, for migration verification, and
for asserting that a write made *through* the product actually landed. For anything observable
through an endpoint, a request spec is cheaper and more stable. If this is the second kind, say so
before writing it.

## Preconditions

1. `.claude/test-profile.md` exists and lists a `database` group. If it does not, this repo may have
   no database access wired at all — ask before adding the first one, because connection handling and
   credential storage are decisions the user owns.
2. You know which table or entity, in which environment, and whether the spec writes.

## Procedure

### 1. Read the existing database specs

Record the verbatim import line, how a query is issued, what the query helper returns (a driver
result object, an array of rows, something custom), how connection details reach it, and how existing
specs handle cleanup.

### 2. Write the spec

```
<the profile's verbatim database import line>

// entity and seed values as module-level constants, not scattered literals

test('<what should be true of the stored data>', async ({ <fixtures from the profile> }) => {
  // act    — the query, always parameterised
  // assert — on the specific column that carries the meaning
});
```

**Parameterised statements, always.** Values are passed as parameters; they are never concatenated or
interpolated into the statement. This is not only about injection — although it is that too, and test
utilities have a habit of graduating into production tooling. It is also that a concatenated
statement silently mangles quotes, nulls and dates, and the resulting failure looks like a data
problem rather than a formatting one.

**Assert on a column, not on the row count alone.** `rowCount === 1` passes against a row with every
value wrong.

**Ordering.** Specs that create or mutate shared rows cannot run concurrently against the same
database — use whatever serialisation the repo already uses, and add a comment naming the shared rows.
Read-only specs stay parallel.

**Teardown that survives failure.** Cleanup goes in an after-hook, not at the end of the test body,
because a mid-spec failure skips the rest of the body and leaves the row behind. The next run then
fails on a unique constraint and looks like a product bug. Make cleanup idempotent — deleting nothing
is a success.

**Seed distinctively.** Prefix seeded values so a leaked row is instantly recognisable as test debris
and never collides with real data.

### 3. Verify

Run the spec alone, twice, using the profile's `run one` command. The second run is the one that
matters: it proves teardown worked. Then check the table is genuinely back to its prior state — the
teardown that deletes by the wrong predicate passes both runs while quietly deleting nothing.

Run lint and typecheck if the profile lists them.

## Guardrails

- **Connection details never appear in a spec.** They belong wherever the profile says configuration
  lives. A spec containing a host and password is a credential leak with a long half-life once
  pushed.
- **Never point a spec at production**, and never write to a shared environment whose data other
  people depend on. If the only available database is shared, say so and stop — that is an
  infrastructure decision, not something to work around.
- **Never `DELETE`/`UPDATE` without a predicate** scoped to rows this spec created.
- Do not read a value from the database and then assert it against itself. Assert against the
  expected constant.

## Done when

The spec passes twice consecutively, the table is verifiably unchanged afterwards, every statement is
parameterised, no connection details are in the file, and you have reported the path plus which
environment's database it ran against.
