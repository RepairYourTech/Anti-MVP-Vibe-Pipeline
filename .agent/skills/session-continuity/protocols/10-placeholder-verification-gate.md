# Placeholder Verification Gate Protocol

## Overview

Specification workflows that read `{{PLACEHOLDER}}`-dependent skill paths must verify those placeholders are filled **before** any skill reads. This prevents spec authoring from running with unfilled stack placeholders, which would produce patterns incompatible with the chosen tech stack.

## When to Use

Add a **Step 0 — Placeholder guard** to any specification workflow that reads `{{PLACEHOLDER}}` values as skill paths. The guard runs before all other steps, including re-run checks and prerequisite validations.

## Key Constraint

**No auto-refire of bootstrap.** The agent stops and tells the user exactly what to run. The guard does not attempt to call `/bootstrap-agents` itself — it emits a HARD STOP with the exact recovery command.

## Four-Part Hard Stop Message Structure

When any placeholder contains literal `{{` characters, emit this message for **each** unfilled placeholder:

> ❌ **Bootstrap incomplete — cannot proceed.**
>
> **Unfilled placeholder:** `{{PLACEHOLDER_NAME}}`
>
> **Recovery:** If `docs/plans/*-architecture-design.md` exists, read it and extract the confirmed [tech decision] value, then run `/bootstrap-agents` with `KEY=<confirmed-value>`. If no architecture design document exists, run `/create-prd-stack` first to confirm tech stack decisions.
>
> **Why this matters:** [specific step] cannot produce correct output without this skill — [concrete consequence of proceeding without it].

---

## Per-Workflow Placeholder Mappings

### `create-prd-architecture`

**Frontmatter:** `requires_placeholders: [HOSTING_SKILL, ORM_SKILL, DATABASE_SKILLS]`

| Placeholder | Filled by | Recovery | Why this matters |
|---|---|---|---|
| `{{HOSTING_SKILL}}` | `/create-prd-stack` when hosting provider is confirmed | If `docs/plans/*-architecture-design.md` exists, read it and extract the confirmed hosting provider, then run `/bootstrap-agents HOSTING=<confirmed-provider>`. Otherwise run `/create-prd-stack` first. | Data strategy and schema design steps (Steps 4 and 5) cannot run without the hosting skill — deployment topology decisions will lack provider-specific conventions. |
| `{{ORM_SKILL}}` | `/create-prd-stack` when ORM is confirmed | If `docs/plans/*-architecture-design.md` exists, read it and extract the confirmed ORM, then run `/bootstrap-agents ORM=<confirmed-orm>`. Otherwise run `/create-prd-stack` first. | Migration strategy (Step 5.4) cannot run without the ORM skill — schema conventions and migration patterns will be generic instead of stack-specific. |
| `{{DATABASE_SKILLS}}` | `/create-prd-stack` when database technology is confirmed | If `docs/plans/*-architecture-design.md` exists, read it and extract the confirmed database, then run `/bootstrap-agents DATABASE=<confirmed-db>`. Otherwise run `/create-prd-stack` first. | Data strategy (Step 5) cannot run without the database skill — schema design patterns will be incompatible with the chosen database. |

### `write-architecture-spec-design`

**Frontmatter:** `requires_placeholders: [DATABASE_SKILLS, SECURITY_SKILLS]`

| Placeholder | Filled by | Recovery | Why this matters |
|---|---|---|---|
| `{{DATABASE_SKILLS}}` | `/create-prd-stack` when database technology is confirmed | If `docs/plans/*-architecture-design.md` exists, read it and extract the confirmed database, then run `/bootstrap-agents DATABASE=<confirmed-db>`. Otherwise run `/create-prd-stack` first. | Data model design (Step 4) would produce schema patterns incompatible with the chosen database. |
| `{{SECURITY_SKILLS}}` | `/create-prd-stack` when security tooling is confirmed | If `docs/plans/*-architecture-design.md` exists, read it and extract the confirmed security framework, then run `/bootstrap-agents SECURITY=<confirmed-framework>`. Otherwise run `/create-prd-stack` first. | Edge case and attack surface analysis (Step 7) would run without the project's security skill, missing stack-specific threat vectors. |

### `write-fe-spec-classify`

**Frontmatter:** `requires_placeholders: [LANGUAGE_SKILL, FRONTEND_FRAMEWORK_SKILL, FRONTEND_DESIGN_SKILL, ACCESSIBILITY_SKILL, STATE_MANAGEMENT_SKILL]`

| Placeholder | Filled by | Recovery | Why this matters |
|---|---|---|---|
| `{{LANGUAGE_SKILL}}` | `/create-prd-stack` when the primary language is confirmed | If `docs/plans/*-architecture-design.md` exists, read it and extract the confirmed language, then run `/bootstrap-agents LANGUAGE=<confirmed-language>`. Otherwise run `/create-prd-stack` first. | FE spec components would be written without the correct language skill, producing patterns incompatible with the chosen language. |
| `{{FRONTEND_FRAMEWORK_SKILL}}` | `/create-prd-stack` when the frontend framework is confirmed | If `docs/plans/*-architecture-design.md` exists, read it and extract the confirmed frontend framework, then run `/bootstrap-agents FRONTEND_FRAMEWORK=<confirmed-framework>`. Otherwise run `/create-prd-stack` first. | FE spec components would be written without the correct framework skill, producing patterns incompatible with the chosen framework. |
| `{{FRONTEND_DESIGN_SKILL}}` | `/create-prd-stack` when the frontend design system is confirmed | If `docs/plans/*-architecture-design.md` exists, read it and extract the confirmed design system, then run `/bootstrap-agents FRONTEND_DESIGN=<confirmed-design>`. Otherwise run `/create-prd-stack` first. | FE spec components would be written without the correct design skill, producing visual patterns incompatible with the chosen design system. |
| `{{ACCESSIBILITY_SKILL}}` | `/create-prd-stack` when accessibility tooling is confirmed | If `docs/plans/*-architecture-design.md` exists, read it and extract the confirmed accessibility tooling, then run `/bootstrap-agents ACCESSIBILITY=<confirmed-tooling>`. Otherwise run `/create-prd-stack` first. | FE spec components would lack stack-specific accessibility patterns and WCAG implementation guidance. |
| `{{STATE_MANAGEMENT_SKILL}}` | `/create-prd-stack` when state management is confirmed | If `docs/plans/*-architecture-design.md` exists, read it and extract the confirmed state management library, then run `/bootstrap-agents STATE_MANAGEMENT=<confirmed-library>`. Otherwise run `/create-prd-stack` first. | FE spec components would be written without the correct state management conventions, producing patterns incompatible with the chosen state library. |

---

## Reference Implementation

`write-be-spec-classify.md` Step 2.5 is the canonical example of a correctly implemented placeholder guard. It follows the exact four-part message structure and demonstrates the scan-then-stop pattern.

---

## Additional Per-Workflow Placeholder Mappings

### `create-prd-security`

**Frontmatter:** `requires_placeholders: [SECURITY_SKILLS, AUTH_SKILL]`

| Placeholder | Filled by | Recovery | Why this matters |
|---|---|---|---|
| `{{SECURITY_SKILLS}}` | `/create-prd-stack` when security tooling is confirmed | If `docs/plans/*-architecture-design.md` exists, read it and extract the confirmed security framework, then run `/bootstrap-agents SECURITY=<confirmed-framework>`. Otherwise run `/create-prd-stack` first. | Security model (Step 6) cannot produce stack-specific threat analysis without the security skill — the model will miss framework-specific attack vectors. |
| `{{AUTH_SKILL}}` | `/create-prd-stack` when auth provider is confirmed | If `docs/plans/*-architecture-design.md` exists, read it and extract the confirmed auth provider, then run `/bootstrap-agents AUTH=<confirmed-provider>`. Otherwise run `/create-prd-stack` first. | Authentication design (Step 6.1) cannot produce correct auth flows without the auth skill — identity provider conventions and token handling will be generic. |

### `create-prd-compile`

**Frontmatter:** `requires_placeholders: [UNIT_TESTING_SKILL, E2E_TESTING_SKILL, CI_CD_SKILL]`

| Placeholder | Filled by | Recovery | Why this matters |
|---|---|---|---|
| `{{UNIT_TESTING_SKILL}}` | `/create-prd-stack` when the unit testing framework is confirmed | If `docs/plans/*-architecture-design.md` exists, read it and extract the confirmed unit testing framework, then run `/bootstrap-agents UNIT_TESTING=<confirmed-framework>`. Otherwise run `/create-prd-stack` first. | Development methodology (Step 8) cannot document correct test conventions without the unit testing skill — TDD patterns will be generic instead of framework-specific. |
| `{{E2E_TESTING_SKILL}}` | `/create-prd-stack` when the E2E testing framework is confirmed | If `docs/plans/*-architecture-design.md` exists, read it and extract the confirmed E2E testing framework, then run `/bootstrap-agents E2E_TESTING=<confirmed-framework>`. Otherwise run `/create-prd-stack` first. | Development methodology (Step 8) cannot document correct E2E test conventions without the E2E testing skill — integration test patterns will be generic. |
| `{{CI_CD_SKILL}}` | `/create-prd-stack` when CI/CD platform is confirmed | If `docs/plans/*-architecture-design.md` exists, read it and extract the confirmed CI/CD platform, then run `/bootstrap-agents CI_CD=<confirmed-platform>`. Otherwise run `/create-prd-stack` first. | Phasing strategy (Step 9) cannot produce correct pipeline configuration without the CI/CD skill — deployment and quality gate patterns will lack platform-specific conventions. |

### `write-be-spec-classify`

**Frontmatter:** `requires_placeholders: [LANGUAGE_SKILL, DATABASE_SKILLS, AUTH_SKILL, BACKEND_FRAMEWORK_SKILL, ORM_SKILL, UNIT_TESTING_SKILL]`

| Placeholder | Filled by | Recovery | Why this matters |
|---|---|---|---|
| `{{LANGUAGE_SKILL}}` | `/create-prd-stack` when the primary language is confirmed | If `docs/plans/*-architecture-design.md` exists, read it and extract the confirmed language, then run `/bootstrap-agents LANGUAGE=<confirmed-language>`. Otherwise run `/create-prd-stack` first. | BE spec code conventions would be written without the correct language skill, producing patterns incompatible with the chosen language. |
| `{{DATABASE_SKILLS}}` | `/create-prd-stack` when database technology is confirmed | If `docs/plans/*-architecture-design.md` exists, read it and extract the confirmed database, then run `/bootstrap-agents DATABASE=<confirmed-db>`. Otherwise run `/create-prd-stack` first. | Schema design and query patterns would be incompatible with the chosen database. |
| `{{AUTH_SKILL}}` | `/create-prd-stack` when auth provider is confirmed | If `docs/plans/*-architecture-design.md` exists, read it and extract the confirmed auth provider, then run `/bootstrap-agents AUTH=<confirmed-provider>`. Otherwise run `/create-prd-stack` first. | Authentication middleware and token handling would use generic patterns instead of provider-specific conventions. |
| `{{BACKEND_FRAMEWORK_SKILL}}` | `/create-prd-stack` when the backend framework is confirmed | If `docs/plans/*-architecture-design.md` exists, read it and extract the confirmed backend framework, then run `/bootstrap-agents BACKEND_FRAMEWORK=<confirmed-framework>`. Otherwise run `/create-prd-stack` first. | Route definitions, middleware patterns, and request handling would be framework-agnostic instead of following confirmed framework conventions. |
| `{{ORM_SKILL}}` | `/create-prd-stack` when ORM is confirmed | If `docs/plans/*-architecture-design.md` exists, read it and extract the confirmed ORM, then run `/bootstrap-agents ORM=<confirmed-orm>`. Otherwise run `/create-prd-stack` first. | Migration patterns and schema conventions would be generic instead of ORM-specific. |
| `{{UNIT_TESTING_SKILL}}` | `/create-prd-stack` when the unit testing framework is confirmed | If `docs/plans/*-architecture-design.md` exists, read it and extract the confirmed unit testing framework, then run `/bootstrap-agents UNIT_TESTING=<confirmed-framework>`. Otherwise run `/create-prd-stack` first. | Test writing conventions would use generic patterns instead of framework-specific assertions and utilities. |
