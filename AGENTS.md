# AGENTS.md

Guidance for AI agents working **on this repository**. If you are here to work on a Playwright test
suite, you want the skills in `skills/` — not this file.

## What this repository is

A pack of Claude Code skills and subagents for Playwright suites. It contains **no test code, no
framework, and no build step.** Every file is markdown, and the markdown *is* the product.

There is one architectural rule, and it is the reason the pack exists:

> **No skill may assume a convention.**

Skills do not know where specs live, what fixtures are called, which locator style the host repo
prefers, or what the run command is. `skills/probe-conventions` reads the host repository and writes
`.claude/test-profile.md`; every other skill reads that file. A change that hardcodes a path, an
import, a command, or a naming rule breaks the pack's only real promise, however reasonable the
convention looks in isolation.

The second rule is about what the skills are for:

> **A green suite that cannot fail is a regression, not a fix.**

Every skill forbids loosening an assertion or a schema, disabling a test to reach a number, adding a
fixed wait, or absorbing a product bug into the test. If you are editing a skill and find yourself
softening one of those, stop — that is the substance, not the packaging.

## Layout

```
skills/<name>/SKILL.md      13 skills. Directory name MUST equal the frontmatter `name`.
agents/<name>.md             4 subagents. Filename MUST equal the frontmatter `name`.
.claude-plugin/plugin.json   Plugin manifest. `skills` array lists every skill directory.
.claude-plugin/marketplace.json
README.md                    The public surface. Counts in it must match reality.
```

## Adding or editing a skill

1. **Frontmatter is two fields:** `name` (kebab-case, equal to the directory name) and
   `description`. The description is how the model decides to invoke the skill, so write it as *when
   to use this*, with the trigger phrasing a user would actually say — not as a summary of the
   contents. Include when **not** to use it; that line prevents most misfires.
2. **Body structure**, loosely: preconditions → procedure → guardrails → done-when. Keep skills
   under about 140 lines. A skill nobody finishes reading is a skill that does not run.
3. **Every skill that authors or changes code must require verification** — run it, and for a new
   test, confirm it can fail. Reporting an unrun spec as done is the failure mode these skills exist
   to prevent.
4. **Cross-reference by exact skill name in bold** (`**harden-locators**`) so references stay
   checkable.
5. **Update three places** when adding a skill: the `skills` array in `plugin.json`, the skills table
   in `README.md`, and the count in the README's opening line.

## Checks to run before finishing

There is no test suite. These are the checks, and they are cheap:

- **Frontmatter valid** — every `skills/*/SKILL.md` has `name` matching its directory and a
  non-trivial `description`; every `agents/*.md` has `name`, `description`, `tools`, `model`.
- **Manifests parse**, and the `skills` array matches the directories on disk.
- **Cross-references resolve** — every bolded skill name refers to a real directory or agent file.
- **No leaks.** This repository is public and the skills are written for people working in private
  codebases. Nothing may contain an employer or product name, an internal hostname, a real endpoint,
  a credential, a token, an absolute local path, or a personal email address. `.gitignore` already
  excludes `settings.local.json` and `.claude/` for this reason — do not commit either, and do not
  add a permissions file.
- **README counts match** the number of skills and agents.

## Conventions in the prose

Skills are read by a model and skimmed by a human, so: second person, imperative, one idea per
paragraph. State the reason a rule exists when the rule is counter-intuitive — a guardrail whose
purpose is unclear gets rationalised away at the moment it matters. Prefer a table when comparing
options and prose when explaining a judgement. British spelling, sentence case in headings.

## What not to do here

- Do not add a Playwright framework, an example suite, config, or CI templates. The pack works on the
  suite the user already has.
- Do not add a skill that assumes a convention (see above).
- Do not rename the four `playwright-*` agents. Those names match what Playwright's own tooling
  ships, so existing setups keep working.
- Do not add a permissions or settings file.
- Do not claim adoption, usage numbers, or endorsements in the README.
