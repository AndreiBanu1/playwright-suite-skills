---
name: harden-locators
description: Use when a locator broke, is about to break, or is XPath — replacing positional and structural selectors with ones tied to what the user perceives. Covers XPath and brittle-CSS replacement patterns, disambiguating between multiple matches, handling text that varies across environments, and when a missing hook should be requested from the app team instead.
---

# Skill: harden-locators

Replace selectors that describe where an element *sits* with selectors that describe what it *is*.

A locator is a claim about the page. `.col-md-4 > div:nth-child(2) > span` claims a layout will never
change, which is the one thing layouts do. `getByRole('status')` claims an element announces itself as
a status, which changes only when the meaning changes — and if the meaning changed, the test *should*
break.

## Preconditions

`.claude/test-profile.md` exists and records the repo's observed locator habit and its configured
test-id attribute. Without it you may "harden" a suite into a form inconsistent with the other 200
locators, which trades one problem for a worse one.

## The order to prefer

Consult the profile first — match the repo's habit unless it is XPath or positional CSS, which are
never the right habit. Absent a strong repo habit, prefer in this order:

1. **Role plus accessible name** — how assistive technology identifies the element. Survives styling,
   markup and framework changes.
2. **Label, placeholder, or associated text** for form controls — what the user reads to know what the
   field is for.
3. **Visible text** for content and links, exact-matched to avoid catching substrings.
4. **A dedicated test attribute** — stable and explicit, but invisible to users, so it cannot tell you
   the element is still labelled correctly. Prefer it over CSS, not over role.
5. **Semantic CSS** — a component-level class or tag that a developer would have to *decide* to
   rename.
6. Never XPath. Never index-based structural CSS.

## Replacement patterns

| Brittle                                   | Harden to                                                       |
| ----------------------------------------- | --------------------------------------------------------------- |
| XPath matching a button by its text       | role `button` with that accessible name                         |
| XPath matching a link by its text         | role `link` with that name                                      |
| XPath on an `@id`                         | the id selector directly, or better, the element's role          |
| XPath on an `@class`                      | the class selector, or better, the role                          |
| XPath on an input's `@type`               | the matching form role, or the field's label                     |
| XPath walking down a hierarchy            | a scoped locator: find the container by role, search within it   |
| `nth-child` / `:eq()` / an index          | filter by the text or child the target actually contains         |
| A generated or hashed CSS class           | role, or ask for a test attribute — the hash changes every build |
| A locator built from a full DOM path       | the nearest meaningful ancestor, then a role query inside it     |

## Techniques for the hard cases

**Several elements match.** Do not reach for an index — indices reorder. Narrow by content: locate the
repeated container, filter it to the one containing the distinguishing text, then find the control
inside it. This survives reordering and reads like the user's own reasoning: *the row for X, then its
delete button*.

**Text varies by environment, tenant, or locale.** Two options, in order of preference: locate by role
alone if the role is unambiguous within its scope, or accept either name via an alternation. Do not
match a shared substring — that is how a locator starts matching two elements a release later.

**Text is dynamic** (counts, currency, dates). Match the stable part with a pattern anchored at the
start, or assert on the container and read the value out separately. A locator that embeds today's
date works for one day.

**Nothing distinguishes the element.** Sometimes the honest answer is that the page is not testable at
this point, and the fix belongs in the application: a label, an `aria-label`, a test attribute. Say so
and request it. Building an elaborate structural locator around a missing hook is a slow-motion
failure that someone else inherits.

## Procedure

1. Read the whole file first. Locators cluster, and fixing one while leaving four identical ones is
   half a job.
2. Confirm what the element actually is — inspect the live page rather than inferring from the old
   selector. A guessed role produces a locator that resolves to nothing, and the failure looks
   identical to the one you were fixing.
3. Replace **only the selector**. Do not rename methods or change logic in the same pass; a mixed diff
   is much harder to review, and if it regresses nobody can tell which half did it.
4. Keep the locator where the repo keeps locators — for a model class, that is the constructor.
5. Run the affected specs. A locator change is verified by execution, never by reading.
6. Where the element had no good hook and you worked around it, leave a one-line comment naming what
   the application should expose. That comment is what turns a workaround into a request someone can
   act on.

## Guardrails

- Never introduce XPath, even as a temporary measure.
- Never swap a broken locator for one that matches more elements just to get a pass. Silently
  operating on the wrong element is worse than a red test.
- Do not add a wait to compensate for a locator that is wrong.
- Do not change a locator you have not seen the page for.
- If hardening a locator would require an application change, stop and say so rather than shipping a
  fragile substitute.

## Done when

Every brittle selector in the file is replaced or explicitly deferred with a reason, the affected specs
pass, each locator resolves to exactly one element, no XPath remains, and any application-side hooks
you need are listed for the user.
