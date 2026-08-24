---
name: handoff-notes
description: Use when test-automation work must continue in a fresh session or be picked up by someone else — the context window is filling mid-task, the day is ending, or the work is being passed on. Writes a handoff document to a temporary location (never the repo) containing what a successor needs and nothing they could read for themselves, with secrets excluded and open decisions recorded so they are not re-litigated. Not for finished work.
---

# Skill: handoff-notes

Compress the session into the smallest document from which someone else could continue.

The failure mode is a transcript. A successor who has to read four pages to find the one broken spec
would have been better off starting fresh. Aim for something a competent person reads in ninety
seconds and then knows exactly what to do next.

## When to use

- Context is filling mid-task and the work must continue elsewhere.
- Stopping for the day mid-flow.
- Passing the work to someone else.

**Not** when the work is finished. Completed and committed work needs a summary, not a handoff — say
what you did and stop.

## Rules

1. **Write outside the repository.** A temporary path — `/tmp/handoff-<short-name>.md` or the platform
   equivalent. A handoff is scratch: it must not appear in the working tree, in a commit, or in a diff
   someone reviews next week.
2. **Point, do not paste.** The branch by name, files by path, commits by reference, the diff by the
   command that produces it. Never copy file contents a successor can read themselves — that is what
   turns a handoff into a transcript.
3. **No secrets, no personal data.** If a configuration value matters, name the key, never the value.
   Assume the file will be pasted into a chat window or a ticket, because it usually is.
4. **Record decisions, not deliberation.** What was settled, so nobody reopens it. Not the reasoning
   that got there.
5. **State what is uncertain.** A successor acting on your guess as if it were established fact is the
   most expensive way this document fails. Mark unverified things unverified.
6. **Shape it around the goal.** If the user named a focus when invoking this, that focus is the
   successor's goal, and everything else is context.

## Ground it in facts, not recollection

Before writing, establish from the repository rather than memory: the current branch, what is
uncommitted, which commits are on the branch, the scope of the change, and the last result of the
relevant test run. Getting any of these wrong sends the successor in a direction with confidence,
which is worse than sending them nowhere.

## The document

```markdown
# Handoff — <one-line task>

## Next goal
<the single thing the successor should achieve, in one or two sentences>

## Where things stand
- Branch: <name> → base: <name>
- Uncommitted: <summary, or "clean">
- Last run: <what ran, against what, result — or "not run since <change>">
- Target/environment: <name>

## Done
- <what is finished, with paths — one line each>

## Not done
- <what is half-built, what is untouched, what is blocked and by what>
- <e.g. "the refund spec is written but red — locator drift on the confirmation dialog, not yet fixed">

## Settled — do not reopen
- <decisions already made: layer, environment, scope of negative paths, reuse vs new>

## Uncertain
- <assumptions made without verifying, and how to check each>

## Next steps
1. <concrete action> — <which skill or agent, and why that one>
2. <…>

## Watch out for
- <the flaky spec, the environment that is down, the check that must pass, the file not to touch>
```

Cut any section that would be empty. An empty heading implies you looked and found nothing, which is a
claim in itself.

## Guardrails

- Never write the handoff into the repository, even temporarily. "I'll delete it after" is how these
  get committed.
- Never include a token, password, key, connection string, or personal data — not even redacted in a
  way that reveals its shape.
- Do not claim a spec passes unless you ran it. "Written, not yet run" is a perfectly good status and
  a false green wastes hours.
- Do not include background the successor can get from the repo. Volume is the enemy of use.
- Do not use this to end a task early. If the work is done, finish it.

## Done when

The file is written outside the repo, states one goal, distinguishes verified from assumed, names the
next skill to invoke, and you have given the user the path and a one-line summary of what it says.
