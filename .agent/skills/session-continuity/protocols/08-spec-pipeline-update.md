> **Framework context required**: This is a protocol excerpt. Before following these steps, read `.agent/skills/session-continuity/SKILL.md` for the complete framework — including the Adaptive Granularity Rule, Level Hierarchy Reference, Frozen Files concept, and Parallel Claim protocol. Protocol files are reference documents for specific steps, not standalone instructions.

# Protocol 8: Spec Pipeline Update

> Part of [Session Continuity](../SKILL.md) — read the index for overview, directory structure, and integration points.

**Invoked by**: `/write-architecture-spec`, `/write-be-spec`, `/write-fe-spec`
(after updating the layer index)

**Purpose**: Mark a spec column done and report pipeline progress.

## Steps

1. **Identify the shard and layer** — which shard just got its spec completed,
   and which layer (IA, BE, or FE)?

2. **Update `.agent/progress/spec-pipeline.md`** — before updating any cell, perform these validation checks:

   (a) Read the `File` column for the shard from the tracker to get the expected file path.
   (b) Confirm the file exists at that path. If not → **skip the update** and report: `"Cannot mark [shard] [layer] as complete — file not found at [path]"`
   (c) Read the file and confirm it contains no skeleton placeholder markers (`[To be filled`, `<!-- TODO -->`, `Status: Skeleton`). If any are present → **skip the update** and report: `"Cannot mark [shard] [layer] as complete — skeleton placeholders remain in [path]: [list of markers found]"`
   (d) Only on all-pass: update the column to `complete`:
   ```markdown
   | 03 | user-profiles | `docs/plans/ia/03-user-profiles.md` | complete | complete | not-started |
   ```
   (e) On any failure: skip the update and report the exact path and failure reason.

3. **Recalculate overall progress**:
   ```markdown
   **Overall**: 8/45 specs (18%)
   ```

4. **Update "Spec Completion Tracking"** — if a shard now has all three specs
   complete (IA ✅ + BE ✅ + FE ✅), add it to the completion tracking list. Note: /plan-phase requires ALL shards to be complete, not just this one.
   ```markdown
   ## Spec Completion Tracking

   Shards with all three specs complete (tracking only — /plan-phase requires ALL shards to be complete, not just individual ones):
   - ✅ Shard 00: API conventions
   - ✅ Shard 01: Authentication
   ```

5. **Report status** — log what was completed and what's next.
