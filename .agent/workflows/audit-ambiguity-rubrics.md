---
description: Scope determination, document loading, and scoring rubrics for all 5 pipeline layers for the audit-ambiguity workflow
parent: audit-ambiguity
shard: rubrics
standalone: true
position: 1
pipeline:
  position: utility
  stage: quality-gate
  predecessors: []
  successors: [audit-ambiguity-execute]
  skills: [code-review-pro]
  calls-bootstrap: false
---

# Ambiguity Audit — Rubrics

Determine audit scope, load source documents, and apply the scoring rubrics for each pipeline layer.

**Prerequisite**: At least one pipeline layer must have completed documents to audit. If no documents exist, tell the user which pipeline steps to run first.

---

## 1. Determine audit scope

Ask the user:
- `vision` — Audit vision document only
- `architecture` — Audit architecture design only
- `ia` — Audit IA shards only
- `be` — Audit BE specs only
- `fe` — Audit FE specs only
- `all` — Full cascade (Vision → Architecture → IA → BE → FE)

## 2. Load source documents

| Layer | Documents to load |
|-------|-------------------|
| Vision | `docs/plans/vision.md` |
| Architecture | `docs/plans/*-architecture-design.md`, `docs/plans/ENGINEERING-STANDARDS.md` |
| IA | `docs/plans/ia/index.md` + each shard listed |
| BE | `docs/plans/be/index.md` + each spec listed |
| FE | `docs/plans/fe/index.md` + each spec listed |

---

## 2.5. Persist audit scope

Write `docs/audits/audit-scope.md` with the determined scope and document list:

```markdown
# Audit Scope

> Generated: [date]

## Layers to Audit
[list of selected layers]

## Documents to Audit
[for each layer, list the exact file paths that will be audited]

## Status
in-progress
```

This file is read by `/audit-ambiguity-execute` as its prerequisite. If this file does not exist, the execute shard cannot proceed.

## Vision Rubric (7 dimensions)

| # | Dimension | ✅ | ⚠️ | ❌ |
|---|-----------|----|----|-----|
| 1 | Problem Clarity | One sentence, specific, testable | Vague or multi-problem | Missing |
| 2 | Persona Specificity | Named, with pain points + success criteria | Named but generic | Missing or "users" |
| 3 | Feature Completeness | MoSCoW with clear scope per item | Some items vague | Missing categories |
| 4 | Constraint Explicitness | All axes addressed with specific values | Some mentioned | Missing |
| 5 | Success Measurability | Concrete numbers/thresholds | Directional only | Missing |
| 6 | Competitive Positioning | Named competitors + differentiation | Vague positioning | Not addressed |
| 7 | Open Question Resolution | All questions have owners + deadlines | Questions listed, no owners | No questions section |

## Architecture Rubric (9 dimensions)

| # | Dimension | ✅ | ⚠️ | ❌ |
|---|-----------|----|----|-----|
| 1 | Tech Stack Decisiveness | Every axis decided with rationale | Some TBDs | Major gaps |
| 2 | System Architecture | All components, flows, failure modes | Some flows missing | Diagram only |
| 3 | Data Strategy | Placement, schema, queries, migrations, PII | Some missing | Sketched only |
| 4 | Security Model | Auth, authz, validation, rate limits, CSP | Some flows vague | Not addressed |
| 5 | Compliance Depth | Full flows for each regulated domain | Mentioned but shallow | Not addressed |
| 6 | API Design | Surface, versioning, conventions, errors, pagination | Some conventions missing | Not addressed |
| 7 | Integration Robustness | All externals with failure + fallback | Some missing fallbacks | Externals unnamed |
| 8 | Phasing Clarity | Dependency-ordered with entry/exit criteria | Phases listed, no criteria | No phasing |
| 9 | Engineering Standards | All thresholds concrete | Some TBDs | Missing |

## IA Rubric (8 dimensions)

| # | Dimension | ✅ | ⚠️ | ❌ |
|---|-----------|----|----|-----|
| 1 | Feature Enumeration | All listed, clear scope | Some vague | Major features missing |
| 2 | Access Model | All roles defined | Some implied | No access model |
| 3 | Data Model | Schema-ready | Structure unclear | Absent |
| 4 | User Flows | All paths covered | Happy path only | None |
| 5 | Cross-Shard Contracts | Bidirectional | One-way | None |
| 6 | Edge Cases | Listed | Some mentioned | None |
| 7 | Deep Dive Coverage | Integrated | Partial | Not reflected |
| 8 | Testability | All testable | Some subjective | Mostly subjective |

## BE Rubric (10 dimensions)

| # | Dimension | ✅ | ⚠️ | ❌ |
|---|-----------|----|----|-----|
| 1 | Upstream Traceability | All traceable | Some inferred | Orphan endpoints |
| 2 | Contract Completeness | All schemas complete | Some vague | Major missing |
| 3 | Error Exhaustiveness | All coded | Some missing | Vague |
| 4 | Schema Completeness | Production-ready | Constraints missing | Sketched only |
| 5 | Middleware Explicitness | Full matrix | Some missing | No matrix |
| 6 | State Transitions | Lifecycle defined | No transition rules | No lifecycle |
| 7 | Concurrency | Strategy defined | Mentioned | Not addressed |
| 8 | Pagination & Limits | All defined | Some missing | No strategy |
| 9 | Integration Seams | All documented | Some vague | Unnamed |
| 10 | Security Rules | All explicit | Some assumed | Not mentioned |

## FE Rubric (10 dimensions)

| # | Dimension | ✅ | ⚠️ | ❌ |
|---|-----------|----|----|-----|
| 1 | Upstream Traceability | All traceable | Some inferred | No mapping |
| 2 | Component Inventory | Full tree | Some missing | Vague |
| 3 | State Management | All mapped | Some missing | Unspecified |
| 4 | Interactions | All defined | Some implicit | Unspecified |
| 5 | Routing | All routes | Some guards missing | Vague |
| 6 | Responsive | All breakpoints | Mobile mentioned | Not addressed |
| 7 | Accessibility | WCAG-ready | Partial | Not addressed |
| 8 | Error/Loading States | All defined | Some missing | None |
| 9 | Performance | Targets set | Some mentioned | Not addressed |
| 10 | Security Rules | All explicit | Some assumed | Not mentioned |

## Scoring

- ✅ = 0 points, ⚠️ = 0.5 points, ❌ = 1 point
- `ambiguity% = (points / applicable_checkpoints) × 100`
