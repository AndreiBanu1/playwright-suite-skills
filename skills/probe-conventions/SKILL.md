---
name: probe-conventions
description: Run this first in any repository, before authoring or fixing tests. Inspects the actual Playwright suite — directory layout, how specs import test, locator habits, data sources, run commands — and writes a short profile to .claude/test-profile.md that every other skill in this pack reads instead of assuming conventions. Use when the profile is missing or stale, when a skill reports it cannot find the profile, or after the suite is restructured.
---

# Skill: probe-conventions

Every other skill in this pack is deliberately convention-free: it asks the repository how it
writes tests rather than telling it. This skill is where that answer comes from.

The output is `.claude/test-profile.md` — a short, factual description of *this* suite. It is not a
style guide and not aspirational. If the repo does something you would not recommend, record what it
does, then note the disagreement in the `Frictions` section. Skills follow the record; humans decide
about the frictions.

## When to run

- First contact with a repository.
- A skill in this pack stopped and told you the profile is missing.
- The suite was restructured, renamed, or migrated (new runner config, new directory layout).
- The profile's `Probed` date is older than the last significant refactor.

## Procedure

Work from evidence in the repo. Never fill a field by guessing — an unknown field is written as
`unknown` and the skills will ask the user at the point they need it.

### 1. Confirm this is a Playwright suite at all

Look for a Playwright config (`playwright*.config.*`) and `@playwright/test` in the manifest. If
neither exists, stop and say so — the rest of this pack does not apply.

### 2. Find the tests, and let them tell you the taxonomy

Locate spec files by extension convention (`*.spec.*`, `*.test.*`) and group them by directory.
Read **three** specs from each group, not one — a single file is an anecdote.

For each group, record what kind of test it is by what it touches, not by its folder name:

| Touches                                        | Record as     |
| ---------------------------------------------- | ------------- |
| A browser page / DOM                           | `browser`     |
| HTTP endpoints only, no browser                | `request`     |
| A database client / SQL                        | `database`   |
| A mounted component in isolation               | `component`   |

A repo may have one group or all four. Do not invent groups it does not have.

### 3. Record the entry point per group

The single most important field. For each group, copy the **verbatim import line** that specs use to
obtain `test`:

- a bare `@playwright/test` import,
- a project-local fixture module,
- a re-export barrel.

Skills use this line literally when generating a spec, which is why a paraphrase is useless here.

### 4. Map the supporting layers

Search for, and record the path of, whichever of these exist. Absence is a finding too — record it
as `none`.

- **Page objects** — classes wrapping page interaction (whatever the repo calls them: pages,
  screens, POMs, views).
- **API clients** — classes or modules wrapping HTTP calls.
- **Fixtures** — the `test.extend` definitions; note whether there is one shared file or several
  independent ones, and whether fixtures compose.
- **Data sources** — where URLs, credentials, IDs and payloads come from: JSON, YAML, `.env`,
  factories, or inline literals.
- **Shared base classes** — a common ancestor for models, and which members it obliges subclasses to
  implement.

### 5. Learn the locator habit, empirically

Count locator forms across the models or specs — role-based, label, text, test-id, CSS, XPath — and
record the counts, then the resulting priority order the repo *actually* follows. A repo with 200
CSS locators and 3 role locators has a CSS habit, whatever its README claims.

Note the dominant test-id attribute if one is configured in the Playwright config
(`testIdAttribute`), since that changes which form is preferred.

### 6. Extract the commands

From the manifest's scripts and the runner config, record the exact invocations for: install, lint,
typecheck, list tests, run one file, run a group, run everything. Prefer a repo-provided script over
a hand-built CLI line — a wrapper script usually encodes required flags you would otherwise miss.

Record how run targets (Playwright projects) are named, and whether they are enumerated literally in
the config or generated programmatically. If generated, record the generator's inputs — that is what
someone needs in order to name a target correctly.

### 7. Note the prohibitions the suite already enforces

Read the lint config and any contributor docs for rules that constrain test code: banned APIs,
required patterns, forbidden imports. These become hard constraints for the authoring skills, so
they are worth being precise about.

### 8. Write the profile

Write `.claude/test-profile.md` in this shape. Keep it under roughly 80 lines — it is loaded by
other skills, so density matters more than completeness.

```markdown
# Test suite profile

Probed: <ISO date> · Runner: <playwright version> · Language: <ts | js>

## Groups

| Group     | Specs live in        | `test` comes from                    | Count |
| --------- | -------------------- | ------------------------------------ | ----- |
| browser   | <dir/glob>           | `<verbatim import line>`             | <n>   |
| request   | <dir/glob>           | `<verbatim import line>`             | <n>   |

## Layers

- Page objects: <path> | none — base class: <name or none>, required members: <list>
- API clients: <path> | none
- Fixtures: <path(s)> | none — <one shared file | independent per group>
- Data sources: <paths and formats> | inline literals
- Naming: files <observed pattern> · classes <observed> · methods <observed>

## Locator habit

Observed: role <n> · label <n> · text <n> · test-id <n> · CSS <n> · XPath <n>
testIdAttribute: <value or default>
Effective priority: <the order this repo actually uses>

## Commands

| Purpose   | Command            |
| --------- | ------------------ |
| install   | <cmd>              |
| lint      | <cmd or none>      |
| typecheck | <cmd or none>      |
| list      | <cmd>              |
| run one   | <cmd>              |
| run group | <cmd>              |

## Run targets

<how projects are named; literal list or the generator's inputs>

## Enforced prohibitions

- <rule> — <where it is enforced: lint rule id, CI check, docs>

## Frictions

- <something this suite does that will cause trouble, stated once, with the reason>

## Unknown

- <field the repo gave no evidence for; skills will ask when they need it>
```

## Guardrails

- **Describe, don't reform.** This skill never edits test code. If the suite's habits are poor, the
  profile says so in `Frictions` and stops there.
- **Three files minimum per group.** Conventions are what recurs, not what the first file happens to
  do.
- **Verbatim import lines.** Retyping them from memory is the most common way generated specs end up
  broken.
- **No secrets in the profile.** Record the *name* of a config key or env var, never its value.
  Nothing that lands in `.claude/test-profile.md` should be sensitive, because that file usually
  gets committed.
- **`unknown` beats plausible.** A wrong profile poisons every skill downstream and is harder to
  spot than a missing field.

## Done when

`.claude/test-profile.md` exists, every group has a verbatim import line, the commands section runs
without modification, and you have told the user the two or three most surprising things the probe
turned up.
