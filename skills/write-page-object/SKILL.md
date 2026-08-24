---
name: write-page-object
description: Use when a spec needs a screen or component wrapper that does not exist yet — the class that owns a page's locators and exposes intent-named methods. Follows whatever model conventions .claude/test-profile.md recorded, including the repo's base class and locator priority. Use harden-locators instead when editing an existing model's selectors.
---

# Skill: write-page-object

Add one class that owns the locators for a screen and exposes what a user can *do* there.

The point of this layer is not encapsulation for its own sake. It is that when a screen changes, the
number of files that must change is one. Every design decision below follows from that, and any
decision that does not serve it is ceremony.

## Preconditions

1. `.claude/test-profile.md` exists and records a model layer. If the profile says `none`, this repo
   has no such layer — do not unilaterally introduce one. Ask; adding an architectural layer is the
   user's call.
2. You know which screen, and which interactions the calling spec needs. Build for the caller in
   front of you, not for imagined future callers.

## Procedure

### 1. Read the nearest existing model, and the base class

From the neighbour, record: the export style, the constructor signature, where locators are declared
and initialised, the visibility modifiers used, and the method naming pattern. From the base class,
record every member subclasses are obliged to implement and every helper they are expected to use
instead of raw calls.

Then check the model does not already exist under a different name. Two models for one screen is
worse than none, because a locator fix lands in only one of them.

### 2. Write the class

```
class <Screen> extends <the repo's base, if it has one> {
  // every locator declared here, with the repo's visibility modifier
  // every locator resolved in the constructor — not inside methods

  // whatever the base class obliges (a readiness check, a URL, a title)

  // action methods:  verb + noun, named for user intent
  // query methods:   get/count/has + noun, returning data
}
```

**Locators resolve in the constructor.** Not because construction time matters — Playwright locators
are lazy — but because it puts every selector for the screen in one visible block. A locator built
inside a method is a selector nobody finds when the DOM changes.

**Locator priority comes from the profile's observed habit, not from ideology.** In a repo whose
suite is role-based, a new CSS selector is the odd one out; in a repo that is uniformly test-id
driven, a role query is. Match the habit. What holds universally: prefer the attribute least likely
to change for cosmetic reasons, and never introduce XPath — see **harden-locators** for the
replacement patterns.

**Methods are named for intent, not mechanics.** `submitPayment()` survives the button becoming a
link; `clickSubmitButton()` becomes a lie. This is the difference between a model that absorbs UI
churn and one that merely relocates it.

**Methods return what the caller needs to assert on.** A query method returns the value. Assertions
about the screen's own integrity — that it loaded, that it is in a valid state — belong in the model.
Assertions about the *behaviour under test* belong in the spec, where a reader can see what the test
claims.

### 3. Keep it thin, on purpose

Four things do not belong in this class:

- **Scenario branching.** An `if` deciding between two test paths puts the test logic somewhere the
  test does not read.
- **Data generation.** Accept values as parameters. A model that invents its own data makes failures
  unreproducible.
- **Waits on durations.** Wait on the condition.
- **Cross-screen orchestration.** A method that navigates through three screens is a flow, and it
  belongs wherever the repo keeps flows — or in the spec.

### 4. Wire and verify

Register the model wherever the profile says models reach specs (**register-fixture**), then run one
spec that uses it. An unregistered model fails at resolution with an error that points at the spec
rather than at the missing registration, which wastes real time.

Run lint and typecheck if the profile lists them.

## Guardrails

- No public locators. Exposing them lets specs reach past every method you just wrote, and then the
  one-file-per-change property is gone.
- If the new screen is a specialisation of an existing one, extend that one.
- Do not add a method with no caller. Speculative methods are untested by definition.
- Do not paste in recorder output. Recorded selectors are positional and brittle; convert them.

## Done when

The class exists, satisfies its base class, exposes only methods, is registered so specs can reach
it, at least one spec exercises it green, and you have reported the path plus the methods you added.
