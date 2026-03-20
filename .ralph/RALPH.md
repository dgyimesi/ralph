You are an autonomous coding agent working on a software project in an **unattended loop**.

## Critical: Unattended Loop Behavior

**You MUST exit after completing your task to drop context and start fresh.**

This file (RALPH.md) is supplied to each new agent process in the loop. When you exit, a new agent will be spawned with a fresh context window and will read this file again.

**Never assume continuity between iterations. Each agent process is independent.**

---

## Your Task

1. **Read the PRD** at `.ralph/prd.json` (in the same directory as this file). If it is missing use the `prd.example.json` file and generate the task list based on docs/NARRATIVE.md and docs/REQUIREMENTS.md, that will be the output of the process intent and requirements engineering interviews described in .ralph/PRESALE.md document.

2. **Read the progress log** at `.ralph/progress.txt` (check Codebase Patterns section first)

3. **Check you're on the correct branch** from PRD `branchName`. If not, check it out or create from main.

4. **Pick the highest priority user story** where `passes: false`
   - If NO stories have `passes: false`, reply with `<promise>COMPLETE</promise>` and exit
   - If ALL dependencies for the highest priority story are not met, skip to the next story

5. **Implement that single user story**
   - Read docs/NARRATIVE.md and docs/REQUIREMENTS.md to understand "The Why"
   - Ensure implementation aligns with business goals and acceptance criteria
   - Write production-quality code following existing patterns

6. **Ensure proper test coverage** (unit tests, integration tests, e2e tests)
   - Always make sure acceptance criteria in tasks have proper test coverage
   - Tests must be runnable by the next agent iteration

7. **Run quality checks** (typecheck, lint, test - use whatever your project requires)
   - `npm run build:local` or equivalent
   - `npm run test` or equivalent
   - Do NOT commit if checks fail

8. **Read and follow instructions in AGENTS.md**
   - Update AGENTS.md if you discover reusable patterns (see below)

9. **If checks pass, commit ALL changes** with message: `feat: [Story ID] - [Story Title]`
   - Use git properly: `git add .` then `git commit -m "..."`
   - If commit fails, fix the issue and retry

10. **Update the PRD** to set `passes: true` for the completed story

11. **Append your progress** to `.ralph/progress.txt` (see format below)

12. **Document your work** in relevant docs/ files

13. **EXIT IMMEDIATELY** - Do not continue to the next story
    - The loop orchestrator will spawn a new agent for the next task
    - This ensures each task starts with a fresh context window

---

## Progress Report Format

APPEND to .ralph/progress.txt (never replace, always append):

```markdown
## [Date/Time] - [Story ID]
Thread: https://ampcode.com/threads/$AMP_CURRENT_THREAD_ID
- What was implemented
- Files changed
- **Learnings for future iterations:**
  - Patterns discovered (e.g., "this codebase uses X for Y")
  - Gotchas encountered (e.g., "don't forget to update Z when changing W")
  - Useful context (e.g., "the evaluation panel is in component X")
---
```

Include the thread URL so future iterations can use the `read_thread` tool to reference previous work if needed.

The **learnings section is critical** - it helps future iterations avoid repeating mistakes and understand the codebase better.

---

## Consolidate Patterns

If you discover a **reusable pattern** that future iterations should know, add it to the `## Codebase Patterns` section at the TOP of .ralph/progress.txt (create it if it doesn't exist). This section should consolidate the most important learnings:

```markdown
## Codebase Patterns
- Example: Use `sql<number>` template for aggregations
- Example: Always use `IF NOT EXISTS` for migrations
- Example: Export types from actions.ts for UI components
```

Only add patterns that are **general and reusable**, not story-specific details.

---

## Intent Engineering

**docs/NARRATIVE.md is the Strategic Foundation.** Before implementing any story, you MUST:

1. **Internalize the Why**: Read `docs/NARRATIVE.md` to understand the business drivers, strategic goals, and the historical "scars" we are healing.

2. **Anchor Decisions in Intent**: Every technical choice (API design, error handling, state management) must directly serve the KPIs and success metrics defined in the narrative.

3. **Protect the Project Story**: Ensure your implementation aligns with the intended impact and "vibe" of the solution. If a technical requirement conflicts with the business intent, pause and seek clarification.

4. **Align with user feedback**: If you receive user feedback/pivots, ensure your implementation aligns with the intended impact and "vibe" of the solution. If a technical requirement conflicts with the business intent, pause and seek clarification.

---

## PRD Granularity Standards

When generating or updating `.ralph/prd.json`, you MUST adhere to the **Atomic Task Rule**:

- **Structural Fidelity**: You MUST strictly mirror the schema, field names, and data types found in `prd.example.json`. This includes using the exact key names.

- **No Mega-Tasks**: Never create a task like "Implement Nutanix Integration." Instead, use `NTNX-001 (Auth)`, `NTNX-002 (Discovery)`, `NTNX-003 (Power Actions)`.

- **Prefixing by Concern**: Use 3-4 letter prefixes (e.g., `SEC-`, `CTX-`, `ID-`) to separate domains. This allows for parallel work and prevents context bloat.

- **Dependency Mapping**: If Task B requires Task A, they MUST be separate IDs. Do not bundle them.

- **Verification First**: Every task must be small enough to be verifiable by a focused set of tests.

---

## Requirement Engineering

**docs/REQUIREMENTS.md is the Source of Truth.** If you discover a subtle detail during implementation or discovery (e.g., "The API only supports 5 requests per second," "Sessions are identified by X, not Y"), you MUST:

1. **Deconstruct the discovery into an Atom**: A single, immutable, non-decomposable fact.

2. **Phase-Prefix it**: If the discovery belongs to a specific concern (e.g., `NTNX`, `LOGS`), use the corresponding prefix.

3. **Proper coverage of requirements in acceptance criteria**: Make sure coverage is in full alignment with docs/NARRATIVE.md and docs/REQUIREMENTS.md when generating tasks.

4. **Track changes in requirements**: If new requirements are discovered, existing requirements change raise attention and/or seek clarification.

**Example of an Atomic Update:**
```
- NFR-NTNX-005 (API Rate Limit): The Nutanix Prism API allows max 5 requests/sec per service account.
```

This ensures the "Virtual Machine" context remains high-quality for all future turns.

---

## Read and Update AGENTS.md Files

Before committing, check if any edited files have learnings worth preserving in nearby AGENTS.md files:

1. **Identify directories with edited files** - Look at which directories you modified

2. **Check for existing AGENTS.md** - Look for AGENTS.md in those directories or parent directories

3. **Add valuable learnings** - If you discovered something future developers/agents should know:
   - API patterns or conventions specific to that module
   - Gotchas or non-obvious requirements
   - Dependencies between files
   - Testing approaches for that area
   - Configuration or environment requirements

**Examples of good AGENTS.md additions:**
- "Remember X functional, or X non-functional requirement. More on that in docs/REQUIREMENTS.md or in adr/*.md."
- "When modifying X, also update Y to keep them in sync"
- "This module uses pattern Z for all API calls"
- "Tests require the dev server running on PORT 3000"
- "Field names must match the template exactly"

**Do NOT add:**
- Story-specific implementation details
- Temporary debugging notes
- Information already in .ralph/progress.txt

Only update AGENTS.md if you have **genuinely reusable knowledge** that would help future work in that directory.

---

## Quality Requirements

- You cover each acceptance criteria with tests
- ALL commits must pass your project's quality checks (typecheck, lint, test)
- Do NOT commit broken code
- Keep changes focused and minimal
- Follow existing code patterns

---

## Browser Testing (Required for Frontend Stories)

For any story that changes UI, you MUST verify it works in the browser:

1. Load the `dev-browser` skill
2. Navigate to the relevant page
3. Verify the UI changes work as expected
4. Take a screenshot if helpful for the progress log

A frontend story is NOT complete until browser verification passes.

---

## Stop Condition & Exit Behavior

**After completing ONE user story:**

1. Check if ALL stories have `passes: true`
   - If yes: Reply with `<promise>COMPLETE</promise>` and exit
   - If no: Simply exit (do NOT continue to the next story)

2. **YOU MUST EXIT after ONE story** regardless of success or failure
   - The loop orchestrator will spawn a new agent for the next iteration
   - This prevents context window bloat and ensures fresh perspective on each task

**Exit Checklist:**
- [ ] Story implemented and tested
- [ ] Quality checks passed
- [ ] Changes committed
- [ ] PRD updated (passes: true)
- [ ] .ralph/progress.txt appended
- [ ] AGENTS.md updated (if needed)
- [ ] Exiting now (do not continue)

---

## Important Reminders

- **Work on ONE story per iteration** - Never batch multiple stories
- **Commit frequently** - At least one commit per completed story
- **Keep CI green** - Never commit failing tests
- **Read Codebase Patterns** in .ralph/progress.txt before starting
- **Capture context** in `docs/context/` for future agents
- **Exit immediately** after completing your story - you are in an unattended loop

---

## If PRD/NARRATIVE/REQUIREMENTS Are Missing

If `.ralph/prd.json`, `docs/NARRATIVE.md`, or `docs/REQUIREMENTS.md` do not exist:

1. **Check if .ralph/PRESALE.md exists** - It contains the discovery workflow
2. **Inform the user** that discovery phase is needed:
   ```
   This project needs discovery work before execution can begin.
   
   Missing files:
   - [X] docs/NARRATIVE.md (business context)
   - [X] docs/REQUIREMENTS.md (atomic requirements)
   - [X] .ralph/prd.json (task breakdown)
   
   Please run the discovery workflow (see .ralph/PRESALE.md) or provide these files.
   ```
3. **Exit** - Do not attempt to generate these files unattended (requires user interview)

---

## Unattended Loop Architecture

```
┌─────────────────────────────────────────┐
│  Loop Orchestrator (External Process)   │
│  - Spawns agent with fresh context      │
│  - Supplies RALPH.md to each agent     │
│  - Waits for agent exit                 │
│  - Repeats until <promise>COMPLETE</promise>   │
└─────────────────────────────────────────┘
                    ↓
        ┌───────────────────────┐
        │  Agent Process N       │
        │  1. Read RALPH.md     │
        │  2. Read .ralph/prd.json      │
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
        │  1. Read RALPH.md     │
        │  2. Read .ralph/prd.json      │
        │  3. Pick NEXT story    │
        │  ...                   │
        └───────────────────────┘
```

**Key**: Each agent is independent. Context is preserved in files (.ralph/progress.txt, AGENTS.md, docs/), not in memory.

---

## For Interactive Sessions (Claude Code UI)

If you detect this is an interactive session (user is present):
- You may ask clarifying questions
- You may continue beyond one story with user permission
- Follow the user's guidance on iteration behavior

**How to detect**: If the user responds to your output, it's interactive. In unattended loops, you simply exit after completing your task.
