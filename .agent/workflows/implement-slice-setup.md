---
description: Progress check, skill loading, slice reading, parallel mode detection, and contract writing for the implement-slice workflow
parent: implement-slice
shard: setup
standalone: true
position: 1
pipeline:
  position: 7.1
  stage: implementation
  predecessors: [plan-phase]
  successors: [implement-slice-tdd]
  skills: [session-continuity, tdd-workflow, parallel-agents, parallel-feature-development]
  calls-bootstrap: true
---

# Implement Slice — Setup

Check progress state, load skills, read the slice, detect parallel mode, and write the contract.

**Prerequisite**: Phase plan must exist with slice acceptance criteria. If not, tell the user to run `/plan-phase` first.

---

## -1. Placeholder verification (CRITICAL — run before anything else)

Scan for unfilled placeholders before writing a single line of code. Read each file and scan for any literal `{{` characters:

1. `AGENTS.md`
2. `GEMINI.md`
3. `.agent/instructions/workflow.md`
4. `.agent/instructions/commands.md`
5. `.agent/instructions/structure.md`
6. `.agent/instructions/patterns.md`
7. `.agent/instructions/tech-stack.md`

**If ANY `{{PLACEHOLDER}}` pattern is found**:

> **STOP immediately.** Do not proceed with implementation. Tell the user which files contain unfilled placeholders and provide the specific remediation command for each:
>
> | File(s) with unfilled placeholders | Remediation |
> |------------------------------------|-------------|
> | `AGENTS.md`, `GEMINI.md`, `workflow.md`, `commands.md`, `tech-stack.md` | Run `/create-prd` to make tech stack decisions and trigger bootstrap provisioning |
> | `structure.md` (`{{PROJECT_STRUCTURE}}`, `{{ARCHITECTURE_TABLE}}`) | Run `/create-prd-compile` Step 9.5 to generate and lock the directory structure |
> | `patterns.md` (`{{FRAMEWORK_PATTERNS}}`) | Run `/bootstrap-agents-provision` after confirming the frontend framework |
>
> List each unfilled placeholder with its file and line number.

**Why this is critical**:
- An unfilled `{{VALIDATION_COMMAND}}` makes every validation step in the pipeline a no-op — tests never run, lint never runs, builds never run. The entire TDD cycle silently does nothing.
- An unfilled `{{PROJECT_STRUCTURE}}` in `structure.md` means the infrastructure slice has no canonical directory tree to scaffold — agents will invent a structure, defeating the purpose of locking it in `/create-prd-compile` Step 9.5.
- An unfilled `{{FRAMEWORK_PATTERNS}}` in `patterns.md` means agents writing components have no framework-specific conventions — they'll use generic patterns that may conflict with the chosen framework.
- An unfilled `GEMINI.md` means the Gemini CLI agent operates with zero project context — no project name, no validation command, no architecture doc path.

Only proceed to Step 0 when zero `{{` patterns remain in all seven files.

---

## 0. Check progress state

Read `.agent/skills/session-continuity/protocols/01-session-resumption.md` and follow the **Session Resumption Protocol** to confirm this slice is next and load context from previous sessions.

Also read `.agent/skills/session-continuity/SKILL.md` for cross-session tracking patterns — if this slice spans multiple sessions, the skill ensures no context is lost between sessions.

**Precondition Check**: Look at `.agent/progress/index.md` and the relevant phase file. Verify that the *previous* slice was fully marked as complete before starting this one. If the previous slice's code is done but its progress tracking wasn't updated, **you must update the previous slice's tracking files first**.

## 0.5. Load bundled skills

Read these skill files for the TDD implementation methodology:
1. `.agent/skills/tdd-workflow/SKILL.md` — Red→Green→Refactor methodology
2. `.agent/skills/error-handling-patterns/SKILL.md` — Error handling conventions
3. `.agent/skills/git-workflow/SKILL.md` — Branch, commit, and PR conventions
4. `.agent/skills/code-review-pro/SKILL.md` — Self-review before presenting
5. `.agent/skills/testing-strategist/SKILL.md` — Test strategy and coverage decisions
6. `.agent/skills/clean-code/SKILL.md` — Architecture and code quality principles
7. `.agent/skills/logging-best-practices/SKILL.md` — Structured logging patterns
8. `.agent/skills/minimalist-surgical-development/SKILL.md` — Precision edits, avoiding massive rewrites
9. `.agent/skills/systematic-debugging/SKILL.md` — Evidence-based root cause analysis

Use **find-skills** to discover a test framework skill if needed. Read `.agent/skills/find-skills/SKILL.md` and search for your project's test framework (e.g., "vitest", "jest", "pytest").

If this slice introduces a significant new dependency not already in the project's tech stack:

1. Identify the new dependency and its stack category
2. Read `.agent/workflows/bootstrap-agents.md` and execute its utility instructions immediately with the new value
3. Bootstrap will install any matching skills from the library

---

## 1. Read the slice

Read the slice's acceptance criteria from the phase plan.

## 1.5. Check for parallel mode

Scan the slice's tasks for surface tags (`BE`, `FE`, `QA`):

- **No tags found** → proceed sequentially (steps 2–7 in the tdd shard as normal)
- **Tags found** → enter parallel mode:

### Parallel mode dispatch (TDD order)

| Phase | Agent | Responsibility | Depends On |
|-------|-------|---------------|------------|
| 0 | Orchestrator | Contracts/schemas (untagged tasks) | — |
| 1 | QA (RED) | Write comprehensive failing tests | Phase 0 |
| 2 | BE + FE (parallel) | Write code to make tests pass | Phase 1 |
| 3 | QA (GREEN) | Verify all tests pass, anti-cheat audit | Phase 2 |
| 4 | Orchestrator | Iterative correction loop if needed | Phase 3 |

> **Tests are the rock. Code is malleable.** Comprehensive tests encode the
> acceptance criteria. If tests fail, the code changes — never the tests.

1. **Handle untagged tasks first** — Contract/schema work runs sequentially by the orchestrator (step 2)
2. **Dispatch `QA` agent (QA-RED)** — Write comprehensive failing tests for ALL acceptance criteria:
   - Read `.agent/skills/session-continuity/protocols/09-parallel-claim.md` — Parallel Claim Protocol
   - QA agent claims its tasks, writes test files covering every criterion
   - Tests MUST fail (Red phase) — verify with `{{TEST_COMMAND}}`
   - **Depth matters**: unit tests, integration tests, edge cases, error paths, boundary conditions
   - No shallow test suites — every acceptance criterion gets thorough coverage
   - **Contract cross-reference**: QA agent verifies every field, error type, and permission boundary in the Zod contract has at least one test — not just what the phase plan acceptance criteria mention

> **Log dispatch**: Append to the slice file (`.agent/progress/slices/phase-NN-slice-NN.md`): `## Dispatch Log` section with entry: `- QA-RED dispatched: [timestamp or "now"]`

3. **Dispatch `BE` + `FE` agents in parallel (GREEN)** — Write code to make QA's tests pass:
   - Read `.agent/skills/parallel-agents/SKILL.md` for dispatch protocol
   - Read `.agent/skills/parallel-feature-development/SKILL.md` for file ownership
   - Each agent claims tasks via Protocol 9, works through subtasks
   - Agents code against the **tests and contracts**, not assumptions
   - **Spec traceability**: any implementation decision not explicitly in the spec or contract (enum values, defaults, timeouts, error messages, spacing, retry logic) MUST be annotated with `// DECISION: [what was decided and why]`

> **Log dispatch**: Append to the dispatch log in the slice file: `- BE+FE dispatched: [timestamp or "now"]`

4. **Dispatch `QA` agent again (QA-GREEN)** — Second pass validation:
   - Run full test suite — all tests must pass
   - **Anti-cheating check**: verify tests haven't been weakened, removed, or simplified
   - **Decision audit**: scan for `// DECISION:` comments — flag any that indicate a spec gap that should be resolved upstream rather than decided by the implementer
   - Add integration/E2E tests that exercise BE+FE together
   - Report any failures to orchestrator

> **Log dispatch**: Append to the dispatch log in the slice file: `- QA-GREEN dispatched: [timestamp or "now"]`

5. **Iterative correction loop** — If QA-GREEN reports failures:
   - Re-dispatch `BE` and/or `FE` agents to fix failing code
   - Re-dispatch `QA` agent (QA-GREEN) to re-verify
   - Repeat until all tests pass — **only the orchestrator may end the loop**
6. **After all tests pass** → proceed to step 6.5 (Synthesis) in the tdd shard

---

## 2. Write the contract (Zod schema)

Define the request/response shapes as Zod schemas in the project's contracts directory (see `.agent/instructions/structure.md` for the exact path — typically `src/contracts/`). This is the source of truth.

### Propose next step

Contract (Zod schema) is written. Next: Run `/implement-slice-tdd` to execute the Red→Green→Refactor cycle, run validation, and update all progress tracking files.

For parallel mode: If parallel mode was activated in Step 1.5, the QA-RED and BE/FE/QA-GREEN dispatch is already underway — proceed to `/implement-slice-tdd` to handle synthesis and progress tracking.

> If this shard was invoked standalone (not from `/implement-slice`), surface this via `notify_user`.

