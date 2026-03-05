---
description: Create TDD vertical slices for one phase, with acceptance criteria per item
pipeline:
  position: 6
  stage: planning
  predecessors: [write-be-spec, write-fe-spec] # join point — waits for both
  successors: [implement-slice]
  skills: [resolve-ambiguity, testing-strategist, technical-writer]
  calls-bootstrap: true
---

// turbo-all

# Plan Phase

Break a phase into TDD vertical slices, each spanning all four surfaces (contract, test, implementation, UI).

> **Every slice ships production-grade code.** Slices are ordered by dependency,
> not by quality tier. The first slice and the last slice meet the same bar.

**Input**: Approved specs (IA + BE + FE) and the phasing section from architecture design
**Output**: Phase plan with ordered slices and acceptance criteria

---

## 0. Phase sequencing gate

Read `.agent/progress/index.md` to identify the current phase number N.

- **If N = 1** → this is the first phase. Skip this gate.
- **If N > 1** → read `.agent/progress/phases/phase-[N-1].md` and verify its status is `complete` with a passing `/validate-phase`. **Hard stop** if the previous phase is not complete: "Phase [N-1] must be complete with a passing `/validate-phase` before planning Phase [N]. Run `/validate-phase` for Phase [N-1] first."

  **Architecture map freshness check (N > 1 only)** — warning, not a hard stop. Skipped entirely for Phase 1.

  If `docs/ARCHITECTURE.md` exists, compare its "Last updated" date against the Phase N-1 completion date from `.agent/progress/phases/phase-[N-1].md`. If the map is **older** than Phase N-1 completion:

  > ⚠️ `docs/ARCHITECTURE.md` may not reflect current state. Run `/update-architecture-map` or reply `"confirmed"` if intentionally current.

  **Wait** for user resolution before proceeding to Step 0.1. If dates are equal/newer, or the file doesn't exist, proceed silently.

---

## 0.1. Load planning skills

Read these skills for slice planning guidance:
1. `.agent/skills/testing-strategist/SKILL.md` — Test strategy per slice
2. `.agent/skills/technical-writer/SKILL.md` — Acceptance criteria clarity

---

## 0.5. Application Completeness Audit

Before running checks: read `{{SURFACES}}` from `.agent/instructions/tech-stack.md`. For each row in the table below, only run the check if the "Applies To" column matches the project's confirmed surfaces. Skip rows that don't apply — a missing check for an inapplicable surface is not a failure.

Read all FE specs in `docs/plans/fe/` and check the following table.

| Check | Applies To | What It Verifies |
|---|---|---|
| **Route coverage** | web, mobile, desktop | Every route in the app is specced in at least one FE spec |
| **Navigation coverage** | web, mobile, desktop | Every route is reachable from at least one navigation element |
| **Auth state coverage** | all surfaces (if auth exists) | Every auth state (logged out, logged in, insufficient permissions) has a defined UI or response |
| **Empty state coverage** | web, mobile, desktop | Every data-fetching view has an empty state spec |
| **Error state coverage** | all surfaces | Every data-fetching operation has an error response or error state spec |
| **Onboarding coverage** | all surfaces (if accounts exist) | First-run or onboarding flow is specced |
| **404/error pages** | web, desktop | Global error pages (404, 500, offline) are specced |
| **Help/usage output** | cli | `--help` flag output is specced and documents all commands |
| **Exit codes** | cli | All error exit codes and their meanings are documented in a spec |
| **Command discovery** | cli | Command listing or autocomplete behavior is specced |

**If ANY applicable check fails → stop and tell the user exactly which FE specs need to be updated. Do NOT proceed to create a phase plan.**

Only when all **applicable** checks pass does the workflow continue to the next step.

---

## 0.6. BE↔FE Coverage Cross-Check

Read `docs/plans/be/index.md` and `docs/plans/fe/index.md`.

For every BE spec listed in `docs/plans/be/index.md`, check whether at least one FE spec's `## Source Map` section references it. Apply the following rules:

1. **Covered**: The BE spec appears in at least one FE `## Source Map` → pass.
2. **Internal-only exception**: If the BE spec is annotated `[internal-only — no UI]` in the BE index, it is exempt from FE coverage → pass.
3. **Uncovered**: The BE spec is neither covered by an FE Source Map nor marked `[internal-only — no UI]` → collect it in the uncovered list.

**If the uncovered list is empty** → proceed to the next step.

**If any uncovered non-internal BE specs exist** → **hard stop**. Present the uncovered list and ask:

> "The following BE specs have no FE coverage and are not marked `[internal-only — no UI]`:
>
> - `[be-spec-filename-1]`
> - `[be-spec-filename-2]`
>
> For each, either:
> 1. Add FE coverage via `/write-fe-spec`
> 2. Mark the BE spec as `[internal-only — no UI]` in `docs/plans/be/index.md` if it has no user-facing surface
>
> Confirm when resolved."

Do not proceed until the user confirms all uncovered specs are resolved.

---

## 0.8. Draft continuity check

Check whether `docs/plans/phases/phase-N-draft.md` already exists (where N is the current phase number).

- **If it exists**: Read it and identify which slices are already drafted (have acceptance criteria written) vs which are missing. Present the current state: "A draft exists for Phase N with [X] slices already written. Do you want to **continue from where you left off** (add missing slices only), or **start fresh** (overwrite the draft)?" Wait for the user's answer.
- **If it does not exist**: Proceed normally.

## 1. Read phase scope

Read the file at `docs/plans/*-architecture-design.md` (phasing section) and the file at `docs/plans/be/index.md` (which specs to include).

## 2. Identify slices (spec-anchored derivation)

For each FE spec in the phase scope:
1. Open the FE spec's `## Interaction Specification` section
2. Enumerate every distinct named user flow (e.g., "Submit entity claim form", "Search directory with filters", "View entity detail page")
3. For each user flow, identify its primary BE endpoint from the FE spec's `## Source Map`
4. Group flows into one slice only when:
   a. They share the exact same DB write (true dependency, not just same domain), OR
   b. They are a required sequence (flow B cannot be tested without flow A existing)
5. Flows that do not meet criteria (a) or (b) become individual slices

The resulting list of slices is derived from the spec, not estimated from feature names. Do not aggregate slices by domain name or by "it feels like one feature."

Estimate complexity (S/M/L) per derived slice. Flag any slice estimated L — these are candidates for further splitting before ordering begins.

**Good slice**: "User can submit an entity claim form" (one named user flow from the FE interaction spec)
**Bad slice**: "Implement entity management" (domain name, not a spec-derived user flow)

## 3. Order by dependency

Read .agent/skills/concise-planning/SKILL.md and follow its methodology.

Sort slices so each builds on the last:
- Infrastructure slices first (DB schema, auth middleware)

**Phase 1 special rule**: The `00-infrastructure` shard is always the first slice. Verify it covers: (1) CI/CD pipeline setup — read `.agent/skills/{{CI_CD_SKILL}}/SKILL.md`, (2) environment configuration (`.env.example`), (3) deployment pipeline — read `.agent/skills/{{HOSTING_SKILL}}/SKILL.md`, (4) project scaffolding from `.agent/instructions/structure.md` (directories, `README.md` files, base configs), (5) database initialization. Add missing items before proceeding.

**Verification gates** (hard gates — add explicitly to the phase plan):
- `/verify-infrastructure` MUST pass after the infrastructure slice, before any feature slice.
- `/verify-infrastructure` MUST pass again after the auth slice (with auth smoke test), before any auth-dependent feature slice.

- Core entity CRUD second
- Dependent features next
- Cross-cutting concerns (logging, monitoring, error handling) woven throughout

**Note:** This ordering is about dependencies, not about deferring quality.
Every slice — including the first infrastructure slice — is fully tested,
fully specified, and production-ready.

## 4. Write acceptance criteria

Read `.agent/skills/prd-templates/references/operational-templates.md` for the **Slice Acceptance Criteria** template. For each slice, use the template to define testable acceptance criteria with surface tags:

> **Write as you go**: After completing acceptance criteria for each slice, immediately append that slice's entry to `docs/plans/phases/phase-N-draft.md` (create the file if it doesn't exist). Do not accumulate all slices in context and write them all at once in Step 5.

**Surface tag rules:**
- `BE`: API routes, DB queries, middleware, business logic, server-side validation
- `FE`: Pages, components, styling, interactions, client-side logic
- `QA`: Test writing (RED phase — runs FIRST), test verification (GREEN phase — runs LAST)
- No tag: Contract/schema work, shared infra — handled sequentially by orchestrator

## 4.5. Identify parallel groups (TDD order)

Read .agent/skills/parallel-agents/SKILL.md and apply its workstream decomposition methodology to identify independent parallel groups.

For each slice, determine the execution order following TDD:

> **Tests are the rock. Code is malleable.** Tests encode the acceptance criteria
> and must be comprehensive. Code adapts to pass tests, never the reverse.

Read `.agent/skills/parallel-agents/SKILL.md` and follow its TDD-Order Dispatch methodology for parallel groups and execution order. Flag any tasks that can't be parallelized (shared file dependencies) in the plan.

## 5. Finalize phase plan

Read `docs/plans/phases/phase-N-draft.md` (which was built progressively in Step 4) and write the final formatted phase plan to `docs/plans/phases/phase-N.md`. The draft is the authoritative source — do not add or drop slices during finalization.

## 6. Generate progress files

Read `.agent/skills/session-continuity/protocols/02-progress-generation.md` and follow the **Progress Generation Protocol** to create tracking files for this phase in `.agent/progress/`.

## 6.5. Bootstrap Completeness Gate

Scan these four files for literal `{{` occurrences of `LANGUAGE_SKILL`, `HOSTING_SKILL`, `CI_CD_SKILL`, `ORM_SKILL`, `UNIT_TESTING_SKILL`, `E2E_TESTING_SKILL`:
- `.agent/workflows/implement-slice-setup.md`
- `.agent/workflows/implement-slice-tdd.md`
- `.agent/workflows/verify-infrastructure.md`
- `.agent/workflows/validate-phase.md`

For each unfilled placeholder, read `docs/plans/*-architecture-design.md` to extract the confirmed value, then run `/bootstrap-agents` (see `.agent/workflows/bootstrap-agents.md`) with the corresponding key.

> ❌ STOP — Re-scan the four files. Only proceed to Step 7 when zero `{{` patterns remain. If any remain unfilled after bootstrap, tell the user which placeholders could not be provisioned.

## 7. Request review and next steps

Use `notify_user` to request review of the phase plan and generated progress files.

**Proposed next step**: Once approved, run `/implement-slice` for the first slice in the phase plan. Read `.agent/progress/` to identify which slice to start with.
