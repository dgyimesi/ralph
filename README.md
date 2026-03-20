# Ralph - Autonomous Agent Loop Methodology

> "Complex systems are built from simple ones, not the other way around. Precision in intent engineering correlates directly to precision in execution."

Ralph is an autonomous agent methodology for software development that maintains high-fidelity context across iterations through file-based knowledge preservation.

---

## 🚀 Two Execution Modes

Ralph supports two distinct execution patterns depending on your workflow:

### 1. Interactive Mode (Human-in-the-Loop)
**File:** `.ralph/CLAUDE.md`  
**Use case:** Active development with human oversight and decision-making

```bash
# Provide CLAUDE.md as context to your AI assistant

❯ study .ralph/CLAUDE.md

# Agent responds similarly:

● Three Operating Modes:
  1. Discovery Mode - When no PRD exists, conduct two interview sessions:
    - Business Context Injection → create docs/NARRATIVE.md
    - Requirements Engineering → create docs/REQUIREMENTS.md
    - Then generate .ralph/prd.json
  2. Execution Mode - When PRD exists with incomplete tasks:
    - Work on ONE story per iteration
    - Read context files first (NARRATIVE, REQUIREMENTS, progress.txt, AGENTS.md)
    - Implement, test, commit, document
    - Update PRD and exit
  3. Completion Mode - When all tasks pass → output <promise>COMPLETE</promise>

  Core Principles:
  - Atomic Requirements - Cannot be decomposed further without losing identity
  - Intent Engineering - Every decision must serve business goals in NARRATIVE.md
  - Quality First - All commits must pass typecheck/lint/test
  - Documentation-Driven - Capture learnings in progress.txt and AGENTS.md
  - One Story Per Iteration - Focus and thoroughness over speed

  The Workflow Loop:
  Read Context → Pick Story → Implement → Test → Commit → Document → Exit
```

### 2. Unattended Mode (AFK Execution)
**File:** `.ralph/RALPH.md`  
**Use case:** Autonomous execution of well-defined tasks without human intervention

```bash
# Run the autonomous loop
.ralph/ralph.sh --tool claude [max_iterations]
```

**Characteristics:**
- Agent picks ONE task, implements it, commits, and exits
- Fresh context on each iteration (no context rot if tasks are atomic)
- Continues until all tasks complete or max iterations reached
- Context preserved in files (progress.txt, AGENTS.md, docs/)
- Ideal for executing a well-defined backlog overnight or during meetings

---

## 🎯 Starting Fresh: Intent & Requirements Engineering

Before Ralph can execute tasks, the project needs three foundational documents (see **[PRESALE.md](.ralph/PRESALE.md)**).

### Phase 1: Intent Engineering (The "Why")
**Output:** `docs/NARRATIVE.md`

Capture the business context, strategic goals, and success metrics through structured interviews. 

**What it captures:**
- Business drivers and strategic goals
- Historical context and "system scars"
- Key Performance Indicators (KPIs)
- Success criteria and constraints

---

### Phase 2: Requirements Engineering (The "What")  
**Output:** `docs/REQUIREMENTS.md`

Deconstruct the narrative into atomic, measurable requirements.

**Format:** `[Actor] shall [Action] so that [Value]`

**What it captures:**
- Functional Requirements (FRs) - What the system must do
- Non-Functional Requirements (NFRs) - How the system must behave
- Each requirement is atomic (non-decomposable) and measurable

---

### Phase 3: Task Generation (The "How")
**Output:** `.ralph/prd.json`

Transform requirements into a phased, prioritized task backlog.

**What it captures:**
- User stories with acceptance criteria
- Task dependencies and priorities
- Pass/fail status for each task
- Branch name for isolation

**Note:** Both CLAUDE.md and RALPH.md reference PRESALE.md for the discovery workflow. Start with interactive mode (CLAUDE.md) to conduct the initial interviews.

---

## 📋 Core Agent Files

### [.ralph/RALPH.md](./.ralph/RALPH.md)
**Unattended execution loop protocol** - Supplied to each agent in the loop via `ralph.sh`.

**Purpose**: Defines the workflow for autonomous agents working in an unattended loop where each agent:
- Reads PRD and picks ONE task
- Implements and tests the task
- Commits changes
- Updates PRD and progress
- **Exits immediately** (new agent spawned for next task)

**Key Behavior**: Agent MUST exit after one task to drop context and start fresh.

---

### [.ralph/CLAUDE.md](./.ralph/CLAUDE.md)
**Interactive discovery + execution workflow** - For human-supervised sessions.

**Purpose**: Unified protocol covering both:
- **Discovery Mode**: When PRD/NARRATIVE/REQUIREMENTS don't exist (conduct PRESALE interviews)
- **Execution Mode**: When PRD exists with tasks (implement one task at a time)
- **Completion Mode**: All tasks complete

**Key Difference**: Allows multi-turn interaction, questions, and iterative refinement (vs. RALPH.md's "one task and exit").

---

### [.ralph/PRESALE.md](./.ralph/PRESALE.md)
**Discovery interview methodology** - How to generate PRD from business context.

**Purpose**: Defines the three-phase interview process:
- **Phase 1**: Business Context Injection (capture "The Why") → produces `docs/NARRATIVE.md`
- **Phase 2**: Requirements Engineering (atomic requirements) → produces `docs/REQUIREMENTS.md`
- **Phase 3**: Generate PRD from approved requirements → produces `.ralph/prd.json`

**When to use**: When starting a new project or major feature without existing PRD.

---

### [.ralph/prd.json](./.ralph/prd.json)
**Product Requirements Document** - Current task backlog.

**Structure**:
```json
{
  "project_name": "...",
  "branchName": "feature/branch-name",
  "objective": "...",
  "user_stories": [
    {
      "id": "FEAT-001",
      "title": "...",
      "passes": false,  // ← Agents pick tasks where passes=false
      "priority": 1
    }
  ]
}
```

**Agent workflow**:
1. Read prd.json
2. Find highest priority story where `passes: false`
3. Implement that ONE story
4. Update `passes: true`
5. Exit

---

### [.ralph/progress.txt](./.ralph/progress.txt)
**Execution history and learnings** - Accumulated context from all agent iterations.

**Structure**:
```markdown
## Codebase Patterns
- Pattern 1
- Pattern 2

---
## [Date/Time] - [Story ID]
Thread: https://ampcode.com/threads/...
- What was implemented
- Files changed
- **Learnings for future iterations:**
  - Pattern discovered
  - Gotcha encountered
---
```

**Purpose**: Preserve context across agent iterations (since each agent drops its memory).

**Key Sections**:
- `## Codebase Patterns` - Reusable patterns (read first!)
- Per-task entries - What was done, what was learned

---

## 🔄 Agent Loop Architecture

```
┌─────────────────────────────────────────┐
│  ralph.sh (Loop Orchestrator)           │
│  - Spawns agent with fresh context      │
│  - Supplies RALPH.md to each agent      │
│  - Waits for agent exit                 │
│  - Repeats until <promise>COMPLETE</promise> │
└─────────────────────────────────────────┘
                    ↓
        ┌───────────────────────┐
        │  Agent Process N       │
        │  1. Read RALPH.md      │
        │  2. Read prd.json      │
        │  3. Pick ONE story     │
        │  4. Implement          │
        │  5. Test & Commit      │
        │  6. Update docs        │
        │  7. EXIT ← CRITICAL    │
        └───────────────────────┘
                    ↓
        ┌───────────────────────┐
        │  Agent Process N+1     │
        │  (Fresh context)       │
        │  Starts from step 1... │
        └───────────────────────┘
```

**Key Principle**: Context is preserved in **files** (progress.txt, AGENTS.md, docs/), not in agent memory.

---

## 🎯 Quick Reference

| Task                        | How to Execute                                                                                             |
| --------------------------- | ---------------------------------------------------------------------------------------------------------- |
| **Start new project**       | Interactive mode → tell agent ```"Study .ralph/CLAUDE.md"``` - agent will detect missing narrative and prd |
| **Interactive development** | Interactive mode → tell agent ```"Study .ralph/CLAUDE.md"``` - agent picks up and starts implementing      |
| **Unattended execution**    | Run `.ralph/ralph.sh --tool claude [max_iterations]`                                                       |
| **Track execution history** | Read/append to `.ralph/progress.txt`                                                                       |
| **Find next task**          | Read `.ralph/prd.json`, find `passes: false`                                                               |
| **Understand codebase**     | Read `.ralph/progress.txt` → "Codebase Patterns" section                                                   |

---

## 🏗️ Project Structure

```
README.md               # This file (methodology overview)

.ralph/
├── RALPH.md            # Unattended execution protocol
├── CLAUDE.md           # Interactive execution protocol
├── PRESALE.md          # Discovery interview workflow
├── ralph.sh            # Loop orchestrator script
├── prd.json            # Task backlog (generated)
├── progress.txt        # Execution history (generated)
└── AGENTS.md           # Agent-specific learnings (optional)

docs/
├── NARRATIVE.md        # Business context (generated from PRESALE)
└── REQUIREMENTS.md     # Atomic requirements (generated from PRESALE)
```
