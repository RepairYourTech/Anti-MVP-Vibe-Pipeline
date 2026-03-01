# Spec Pipeline Starter Kit - Architecture

**Purpose:** Provide a high-level map of the agentic machinery that powers the Spec Pipeline Starter Kit.

This document serves as a guide to understanding how the various components within the `.agent/` directory interlock to provide a robust, agent-agnostic development environment.

---

## 1. Code Organization

The intelligence of the kit lives entirely within the `.agent/` directory.

```text
.agent/
├── instructions/    # Core directives (the "brainstem")
├── progress/        # State and memory (the "hippocampus")
├── rules/           # Non-negotiable constraints (the "laws")
├── skills/          # Reusable capabilities (the "tools")
└── workflows/       # Structured processes (the "playbooks")
```

### Core Components

*   **Instructions:** (`workflow.md`, `tech-stack.md`, `structure.md`, `patterns.md`, etc.) Baseline knowledge the agent needs to operate in the specific environment. These files ship as templates with `{{PLACEHOLDER}}` markers — they are not static files. The bootstrap system fills them progressively as tech decisions are confirmed during `/create-prd`. An instruction file with unfilled placeholders is a broken agent context. `workflow.md` enforces the mandatory execution sequence: Understand Context -> Check Skills -> Execute -> Validate.
*   **Rules:** Preemptively loaded constraints that apply to *every* task. Includes security best practices, TDD mandates (`tdd-contract-first.md`), and platform-specific laws (e.g., `child-safety.md`).
*   **Skills:** Modular capabilities (e.g., `technical-writer`, `surrealdb-expert`). Agents load these explicitly when a task requires them, preventing context bloat.
*   **Workflows:** Step-by-step markdown checklists invoked via `/slash-commands` (e.g., `/create-prd`, `/implement-slice`). They chain skills together to achieve complex, multi-stage goals.

---

## 2. Data Flow & State Management

Agents are inherently stateless across conversations. The kit uses the **Session Continuity** protocol to provide a persistent memory system.

### The `.agent/progress/` Directory

This directory acts as the agent's long-term and working memory.

```text
.agent/progress/
├── active-phase.md     # Current phase/sprint tracking
├── memory/
│   ├── blockers.md     # Resolved issues to prevent regression
│   ├── decisions.md    # Architectural choices and their rationale
│   └── patterns.md     # Recurring semantic patterns/idioms
└── sessions/           # State checkpoints for resuming work across chats
```

### Flow

1.  **Read:** Upon initialization, workflows frequently instruct agents to read `.agent/progress/active-phase.md` to understand current goals.
2.  **Act:** The agent executes the workflow.
3.  **Persist:** At defined checkpoints (or via the `/sync-progress` protocol), the agent distills learnings into `patterns.md`, `decisions.md`, and `blockers.md`.
4.  **Resume:** When a new chat starts, the `/resume-session` step in workflows points the agent back to the recent session files to pick up where it left off.

---

## 3. Module Relationships

The power of the kit comes from how these modules interact:

*   **Workflows call Skills:** A workflow like `/create-prd` will explicitly instruct the agent to use the `technical-writer` and `brainstorming` skills.
*   **Rules constrain Workflows:** While a workflow dictates the *steps*, the rules dictate *how* those steps are performed (e.g., `/implement-slice` must obey `tdd-contract-first.md`).
*   **State informs Execution:** Workflows read from `.agent/progress/` to contextualize their execution based on past decisions and current active phases.

---

## 3.5. Bootstrap System

The bootstrap system transforms the kit from a generic template into a project-specific configuration. It runs as a utility workflow called by other pipeline workflows — never directly by the user.

### Components

*   **`bootstrap-agents-fill`**: Receives template key-value pairs and fills `{{PLACEHOLDER}}` markers across all instruction files and root agent config files (`AGENTS.md`, `GEMINI.md`). Idempotent — only fills what's provided, leaves other placeholders untouched.
*   **`bootstrap-agents-provision`**: Reads the skill library manifest, copies matching skills from `.agent/skill-library/` into `.agent/skills/`, fills skill-specific placeholders, composes `FRAMEWORK_PATTERNS` from the installed frontend framework skill, and updates the installed skills list in all root config files.

### Invocation Model

Bootstrap fires **progressively** — once per confirmed tech decision during `/create-prd-stack`, not in a batch at the end:

1. **Database confirmed** → fills DB placeholders (`DATABASE`, `ORM`, etc.) + provisions database skill (e.g., `surrealdb-expert`, `postgresql-patterns`)
2. **Frontend framework confirmed** → fills framework placeholders + provisions framework skill + composes `FRAMEWORK_PATTERNS` for `patterns.md`
3. **Step 9.5 of `/create-prd-compile`** → fills `PROJECT_STRUCTURE` and `ARCHITECTURE_TABLE` in `structure.md`

### Auto-filled vs. "If Provided" Keys

| Key | Auto-filled? | Generated by |
|-----|-------------|--------------|
| `VALIDATION_COMMAND`, `TEST_COMMAND`, etc. | Yes | Derived from confirmed dev tooling |
| `DATABASE`, `AUTH_PROVIDER`, etc. | Yes | Confirmed tech decisions |
| `INSTALLED_SKILLS` | Yes | After skill provisioning |
| `PROJECT_STRUCTURE`, `ARCHITECTURE_TABLE` | **No — requires Step 9.5** | `/create-prd-compile` Step 9.5 |
| `FRAMEWORK_PATTERNS` | **No — requires framework skill** | `bootstrap-agents-provision` after framework provisioning |

### Placeholder Verification Gate

Before any implementation begins, `/implement-slice-setup` (Step -1) scans all seven instruction files for unfilled `{{` patterns. If any are found, implementation stops with a specific remediation path per file. This gate is the last line of defense against broken agent context reaching the implementation phase.

---

## 4. Key Patterns

### Test-Driven Contract-First (CFPA)

The defining architectural pattern of the code produced by this kit.

1.  **Contract (Zod):** Define the schema first.
2.  **Tests (Failing):** Write tests that assert against the contract.
3.  **Implementation:** Write the code to make the tests pass.

*Never reverse this order.*

### Explicit Handoffs

Workflows are designed to end with explicit NEXT STEPS. An agent shouldn't guess what happens after `/ideate`; the workflow tells it to propose `/create-prd`. This ensures continuous, unbroken pipeline progression.

### Placeholder Verification Gate

A safety mechanism in `/implement-slice-setup` (Step -1) that prevents implementation from proceeding with broken agent context. Before any code is written, it scans all seven instruction files for unfilled `{{PLACEHOLDER}}` markers. If any are found, it stops and provides specific remediation commands per file. This gate catches bootstrap failures that would otherwise cause agents to invent project structure, ignore framework conventions, or operate with zero project context.
