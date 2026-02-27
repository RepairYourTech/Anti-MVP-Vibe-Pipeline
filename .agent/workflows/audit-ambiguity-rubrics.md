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
| IA | `docs/plans/ia/index.md` + each shard listed + `docs/plans/ia/deep-dives/*.md` (list directory; include all files present) |
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

| # | Dimension | ✅ (must meet ALL criteria) | ⚠️ | ❌ |
|---|---|---|---|---|
| 1 | Problem Clarity | Single sentence. Names a specific user. Names a specific pain. Is falsifiable. | Vague, multi-problem, or untestable | Missing |
| 2 | Persona Specificity | Each persona has: name + role + specific pain point + current workaround + success criteria + switching trigger. No persona uses "users" or "customers" without a role. | Named but missing ≥1 required field | Missing or generic "users" |
| 3 | Feature Completeness | Every Must Have has ≥2 levels of sub-features. Every sub-feature has at least one edge case. MoSCoW categories are mutually exclusive. | Some Must Haves at Level 1 only, or missing edge cases | Missing MoSCoW or major features absent |
| 4 | Constraint Explicitness | Every axis (budget, timeline, team size, compliance, performance) has a specific value or explicit "not applicable". No axis uses "standard" or "TBD". | Some axes addressed, some missing or vague | Missing |
| 5 | Success Measurability | Every metric is a number with a unit and a timeframe (e.g., "p95 < 500ms at launch"). No metric uses "fast", "good", or "acceptable". | Directional only | Missing |
| 6 | Competitive Positioning | ≥3 named competitors. Differentiation is a specific capability gap. Moat is a defensible mechanism. | Vague positioning or <3 competitors | Not addressed |
| 7 | Open Question Resolution | Every open question has: owner + deadline + what decision it blocks. No question is listed without an owner. | Questions listed, no owners or deadlines | No questions section |

## Architecture Rubric (9 dimensions)

| # | Dimension | ✅ (must meet ALL criteria) | ⚠️ | ❌ |
|---|---|---|---|---|
| 1 | Tech Stack Decisiveness | Every applicable axis has a named technology + rationale + what alternatives were rejected and why. No axis uses "TBD" or "standard". | Some axes decided, some TBD or rationale missing | Major axes undecided |
| 2 | System Architecture | Every component is named. Every communication path has a protocol. Every component has a defined failure mode and fallback. Data flow is traced end-to-end with every hop named. | Some components or flows missing, or failure modes absent | Diagram only, no flows or failure modes |
| 3 | Data Strategy | Every entity has a named canonical store. Every hot query path is identified. Migration strategy names the tool and approach. PII fields are enumerated by name. | Some entities without canonical store, or PII not enumerated | Sketched only |
| 4 | Security Model | Auth flow is step-by-step. Every role has an explicit permission list. Rate limits are numbers (not "standard"). Input validation names the library and where it runs. | Some flows vague or rate limits unspecified | Not addressed |
| 5 | Compliance Depth | Every regulated domain (minors, payments, health) has its own section with: account type hierarchy, consent flows, content filtering rules, audit requirements. No compliance domain is a sub-bullet. | Mentioned but in a sub-bullet without full depth | Not addressed |
| 6 | API Design | Versioning strategy is named. Error format is defined (fields, types). Pagination strategy is named with parameters. Rate limit headers are specified. | Some conventions missing | Not addressed |
| 7 | Integration Robustness | Every external service has: what it provides + failure mode + fallback strategy + cost model. No external is listed without a fallback. | Some externals missing fallback or cost model | Externals unnamed |
| 8 | Phasing Clarity | Every phase has: entry criteria + exit criteria + dependency list. No phase uses "when ready" as a criterion. | Phases listed without entry/exit criteria | No phasing |
| 9 | Engineering Standards | Every threshold is a number (coverage %, response time ms, bundle size KB). No threshold uses "good", "acceptable", or "TBD". | Some thresholds concrete, some vague | Missing |

## IA Rubric (8 dimensions)

| # | Dimension | ✅ (must meet ALL criteria) | ⚠️ | ❌ |
|---|---|---|---|---|
| 1 | Feature Enumeration | Every feature from the vision's Must Have list appears in at least one shard. Every feature has a clear scope boundary (what's in, what's out). | Some features vague or scope unclear | Major Must Have features missing |
| 2 | Access Model | Every role has: a named role + an exhaustive list of what it CAN do + an exhaustive list of what it CANNOT do + escalation path. No role uses "standard access" without defining it. | Some roles defined but missing permission or restriction lists | No access model |
| 3 | Data Model | Every entity has: named fields + field types + constraints + relationships with cardinality. No field uses "standard" or "typical" without defining it. | Structure present but field types or constraints missing | Absent |
| 4 | User Flows | Every user flow has: trigger + step-by-step actions + system responses + error paths. No flow ends at "success" without defining what success looks like. | Happy path only, or flows missing error paths | None |
| 5 | Cross-Shard Contracts | Every cross-shard reference is bidirectional and cites the specific section in the referenced shard. No reference uses "see shard N" without a section name. | One-way references or missing section citations | None |
| 6 | Edge Cases | Every feature has ≥3 edge cases covering: concurrent access, invalid input, and deletion/cascade. No edge case uses "handle gracefully" without defining the handling. | Some edge cases listed but fewer than 3 per feature, or vague handling | None |
| 7 | Deep Dive Coverage | Every referenced deep dive file exists and contains exhaustive subsystem detail (technology choice + rationale + phasing + failure modes + integration contracts). No deep dive is a skeleton. | Some deep dives exist but are partial or skeleton | Referenced deep dives missing or empty |
| 8 | Testability | Every feature has ≥1 acceptance criterion in Given/When/Then format. No criterion uses "should work" or "should be fast" without a measurable threshold. | Some criteria present but not in Given/When/Then or missing thresholds | Mostly subjective |

> **Deep Dive audit note**: When auditing dimension 7, read each deep dive file directly — do not infer its completeness from the parent shard's reference to it. A parent shard that links to a deep dive scores ⚠️ (not ✅) if the deep dive file itself is still a skeleton. Score ✅ only when the deep dive file contains exhaustive subsystem detail that an implementer could work from without asking questions.

## BE Rubric (10 dimensions)

| # | Dimension | ✅ (must meet ALL criteria) | ⚠️ | ❌ |
|---|---|---|---|---|
| 1 | Upstream Traceability | Every endpoint cites its IA source shard + section. Every schema field traces to an IA data model field. No endpoint is "inferred" from context. | Some endpoints or fields missing IA citations | Orphan endpoints with no IA source |
| 2 | Contract Completeness | Every endpoint has: Zod request schema (all fields typed) + Zod response schema (all fields typed) + error schema. No field uses `any`, `unknown`, or untyped `string` where a more specific type exists. | Some schemas present but fields untyped or missing | Major schemas absent |
| 3 | Error Exhaustiveness | Every endpoint has: HTTP status code + application error code (not just HTTP) + error message format + retry guidance. No endpoint uses "standard error handling" without defining it. | Some endpoints missing error codes or retry guidance | Generic 500s or no error section |
| 4 | Schema Completeness | Every database table has: all fields with types + constraints (nullable, unique, indexed) + relationships with foreign keys + indexes for every query pattern. No field uses "standard" constraints without defining them. | Constraints or indexes missing | Sketched only |
| 5 | Middleware Explicitness | Every endpoint has an explicit middleware matrix: auth required (yes/no) + rate limit (number/window) + input validation (where it runs) + CORS policy. No endpoint inherits "default middleware" without defining it. | Some endpoints missing middleware entries | No matrix |
| 6 | State Transitions | Every stateful entity has a state machine: all states named + all valid transitions + what triggers each transition + what's blocked in each state. | Lifecycle mentioned but transitions or triggers missing | No lifecycle |
| 7 | Concurrency | Every write operation has an explicit concurrency strategy: optimistic locking, pessimistic locking, idempotency key, or "single writer" with justification. No operation uses "handle concurrency" without a named strategy. | Mentioned but strategy not named | Not addressed |
| 8 | Pagination & Limits | Every list endpoint has: pagination strategy (cursor/offset) + page size (default + max) + sort options + filter options. No endpoint uses "standard pagination" without defining it. | Some list endpoints missing pagination or limits | No strategy |
| 9 | Integration Seams | Every external service call has: request format + response format + timeout (ms) + retry policy (count + backoff) + circuit breaker behavior. No integration uses "standard retry" without defining it. | Some integrations missing timeout or retry policy | Unnamed externals |
| 10 | Security Rules | Every endpoint has explicit: authentication requirement + authorization check (which roles) + input sanitization + output filtering (what's excluded from response). No endpoint uses "standard security" without defining it. | Some endpoints missing auth or authz specification | Not mentioned |

## FE Rubric (10 dimensions)

| # | Dimension | ✅ (must meet ALL criteria) | ⚠️ | ❌ |
|---|---|---|---|---|
| 1 | Upstream Traceability | Every component cites its BE source spec + field. Every page cites its IA user flow. No component is "inferred" from context. | Some components or pages missing source citations | No mapping |
| 2 | Component Inventory | Every component has: name + props interface (all props typed) + children + variants. No component uses "standard props" or "typical interface". | Some components missing props or variants | Vague component list |
| 3 | State Management | Every piece of state is classified: server state (which query) + client state (which store) + URL state (which param). Loading, error, and empty states are defined for every async operation. | Some state unclassified or async states missing | Unspecified |
| 4 | Interactions | Every interactive element has: trigger + action + feedback (visual + timing). Every form has: validation rules + error display + submission behavior + success behavior. | Some interactions implicit or missing feedback | Unspecified |
| 5 | Routing | Every route has: URL pattern + auth guard (yes/no + redirect target) + page component + meta (title, description). No route uses "standard guard" without defining it. | Some routes missing guards or meta | Vague |
| 6 | Responsive | Every component has behavior defined at: mobile (≤768px) + tablet (769–1024px) + desktop (≥1025px). No component uses "responsive" without defining the behavior at each breakpoint. | Mobile mentioned but tablet/desktop missing | Not addressed |
| 7 | Accessibility | Every interactive element has: ARIA role + keyboard navigation + screen reader label. Every image has alt text policy. WCAG level is named. | Partial — some elements missing ARIA or keyboard nav | Not addressed |
| 8 | Error/Loading States | Every data-fetching view has: loading skeleton/spinner + error message + retry action + empty state. No state uses "standard loading" without defining it. | Some states missing or using generic messages | None |
| 9 | Performance | Every page has: bundle size budget + lazy loading strategy for heavy components + image optimization policy. No page uses "optimized" without a number. | Some pages missing budgets or lazy loading strategy | Not addressed |
| 10 | Security Rules | Every form has: CSRF protection + input sanitization + output encoding. Every auth-gated view has: token validation + expiry handling + redirect on failure. | Some forms or views missing security rules | Not mentioned |

## Scoring

- ✅ = 0 points, ⚠️ = 0.5 points, ❌ = 1 point
- `ambiguity% = (points / applicable_checkpoints) × 100`
- Note: Gaps found during Implementer Simulation (Step 3a) are added directly to the punch list as ❌ items and also count toward the ambiguity score.
