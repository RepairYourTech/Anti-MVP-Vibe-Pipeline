---
description: Domain mapping, problem exploration, and feature inventory for the ideate workflow
parent: ideate
shard: discover
standalone: true
position: 2
pipeline:
  position: 1.2
  stage: vision
  predecessors: [ideate-extract]
  successors: [ideate-validate]
  skills: [idea-extraction, resolve-ambiguity]
  calls-bootstrap: false
---

// turbo-all

# Ideate — Discover

Map domains, explore the problem space, and build the feature inventory.

**Prerequisite**: If invoked standalone, verify the user has already classified their input type (via `/ideate-extract` or by providing context). If not, prompt the user to run `/ideate-extract` first or describe what they want to build.

---

## 3. Domain mapping

Read .agent/skills/idea-extraction/SKILL.md and follow its domain exhaustion methodology.

Read the `## Expansion Mode` section from `docs/plans/ideation.md` and route accordingly. All findings are written to `ideation.md` in real time — not held in context.

**Full Mode** (three-pass orchestration):

- **Pass 1 — Horizontal Sweep**: Audit `ideation.md` for all domains present. Apply domain exhaustion protocol to identify missing domains. Add all confirmed domains with `[SURFACE]` marker. Do NOT drill yet. Present complete domain map to user and get confirmation before Pass 2.
- **Pass 2 — Vertical Drilling**: For each domain (dependency order — foundational first), drill to Level 2-3 depth. Cross-cut watch active: after each sub-feature/edge case, ask "Does this touch any other domain?" If yes, immediately add to `## Cross-cutting Candidates` in `ideation.md` as `[Domain A] × [Domain B]: [connection]`. Do not stop to explore — flag and continue. Update depth markers as drilling progresses. Pause after each domain: "Here's what I captured. Anything missing?"
- **Pass 3 — Cross-cutting Synthesis**: Present full candidates list. For each cross-cutting candidate pair, ask all five of these questions:

  1. **Shared state conflict**: If both features write to the same entity, who wins on conflict? What is the merge/override strategy? Is there a canonical owner?
  2. **Trigger chain**: Does completing Feature A automatically trigger Feature B? If yes, what are the rollback semantics if B fails mid-chain? Is this synchronous or async?
  3. **Permission intersection**: Does a user's permission level in Domain A affect what they can do in Domain B? Are there permission escalation or restriction paths between domains?
  4. **Notification fan-out**: Does an event in Domain A need to notify actors in Domain B? Who owns the notification contract? What happens if the notification fails?
  5. **State transition conflict**: Can Features A and B be triggered simultaneously by the same user? What happens to data consistency if they race?

  These five questions surface the failure modes most commonly discovered as production bugs when cross-cutting interactions are underspecified at the ideation phase. Document confirmed answers in `## Feature Interactions` in `ideation.md`. Discard rejected pairs with a one-line note explaining why they don't interact. Check for second-order cross-cuts.

**Vertical Mode**: Focus on existing domains. Identify shallowest sub-topics. Drive to Level 2-3 depth. Cross-cut watch active — surface candidates list at end. Do not introduce new domains unless user requests.

**Horizontal Mode**: Audit for missing domains. Present gap list. Add confirmed new domains with Level 1 exploration. Cross-cut watch active for new domains vs existing. Offer to continue with vertical drilling after.

**Cross-cutting Mode (standalone)**: Read all features, identify interaction points. For each significant pair, ask about conflicts/dependencies/shared state. Document confirmed interactions. Note: works best after vertical drilling.

**Combination Mode**: User specifies sequence. Apply each mode in order. Cross-cut detection always active.

**As-is Mode**: Skip active expansion. Proceed to domain exhaustion check. Cross-cut watch still active — scan `ideation.md` for obvious interaction points and surface before compiling `vision.md`.

> **Mid-session mode switching**: If you want to change expansion direction, update the `Expansion Mode` in `ideation.md` and re-run `/ideate-discover`.

## 4. Problem exploration

Read .agent/skills/brainstorming/SKILL.md and follow its structured facilitation methodology.

Using the idea-extraction skill's approach for the selected mode, explore:

1. **What problem are we solving?** — Get the user to articulate the core pain point in one sentence. If they can't, help them sharpen it.
2. **Who has this problem?** — Identify 2-4 distinct user personas with different needs. For each persona, explore:
   - Name and role
   - Their primary pain point
   - How they solve it today (workarounds)
   - What success looks like for them
   - What would make them switch from their current solution
3. **How are they solving it today?** — Understand the competitive landscape and current workarounds
4. **Why now?** — What has changed that makes this solvable/valuable now

After each answer, summarize what you've captured and ask the user to confirm before moving on.

**Persona completeness gate (Vision Rubric Dimension 2):** For each persona, verify all 6 fields are explicitly present before moving on:
1. **Name + specific role** — not "a user" or "customers"; a named role with context
2. **Specific pain point** — one sentence naming the exact friction, not "they struggle with X generally"
3. **Current workaround** — how they solve this today: the specific tool, manual process, or coping mechanism
4. **Success criteria** — what "solved" concretely looks like for this persona; measurable if possible
5. **Switching trigger** — what would make them switch from their current approach (price threshold, feature, event)
6. **At least one edge case or constraint unique to this persona** — something that makes their situation different from other personas

If any field is absent → probe for it before proceeding. Do not move to feature collection until all personas have all 6 fields. Reference: `.agent/skills/pipeline-rubrics/references/vision-rubric.md` Dimension 2 for the ✅ criteria this gate enforces.

## 5. Feature inventory — deep exploration

This is where most ideation workflows fail. They collect a feature list and move on.
This workflow explores each feature to multiple levels of depth.

### 5a. Feature collection (MoSCoW)

Read .agent/skills/brainstorming/SKILL.md and follow its methodology.

For each persona identified in Step 4, brainstorm features:

1. **Must-have features** — What features are launch-blocking?
2. **Should-have features** — What features are expected but not day-one critical?
3. **Could-have features** — What features would delight but can be deferred?
4. **Won't-have (now)** — What is explicitly out of scope?

Present the MoSCoW matrix to the user for validation.

> **Note:** Deferred ≠ lower quality. Every feature that makes the cut will be
> built to production standard. The categories control *what* ships, not *how well*
> it's built.

### 5b. Feature deepening — Must Haves

Read .agent/skills/idea-extraction/SKILL.md and follow its domain exhaustion methodology.

For each **Must Have** feature, use the idea-extraction skill's Domain Exhaustion Protocol
to explore at least 2 levels deep:

**Level 1: Sub-features.** What are the component parts of this feature?
- "User authentication" → signup, login, password reset, profile management, account deletion, session management, OAuth providers

**Level 2: Edge cases and failure modes.** For each sub-feature:
- What happens on failure? What error does the user see?
- What happens on partial failure? (network drops mid-operation)
- What permissions are required?
- What are the states? (loading, empty, populated, error)

**Level 3 (for complex features): Interactions and conflicts.**
- How does this feature interact with other Must Haves?
- What if two features trigger simultaneously?
- Are there any conflicts between features?

**Decision routing:** When feature decisions arise, use the decision classification rule:
- Feature inclusion/exclusion → product decision → ask the user
- How to implement the feature → defer to `/create-prd`
- Detailed technical choice → note for later, don't burden the user

### 5c. Feature deepening — Should Haves (lighter touch)

For Should Have features, explore Level 1 (sub-features) only. Note that these will get
full Level 2-3 treatment in `/create-prd` when they enter scope.

### Propose next step

Once the feature inventory is complete and the user has confirmed the MoSCoW matrix, proceed to `/ideate-validate` to explore constraints, success metrics, and compile the vision document.

> If this shard was invoked standalone (not from `/ideate`), surface this via `notify_user`. If invoked by the parent `/ideate`, this is a natural handoff — the parent orchestrates the transition.

