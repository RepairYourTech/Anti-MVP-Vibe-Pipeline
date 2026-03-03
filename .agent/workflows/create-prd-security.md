---
description: Security model and integration points for the create-prd workflow
parent: create-prd
shard: security
standalone: true
position: 3
pipeline:
  position: 2.3
  stage: architecture
  predecessors: [create-prd-architecture]
  successors: [create-prd-compile]
  skills: [security-scanning-security-hardening, resolve-ambiguity, logging-best-practices]
  calls-bootstrap: true
---

// turbo-all

# Create PRD — Security & Integrations

Define the security model with full compliance escalation, and document all integration points with failure modes and fallbacks.

**Prerequisite**: System architecture and data strategy must be designed (from `/create-prd-architecture`). The agent should have component diagrams, data placement, and PII boundaries established.

---

## 6. Security model

Read .agent/skills/security-scanning-security-hardening/SKILL.md and follow its defense-in-depth methodology.

Using `{{AUTH_SKILL}}`:

Read each skill listed in `{{SECURITY_SKILLS}}` (comma-separated). For each skill directory name, read `.agent/skills/[skill]/SKILL.md` before proceeding.

1. **Authentication** — How do users prove identity?
2. **Authorization** — RBAC vs ABAC? Permission model?
3. **Data protection** — PII handling, encryption at rest/transit
4. **Input validation** — Where and how? (Zod recommended for TypeScript)
5. **Age restrictions** — If applicable: age verification model
6. **Rate limiting** — Per-route? Per-user? Per-IP?
7. **CSP/CORS** — Content Security Policy, allowed origins (web surfaces)
8. **Code signing** — App signing certificates, notarization (desktop/mobile surfaces)

### Compliance escalation

If any compliance constraint from `vision.md` involves **minors, payments, health data, or government-regulated domains**, it is NOT a sub-bullet — it becomes its own top-level section in the architecture document with the same depth as System Architecture. For example:

- **Minors/COPPA/age-gating** → requires: account type hierarchy, consent flows, content filtering architecture, Guardian oversight model, age verification mechanism, blocked content categories, notification system for filter triggers
- **Payments/PCI** → requires: token handling architecture, PCI scope boundaries, payment name verification, refund flows, dispute handling
- **Health data/HIPAA** → requires: PHI isolation boundaries, audit logging, access controls, breach notification procedures

Each of these must be specified with the same depth as any other architectural component — field-level detail, flow diagrams, error cases, and permission boundaries. A single bullet saying "age verification model" is not architecture.

Read .agent/skills/resolve-ambiguity/SKILL.md and follow its methodology.

**Present to user**: Show the security model and any compliance sections. Walk through each flow step by step. Ask:
- "Can you think of a way to bypass any of these controls?"
- "Are there edge cases in the age/payment/compliance flows I haven't covered?"

Refine based on discussion before proceeding.

**Bootstrap fire — security decision confirmed**

If the security model confirmed a specific security framework or compliance approach (e.g., crypto patterns, custom HSM approach), read `.agent/workflows/bootstrap-agents.md` and invoke `/bootstrap-agents SECURITY=[confirmed value]` to provision additional skills. Note: surface-triggered security skills (`owasp-web-security`, `csp-cors-headers`, `input-sanitization`, `api-security-checklist`, `dependency-auditing`, `desktop-security-sandboxing`) are provisioned automatically by bootstrap when surfaces are confirmed in `/create-prd-stack` — no manual fire needed for those.

## 6.5. Attack Surface Review

Read .agent/skills/security-scanning-security-hardening/SKILL.md and follow its attack surface analysis methodology.

Using the security model from Step 6 and the confirmed surfaces from `/create-prd-stack`, review each applicable attack surface:

### Universal checks (all projects)

1. **Secret management** — Where are secrets stored? (env vars, vault, cloud secret manager). What is the access policy? What is the rotation cadence? How are secrets injected in CI (environment variables, OIDC, sealed secrets)? Document under `## Security — Attack Surface > Secret Management`.
2. **Dependency audit cadence** — How often are dependencies scanned? What CI gate behaviour on critical CVE? (block merge, warn, ignore). What is the triage policy for high-severity CVEs (patch within N days) and medium-severity CVEs (patch within N days or accept-with-justification)? Document under `## Security — Attack Surface > Dependency Auditing`.

### Web surface checks (if web surface confirmed)

1. **OWASP Top 10 review** — For each of the 10 categories, name the specific mechanism that addresses it. "Handled by framework" is not acceptable — name the framework feature, the configuration, and the fallback if the framework is bypassed. Document under `## Security — Attack Surface > Web > OWASP Top 10`.
2. **Security headers** — Document configured values for: `Content-Security-Policy`, `Strict-Transport-Security`, `X-Content-Type-Options`, `X-Frame-Options`, `Referrer-Policy`, `Permissions-Policy`. Each header must have a specific value, not just "enabled". Document under `## Security — Attack Surface > Web > Security Headers`.

### API surface checks (if API surface confirmed)

1. **OWASP API Security Top 10** — For each category, name the mechanism. Pay special attention to BOLA/IDOR — document the per-endpoint ownership check strategy (how does each endpoint verify the requesting user owns the requested resource?). Document under `## Security — Attack Surface > API > OWASP API Top 10`.

### Desktop surface checks (if desktop surface confirmed)

1. **Sandboxing model** — Document the sandboxing strategy, IPC security boundaries, auto-update verification, and code signing chain.
2. **Notarization** — Document the notarization workflow (macOS notarization, Windows SmartScreen signing). Name the signing certificate source and CI step that performs notarization.
3. **Local data encryption** — Document the strategy for encrypting sensitive data at rest on the user's machine (encryption library, key storage mechanism, what data is encrypted). Document under `## Security — Attack Surface > Desktop`.

### Mobile surface checks (if mobile surface confirmed)

1. **Mobile-specific threats** — Document certificate pinning strategy, secure storage approach, jailbreak/root detection policy, and deep link validation. Document under `## Security — Attack Surface > Mobile`.

**Present to user**: Show the attack surface review findings. Ask:
- "Are there any attack vectors I've missed for your specific domain?"
- "Do the OWASP mechanisms look correct, or are any of them actually handled differently?"

## 7. Integration points

For each external service:

1. **What it provides** — Specific features used
2. **Failure mode** — What happens when it's down?
3. **Fallback strategy** — Graceful degradation plan
4. **Cost model** — Pricing tier, expected usage

## 7.5. Observability Architecture

Read .agent/skills/logging-best-practices/SKILL.md and follow its structured logging methodology.

Design the observability architecture for the project. Each decision must be confirmed before proceeding:

1. **Logging strategy** — Name the logging library. Structured JSON in production (yes/no)? Log levels per environment (dev: debug, staging: info, prod: warn). PII field names that are never logged (enumerate explicitly). Log destination (stdout, file, cloud service — name it). When logging is confirmed, always fire `/bootstrap-agents OBSERVABILITY=structured-logging` first to provision baseline logging guidance. If the confirmed library or stack maps to an additional observability tool (e.g., Datadog, OpenTelemetry, Pino), also fire `/bootstrap-agents OBSERVABILITY=[tool-specific value]`.

2. **Tracing strategy** — Which service boundaries are traced? Sampling rate per environment. Trace ID propagation to API clients (header name). If a specific tracing tool is confirmed, invoke `/bootstrap-agents OBSERVABILITY=[confirmed value]`.

3. **Alerting thresholds** — Error rate percentage that triggers alert. Latency threshold (ms) + duration before alert. Queue depth warning level. Delivery mechanism (PagerDuty, Slack, email — name it). If a specific monitoring tool is confirmed, invoke `/bootstrap-agents MONITORING=[confirmed value]`.

4. **Launch dashboards** — Minimum required panels (name each). Tool (Grafana, Datadog, CloudWatch — name it). Dashboard owner (role, not person).

5. **Retention** — Log retention duration. Trace retention duration. Compliance alignment (if applicable).

**Present to user**: Show the observability architecture decisions. Ask:
- "Are these logging levels and PII exclusions correct for your compliance requirements?"
- "Are the alerting thresholds appropriate for your expected traffic?"

After the security model (Step 6) is completed and confirmed, write the `## Security Model` section to `docs/plans/architecture-draft.md`. After the attack surface review (Step 6.5) is completed and confirmed, write the `## Security — Attack Surface` section to `docs/plans/architecture-draft.md`. After the integration points (Step 7) are completed and confirmed, write the `## Integration Points` section to `docs/plans/architecture-draft.md`. After the observability architecture (Step 7.5) is completed and confirmed, write the `## Observability Architecture` section to `docs/plans/architecture-draft.md`. Do not wait until the end — write each section as it is completed.

### Propose next step

Security model and integration points are defined. Next: Run `/create-prd-compile` to document the development methodology, phasing strategy, and compile the final architecture design document and Engineering Standards.

> If this shard was invoked standalone (not from `/create-prd`), surface this via `notify_user`.

