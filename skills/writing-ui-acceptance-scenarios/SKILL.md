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
|--------|-----|
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
