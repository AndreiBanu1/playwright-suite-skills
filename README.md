# Playwright suite skills for Claude Code

A pack of **13 skills and 4 subagents** for working on Playwright test suites with
[Claude Code](https://claude.com/claude-code): scaffolding specs, hardening locators, triaging a red
suite, reviewing a test diff, scoping the work before any of it starts.

The pack is **project-agnostic by construction.** It contains no conventions of its own. Instead, one
skill reads your repository and writes down what it actually does; the other twelve read that
description and follow it.

**Want to see it work first?** [A worked example](#a-worked-example-playwright-test-healer) — 17 of 53
tests broken on purpose in a real suite, then healed back to green by one of the agents with git history
withheld from it, CI red and green on the record.

## Why not just prescribe conventions?

Because a skill that asserts your fixtures live at one particular path is useful in exactly one
repository, and subtly harmful in every other — it generates specs that cannot resolve `test`,
flags your house style as a violation, and puts locators in a file you do not have.

So nothing here tells you how to write tests. `probe-conventions` inspects your suite — layout, the
verbatim import line your specs use, your observed locator habit, your run commands, the rules your
lint config already enforces — and writes `.claude/test-profile.md`. Every other skill starts by
reading that file.

The practical consequence: `write-e2e-test` in a role-locator, single-fixture repo and the same
skill in a test-id, per-group-fixture repo produce specs that look nothing alike, and each looks like
its neighbours.

**What the pack *does* hold opinions about** is a much shorter list, and it is about whether the suite
can still tell you the truth: no assertion or schema is ever loosened to clear a failure, no test is
disabled to reach a green number, product bugs get escalated rather than absorbed, nothing waits on a
duration, and no secret goes near a spec. Those are not house style — they are the difference between
a test suite and a test-shaped ritual.

## Install

**As a plugin**

```
/plugin marketplace add AndreiBanu1/playwright-suite-skills
/plugin install playwright-suite-skills
```

**Or copy the directories** into the project you want them in:

```bash
cp -R skills agents /path/to/your-repo/.claude/
```

Then, once, in the target repository:

```
Run probe-conventions
```

It writes `.claude/test-profile.md`. Commit that file — it is useful to humans too, and every skill
degrades to asking you questions without it.

> **A note on the subagents.** The four Playwright agents declare tools from the
> [Playwright Test MCP](https://github.com/microsoft/playwright-mcp) server (`mcp__playwright-test__*`).
> If you run a different Playwright MCP server, adjust the `tools:` line in each agent's frontmatter to
> match your server's tool names. The skills need no MCP server at all.

## Using this with other tools

The skills are Claude Code's format. Tools that read `AGENTS.md` — Codex, Amp, Zed, Jules, Gemini CLI,
the Copilot coding agent — will not load them, and their formats have no equivalent of a skill that the
model selects by description and loads on demand.

What ports cleanly is the short list of invariants. Paste this into your own repository's `AGENTS.md`
and you get the part that matters most, in any tool:

```markdown
## Test suite rules

- Never loosen an assertion, widen a status check, or relax a schema to make a failing test pass. If
  the test is right and the application is wrong, that is a bug report, not a test change.
- Never disable, skip, or delete a test to reach a green run. Only ever to record a confirmed product
  bug, with a comment naming the actual behaviour.
- Never add a fixed wait, and never raise retries or timeouts to stabilise something. Wait on the
  condition. Retries reveal flakes; they do not fix them.
- Never put a credential, token, connection string, hostname, or real personal data in a spec, a
  fixture, or a config file.
- Match the surrounding suite's conventions over any external best practice.
- A test that has not been run is a draft. Say so.
- A new test must be shown to fail: break the expected value, confirm red, restore it.
```

If you use VS Code, the four subagents work in Copilot as-is — it reads custom agents from
`.claude/agents` as well as `.github/agents`. Only the MCP tool names may need adjusting.

## The skills

| Skill | Use it when |
| --- | --- |
| `probe-conventions` | First contact with a repo, or after the suite is restructured. Writes the profile everything else reads. |
| `clarify-scope` | The ask is a feature, not a behaviour. Resolves it one question at a time into a confirmed brief. |
| `write-e2e-test` | Adding an end-to-end spec that drives a real page. |
| `write-api-test` | Adding a REST or GraphQL test with no browser. |
| `write-db-test` | Adding a test that queries a database directly. |
| `write-page-object` | A spec needs a page object — the class owning one screen's locators — and it does not exist yet. |
| `write-api-client` | An API test needs an endpoint that no existing client method wraps. |
| `register-fixture` | Making a new page object, client, or helper reachable from specs. |
| `run-suite` | Running tests when the right command, target, or flags are not obvious. |
| `harden-locators` | A locator broke, is about to break, or is XPath. |
| `triage-red-suite` | Many failures at once. Classify before fixing; report findings separately from fixes. |
| `audit-changeset` | Reviewing a test diff, PR, or branch. |
| `handoff-notes` | The work has to continue in a fresh session or someone else's hands. |

## The subagents

| Agent | Role |
| --- | --- |
| `playwright-test-planner` | Explores a live flow and writes a prioritised plan. Writes no code. |
| `playwright-test-generator` | Drives the live app, then rewrites the result into your conventions. Verifies before finishing. |
| `playwright-test-healer` | Repairs one failing browser test. Reproduces first, escalates product bugs. |
| `playwright-api-test-healer` | Repairs one failing request test, including the schema-versus-contract question. |

The agent names match the ones Playwright's own tooling ships, so an existing setup keeps working.

`playwright-test-healer` is the one with [a worked example](#a-worked-example-playwright-test-healer)
below, on a real repository.

## How they fit together

```
clarify-scope                       turn the ask into a confirmed brief
      │
probe-conventions                   write .claude/test-profile.md
      │
      ├─ playwright-test-planner    explore the live flow, plan, write no code
      │
      ├─ write-page-object ─┐
      ├─ write-api-client ──┴─ register-fixture
      │
      ├─ write-e2e-test · write-api-test · write-db-test
      │      or playwright-test-generator
      │
run-suite
      │
      ├─ triage-red-suite           many failures
      ├─ playwright-test-healer     one failure
      │
harden-locators
      │
audit-changeset
```

The middle three groups are the usual chain: build the wrapper a spec needs, register it as a fixture
so the spec can reach it, then write the spec. If the page object and client already exist, you skip
straight to `write-*-test`.

`handoff-notes` cuts across all of it, at whatever point the session runs out.

## A worked example: `playwright-test-healer`

[**demoqa-playwright-framework#2**](https://github.com/AndreiBanu1/demoqa-playwright-framework/pull/2) —
a real 53-test suite broken on purpose, then repaired by the healer agent, with CI red and green on the
record. If you want to know what this pack actually does before installing it, read that PR.

**1. The suite was broken.** Twelve locators in two page objects were rotted deliberately, in two different
ways so the agent had to diagnose rather than pattern-match: `login.page.ts` to off-by-one positional CSS
under a `#userForm` wrapper that does not exist in the DOM, `books.page.ts` to react-table v6 markup
(`div.ReactTable`, `div.rt-tbody`, `span.-pageInfo`) that the app no longer renders.

That took **17 of 53 tests red across six spec files** while only two files were touched — page objects are
shared, which is exactly why locator rot is worth practising on. All 18 API tests stayed green; the fault
was purely in the UI layer.

The detail that matters most: **the CI quality gate passed on the red commit.** Typecheck, eslint, prettier
and spec-placement were all green while 17 tests failed. The breakage is valid, formatted, type-correct
TypeScript. No amount of static analysis catches it — only tests do.

**2. The healer was run, once per failing spec**, headless, each in a fresh session:

```
src/tests/ui/account/login.ui.spec.ts is failing (3 of 3 tests).
Use the playwright-test-healer subagent to repair it.
- Do NOT read git history, diffs, git show, or the reflog, and do not look for
  previous versions of any file. Derive locators from the live page.
- Repair the cause in the page object, not the spec.
- Do the mutation check your procedure requires, and report its result.
```

It was **denied the answer** on purpose: `git diff HEAD~1` would have handed it the solution and the
exercise would have proved nothing. That constraint was verified afterwards against both session
transcripts — zero git-history commands in either run. It worked from the live DOM alone, running the suite
through the Playwright Test MCP server, reading the failure, snapshotting the real page, and generating
locators from what was actually rendered.

**3. What came back.** Its repairs, in its own words, with the specs untouched:

| | was | now |
| --- | --- | --- |
| `userNameInput` | `#userForm > div:nth-child(1) input.form-control` | `getByPlaceholder('UserName')` |
| `loginButton` | positional `.first()` on a button chain | `getByRole('button', { name: 'Login' })` |
| `searchBox` | `div.ReactTable input#searchBox` | `getByPlaceholder('Type to search')` |
| `tableRows` | `div.rt-tbody div.rt-tr-group` filtered `has: a` | `booksTable.locator('tbody').getByRole('row')` |

| | tests | result |
| --- | --- | --- |
| baseline, before the break | 53 | 53 passed, three consecutive runs, `retries: 0` |
| after the break | 53 | **17 failed / 36 passed**, reproduced twice — deterministic, not flake |
| after the heal | 53 | **53 passed**, `retries: 0`, full quality gate clean |

No spec modified, no assertion widened, no timeout raised, no wait added, no retry introduced — the
prohibitions at the top of this README, holding under the one condition that tempts you to break them.

Two things there are worth borrowing whether or not you use this pack:

- **The mutation check is what makes the green mean anything.** Eight expectations were broken in turn —
  `Publisher`→`Publishers`, row count +1, a bogus title, `totalPages` 1→2, both pagination flags, a
  no-match query swapped for a matching one — each confirmed red *for its own specific reason*, then
  restored. All eight behaved as predicted. A suite that was never shown to be capable of failing is not
  evidence that anything works.
- **It improved on what it replaced, and said why.** On two locators it declined to port the original:
  it dropped a `has: a` row filter, because the old DOM emitted empty filler rows while the new one emits a
  genuinely empty `<tbody>` on a no-match search — so the filter would now hide a real regression in
  `expectRowCount`. And it swapped `:nth-child(2)` for a `span[id^="see-book-"]` anchor, binding titles to
  book identity instead of column position.

One honest caveat, since a demo that hides its rough edges is not much use: the agent's written diagnosis
attributes the rot to an upstream rewrite of the application. That is the correct inference from live-DOM
evidence alone — it had no way to know a human had broken it deliberately ten minutes earlier, because that
evidence was deliberately withheld. And the first attempt had to be killed: it wrote a probe spec
containing `page.pause()`, which blocks forever in an unattended run with no one to click *Resume*.

## Three decisions worth knowing about

**Verification is part of every authoring skill, not a separate step.** A spec that has not been run is
reported as a draft. Every scaffolding skill also asks for the ten-second check that the test can *fail*
— break the expected value, confirm red, restore — because a spec that passes against both the right and
the wrong answer asserts nothing, and nothing else catches that.

**`triage-red-suite` reports two columns, not one number.** Fixed test defects on one side, product
findings on the other. "Now green" is not an outcome; it is a number that can be reached by deleting
tests.

**`audit-changeset` reviews by consequence, not by structure.** Weakened assertions, disabled tests and
committed secrets are checked before naming and layout, because a review that flags a filename and
misses a widened assertion has actively done harm — it produced an approval.

## What is deliberately not here

No test framework, no example suite, no config, no CI templates. This pack works on the Playwright suite
you already have; if you need a framework, use Playwright's own scaffolding. The worked example above is
deliberately a *link* to a separate repository rather than a suite vendored into this one — installing the
pack should not drop someone else's tests into your project.

No mocking, visual-regression, load-testing, or accessibility-audit skills. Each is a real specialism and
a thin skill about it would be worse than none.

## Prior art

The one-question-at-a-time interrogation pattern in `clarify-scope` is adapted from
[Matt Pocock's "grilling"](https://github.com/mattpocock/skills) technique, narrowed to test-automation
scoping. The Playwright agent roles follow the planner / generator / healer split established by
Playwright's own agent definitions.

## Contributing

Issues and pull requests are welcome. One thing to know before proposing a skill: a change that makes a
skill assume a convention will be declined, however reasonable the convention. That assumption is the
specific failure this pack exists to avoid.

## License

MIT — see [LICENSE](LICENSE).
