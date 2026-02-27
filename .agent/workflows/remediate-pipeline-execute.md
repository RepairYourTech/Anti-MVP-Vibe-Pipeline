---
description: Execute adversarial audit, remediate gaps, enforce fresh-session confirmation, and advance layers
parent: remediate-pipeline
shard: execute
standalone: true
position: 2
pipeline:
  position: utility
  stage: quality-gate
  predecessors: [remediate-pipeline-assess]
  successors: [] # returns to caller after stop
  skills: [resolve-ambiguity, technical-writer, code-review-pro]
  calls-bootstrap: false
---

# Remediate Pipeline — Execute

Layer-by-layer audit → remediate → fresh-run instruction loop.

> **Prerequisite**: Read `docs/audits/remediation-state.md` to determine the current layer and status. If this file does not exist, run `/remediate-pipeline-assess` first.

---

## 1. Read remediation state

Read `docs/audits/remediation-state.md` to determine:
- The current layer to process
- The status of all layers
- Any layers already confirmed clean this session

---

## 2. Re-invocation detection

Read `docs/audits/audit-scope.md` and look for a `## Gaps Fixed` section.

- **If absent** → This is a first-pass invocation. Proceed to Step 3.
- **If present** → Read the `layer` and `fixed_by_session` fields.
  - If `layer` matches the current layer AND `fixed_by_session` is different from the current session identifier → The user has run a fresh audit and it passed (they would not be re-invoking otherwise). Mark the current layer as confirmed clean in `remediation-state.md`. Advance to the next layer (update `## Current Layer` in `remediation-state.md`). If no more layers need auditing, go to Step 7. Otherwise, proceed to Step 3 for the next layer.
  - If `layer` matches but `fixed_by_session` is the same session → **STOP**: "The session that fixed gaps cannot confirm them clean. Please run `/audit-ambiguity [layer]` as a fresh invocation (new session or new conversation), then re-run `/remediate-pipeline`."

---

## 3. Run adversarial audit inline

**CRITICAL**: Run the audit inline — do not invoke `/audit-ambiguity` as a sub-workflow.

**CRITICAL ANTI-HALLUCINATION RULE**: Process one document at a time through 3a → 3b → 3c before moving to the next. Never batch-read and score from memory.

For the current layer, audit each document using the following three sub-steps:

### 3a. Implementer Simulation

Attempt to write a stub implementation of each feature/endpoint/component using only what's in the spec — no external context or prior project knowledge.

List every decision you had to make that isn't explicitly specified (enum values, defaults, timeouts, error messages, retry counts, field types, HTTP status codes, validation rules, rate limit numbers, role permission lists, etc.).

Write each such decision immediately to `docs/audits/[layer]-ambiguity-report.md` as a punch list item with severity ❌. These gaps are unconditional — the rubric in 3b cannot override them.

### 3b. Rubric Scoring with Two-Implementer Test

Score each dimension of the relevant rubric (use the rubrics from `.agent/workflows/audit-ambiguity-rubrics.md`). For each dimension:

- **✅** → "Two implementers would make the same decision because: [specific reason citing exact text from the document]." If cannot confidently complete this sentence, score ⚠️ instead.
- **⚠️** → Quote what exists AND state precisely what is missing AND what decision an implementer would have to make without it.
- **❌** → List the section headings checked and confirm the content is absent.

After scoring all dimensions for a document, immediately write the complete score table and all supporting citations/gap descriptions to `docs/audits/[layer]-ambiguity-report.md` before moving to 3c.

### 3c. Devil's Advocate Pass

After scoring all dimensions for a document:
1. For each ✅, ask: "What would a junior developer get wrong?" and "What would a malicious implementer exploit?"
2. If either question reveals a gap → downgrade to ⚠️ and add the gap to the punch list.
3. Document the devil's advocate finding alongside the score in the report.

### Layer-to-documents mapping

| Layer | Documents to load |
|-------|-------------------|
| Vision | `docs/plans/vision.md` |
| Architecture | `docs/plans/*-architecture-design.md`, `docs/plans/ENGINEERING-STANDARDS.md` |
| IA | `docs/plans/ia/index.md` + each shard + `docs/plans/ia/deep-dives/*.md` |
| BE | `docs/plans/be/index.md` + each spec |
| FE | `docs/plans/fe/index.md` + each spec |

Write findings to `docs/audits/[layer]-ambiguity-report.md`.

---

## 3.5. Cross-Layer Consistency

This step runs after all per-document scoring is complete, whenever the current layer is BE, FE, or the scope is `all`.

| Check | How to Verify |
|-------|---------------|
| IA → BE coverage | For each user flow in each IA shard, verify a BE endpoint exists that handles it. Orphan endpoints or uncovered flows are ❌. |
| BE → FE field mapping | For each BE response field, verify at least one FE component prop consumes it. Unmapped fields or props are ❌. |
| IA → FE access control | For each access control rule in each IA shard, verify the FE spec has corresponding conditional rendering. Missing conditional rendering is ❌. |
| Error code coverage | For each BE error code, verify the FE spec has a corresponding error state. Missing error states are ❌. |

Write all cross-layer findings to a `## Cross-Layer Consistency` section in the layer's ambiguity report.

---

## 4. Zero-gap first-pass path

If 0% ambiguity on first pass, the layer is clean but this is the "unverified clean" case.

> **The fresh-run rule still applies.** This session performed the audit, so it cannot also confirm the result. A different session must run `/audit-ambiguity [layer]` to confirm.

Add the summary section to the layer's ambiguity report:
- Overall ambiguity: 0%
- No gaps found

Persist `## Gaps Fixed` in `docs/audits/audit-scope.md` with `gaps_resolved: 0` and a generated session identifier (UUID or timestamp-based). Then proceed to Step 6.

---

## 5. Remediate gaps

Read `.agent/skills/resolve-ambiguity/SKILL.md` for the full resolution methodology.

**Classify all gaps for the entire layer first, then present them together:**

- **Judgment calls** (Intent/Choice decisions that require product direction, UX preference, or scope choice): Collect ALL judgment calls across all documents in the layer, grouped by document with smart options where possible, and present them to the user before applying any fixes.
- **Mechanical fixes** (Technical/factual gaps resolvable via tiered lookup — project docs, architecture files, upstream specs, official docs, web search): Propose all mechanical fixes with citations. Apply after user approval.

### Resolution order

1. Present all judgment calls to the user. Wait for decisions.
2. Apply all approved mechanical fixes using the tiered lookup from the `resolve-ambiguity` skill.
3. Apply all user-resolved judgment-call fixes. These may change what mechanical fixes are needed — re-check after applying.

After all fixes are applied, add the summary section to the layer's ambiguity report:
- Overall ambiguity before remediation: [X]%
- Gaps resolved: [N]
- Judgment calls resolved: [N]
- Mechanical fixes applied: [N]

**Restart recommendation**: If >70% of the layer's documents scored ❌ on the majority of rubric dimensions, add a note: "Consider rewriting `[layer]` from scratch using the current pipeline — the volume of gaps suggests the documents are structurally shallow rather than just missing details."

### Persist fix metadata

If `## Gaps Fixed` already exists in `docs/audits/audit-scope.md`, replace it. Otherwise append:

```markdown
## Gaps Fixed

- **layer**: [current layer]
- **fixed_at**: [ISO 8601 timestamp]
- **fixed_by_session**: [generated UUID e.g. "remediate-[timestamp]"]
- **gaps_resolved**: [count of ⚠️ and ❌ items resolved]
- **report_file**: `docs/audits/[layer]-ambiguity-report.md`
```

---

## 6. Instruct fresh-run confirmation

Use `notify_user` with the following message:

> ## [Layer] Remediated
>
> [N] gaps were found and fixed across [M] documents.
>
> **The fresh-run rule requires a different invocation to confirm this layer is clean.**
>
> Next steps:
> 1. Run `/audit-ambiguity [layer]` as a fresh invocation (new session or new conversation)
> 2. When it scores 0% ambiguity, come back and run `/remediate-pipeline` again
> 3. This workflow will detect the confirmed-clean state automatically and advance to [next layer]
>
> Do NOT re-run `/remediate-pipeline` in this same session — the session that fixed gaps cannot grade its own homework.

**STOP** — Do not execute any further steps in this invocation.

---

## 7. Advance or complete

This step runs when re-invoked after `/audit-ambiguity [layer]` has confirmed the layer clean (detected in Step 2).

### If more layers remain

Update `docs/audits/remediation-state.md`:
- Move the current layer to `confirmed-clean`
- Set the next `needs-audit` or `unverified-clean` layer as `## Current Layer`
- Add the confirmed layer to `## Layers Confirmed Clean This Session`

Use `notify_user` to inform the user that the layer is confirmed clean and instruct them to re-run `/remediate-pipeline` to continue with the next layer.

### If all layers are confirmed clean

Present the completion summary:

> ## Pipeline Remediation Complete
>
> All layers have been audited and confirmed clean across multiple invocations.
>
> | Layer | Status |
> |-------|--------|
> | Vision | ✅ Confirmed clean |
> | Architecture | ✅ Confirmed clean |
> | IA | ✅ Confirmed clean |
> | BE | ✅ Confirmed clean |
> | FE | [✅ Confirmed clean | ⬜ Not started] |

Propose next pipeline step:
- All IA + BE clean, FE not started → "Next: Run `/write-fe-spec` for the first IA shard that needs a FE spec."
- All layers including FE → "/plan-phase to create dependency-ordered TDD slices."
- Vision + Architecture clean, IA not started → "/decompose-architecture to break architecture into IA shards."
