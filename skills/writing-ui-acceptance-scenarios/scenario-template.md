# <feature> — UI Acceptance Scenarios

**Source spec:** `docs/superpowers/specs/YYYY-MM-DD-<feature>-design.md`
**Source plan:** `docs/superpowers/plans/YYYY-MM-DD-<feature>.md`
**Status:** Draft
**Date:** YYYY-MM-DD

## Surface under test

<one paragraph naming the routes, components, or screens this feature exposes to users>

## Setup

Before scenario 1 can run, the runner needs:

- **Dev server URL:** <url>
- **Test account:** <credentials or "create a fresh test user via /signup">
- **Fixture data:** <seeds required, or "none">
- **Environment variables / feature flags:** <list, or "none">
- **External service dependencies:** <list, or "none">

If nothing beyond a running dev server is needed, state explicitly: "No setup beyond a running dev server at <url>."

## Scenarios

### Scenario 1: <primary user flow, stated as user intent>

Given <preconditions>
  And <additional preconditions if any>

When <user action>
  And <additional actions in order>

Then <observable outcome>
  And <additional outcomes>

### Scenario 2: <covers acceptance criterion 1 from spec>

Given ...

When ...

Then ...

<repeat one scenario per acceptance criterion from the spec>

## Hard fail conditions

These apply to every scenario. If any triggers, the scenario fails regardless of `Then` clauses.

- Any HTTP 5xx during the flow
- Browser console errors of severity `error`
- Components mount but render empty / undefined
- Auth or session state breaks unexpectedly

<plus any feature-specific fails inferred from spec, plan, or supplied by the human>

## Pass criteria

The doc passes when:

- Every scenario's `Then` clauses are observed in the running app
- No hard fail condition triggered during any run

Pass/fail is binary per scenario. There are no partial passes.

## Reporting failures

Log each run in the sibling results file at `docs/superpowers/ui-acceptance-scenarios/YYYY-MM-DD-<feature>-results.md`. For each scenario, record:

- Date and time of the run
- Scenario number and title
- Pass / Fail
- For failures: observed behaviour, screenshots, relevant network or console output

If a regression is found, hand back to the development agent with the path to the results file.
