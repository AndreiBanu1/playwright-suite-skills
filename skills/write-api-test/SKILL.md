---
name: write-api-test
description: Use when adding a Playwright test that exercises an HTTP endpoint directly with no browser — REST or GraphQL — including response-shape validation, error paths, and cleanup after writes. Reads conventions from .claude/test-profile.md and mirrors the existing request specs. Not for tests that drive a page.
---

# Skill: write-api-test

Add one request-level spec: no browser, no DOM, just a call and what came back.

These tests are cheap and fast, which makes them the ones most often written carelessly. The two
failure modes to avoid are asserting so little that the test cannot fail (`status < 500`,
`toBeTruthy()`) and asserting so much that every unrelated field change turns it red.

## Preconditions

1. `.claude/test-profile.md` exists and lists a `request` group. If not, run
   **probe-conventions** first.
2. You know the endpoint, the method, and what the call is supposed to prove. If the ask is
   "add tests for the orders API", use **clarify-scope** — that is a backlog, not a test.

## Procedure

### 1. Read two existing request specs

Record: the verbatim import line, how the call is made (a shared client, a domain-specific client,
the raw request context), how credentials are obtained, how the response is unwrapped, and whether
response shape is validated against a schema or asserted field by field. Copy all of it.

### 2. Route the call the way the repo routes calls

If the profile names an endpoint-client layer, use it. If the endpoint has no client method yet, add
one with **write-api-client** rather than reaching around the layer — one spec bypassing the
client is how the layer stops being true.

If the profile says `none`, calls go through whatever the request group already uses, and you leave
the architecture alone.

### 3. Write the spec

```
<the profile's verbatim request import line>

test('<the behaviour, not the endpoint>', async ({ <fixtures from the profile> }) => {
  // arrange — request inputs, sourced the way the profile says data is sourced
  // act     — the call, through the repo's client layer
  // assert  — status, then shape, then the one or two values that carry the meaning
});
```

Assert in that order and stop there. Status alone is not a test. Every field is not a test either.

**Shape validation.** If the repo validates response shape against schemas, add or reuse a schema
alongside the spec the way the neighbours do. Treat a schema as a contract: when a response stops
matching, the first question is whether the *contract* changed intentionally. Only then does the
schema get updated. Loosening a schema to clear a red run destroys the only thing it was protecting.

**GraphQL.** A `200` with a populated `errors` array is a failure — assert on the absence of errors
explicitly, since the transport status will not tell you. Send variables as variables; never
string-interpolate values into a query document.

**Error paths.** For every write endpoint, one negative case is worth more than three more happy
paths: rejected input, missing record, or unauthenticated. Assert the status *and* the error body's
shape — a 400 with an HTML error page is not the same as a 400 with the documented error object.

**Writes clean up after themselves.** Anything created gets removed in teardown, and teardown must
survive the spec having failed halfway. Then read the resource back — a write endpoint that returns
`201` while persisting nothing is a real bug class, and only the read-back finds it.

### 4. Verify

Run the spec alone with the profile's `run one` command, then run it a second time. Request specs
that pass once and fail on the rerun are leaking state, and that is far easier to fix now than after
it lands in CI.

Run lint and typecheck if the profile lists them.

## Guardrails

- Never hardcode a base URL, token, tenant, or account identifier. Those come from the repo's data
  source, and a spec with an embedded token is a leak the moment it is pushed.
- Never assert on a raw secret's value, and never log a response that contains one.
- Never widen an assertion or a schema to clear a failure. Report the disagreement instead.
- Do not add a retry to hide a race. Find what you should have waited for.
- If the endpoint needs a fixture the repo has no pattern for (an impersonated user, a seeded
  tenant), stop and ask.

## Done when

The spec passes twice consecutively in isolation, asserts status and shape and meaning, cleans up
anything it created, and you have reported the path, the target, and any client method or schema you
added.
