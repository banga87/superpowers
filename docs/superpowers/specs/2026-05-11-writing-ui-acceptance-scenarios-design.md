# Writing UI Acceptance Scenarios — Design Spec

A new core superpowers skill, `writing-ui-acceptance-scenarios`, that produces a zero-dependency markdown artifact capturing how a user verifies a feature works in the running app. The doc lives parallel to specs and plans and can be picked up later by any human or MCP-equipped agent to drive a runtime walkthrough.

This skill is an MVP. Its purpose is to prove that a markdown-only artifact can fill the runtime-verification gap without violating superpowers' zero-dependency policy. Submission to upstream is the first step of a deliberate dialogue with core maintainers; broader follow-on capabilities are explicitly out of scope until that dialogue concludes.

## Motivation

Three open superpowers issues converge on the same gap:

- **Issue #1027** (Mori, "Separated browser-based QA evaluator skill") — a four-file design with a Playwright/Chrome-DevTools MCP-driven QA subagent. Cannot land in core because it depends on an external MCP server, which violates superpowers' zero-dependency policy.
- **Issue #374** (SamuelMiller, "Add E2E browser testing for web app development") — requests BDD-style end-to-end browser verification by default.
- **PR #946** (chickencyj) — narrower attempt: a hard smoke-check gate inside `finishing-a-development-branch` that forces the agent to walk the primary user flow before merge options appear. Open, unmerged.

The empirical case for separated runtime verification is in Anthropic's *Harness design for long-running application development* (March 2026), which observed that agents reliably skew positive when evaluating their own code, and that a separated evaluator catches integration bugs invisible to spec compliance and code-quality review.

The structural problem all three prior attempts share: any solution that *runs* a browser depends on an MCP server, and core cannot ship that dependency. PR #946 sidesteps the dependency by making the agent start a dev server and `curl` it, but its `curl`-based smoke check cannot exercise UI behaviour (rendered state, click handlers, post-interaction DOM updates).

This spec proposes a different cut at the problem: produce a tool-agnostic markdown plan during the planning phase. The plan describes scenarios in BDD form with no execution syntax. Whoever executes it later — a human in their own browser, an MCP-equipped plugin agent, a contractor running QA — picks it up and uses whatever tools they have. Core stays zero-dep. Plugins and humans compose on top.

## Scope (v1)

This spec describes only what ships in v1. Every decision below is deliberate; the corresponding v2 capability is listed in §Deferred so the PR description can use the spec as a dialogue artifact with maintainers.

**In scope:**

- A new skill at `skills/writing-ui-acceptance-scenarios/`.
- A scenario template file the skill writes from.
- The skill produces a single markdown plan at `docs/superpowers/ui-acceptance-scenarios/YYYY-MM-DD-<feature>.md`.
- Single authoring pass. No two-pass authoring.
- Human-invoked only. No auto-triggering from `writing-plans`, no description tuned for self-triggering, no modifications to any other skill.
- UI flows only. Front-end surface must be present in the plan; otherwise the skill stops.
- Output is tool-agnostic. No Playwright/Chrome DevTools/curl syntax in the doc.
- Plan names a sibling results file but does not author or update it.
- Adversarial pressure testing per `superpowers:writing-skills` methodology.

**Out of scope for v1, deferred to v2 conditional on maintainer approval:**

- Two-pass authoring (planning-time skeleton + pre-merge contract refinement).
- Auto-invocation from `writing-plans` or any other skill.
- Auto-invocation via strong skill description (self-triggering on plan-completion conditions).
- Backend-only flows (API contract scenarios via `curl`, or CLI scenarios).
- Diff-aware sourcing (reading the working branch diff in addition to spec/plan).
- Execution and results tracking (skill writes results file, not just names it).
- Hard gating in `finishing-a-development-branch` (refusing merge until results file signed off).
- Calibrated grading rubrics or few-shot scoring anchors.
- Failure-flow automation (formal path back from a found regression into the agent loop).

## Architecture

### Skill identity

```yaml
---
name: writing-ui-acceptance-scenarios
description: Use after writing-plans has produced an implementation plan for a feature with UI surface, before implementation starts, to capture how a user verifies the feature works in the running app
---
```

Type: technique skill. Verb-prefixed name, third-person "Use after..." description, names trigger conditions explicitly (plan exists, UI surface present).

### Files

```
skills/writing-ui-acceptance-scenarios/
  SKILL.md
  scenario-template.md
```

Two files. No subagent prompt, no calibration data, no example docs. SKILL.md is the skill; `scenario-template.md` is the empty artifact shape the skill fills in.

### Inputs the skill reads

- The spec at `docs/superpowers/specs/YYYY-MM-DD-<feature>-design.md` — for acceptance criteria, motivation, surface description.
- The plan at `docs/superpowers/plans/YYYY-MM-DD-<feature>.md` — for file list, surface confirmation, and corroborating acceptance criteria.
- Human, via direct question, for anything not derivable: dev URL, test accounts, fixture data, env vars, feature flags, external service dependencies.

The skill resolves which spec/plan pair to read at the start of its process by asking the human which feature is being targeted (see §Process step 1). It then locates the most recent matching spec and plan in those directories. If the human supplies an exact path, that path is used directly.

Spec and plan are read-only inputs. The skill never modifies them.

### Output

A single markdown file at `docs/superpowers/ui-acceptance-scenarios/YYYY-MM-DD-<feature>.md`, committed with message `docs: add UI acceptance scenarios for <feature>`.

### Doc shape

The template defines these sections in order:

1. **Header.** Title `<feature> — UI Acceptance Scenarios`. Source spec and plan paths. Status (`Draft` / `Ready to run`). Date.
2. **Surface under test.** One paragraph naming routes, components, or screens the feature exposes.
3. **Setup.** Dev server URL, test accounts, fixture data, env vars, feature flags, external service dependencies. Items not derivable from spec/plan come from the human during authoring. If nothing beyond a running dev server is needed, state that explicitly.
4. **Scenarios.** Numbered BDD `Given / When / Then`. Scenario 1 is always the primary user flow. Scenarios 2..N are one-per-acceptance-criterion from the spec. Each scenario:
    - Short title describing user intent (`### Scenario N: <user does X>`)
    - `Given` preconditions
    - `When` user actions in order
    - `Then` observable outcomes the runner must verify
5. **Hard fail conditions.** Categorical fails that apply to every scenario. Seeded with:
    - Any HTTP 5xx during the flow
    - Browser console errors of severity `error`
    - Components mount but render empty / undefined
    - Auth or session state breaks unexpectedly
   Plus any feature-specific fails the agent can infer or the human adds.
6. **Pass criteria.** Doc passes when every scenario's `Then` clauses are observed and no hard fail triggered during any run. Binary per scenario.
7. **Reporting failures.** One-paragraph instruction: log each run in the sibling results file at `docs/superpowers/ui-acceptance-scenarios/YYYY-MM-DD-<feature>-results.md` with date, scenario number, pass/fail, observed behaviour, and (on failure) hand back to the development agent with the results-file path. v1 does not define the results file's format — that is the runner's call.

### Scenario format — worked example

```markdown
### Scenario 2: User edits their display name from settings

Given a signed-in user with display name "Pat"
  And the user is on /settings/profile

When the user clears the "Display name" field
  And enters "Patricia"
  And clicks "Save changes"

Then the page shows a "Saved" confirmation within 2 seconds
  And the navigation bar greeting updates to "Patricia"
  And reloading the page still shows "Patricia"
```

### What the doc never contains

- Execution syntax: no `await page.click(...)`, no `cy.visit(...)`, no `curl`, no DevTools snippets.
- Grading rubrics or scored criteria.
- Regression test pinning to past bugs.
- Environment teardown instructions.
- Pre-filled execution results, dates, pass/fail flags.

## Process the skill runs

When invoked, the skill executes the following steps in order:

1. **Identify the feature and locate inputs.** Ask the human which feature this skill is being run for. Accept either a short feature slug (e.g. `dashboard-search`) or an explicit path. Resolve to the most recent matching spec at `docs/superpowers/specs/YYYY-MM-DD-<feature>-design.md` and plan at `docs/superpowers/plans/YYYY-MM-DD-<feature>.md`. If either cannot be located after asking, stop with a clear message that a plan and spec must exist first — direct the human to run `superpowers:brainstorming` and `superpowers:writing-plans`.
2. **Confirm UI scope.** Inspect the plan's file list for front-end files (extensions `.tsx`, `.jsx`, `.vue`, `.svelte`, `.html`, or paths containing `components/`, `pages/`, `app/`) **or** any spec section describing user-facing behaviour. If no front-end surface is detected, stop: "v1 is UI-only; backend and CLI surfaces are deferred."
3. **Extract acceptance criteria.** Pull every acceptance criterion from the spec. These map to scenarios 2..N. If the primary user flow is not explicit in the spec, ask the human to state it in one sentence; this becomes scenario 1.
4. **Identify Setup needs.** From plan + spec, list known surfaces (routes, components). Ask the human:
    - Dev server URL
    - Test account credentials (or how to create a fresh test user)
    - Any fixture data or seeds required
    - Any feature flags or env vars that gate the feature
    - Any external service dependencies (payment sandbox, email inbox, etc.)
5. **Draft the doc.** Fill `scenario-template.md` with: header, surface, setup, scenarios in BDD form, hard fails (seed list + inferred), pass criteria, reporting-failures paragraph.
6. **Self-review.** Run the inline self-review checklist (see §Self-review). Fix any failures in place.
7. **Write and commit.** Save to `docs/superpowers/ui-acceptance-scenarios/YYYY-MM-DD-<feature>.md`. Commit with `docs: add UI acceptance scenarios for <feature>`.
8. **Terminal handback.** Report the path to the human. Suggest manual review and execution via their preferred tooling. Do **not** invoke any other skill.

The skill runs inline in the agent's session. No subagent dispatch. No parallel execution.

## Self-review

The skill runs this checklist against its own draft before writing the file. Failures are fixed inline.

- Scenario 1 is the primary user flow, stated as user intent.
- Every acceptance criterion in the spec is covered by exactly one scenario.
- No scenarios exist that are not anchored in either the primary flow or a spec acceptance criterion.
- Each scenario uses Given / When / Then. No prose-only scenarios.
- Every `Then` clause names something the runner can observe in the running app. "Then the database row is updated" is invalid — that is not user-observable.
- No execution syntax appears anywhere in the doc.
- Setup section names dev URL, accounts, fixtures, env vars, external services — or explicitly states "no setup beyond a running dev server".
- Hard fail conditions include the seed list plus any feature-specific fails.
- No TBD, TODO, "fill in later", or placeholder text.
- No pre-filled execution results, dates, or pass/fail flags.
- Pass criteria block is binary and unambiguous.

## Discipline content — red flags

The skill embeds the following STOP conditions to harden against rationalisations under pressure:

- About to write a scenario whose `Then` is a code-level assertion → that belongs in TDD; delete from this doc.
- About to add Playwright, Cypress, DevTools, or `curl` syntax → strip it; the doc is tool-agnostic.
- About to author without reading the spec → stop; locate the spec or ask the human.
- About to author for a project with no front-end surface → stop; tell the human v1 is UI-only.
- About to invent an acceptance criterion the spec does not contain → ask the human whether to add it to the spec first; do not smuggle new requirements into scenarios.
- About to fill in a `Last run` or `Result` field during authoring → stop; that belongs in the sibling results file written by the executor.
- About to invoke another skill at the end → don't; v1 is a terminal leaf skill.

## Common mistakes (in SKILL.md)

| Mistake | Fix |
|---------|-----|
| Scenarios re-state spec prose verbatim | Re-phrase as a user *observing* behaviour: "Then the saved indicator appears" not "Then the save endpoint succeeds" |
| Then clauses are vague ("Then it works") | Name the visible artifact: rendered text, route change, badge state |
| Exhaustive edge cases included | Cap depth at primary flow + per-acceptance-criterion + hard fails. Edge cases go in TDD. |
| Setup section missing dev URL | Ask the human; never invent a URL |
| Skill invoked before any plan exists | Ask the human to run `writing-plans` first |
| Backend-only project | v1 out of scope; tell the human cleanly |

## What the skill explicitly does not do

- Does not run any test.
- Does not invoke any MCP.
- Does not modify code under `src/`, `app/`, or any source path.
- Does not edit the spec or plan it reads (read-only inputs).
- Does not call `finishing-a-development-branch`, `writing-plans`, `subagent-driven-development`, or any other skill.
- Does not produce or update the sibling results file.
- Does not modify any existing skill file.

## Relationship to existing skills

- **`test-driven-development`** — orthogonal. TDD is code-level red-green for programmatic tests. This skill is user-observable verification in the running app. Naming uses "scenarios" rather than "tests" to keep the boundary clear; SKILL.md states the boundary explicitly so agents do not conflate the two.
- **`writing-plans`** — runs strictly *after* it. Skill expects a plan and spec to exist at conventional paths; stops cleanly if not.
- **`finishing-a-development-branch`** — no relationship in v1. Skill does not call it, does not gate it, does not change it.
- **`subagent-driven-development`, `executing-plans`** — no relationship. Skill produces an artifact; execution is a downstream concern out of scope.
- **`verification-before-completion`** — orthogonal. That skill enforces evidence before completion claims; this skill produces a written verification *plan*. Both can coexist.

## PR strategy

The PR is the dialogue artifact. It must do specific work to clear the 94% rejection rate.

**Title:** `feat(writing-ui-acceptance-scenarios): zero-dep skill for capturing UI verification scenarios`

**Required PR description content (per `CLAUDE.md` and `.github/PULL_REQUEST_TEMPLATE.md`):**

1. **Problem statement, lived experience.** Cite the gap directly: "agents can write code, pass tests, pass review, and ship broken UI because nothing in core asks `did you walk this in a browser?`" Reference the original incident pattern from PR #946's RED scenario.
2. **Existing PRs.** Reference Issue #1027 (Mori, browser-MCP-dependent design), Issue #374 (Samuel, BDD framing request), and PR #946 (chickencyj, runtime smoke-check gate). Explain how this PR differs: it is a *planning-time authoring* skill that produces a tool-agnostic artifact, not a runtime gate and not an MCP-dependent runner. The artifact is what plugins and humans need to act independently of core.
3. **Explicit scope cuts.** The §Deferred section above, reproduced verbatim. The point: every cut is a dialogue lever. Maintainers approving v1 implicitly indicate which v2 capabilities they would entertain.
4. **Rigor.** Adversarial pressure testing per `superpowers:writing-skills`:
    - RED baseline: agent without skill, after writing-plans finishes, asked to "verify the feature will work in the browser". Expected: vague suggestions or skipping. Captured rationalisations.
    - GREEN: agent with skill produces a scenarios doc matching the template, covering primary flow + per-criterion + hard fails, no execution syntax.
    - Refactor rounds plug loopholes identified in RED.
5. **Environment table.** Claude Code, latest, Opus 4.7.
6. **Human review.** A human has reviewed the complete diff.

**Files the PR touches:**

- New: `skills/writing-ui-acceptance-scenarios/SKILL.md`
- New: `skills/writing-ui-acceptance-scenarios/scenario-template.md`
- New: docs spec entry (this file, plus the plan that follows)
- Modified: none of the existing skills

## Open questions for v2 dialogue

These are not v1 decisions. They are listed so the PR can pose them to maintainers without committing the v1 PR to a particular answer.

1. **Trigger placement.** Should v2 auto-invoke from `writing-plans` self-review when the plan produces observable output, or self-trigger via skill description, or stay human-invoked with a plan-template pointer?
2. **Two-pass authoring.** Planning-time skeleton + pre-merge contract refinement (Mori's pattern), or stay single-pass?
3. **Backend support.** API contract scenarios via `curl`-style assertions; CLI scenarios via shell invocations. Same skill or sibling skills?
4. **Diff-aware sourcing.** Post-implementation pass that reads the working branch diff to refine scenarios against what was actually built?
5. **Execution and results.** Does a future companion skill or plugin own running the doc and writing the sibling results file? What is the format of the results file?
6. **Gate enforcement.** Does `finishing-a-development-branch` consult the results file before presenting merge options (addressing PR #946's lesson that bullet suggestions get skipped under pressure)?
7. **Calibration.** Adopt Mori's few-shot grading anchors for v2 execution, or stay binary pass/fail?
