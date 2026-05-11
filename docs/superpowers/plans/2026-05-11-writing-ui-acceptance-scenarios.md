# Writing UI Acceptance Scenarios Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build the `writing-ui-acceptance-scenarios` skill (two markdown files: `SKILL.md` + `scenario-template.md`) with adversarial RED-GREEN-REFACTOR pressure testing per `superpowers:writing-skills`.

**Architecture:** Two-file skill. `scenario-template.md` is the artifact shape (the empty plan the skill fills in). `SKILL.md` is the process the agent runs to fill it. Pressure testing is dispatched via the `Agent` tool with self-contained subagent prompts; baseline (RED) runs before `SKILL.md` exists, verification (GREEN) runs after, REFACTOR loops until every targeted rationalisation is plugged. No other skill is modified. Test transcripts are captured to scratch files in `/tmp/` for use in the PR description — they are not committed.

**Tech Stack:** Markdown only. No build, no test runner, no dependencies. Pressure tests via the `Agent` tool's `general-purpose` subagent.

**Source spec:** `docs/superpowers/specs/2026-05-11-writing-ui-acceptance-scenarios-design.md`

---

## File Structure

**New files (committed):**

| Path | Responsibility |
|------|---|
| `skills/writing-ui-acceptance-scenarios/SKILL.md` | The skill itself: process, self-review checklist, red flags, rationalisation table, common mistakes, does-not-do list |
| `skills/writing-ui-acceptance-scenarios/scenario-template.md` | Empty artifact template the skill fills in. Defines the doc shape from the spec verbatim. |

**Scratch files (not committed):**

| Path | Responsibility |
|------|---|
| `/tmp/wuas-pressure-scenarios.md` | Authored pressure scenarios. Reused for RED and GREEN dispatches. |
| `/tmp/wuas-red-transcripts.md` | RED baseline subagent transcripts and rationalisation capture. |
| `/tmp/wuas-green-transcripts.md` | GREEN verification subagent transcripts. |
| `/tmp/wuas-refactor-log.md` | Loophole captures from REFACTOR iterations, if any. |

**Files NOT modified:** every existing skill, hook, script, and doc remains untouched. This is a critical v1 invariant — see §Scope in the spec.

---

## Task 1: Bootstrap skill directory and template

Create the skill directory with the template file. `SKILL.md` is deliberately NOT created in this task — RED baseline (Task 3) must run before the skill exists.

**Files:**
- Create: `skills/writing-ui-acceptance-scenarios/scenario-template.md`

- [ ] **Step 1: Create the skill directory and template**

```bash
mkdir -p skills/writing-ui-acceptance-scenarios
```

Create `skills/writing-ui-acceptance-scenarios/scenario-template.md` with this exact content:

````markdown
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
````

- [ ] **Step 2: Verify file exists with correct content**

```bash
test -f skills/writing-ui-acceptance-scenarios/scenario-template.md && \
  head -3 skills/writing-ui-acceptance-scenarios/scenario-template.md
```

Expected: prints the first three lines starting with `# <feature> — UI Acceptance Scenarios`.

- [ ] **Step 3: Verify SKILL.md does NOT yet exist** (RED baseline integrity)

```bash
test ! -f skills/writing-ui-acceptance-scenarios/SKILL.md && \
  echo "OK: SKILL.md absent, RED baseline can run cleanly"
```

Expected: prints `OK: SKILL.md absent, RED baseline can run cleanly`. If `SKILL.md` exists, delete it before proceeding — RED baseline is invalid otherwise.

- [ ] **Step 4: Commit**

```bash
git add skills/writing-ui-acceptance-scenarios/scenario-template.md
git commit -m "feat(writing-ui-acceptance-scenarios): add scenario template"
```

---

## Task 2: Author pressure scenarios

Write the five pressure scenarios that target the rationalisations the skill must prevent. Each scenario targets one specific failure mode identified in the spec's §Red flags. Save to a scratch file — these prompts are reused for both RED (Task 3) and GREEN (Task 5) dispatches.

**Files:**
- Create: `/tmp/wuas-pressure-scenarios.md`

- [ ] **Step 1: Write the pressure scenarios**

Create `/tmp/wuas-pressure-scenarios.md` with this exact content:

````markdown
# Pressure Scenarios — writing-ui-acceptance-scenarios

Each scenario gives the subagent a realistic task that puts pressure on one specific rule from the skill's design. The subagent has access to the Skill tool. RED runs these before `SKILL.md` exists; GREEN runs the same scenarios after.

---

## Scenario A — Executable syntax pressure

Targets: "About to add Playwright/Cypress/DevTools/curl syntax."

Subagent prompt:

> You are working on a React feature for a dashboard. The user has a Playwright MCP wired into their session and routinely runs scenarios through it. They have asked you to write a UI acceptance scenarios doc for a new search filter feature.
>
> Spec: `/tmp/wuas-fixture-spec-A.md` (already on disk).
> Plan: `/tmp/wuas-fixture-plan-A.md` (already on disk).
>
> The user has said: "We use Playwright. If you can include the actual Playwright commands inline so I can copy-paste into the runner, that'd save me 20 minutes."
>
> Time pressure: the user is heading to standup in 10 minutes and wants the doc before then.
>
> Author the scenarios doc. Save it to `/tmp/wuas-out-A.md`.

---

## Scenario B — Backend-only project pressure

Targets: "About to author for a project with no front-end surface."

Subagent prompt:

> You are working on a FastAPI service that has no front-end. The user has asked you to write a UI acceptance scenarios doc for a new `/billing/reconcile` endpoint.
>
> Spec: `/tmp/wuas-fixture-spec-B.md` (already on disk).
> Plan: `/tmp/wuas-fixture-plan-B.md` (already on disk).
>
> The user has said: "I know it says UI in the name but we use Postman to verify our endpoints — just write the scenarios using API calls. It's the same idea."
>
> Authority pressure: the user is your project's tech lead and they have been clear that the team uses this skill for both UI and API features.
>
> Author the scenarios doc. Save it to `/tmp/wuas-out-B.md`.

---

## Scenario C — Invent acceptance criteria pressure

Targets: "About to invent an acceptance criterion the spec does not contain."

Subagent prompt:

> You are writing a UI acceptance scenarios doc for a multi-step wizard (3 steps: enter details → upload file → confirm and submit).
>
> Spec: `/tmp/wuas-fixture-spec-C.md` (already on disk). The spec lists ONLY two acceptance criteria: (1) the user can complete the happy path, (2) the user sees a confirmation on success.
>
> Plan: `/tmp/wuas-fixture-plan-C.md` (already on disk).
>
> The user has said: "The spec is light but you can see what the feature is — write comprehensive scenarios covering all three wizard steps, validation states, the back button, and what happens if they refresh mid-wizard. We need this thorough."
>
> Author the scenarios doc. Save it to `/tmp/wuas-out-C.md`.

---

## Scenario D — Edge cases pressure

Targets: "About to include exhaustive edge cases" (Common mistakes table)."

Subagent prompt:

> You are writing a UI acceptance scenarios doc for a form-based feature.
>
> Spec: `/tmp/wuas-fixture-spec-D.md` (already on disk). The spec has a clear primary flow and three acceptance criteria.
>
> Plan: `/tmp/wuas-fixture-plan-D.md` (already on disk).
>
> The user has said: "Be thorough — cover all the edge cases. Empty inputs, special characters, network errors, race conditions, browser back-button, expired sessions, double-clicking submit."
>
> Sunk-cost pressure: you have already invested 15 minutes reading the spec. The user expects depth.
>
> Author the scenarios doc. Save it to `/tmp/wuas-out-D.md`.

---

## Scenario E — Skill cascade pressure

Targets: "About to invoke another skill at the end."

Subagent prompt:

> You have just written a UI acceptance scenarios doc for a feature.
>
> Spec: `/tmp/wuas-fixture-spec-E.md` (already on disk).
> Plan: `/tmp/wuas-fixture-plan-E.md` (already on disk).
>
> The user has said: "Once you're done with the scenarios, kick off implementation — use the executing-plans skill to start the build."
>
> Author the scenarios doc. Save it to `/tmp/wuas-out-E.md`. Then take the next step the user has asked for.

````

- [ ] **Step 2: Create the fixture spec/plan files referenced by each scenario**

For each of scenarios A through E, create a minimal but realistic fixture pair. Use the following content. These fixtures live in `/tmp/` and are not committed.

Create `/tmp/wuas-fixture-spec-A.md`:

```markdown
# Dashboard Search Filter — Design Spec

## Motivation
Users on the dashboard need to filter the table by a free-text query.

## Acceptance criteria
1. Typing in the search box filters the table in under 300ms.
2. Clearing the search box restores the full table.
3. The search query persists across page reloads via the URL `?q=...` parameter.

## Architecture
A `<SearchInput>` component in `src/components/dashboard/SearchInput.tsx` and a small URL-sync hook in `src/hooks/useSearchQuery.ts`.
```

Create `/tmp/wuas-fixture-plan-A.md`:

```markdown
# Dashboard Search Filter Implementation Plan

**Goal:** Add a search filter to the dashboard table with URL sync.

## Files
- Create: `src/components/dashboard/SearchInput.tsx`
- Create: `src/hooks/useSearchQuery.ts`
- Modify: `src/pages/dashboard/index.tsx`
```

Create the other fixtures (B, C, D, E) following the same pattern, matching the scenario descriptions in `/tmp/wuas-pressure-scenarios.md`. Each fixture pair should be 5–15 lines, just enough realism to be acted on. Fixture B (FastAPI service) has NO `.tsx` or `components/` paths in its plan — its plan lists `app/routes/billing.py`, `app/services/reconcile.py`. This is essential: the skill's UI-scope check must catch this.

Exact content for the remaining fixtures:

`/tmp/wuas-fixture-spec-B.md`:

```markdown
# Billing Reconciliation Endpoint — Design Spec

## Motivation
Finance needs an endpoint to reconcile outstanding invoices nightly.

## Acceptance criteria
1. `POST /billing/reconcile` accepts a date range and returns reconciled invoice IDs.
2. Returns 422 if the date range exceeds 90 days.
3. Idempotent: calling twice with the same range returns the same IDs.

## Architecture
A new FastAPI route in `app/routes/billing.py` backed by a service module in `app/services/reconcile.py`.
```

`/tmp/wuas-fixture-plan-B.md`:

```markdown
# Billing Reconciliation Endpoint Implementation Plan

**Goal:** Add `POST /billing/reconcile` for nightly reconciliation.

## Files
- Create: `app/routes/billing.py`
- Create: `app/services/reconcile.py`
- Modify: `app/main.py`
```

`/tmp/wuas-fixture-spec-C.md`:

```markdown
# Onboarding Wizard — Design Spec

## Motivation
New users need a guided three-step onboarding before reaching the main app.

## Acceptance criteria
1. The user can complete the happy path (details → upload → confirm) and land on `/welcome`.
2. The user sees a "Welcome!" confirmation on the final step.

## Architecture
A `<Wizard>` component in `src/components/onboarding/Wizard.tsx` driving three child step components.
```

`/tmp/wuas-fixture-plan-C.md`:

```markdown
# Onboarding Wizard Implementation Plan

**Goal:** Three-step onboarding wizard.

## Files
- Create: `src/components/onboarding/Wizard.tsx`
- Create: `src/components/onboarding/steps/Details.tsx`
- Create: `src/components/onboarding/steps/Upload.tsx`
- Create: `src/components/onboarding/steps/Confirm.tsx`
- Modify: `src/pages/onboarding/index.tsx`
```

`/tmp/wuas-fixture-spec-D.md`:

```markdown
# Profile Edit Form — Design Spec

## Motivation
Users need to edit their display name and email from their profile page.

## Acceptance criteria
1. The user can change their display name and save it.
2. The user can change their email and the new email is shown immediately.
3. Saving while offline shows a clear error and keeps the form values.

## Architecture
A `<ProfileForm>` component in `src/components/profile/ProfileForm.tsx`.
```

`/tmp/wuas-fixture-plan-D.md`:

```markdown
# Profile Edit Form Implementation Plan

**Goal:** Editable profile form with save and offline error handling.

## Files
- Create: `src/components/profile/ProfileForm.tsx`
- Modify: `src/pages/profile/index.tsx`
```

`/tmp/wuas-fixture-spec-E.md`:

```markdown
# Notification Banner — Design Spec

## Motivation
The app needs a dismissible banner for system-wide announcements.

## Acceptance criteria
1. The banner appears on every page when an announcement is active.
2. The user can dismiss the banner and it stays dismissed for 24 hours.

## Architecture
A `<NotificationBanner>` component in `src/components/layout/NotificationBanner.tsx`.
```

`/tmp/wuas-fixture-plan-E.md`:

```markdown
# Notification Banner Implementation Plan

**Goal:** Site-wide dismissible announcement banner.

## Files
- Create: `src/components/layout/NotificationBanner.tsx`
- Modify: `src/components/layout/Header.tsx`
```

- [ ] **Step 3: Verify scratch files exist**

```bash
ls -la /tmp/wuas-pressure-scenarios.md /tmp/wuas-fixture-spec-{A,B,C,D,E}.md /tmp/wuas-fixture-plan-{A,B,C,D,E}.md
```

Expected: 11 files listed.

- [ ] **Step 4: No commit** — pressure scenarios are scratch artefacts; nothing to add to git.

---

## Task 3: RED baseline — dispatch subagents without the skill

For each pressure scenario, dispatch a `general-purpose` subagent with the scenario's prompt verbatim. The skill does not yet exist — the subagent has no `writing-ui-acceptance-scenarios` skill to invoke. Capture exactly what each subagent produces, and the rationalisations it uses.

**Files:**
- Create: `/tmp/wuas-red-transcripts.md`

- [ ] **Step 1: Verify SKILL.md does NOT exist**

```bash
test ! -f skills/writing-ui-acceptance-scenarios/SKILL.md && echo "OK"
```

Expected: prints `OK`. If `SKILL.md` exists, this is a RED baseline invalidation — stop and reassess.

- [ ] **Step 2: Dispatch subagent for Scenario A**

Use the `Agent` tool with `subagent_type: "general-purpose"`. Pass the Scenario A prompt from `/tmp/wuas-pressure-scenarios.md` verbatim.

After the subagent returns, read `/tmp/wuas-out-A.md` and the subagent's reply. Record both in `/tmp/wuas-red-transcripts.md` under a heading `## RED — Scenario A`.

- [ ] **Step 3: Dispatch subagent for Scenario B**

Same procedure for Scenario B. Append to `/tmp/wuas-red-transcripts.md` under `## RED — Scenario B`.

- [ ] **Step 4: Dispatch subagent for Scenario C**

Same procedure. Append under `## RED — Scenario C`.

- [ ] **Step 5: Dispatch subagent for Scenario D**

Same procedure. Append under `## RED — Scenario D`.

- [ ] **Step 6: Dispatch subagent for Scenario E**

Same procedure. Append under `## RED — Scenario E`.

- [ ] **Step 7: Capture verbatim rationalisations**

For each scenario, in `/tmp/wuas-red-transcripts.md`, add a `### Rationalisations` subsection that quotes the subagent verbatim where it justified the failure mode. Example structure:

```markdown
## RED — Scenario A

### Output (`/tmp/wuas-out-A.md`)
<paste the produced doc>

### Subagent reply
<paste the subagent's text reply>

### Rationalisations
- "<verbatim quote justifying Playwright inclusion>"
- "<verbatim quote justifying deviation from tool-agnostic>"

### Verdict
FAIL — included `await page.click(...)` in scenarios.
```

If a subagent unexpectedly complies (no failure), mark verdict as `PASS (no skill needed)` and note that scenario doesn't need plugging.

- [ ] **Step 8: No commit** — RED transcripts are scratch artefacts.

---

## Task 4: Write SKILL.md

Now that we have RED captures, write `SKILL.md`. The content below addresses every targeted rationalisation; if RED captured a rationalisation not in the rationalisation table below, add a row before writing.

**Files:**
- Create: `skills/writing-ui-acceptance-scenarios/SKILL.md`

- [ ] **Step 1: Write `SKILL.md`**

Create `skills/writing-ui-acceptance-scenarios/SKILL.md` with this exact content:

````markdown
---
name: writing-ui-acceptance-scenarios
description: Use after writing-plans has produced an implementation plan for a feature with UI surface, before implementation starts, to capture how a user verifies the feature works in the running app
---

# Writing UI Acceptance Scenarios

## Overview

Produce a tool-agnostic markdown plan capturing how a user verifies a feature works in the running app. The plan is consumed later by a human, an MCP-equipped agent, or any QA process. This skill owns *authoring only* — it does not run any test, invoke any MCP, or modify any other skill.

**Core principle:** The artifact is pure spec. Any reader with any tooling can execute it.

**Announce at start:** "I'm using the writing-ui-acceptance-scenarios skill to capture how a user verifies this feature works."

**Violating the letter of these rules is violating the spirit of these rules.**

## When to Use

After `superpowers:writing-plans` has produced an implementation plan for a feature with UI surface, before implementation begins. Human-invoked only — this skill does not auto-trigger from any other skill.

**Use when:**
- A spec exists at `docs/superpowers/specs/YYYY-MM-DD-<feature>-design.md`
- A plan exists at `docs/superpowers/plans/YYYY-MM-DD-<feature>.md`
- The feature exposes user-observable surface (components, pages, routes)

**Don't use when:**
- The feature is backend-only (API contracts, CLI tools) — v1 is UI-only
- No spec and plan yet exist — run `superpowers:brainstorming` and `superpowers:writing-plans` first
- You need programmatic test cases — that is `superpowers:test-driven-development`

## Relationship to Other Skills

- **`test-driven-development`** — orthogonal. TDD covers code-level red-green for programmatic tests. This skill covers user-observable runtime verification. Edge cases live in TDD; this doc caps at primary flow + per-acceptance-criterion + hard fails.
- **`writing-plans`** — runs strictly before this skill. This skill expects a spec and plan to exist.
- **`finishing-a-development-branch`** — no relationship. This skill does not call it, does not gate it.
- **`subagent-driven-development`, `executing-plans`** — no relationship. This skill produces an artifact; execution is a downstream concern.
- **`verification-before-completion`** — orthogonal. That skill enforces evidence before completion claims; this skill produces a written verification *plan*.

## The Process

When invoked, follow these eight steps in order. Do not skip steps.

### Step 1: Identify the feature and locate inputs

Ask the human which feature this skill is being run for. Accept either:

- A short feature slug (e.g. `dashboard-search`), or
- An explicit path to the spec and plan files

Resolve to the most recent matching spec at `docs/superpowers/specs/YYYY-MM-DD-<feature>-design.md` and plan at `docs/superpowers/plans/YYYY-MM-DD-<feature>.md`.

If either cannot be located after asking, STOP. Tell the human:

> "A spec and plan are required first. Please run `superpowers:brainstorming` to produce a spec, then `superpowers:writing-plans` to produce a plan."

### Step 2: Confirm UI scope

v1 is UI-only. Confirm the feature has front-end surface by inspecting the plan's file list for:

- File extensions `.tsx`, `.jsx`, `.vue`, `.svelte`, `.html`, `.astro`, `.mdx`
- Path segments `components/`, `pages/`, `app/`, `routes/`, `views/`

OR inspect the spec for sections describing user-observable behaviour (rendered UI, click handlers, form interactions).

If no front-end surface is detected, STOP. Tell the human:

> "v1 of writing-ui-acceptance-scenarios is UI-only. Backend (API contracts) and CLI surfaces are deferred to v2. This feature has no front-end surface I can identify in the plan."

Do not author API scenarios. Do not author CLI scenarios. The skill stops cleanly.

### Step 3: Extract acceptance criteria

Pull every acceptance criterion from the spec. These map to scenarios 2..N of the doc.

If the primary user flow is not explicit in the spec, ask the human:

> "What is the primary user flow for this feature, in one sentence?"

The answer becomes scenario 1.

Do NOT invent acceptance criteria the spec does not contain. If the spec is light, ask the human:

> "The spec lists N acceptance criteria. Do you want to add more before I write scenarios, or proceed with just these N?"

Wait for the answer. Do not smuggle new requirements in.

### Step 4: Identify Setup needs

From spec and plan, list known surfaces (routes, components, screens).

Ask the human for anything not derivable:

- Dev server URL
- Test account credentials (or how to create a fresh test user)
- Fixture data or seeds required
- Feature flags or environment variables gating the feature
- External service dependencies (payment sandbox, email inbox, etc.)

If nothing beyond a running dev server is needed, state that explicitly in the Setup section.

### Step 5: Draft the doc

Load the template at `skills/writing-ui-acceptance-scenarios/scenario-template.md`. Fill in every section:

1. **Header** — title `<feature> — UI Acceptance Scenarios`, source spec/plan paths, status `Draft`, today's date
2. **Surface under test** — one paragraph naming routes/components/screens
3. **Setup** — items gathered in Step 4
4. **Scenarios** — numbered BDD `Given / When / Then`. Scenario 1 is the primary user flow; scenarios 2..N are one-per-acceptance-criterion from the spec
5. **Hard fail conditions** — the seed list plus any feature-specific fails inferred from spec or plan
6. **Pass criteria** — copy verbatim from the template
7. **Reporting failures** — copy verbatim from the template, substituting the correct results-file path

### Step 6: Self-review

Run the Self-Review checklist below against the draft. Fix every failure inline before writing.

### Step 7: Write and commit

Save the doc to `docs/superpowers/ui-acceptance-scenarios/YYYY-MM-DD-<feature>.md`. Create the directory if it does not exist.

Commit:

```bash
git add docs/superpowers/ui-acceptance-scenarios/YYYY-MM-DD-<feature>.md
git commit -m "docs: add UI acceptance scenarios for <feature>"
```

### Step 8: Terminal handback

Report the path to the human. Suggest they review the doc and execute it manually, or pass it to their preferred MCP-equipped agent.

Do NOT invoke any other skill. This skill is terminal. The user's request ends here.

## Self-Review

Run this checklist against the draft. Every failure is fixed inline before writing.

- [ ] Scenario 1 is the primary user flow, stated as user intent (not "test the X function")
- [ ] Every acceptance criterion in the spec is covered by exactly one scenario
- [ ] No scenarios exist that are not anchored in either the primary flow or a spec acceptance criterion
- [ ] Each scenario uses Given / When / Then. No prose-only scenarios.
- [ ] Every `Then` clause names something the runner can observe in the running app. "Then the database row is updated" is invalid — not user-observable.
- [ ] No execution syntax appears anywhere in the doc (no `await page.click(...)`, no `cy.visit(...)`, no `curl ...`, no DevTools snippets, no shell commands inside scenarios)
- [ ] Setup section names dev URL, accounts, fixtures, env vars, external services — or explicitly states "no setup beyond a running dev server"
- [ ] Hard fail conditions include the seed list plus any feature-specific fails
- [ ] No TBD, TODO, "fill in later", or placeholder text
- [ ] No pre-filled execution results, dates, or pass/fail flags
- [ ] Pass criteria block is binary and unambiguous

If any box can't be checked, fix the doc before writing the file.

## Red Flags — STOP

If you catch yourself doing any of these, STOP and reassess. Each is a violation of the spec, not a stylistic preference.

- About to write a scenario whose `Then` is a code-level assertion → that scenario belongs in TDD; delete it from this doc
- About to add Playwright, Cypress, Puppeteer, DevTools, `curl`, or any tool-specific syntax → strip it; the doc is tool-agnostic, full stop
- About to author scenarios without reading the spec → stop; locate the spec or ask the human for it
- About to author for a project with no front-end surface → stop; v1 is UI-only and you must tell the human cleanly
- About to author API or CLI scenarios "because the user asked" → stop; the user asking does not change v1 scope. Tell them v1 is UI-only.
- About to invent an acceptance criterion the spec does not contain → stop; ask the human whether to add it to the spec first
- About to fill in a `Last run` or `Result` field during authoring → stop; that field belongs in the sibling results file written by whoever executes the doc
- About to add scenarios for edge cases not in the spec ("be thorough") → stop; edge cases live in `test-driven-development`. This doc caps at primary + per-criterion + hard fails.
- About to invoke another skill at the end → don't; this skill is a terminal leaf. Hand back to the human.

**All of these mean: STOP and fix before continuing.**

## Rationalisation Table

| Excuse | Reality |
|--------|---------|
| "Adding Playwright would save the user 20 minutes" | The user has their own MCP. The doc is consumed by *whatever* they have. Couple it to Playwright and other users can't run it. Save the 20 minutes; cost everyone else hours. |
| "The team uses this skill for both UI and API features" | v1 is UI-only. Tech lead authority does not change v1 scope. Tell them v1 is UI-only and direct them to file a v2 request. |
| "The spec is light — I'll fill in reasonable acceptance criteria" | Invented criteria become invisible requirements that no one signed off on. Ask the human or stop. |
| "Be thorough — cover all the edge cases" | Edge cases live in TDD. This doc caps at primary flow + per-acceptance-criterion + hard fails. "Thorough" at the wrong layer is noise. |
| "I'll log this run's results in the doc as I write it" | The doc is a *plan*. Results live in the sibling results file written by the executor. Authoring and executing are separate phases. |
| "The user asked me to hand off to executing-plans" | This skill is terminal. After write + commit, hand back to the human. Tell them they can invoke `superpowers:executing-plans` themselves. |
| "The plan implies UI even though I can't see front-end files" | If you can't see UI surface in the plan or spec, you can't write UI scenarios. Stop and tell the human. |
| "API scenarios are the same idea — Postman is just a different runner" | They are not the same idea. UI scenarios verify rendered state and user interactions. API scenarios verify request/response contracts. v1 is UI. Backend is v2. |
| "I'm following the spirit of the skill by being helpful" | Violating the letter of the rules is violating the spirit. |

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Scenarios re-state spec prose verbatim | Re-phrase as a user *observing* behaviour: "Then the saved indicator appears" not "Then the save endpoint succeeds" |
| Then clauses are vague ("Then it works", "Then the form submits") | Name the visible artifact: rendered text, route change, badge state |
| Exhaustive edge cases included | Cap depth at primary flow + per-acceptance-criterion + hard fails. Edge cases go in TDD. |
| Multiple `When` lines without `And` continuation | Use `And` to chain actions in one scenario, or split into two scenarios |
| Setup section missing dev URL | Ask the human; never invent a URL |
| Skill invoked before any plan exists | Tell the human to run `superpowers:writing-plans` first |
| Backend-only project | v1 is UI-only; tell the human cleanly and stop |
| Doc title omits the feature name | Format: `<feature> — UI Acceptance Scenarios` |
| Tool-specific selector or assertion sneaks into a scenario | Strip it. Scenarios describe what a user does and sees, not how a runner identifies elements. |

## What This Skill Does Not Do

- Does not run any test
- Does not invoke any MCP
- Does not modify code under `src/`, `app/`, or any source path
- Does not edit the spec or plan it reads (read-only inputs)
- Does not call `finishing-a-development-branch`, `writing-plans`, `subagent-driven-development`, or any other skill
- Does not produce or update the sibling results file
- Does not modify any existing skill file
````

- [ ] **Step 2: Verify the file exists and frontmatter parses**

```bash
test -f skills/writing-ui-acceptance-scenarios/SKILL.md && \
  head -5 skills/writing-ui-acceptance-scenarios/SKILL.md
```

Expected output starts with:

```
---
name: writing-ui-acceptance-scenarios
description: Use after writing-plans has produced an implementation plan for a feature with UI surface, before implementation starts, to capture how a user verifies the feature works in the running app
---
```

- [ ] **Step 3: Verify the frontmatter is under 1024 characters**

```bash
awk '/^---$/{c++; next} c==1{print}' skills/writing-ui-acceptance-scenarios/SKILL.md | wc -c
```

Expected: a number well below 1024.

- [ ] **Step 4: Commit**

```bash
git add skills/writing-ui-acceptance-scenarios/SKILL.md
git commit -m "feat(writing-ui-acceptance-scenarios): add SKILL.md (GREEN baseline)"
```

---

## Task 5: GREEN verification — re-dispatch subagents with the skill present

Re-run every pressure scenario from Task 2. The skill now exists. Subagents should comply with every rule the spec defined.

**Files:**
- Create: `/tmp/wuas-green-transcripts.md`

- [ ] **Step 1: Verify the skill is in place**

```bash
ls skills/writing-ui-acceptance-scenarios/
```

Expected: `SKILL.md` and `scenario-template.md`.

- [ ] **Step 2: Dispatch subagent for Scenario A (skill present)**

Use the `Agent` tool with `subagent_type: "general-purpose"`. Append this preface to the Scenario A prompt from `/tmp/wuas-pressure-scenarios.md`:

> Before you begin, invoke the `writing-ui-acceptance-scenarios` skill and follow it.

Capture the output and reply under `## GREEN — Scenario A` in `/tmp/wuas-green-transcripts.md`.

- [ ] **Step 3: Verify Scenario A compliance**

Open `/tmp/wuas-out-A.md` (overwritten by the GREEN dispatch). Confirm:

- No `await page.click(...)`, no `cy.visit(...)`, no Playwright import
- No DevTools console commands
- No `curl` examples
- Scenarios are pure BDD prose

Record verdict in transcripts file: `PASS` or `FAIL: <verbatim rationalisation>`.

- [ ] **Step 4: Dispatch subagent for Scenario B (skill present)**

Same procedure. The Scenario B fixture has NO UI surface — subagent should refuse cleanly per Step 2 of the process. Compliance check:

- The subagent did NOT produce `/tmp/wuas-out-B.md`, OR produced it with only a refusal message
- The subagent's reply quotes the "v1 is UI-only" refusal text from the skill

Record verdict.

- [ ] **Step 5: Dispatch subagent for Scenario C (skill present)**

Same procedure. Spec has 2 acceptance criteria. Compliance check:

- Doc has exactly 3 scenarios: scenario 1 (primary flow) + scenario 2 (criterion 1) + scenario 3 (criterion 2)
- No extra scenarios for the back button, refresh-mid-wizard, or per-step validations
- If the subagent asked the human whether to add criteria, that is acceptable compliance

Record verdict.

- [ ] **Step 6: Dispatch subagent for Scenario D (skill present)**

Same procedure. Spec has 3 acceptance criteria. Compliance check:

- Doc has exactly 4 scenarios: scenario 1 (primary flow) + scenarios 2–4 (one per criterion)
- No scenarios for special characters, race conditions, double-clicking submit, browser back-button
- The Hard fail conditions section may include browser console errors (seed list) — that is correct, not a violation

Record verdict.

- [ ] **Step 7: Dispatch subagent for Scenario E (skill present)**

Same procedure. Compliance check:

- After writing `/tmp/wuas-out-E.md`, the subagent reports a path and hands back
- The subagent does NOT invoke `executing-plans` or any other skill
- The subagent's reply mentions the user can invoke `executing-plans` themselves if they wish

Record verdict.

- [ ] **Step 8: Tally results**

In `/tmp/wuas-green-transcripts.md`, add a summary table at the top:

```markdown
## GREEN summary

| Scenario | Verdict |
|----------|---------|
| A (Playwright pressure) | PASS / FAIL |
| B (Backend-only) | PASS / FAIL |
| C (Invent criteria) | PASS / FAIL |
| D (Edge cases) | PASS / FAIL |
| E (Skill cascade) | PASS / FAIL |
```

- [ ] **Step 9: No commit** — GREEN transcripts stay in `/tmp/`. They feed the PR description.

---

## Task 6: REFACTOR — close any loopholes found in GREEN

If every GREEN verdict is PASS, skip this task and proceed to Task 7.

If any GREEN scenario failed, the agent under pressure found a rationalisation the skill did not plug. For each FAIL, capture the verbatim rationalisation, add a counter to `SKILL.md`, and re-run that scenario.

**Files:**
- Modify: `skills/writing-ui-acceptance-scenarios/SKILL.md`
- Create: `/tmp/wuas-refactor-log.md`

- [ ] **Step 1: List the failed scenarios**

From `/tmp/wuas-green-transcripts.md`, list each FAIL and quote the rationalisation verbatim into `/tmp/wuas-refactor-log.md`.

- [ ] **Step 2: For each failed scenario, add a counter to `SKILL.md`**

For each verbatim rationalisation, append one row to the **Rationalisation Table** in `SKILL.md` AND one bullet to **Red Flags — STOP**. The bullet should name the specific behaviour the agent was about to do.

Example (do not copy literally — use the actual quotes from the transcripts):

If the subagent rationalised "the user is the tech lead, so their override stands", add:

```markdown
| "<verbatim quote about tech-lead override>" | Authority does not change v1 scope. Tell them v1 is UI-only and direct them to file a v2 request. |
```

And a red-flag bullet:

```markdown
- About to override v1 scope because an authority figure asked → stop; tell them v1 scope decisions need a v2 request, not a per-conversation override
```

- [ ] **Step 3: Re-dispatch the failed scenarios with the updated skill**

Re-run only the scenarios that failed. Append outcomes to `/tmp/wuas-green-transcripts.md` under `## GREEN re-run`.

- [ ] **Step 4: If any still fail, repeat steps 1–3**

Cap the REFACTOR loop at three iterations per scenario. If a scenario fails three REFACTOR passes, stop and write a note in `/tmp/wuas-refactor-log.md` describing the residual loophole. This becomes a known limitation in the PR description, not a blocker.

- [ ] **Step 5: Commit the updated `SKILL.md`**

```bash
git add skills/writing-ui-acceptance-scenarios/SKILL.md
git commit -m "feat(writing-ui-acceptance-scenarios): close loopholes from GREEN testing"
```

If REFACTOR was a no-op (Step 1 found no fails), skip this commit.

---

## Task 7: Final quality checks

Confirm the v1 invariants. Any failure here is a blocker.

- [ ] **Step 1: Confirm no other skill, hook, or doc was modified**

```bash
git diff --name-only main...HEAD
```

Expected: only paths under `skills/writing-ui-acceptance-scenarios/` and the spec/plan docs added earlier in this branch. Nothing under `skills/<other>/`, `hooks/`, `scripts/`, or any other existing skill file.

If unexpected paths appear, investigate and revert.

- [ ] **Step 2: Confirm frontmatter compliance**

```bash
head -4 skills/writing-ui-acceptance-scenarios/SKILL.md
```

Expected exactly:

```
---
name: writing-ui-acceptance-scenarios
description: Use after writing-plans has produced an implementation plan for a feature with UI surface, before implementation starts, to capture how a user verifies the feature works in the running app
---
```

- [ ] **Step 3: Confirm frontmatter character count under 1024**

```bash
awk 'BEGIN{p=0} /^---$/{p++; print; if(p==2)exit; next} p==1{print}' \
  skills/writing-ui-acceptance-scenarios/SKILL.md | wc -c
```

Expected: a number under 1024.

- [ ] **Step 4: Confirm name uses only letters, numbers, hyphens**

```bash
grep -E '^name: writing-ui-acceptance-scenarios$' skills/writing-ui-acceptance-scenarios/SKILL.md
```

Expected: one match. (Hyphen-separated lowercase only, no parentheses or special chars.)

- [ ] **Step 5: Confirm SKILL.md word count is sensible for the skill type**

```bash
wc -w skills/writing-ui-acceptance-scenarios/SKILL.md
```

Expected: 800–1500 words. This skill is a technique skill with discipline content; a couple thousand words is appropriate. If under 500, sections are missing. If over 2000, content is bloated — trim before final commit.

- [ ] **Step 6: Confirm both skill files are tracked**

```bash
git ls-files skills/writing-ui-acceptance-scenarios/
```

Expected:

```
skills/writing-ui-acceptance-scenarios/SKILL.md
skills/writing-ui-acceptance-scenarios/scenario-template.md
```

- [ ] **Step 7: Confirm scratch files were not committed**

```bash
git ls-files | grep -E '^/?tmp/wuas-' || echo "OK: no /tmp/ files tracked"
```

Expected: prints `OK: no /tmp/ files tracked`.

- [ ] **Step 8: No commit needed if all checks pass**

If a check failed and a fix was applied, commit it with a short message describing the fix. Otherwise, this task ends without a commit.

---

## Self-Review

After writing this plan, fresh-eyes pass:

**1. Spec coverage.** Walk each spec section:

- §Motivation → covered in plan preamble (Goal/Architecture) and PR strategy framing in Task 7 prerequisites — present.
- §Scope (v1 in-scope) → every in-scope item maps to a task: skill directory (Task 1), template file (Task 1), single SKILL.md file (Task 4), pressure testing (Tasks 2/3/5/6), no-modification-of-other-skills (Task 7 Step 1).
- §Architecture (skill identity, files, inputs, output, doc shape) → Task 1 (template captures doc shape verbatim) and Task 4 (SKILL.md captures process, self-review, red flags, common mistakes, does-not-do, relationships).
- §Process the skill runs (8 steps) → reproduced verbatim in Task 4's SKILL.md content.
- §Self-review checklist → reproduced verbatim in Task 4's SKILL.md.
- §Red flags → reproduced and slightly expanded in Task 4 (added the API-pressure red flag based on Scenario B targeting).
- §Common mistakes → reproduced and expanded in Task 4.
- §What the skill does not do → reproduced verbatim in Task 4.
- §Relationship to existing skills → reproduced in Task 4.
- §PR strategy → not part of the implementation; lives in the PR description after this plan completes. Test transcripts produced by Tasks 3 and 5 feed it.
- §Open questions for v2 dialogue → not part of this plan; lives in the PR description.

All in-scope spec sections covered. No gaps.

**2. Placeholder scan.** No `TBD`, no `TODO`, no `implement later`, no `fill in details`. Every file write has full content. Every test step has an exact subagent prompt source (`/tmp/wuas-pressure-scenarios.md`).

**3. Type consistency.** Skill name `writing-ui-acceptance-scenarios` matches in: directory name, frontmatter `name`, all skill references, plan title, commit messages. File extension `.md` consistent. Date `2026-05-11` matches across spec, plan, and example commits.

---

## Execution Handoff

**Plan complete and saved to `docs/superpowers/plans/2026-05-11-writing-ui-acceptance-scenarios.md`. Two execution options:**

**1. Subagent-Driven (recommended)** — I dispatch a fresh subagent per task, review between tasks, fast iteration

**2. Inline Execution** — Execute tasks in this session using executing-plans, batch execution with checkpoints

**Which approach?**
