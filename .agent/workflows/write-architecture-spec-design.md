---
description: Explore requirements, map interactions, and design contracts/data models/access control for the write-architecture-spec workflow
parent: write-architecture-spec
shard: design
standalone: true
position: 1
pipeline:
  position: 4.1
  stage: specification
  predecessors: [decompose-architecture]
  successors: [write-architecture-spec-deepen]
  skills: [brainstorming, resolve-ambiguity, database-schema-design]
  calls-bootstrap: false
requires_placeholders: [DATABASE_SKILLS, SECURITY_SKILLS]
---

// turbo-all

# Write Architecture Spec — Design

Explore requirements, map all interactions, and define contracts, data models, access control, event schemas, and edge cases.

**Prerequisite**: Skeleton IA shard must exist in `docs/plans/ia/`. If it does not, tell the user to run `/decompose-architecture` first.

## 0. Placeholder guard

Before any skill reads, verify that the following placeholder values have been filled by `/bootstrap-agents`. For each placeholder, check whether the literal characters `{{` still appear in the value. If **any** are unfilled, emit a **HARD STOP** and do not proceed to Step 0.5.

| Placeholder | Filled by | Recovery | Why this matters |
|---|---|---|---|
| `{{DATABASE_SKILLS}}` | `/create-prd-stack` when database technology is confirmed | If `docs/plans/*-architecture-design.md` exists, read it and extract the confirmed database, then run `/bootstrap-agents DATABASE=<confirmed-db>`. Otherwise run `/create-prd-stack` first. | Data model design (Step 4) would produce schema patterns incompatible with the chosen database. |
| `{{SECURITY_SKILLS}}` | `/create-prd-stack` when security tooling is confirmed | If `docs/plans/*-architecture-design.md` exists, read it and extract the confirmed security framework, then run `/bootstrap-agents SECURITY=<confirmed-framework>`. Otherwise run `/create-prd-stack` first. | Edge case and attack surface analysis (Step 7) would run without the project's security skill, missing stack-specific threat vectors. |

**Hard stop message format** (emit one block per unfilled placeholder):

> ❌ **Bootstrap incomplete — cannot proceed.**
>
> **Unfilled placeholder:** `{{PLACEHOLDER_NAME}}`
>
> **Recovery:** If `docs/plans/*-architecture-design.md` exists, read it and extract the confirmed [tech decision] value, then run `/bootstrap-agents` with `KEY=<confirmed-value>`. If no architecture design document exists, run `/create-prd-stack` first to confirm tech stack decisions.
>
> **Why this matters:** [specific step] cannot produce correct output without this skill — [concrete consequence of proceeding without it].

Only proceed to Step 0.5 when both placeholders report no literal `{{` characters.

---

## 0.5. Re-run check

Before loading skills, check whether the shard file at `docs/plans/ia/[shard-name].md` already has content beyond skeleton placeholders (look for filled-in sections vs empty `<!-- TODO -->` markers).

- **If sections are already filled**: Present the current state of the file and ask: "Some sections are already written. Do you want to **continue from where you left off** (skip filled sections), or **redo specific sections** (tell me which ones)?" Wait for the user's answer before proceeding.
- **If the file is still a skeleton**: Proceed normally.

---

## 1. Explore requirements

### 1a. Read the authoritative sources

Read the following files and build a **reconciliation table** comparing what each source says about this shard's features. `docs/plans/ideation.md` is the **primary source of truth** for sub-features — the architecture design is secondary context.

1. `docs/plans/ideation.md` — the primary sub-feature inventory (locate the section for this shard's domain)
2. The shard's `## Features` section (seeded during `/decompose-architecture-structure`)
3. `docs/plans/vision.md` — Must Have features relevant to this domain

Read .agent/skills/architecture-mapping/SKILL.md and follow its Sub-Feature Reconciliation Protocol. Build the reconciliation table comparing ideation.md against the shard skeleton and apply mismatch handling rules before proceeding to Step 1b.

### 1b. Scope confirmation

Present the reconciled `## Features` list to the user, including a count of newly added sub-features:

> **Reconciled features for [Shard NN — Domain Name]:**
> 
> [bullet list of all sub-features, with `[Architecture-only]` markers where applicable]
> 
> **[N] sub-features added from ideation.md** that were missing from the shard skeleton.
> **[M] sub-features marked `[Architecture-only]`** — not found in ideation.md, added during decomposition.
> 
> "Does this feature list match your intent for this domain? Any sub-features to add, remove, or re-scope?"

**Wait for explicit user confirmation before proceeding.** If the user modifies the list, update the shard's `## Features` section in `docs/plans/ia/[shard-name].md` immediately.

### 1c. Load brainstorming skill

Read `.agent/skills/brainstorming/SKILL.md` and use it to explore any remaining ambiguous sub-features — those marked `[Architecture-only]` that the user hasn't explicitly confirmed, or any sub-feature whose scope boundary (what's in, what's out) is still unclear after 1b.

## 2. Map all interactions

For each feature in the shard, document:
- User action → API call → data mutation → response
- Which surfaces are involved (public, dashboard, admin, API)
- What events are emitted
- Error states and edge cases

**Present to user**: Show the interaction map and ask:
- "Does this capture all the ways a user touches this domain?"
- "Are there admin/system-initiated actions I'm missing?"
- "What happens in each failure case?"

Refine based on discussion before proceeding.

> **Write now**: Write the completed `## User Interactions` content to `docs/plans/ia/[shard-name].md`, replacing the skeleton placeholder for this section. Do not wait until Step 8.

## 3. Define contracts

Read .agent/skills/rest-api-design/SKILL.md and follow its methodology.

For each interaction, define the contract shape:
- Request shape (params, query, body)
- Response shape (all fields typed)
- Error shape (specific error codes)
- Note: actual Zod schemas written in BE spec phase

**Present to user**: Show the contract shapes and ask:
- "Are there fields I'm missing from these requests/responses?"
- "Are these error codes specific enough?"

Refine based on discussion before proceeding.

> **Write now**: Write the completed `## Contracts` content to `docs/plans/ia/[shard-name].md`, replacing the skeleton placeholder for this section. Do not wait until Step 8.

## 4. Design data models

Read each skill listed in `{{DATABASE_SKILLS}}` (comma-separated). For each skill directory name, read `.agent/skills/[skill]/SKILL.md` before proceeding. Also load these community skills for guidance:
- `.agent/skills/database-schema-design/SKILL.md` — Schema design principles
- `.agent/skills/error-handling-patterns/SKILL.md` — Error categories for contracts
- `.agent/skills/technical-writer/SKILL.md` — Specification clarity
- Tables/collections, fields, types
- Relationships (graph edges, foreign keys, etc.)
- Indexes for query patterns
- Constraints and validation rules

**Present to user**: Show the data model and ask:
- "Does this schema capture everything this domain needs to store?"
- "Are the relationships and cardinalities correct?"
- "Are there derived/computed fields I should account for?"

Refine based on discussion before proceeding.

> **Write now**: Write the completed `## Data Model` content to `docs/plans/ia/[shard-name].md`, replacing the skeleton placeholder for this section. Do not wait until Step 8.

## 5. Design access control

Read .agent/skills/security-scanning-security-hardening/SKILL.md and apply its access control and authorization design methodology.

- Permission matrix (who can read/write/delete what)
- Role-based restrictions
- Age restrictions (if applicable)
- Escalation paths (Guardian, Admin)
- Admin-only operations

**Present to user**: Show the permission matrix and ask:
- "Can you think of a scenario where a user should be blocked that this matrix allows?"
- "Can you think of a scenario where a user should be allowed that this matrix blocks?"

Refine based on discussion before proceeding.

> **Write now**: Write the completed `## Access Control` content to `docs/plans/ia/[shard-name].md`, replacing the skeleton placeholder for this section. Do not wait until Step 8.

## 5.5. Accessibility specifications

Read `{{SURFACES}}` to determine the project's target surfaces.

**If surfaces include `web`, `mobile`, or `desktop`:**

Read `.agent/skills/accessibility/SKILL.md` and apply its WCAG 2.1 AA methodology.

For each user interaction documented in Step 2 (`## User Interactions`), identify and document:
- **Keyboard navigation path** — tab order and focus management for this interaction
- **Screen reader semantics** — ARIA roles, labels, and live regions required
- **Color contrast** — requirements for any new visual states (loading, error, empty, disabled)
- **Motion/animation** — `prefers-reduced-motion` implications for any transitions or animations
- **Touch target sizes** — minimum 44×44 px (mobile surfaces only)

**Present to user**: Show the accessibility specifications and ask:
- "Are there any interactions in this domain that could be problematic for keyboard-only users?"
- "Are there any visual states (loading, error, empty, disabled) that need specific ARIA announcements?"
- For mobile: "Do any touch targets fall below 44×44px minimum?"

Refine based on discussion before proceeding.

**If surfaces are `api`, `cli`, or `extension` only:**

Write `"Not applicable — no visual surfaces"` in the `## Accessibility` section and proceed directly to the Write now instruction.

> **Write now**: Write the completed `## Accessibility` content to `docs/plans/ia/[shard-name].md`, replacing the skeleton placeholder for this section. Do not wait until Step 8.

## 6. Design event schemas (if applicable)

- Event name, payload shape, emitter, consumers
- Async vs sync processing
- Retry semantics

> **Write now**: Write the completed `## Event Schemas` content to `docs/plans/ia/[shard-name].md`, replacing the skeleton placeholder for this section. Do not wait until Step 8.

## 7. Document edge cases

Read .agent/skills/resolve-ambiguity/SKILL.md and follow its methodology.

Read each skill listed in `{{SECURITY_SKILLS}}` (comma-separated). For each skill directory name, read `.agent/skills/[skill]/SKILL.md` before proceeding.
For each skill in {{SECURITY_SKILLS}}, follow its attack surface methodology for edge case identification.

- Rate limits and abuse scenarios
- Concurrent access handling
- Deletion cascades
- State transition conflicts
- Empty/null states

**Present to user**: Show the edge cases and ask:
- "What's the worst thing a malicious user could try in this domain?"
- "What happens if two users do the same thing at the same time?"
- "What happens when related data is deleted?"

Refine based on discussion before proceeding.

> **Write now**: Write the completed `## Edge Cases` content to `docs/plans/ia/[shard-name].md`, replacing the skeleton placeholder for this section. Do not wait until Step 8.

## 7.5. Write deep dive files (if applicable)

Check whether the current shard references any deep dive candidates. To do this:
1. Read the shard file at `docs/plans/ia/[shard-name].md` and look for any links to `docs/plans/ia/deep-dives/` files in the "Deep Dives Needed" section or anywhere in the document.
2. List the files in `docs/plans/ia/deep-dives/` and identify which ones are referenced by this shard.

**If no deep dives are referenced**: Skip this step entirely.

**If deep dives are referenced**: For each referenced deep dive file:

1. Read the current content of `docs/plans/ia/deep-dives/[feature-name].md` to check if it is still a skeleton (empty or contains only placeholder comments).
2. **If already has full content**: Skip — do not overwrite.
3. **If still a skeleton**: Write the exhaustive detail for this subsystem to the deep dive file. The deep dive should contain everything that is too complex or too long to live in the parent shard:
   - Detailed algorithm or state machine description
   - Technology choices with rationale (e.g., why this protocol, why this library)
   - Phasing strategy (what ships in Phase 1 vs later)
   - Detailed data schemas specific to this subsystem
   - Failure modes and recovery strategies
   - Integration contracts with external systems
   - Performance characteristics and limits
   - Security considerations specific to this subsystem

   The parent shard's "Deep Dives Needed" section should already contain a summary + link. Do not duplicate the full content in the parent shard — the deep dive file IS the content.

> **Write now**: Write the completed deep dive content to `docs/plans/ia/deep-dives/[feature-name].md`. Do not wait until Step 8.

## 8. Present all sections and request approval

All sections are now written to `docs/plans/ia/[shard-name].md`. Please review the file directly and confirm it's ready for deepening passes.

> **Do NOT proceed to `/write-architecture-spec-deepen` until the user approves all sections. Proposing next steps is not the same as receiving approval.**

Once approved, run `/write-architecture-spec-deepen`.

> **Note**: The deepen shard will run iterative passes over the approved content — the user does not need to re-approve individual sections during deepening.

