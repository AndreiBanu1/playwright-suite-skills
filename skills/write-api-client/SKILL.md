---
name: write-api-client
description: Use when a request spec needs a call that no existing client method covers — adding a domain client, adding a method to one, or extending the shared transport helper. Covers where the endpoint path belongs, what the method should return, and when a new client is warranted at all. Reads conventions from .claude/test-profile.md.
---

# Skill: write-api-client

Add the reusable call a spec is missing, at the right level.

## Preconditions

1. `.claude/test-profile.md` exists. If the profile records no client layer, calls in this repo are
   made directly from specs — adding a layer is an architectural change, so ask first rather than
   introducing one for a single spec.
2. You know the endpoint, its method, its inputs, and the shape it returns.

## Procedure

### 1. Decide the level before writing anything

Three levels, and picking the wrong one is the mistake this skill exists to prevent:

| Level                      | Choose it when                                                                 |
| -------------------------- | ------------------------------------------------------------------------------ |
| **Existing client method** | The endpoint's domain already has a client. Add a method. This is the common case. |
| **New domain client**      | The domain has no client *and* you can name at least two endpoints it will own. |
| **Shared transport**       | The need is genuinely cross-cutting: a verb wrapper, a response envelope, a retry policy. Rare. |

One endpoint does not justify a new client. Put the method on the nearest existing client and split
later when a second endpoint arrives — a client with one method is indirection with no payoff, and
premature domain boundaries are harder to undo than a later split.

Never reach for the shared transport to solve a domain problem. Anything domain-specific there is
paid for by every caller in the suite.

### 2. Read the neighbouring client

Record the export style, the constructor dependencies, where paths are held, what methods return
(raw driver response, unwrapped body, typed envelope), how headers and auth are threaded, and whether
errors throw or come back as data. Match all of it — a client that returns a different shape from its
siblings makes every spec that uses it read differently.

### 3. Write it

```
class <Domain>Client {
  // endpoint paths as named constants or functions — never inline strings in methods
  // constructor takes only its transport dependency

  // one method per operation, named for the operation, not the verb
}
```

**Paths as named constants.** Inline path strings scattered through methods are the reason a version
bump becomes a twelve-file change.

**Base URL, tenant and credentials are injected, not embedded.** The client knows the path shape; the
environment supplies the rest. A client with a hostname in it works in exactly one environment and
looks like it works everywhere.

**Return the whole response, not just the body**, unless the neighbours do otherwise. Specs need to
assert on status, and a client that swallows it forces callers to guess.

**Do not assert inside a client.** A client that throws on a non-2xx cannot be used to test error
paths — and error paths are where the interesting bugs are. Return the failure; let the spec judge it.

**No test logic.** No branching on scenario, no conditional payload assembly for a particular test.
Parameters in, request out.

**Types over `any`.** If the repo is typed, type the payload and the response. An untyped client is a
client whose contract lives only in the specs that happen to call it.

### 4. GraphQL specifics

Wrap the existing query/mutate transport rather than assembling documents by hand. Keep documents
where the repo keeps them — colocated files or exported constants — and pass variables as variables.
Interpolating a value into a document string breaks on quotes and nulls, and defeats server-side
query caching.

### 5. Wire and verify

Register the client wherever the profile says clients reach specs (**register-fixture**), then prove
it with one real call from a spec. A client verified only by reading is not verified.

Run lint and typecheck if the profile lists them.

## Guardrails

- No hardcoded hosts, tokens, tenant identifiers, or account numbers. This is the file where such
  values most often get committed, because it feels like infrastructure rather than a test.
- Do not log full responses from a client. Auth-adjacent endpoints return tokens, and CI logs are
  usually far more widely readable than the repo.
- Do not add a method with no caller.
- Do not duplicate an existing method under a new name because the old one returns an inconvenient
  shape. Fix the shape, or take the inconvenience.

## Done when

The method exists at the justified level, paths are constants, no environment values are embedded, one
spec calls it green, and you have reported the path plus what you added and why at that level.
