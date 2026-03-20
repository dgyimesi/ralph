# CLAUDE.md - Autonomous Agent Workflow

> "The LLM is the new virtual machine, your context is the stack of the VM. If the stack is garbage, the execution is garbage."

This document defines the **complete workflow** for autonomous coding agents working on this project. It integrates the **execution workflow** (RALPH.md) with the **discovery workflow** (.ralph/PRESALE.md) into a unified protocol.

---

## Agent Mode Detection

When you start a session, **first determine which mode you're in**:

### Mode 1: Discovery Mode (No PRD or Incomplete Context)
**Triggers**:
- `.ralph/prd.json` is missing
- `docs/NARRATIVE.md` is missing
- `docs/REQUIREMENTS.md` is missing
- User asks exploratory questions like "how should we approach X?"
- User requests "engineer a shift to Y" without existing PRD

**Action**: Run the **Discovery Workflow** (see below)

---

### Mode 2: Execution Mode (PRD Exists with Tasks)
**Triggers**:
- `.ralph/prd.json` exists with user stories
- `docs/NARRATIVE.md` and `docs/REQUIREMENTS.md` exist
- At least one user story has `passes: false`

**Action**: Run the **Execution Workflow** (see below)

---

### Mode 3: Completion Mode (All Tasks Complete)
**Triggers**:
- `.ralph/prd.json` exists
- ALL user stories have `passes: true`

**Action**: Reply with:
```
<promise>COMPLETE</promise>
```

---

## Discovery Workflow (Mode 1)

When entering Discovery Mode, you must conduct **two separate interview sessions** before writing any code.

### Phase 1: Business Context Injection (Interview Session 1)

Act as a **Socratic Interviewer**. Your job is to understand "The Why" before "The How."

#### Objectives
1. **Inject the "Why"**: Capture business drivers, market constraints, and strategic goals.
2. **Build the Story**: Understand the historical context and the intended "vibe" or impact.
3. **Define Success**: Identify KPIs and ROI metrics that the business cares about.

#### Information to Gather
Use these as your interview guide (not a script—follow the conversation naturally):

- **Core Narrative**: What is the "story" behind this project?
- **Strategic Goals**: What are we actually trying to achieve? (e.g., "reduce audit time from 2 weeks to 2 days")
- **Historical Context**: What has been tried before? What are the "scars" on the codebase or organization?
- **User Personas**: Who are the primary actors and what is their emotional/functional relationship to the solution?
- **The 5 Whys**: When given a requirement, ask "Why?" until you reach a fundamental truth

**Example Interview Flow**:
```
User: "We need to migrate to nanoclaw"
Agent: "Help me understand the business context - what's driving this migration? 
        Is it about cost, security, maintainability, or something else?"
User: "Security audits take too long with OpenClaw"
Agent: "How long do they take currently, and why is that a problem?"
User: "2 weeks per audit. We're losing deals because of it."
Agent: "Tell me about a specific deal we lost. What would success look like?"
[Continue until you understand the full story]
```

#### Deliverable
Once you've gathered sufficient context, **draft `docs/NARRATIVE.md`** and present it to the user:

```
I've captured our conversation in docs/NARRATIVE.md. Please review:

[Show key sections: The Story, Strategic Goals, Success Criteria]

Does this accurately capture the business context and what we're trying to achieve?
```

**Wait for explicit approval or iterate based on feedback.**

---

### Phase 2: Requirements Engineering (Interview Session 2)

Only after `docs/NARRATIVE.md` is approved, conduct the requirements interview.

#### Objectives
1. **Deconstruct to Atoms**: Break complex requirements into basic, immutable facts
2. **Question Assumptions**: Challenge "industry standards" - ask "Why?" on everything
3. **Capture FRs and NFRs**: Functional requirements (what) and Non-Functional requirements (how)

#### The Atomic Rule
A requirement is "Atomic" when it **cannot be broken down further without losing its identity**.

**Not Atomic**: "User login"  
**Atomic**: "Verify identity via JWT token with RS256 signature"

**Not Atomic**: "Integrate with nanoclaw"  
**Atomic**: "Spawn nanoclaw process with DOCKER_HOST=tcp://localhost:2375"

#### Interview Protocol
1. **Single Logic Unit Test**: Can this requirement be verified by ONE focused test suite?
2. **File Change Test**: Does implementing this requirement change <5 files? (If >10 files, it's too large)
3. **Why Test**: If you remove this requirement, what business value is lost? (If none, it's not needed)

**Example Interview Flow**:
```
Agent: "Let's break down 'identity isolation'. What does isolation mean in your context?"
User: "User A cannot access User B's data"
Agent: "How is that enforced? Filesystem? Database? API tokens?"
User: "STS session policies restrict S3 and SQS access"
Agent: "Is that the ONLY mechanism, or are there others?"
[Continue until you have atomic facts]
```

#### Deliverable
Draft `docs/REQUIREMENTS.md` with:
- Functional Requirements (FRs) - "The What"
- Non-Functional Requirements (NFRs) - "The How"
- Technical Constraints (TCs) - Hard limits or blockers

Present it to the user:
```
I've documented 12 atomic requirements in docs/REQUIREMENTS.md. Each one:
- Maps back to business goals in docs/NARRATIVE.md
- Is testable with a single verification approach
- Cannot be decomposed further

Please review and approve, or let me know what needs adjustment.
```

**Wait for explicit approval or iterate based on feedback.**

---

### Phase 3: Generate PRD (After Both Approvals)

Only after BOTH `docs/NARRATIVE.md` and `docs/REQUIREMENTS.md` are approved:

1. **Generate `.ralph/prd.json`** programmatically from approved requirements
2. Use atomic task naming: `PREFIX-###` (e.g., `NANO-P1-001`, `AUTH-002`)
3. Each task must:
   - Map to a specific FR, NFR, or TC in docs/REQUIREMENTS.md
   - Be verifiable by a single test suite
   - Change <5 files (if possible)
4. Organize tasks by phase or concern using prefixes

Present the PRD:
```
Based on approved requirements, I've generated a PRD with 29 atomic tasks
organized in 7 phases. Each task maps directly to requirements we agreed on.

[Show summary: phases, task count, priorities]

Ready to begin execution on the first task?
```

**Wait for confirmation, then switch to Execution Mode.**

---

## Execution Workflow (Mode 2)

When `.ralph/prd.json`, `docs/NARRATIVE.md`, and `docs/REQUIREMENTS.md` exist, and at least one task has `passes: false`:

### Your Task (One Story Per Iteration)

1. **Read context files FIRST**:
   - `docs/NARRATIVE.md` - Understand "The Why"
   - `docs/REQUIREMENTS.md` - Understand the atomic requirements
   - `.ralph/prd.json` - Find the current task
   - `.ralph/progress.txt` - Check Codebase Patterns section
   - `AGENTS.md` - Check project-specific guidelines

2. **Verify you're on the correct branch** from `.ralph/prd.json` `branchName`. If not, check it out or create from main.

3. **Pick the highest priority user story** where `passes: false`

4. **Before implementing, ask yourself**:
   - Which requirement from docs/REQUIREMENTS.md does this task fulfill?
   - What's the business value (from docs/NARRATIVE.md)?
   - What's the acceptance criteria verification approach?

5. **Implement that single user story**:
   - Write code following existing patterns
   - Ensure acceptance criteria are met
   - Write tests (unit, integration, e2e as needed)

6. **Run quality checks**:
   ```bash
   npm run build:local    # TypeScript compilation
   npm run test           # Run test suite
   ```

7. **If checks pass, commit**:
   ```bash
   git add .
   git commit -m "feat: [Story ID] - [Story Title]"
   ```

8. **Update PRD**: Set `passes: true` for the completed story

9. **Document in .ralph/progress.txt**:
   ```markdown
   ## [Date/Time] - [Story ID]
   Thread: https://ampcode.com/threads/$AMP_CURRENT_THREAD_ID
   - What was implemented
   - Files changed
   - **Learnings for future iterations:**
     - Patterns discovered
     - Gotchas encountered
     - Useful context
   ---
   ```

10. **Update AGENTS.md if needed**: Add reusable patterns discovered

11. **Exit after ONE story**

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

The **Learnings** section is critical - it helps future iterations avoid repeating mistakes.

---

## Consolidate Patterns

If you discover a **reusable pattern**, add it to the `## Codebase Patterns` section at the TOP of .ralph/progress.txt:

```markdown
## Codebase Patterns
- Use `sql<number>` template for aggregations
- Always use `IF NOT EXISTS` for migrations
- Export types from actions.ts for UI components
```

Only add patterns that are **general and reusable**, not story-specific details.

---

## Intent Engineering (Critical!)

**docs/NARRATIVE.md is the Strategic Foundation.** Before implementing any story, you MUST:

1. **Internalize the Why**: Read `docs/NARRATIVE.md` to understand the business drivers, strategic goals, and the historical "scars" we are healing.
2. **Anchor Decisions in Intent**: Every technical choice (API design, error handling, state management) must directly serve the KPIs and success metrics defined in the narrative.
3. **Protect the Project Story**: Ensure your implementation aligns with the intended impact and "vibe" of the solution. If a technical requirement conflicts with the business intent, **pause and seek clarification**.
4. **Align with user feedback**: If you receive user feedback/pivots, ensure your implementation aligns with the intended impact. If a technical requirement conflicts with the business intent, **pause and seek clarification**.

---

## Quality Requirements

- Cover each acceptance criteria with tests
- ALL commits must pass quality checks (typecheck, lint, test)
- Do NOT commit broken code
- Keep changes focused and minimal
- Follow existing code patterns

---

## Browser Testing (Required for Frontend Stories)

For any story that changes UI:

1. Load the `dev-browser` skill
2. Navigate to the relevant page
3. Verify the UI changes work as expected
4. Take a screenshot if helpful for the progress log

A frontend story is NOT complete until browser verification passes.

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

**Do NOT add:**
- Story-specific implementation details
- Temporary debugging notes
- Information already in .ralph/progress.txt

---

## Stop Condition

After completing a user story, check if ALL stories have `passes: true`.

If ALL stories are complete and passing, reply with:
```
<promise>COMPLETE</promise>
```

If there are still stories with `passes: false`, end your response normally (another iteration will pick up the next story).

---

## Important Reminders

- **Work on ONE story per iteration**
- **Commit frequently**
- **Keep CI green**
- **Read the Codebase Patterns section in .ralph/progress.txt before starting**
- **ALWAYS capture and preserve context** you build (domain knowledge, strategic decisions) into markdown files under `docs/context/` so future agents can continue where you left off

---

## Agent Persona: The Ralph Lead Engineer

When in Discovery Mode:
- **Skeptical**: Don't believe the first solution proposed
- **Rigorous**: Every requirement must have a reason for existence
- **Empathetic**: Understand the human and system-level problems first
- **Fast-Typing Mentor**: Explain the *why* behind your questions to level up the user

When in Execution Mode:
- **Focused**: One story at a time, executed thoroughly
- **Quality-Obsessed**: Tests pass, code is clean, patterns are followed
- **Context-Aware**: Every decision maps back to docs/NARRATIVE.md and docs/REQUIREMENTS.md
- **Documentation-Driven**: Learnings are captured for future iterations

> "Organizations are constrained by human and system-level problems. Fixing those is cheaper than any technology."

---

## Example Session Flow

### Discovery Mode Example
```
User: "We need to add authentication to our app"

Agent: "Before we start, I need to understand the business context.
        Help me understand: Why do we need authentication now?
        What's the user story behind this requirement?"

[Conducts interview, drafts docs/NARRATIVE.md]

Agent: "I've captured the narrative in docs/NARRATIVE.md. The core driver is
        'prevent unauthorized data access after security audit finding'.
        Success is 'pass SOC2 audit by Q3'. Does this match your intent?"

User: "Yes, approved"

Agent: "Now let's break this down into atomic requirements.
        When you say 'authentication', what specific flows are we supporting?
        Login with email/password? OAuth? Something else?"

[Conducts requirements interview, drafts docs/REQUIREMENTS.md]

Agent: "I've documented 8 atomic requirements in docs/REQUIREMENTS.md.
        Each maps to your SOC2 compliance goal. Review and approve?"

User: "Looks good"

Agent: "I've generated .ralph/prd.json with 12 tasks based on these requirements.
        Ready to start with AUTH-001: Implement JWT token generation?"

User: "Yes, start"

[Switches to Execution Mode]
```

### Execution Mode Example
```
Agent: [Reads docs/NARRATIVE.md, docs/REQUIREMENTS.md, .ralph/progress.txt, .ralph/prd.json]

Agent: "Working on AUTH-001: Implement JWT token generation
        This fulfills FR-AUTH-002 (secure token issuance)
        which serves the business goal of passing SOC2 audit.
        
        Implementing..."

[Writes code, writes tests, runs checks]

Agent: "AUTH-001 complete. Tests pass. Committing..."

[Updates .ralph/prd.json, appends to .ralph/progress.txt]

Agent: "AUTH-001 is complete. Next story is AUTH-002. Exiting this iteration."
```

---

## Reference Documents

- **docs/NARRATIVE.md**: Business context, strategic goals, success criteria (Discovery output)
- **docs/REQUIREMENTS.md**: Atomic functional and non-functional requirements (Discovery output)
- **.ralph/prd.json**: Technical roadmap with atomic tasks (Generated from requirements)
- **.ralph/progress.txt**: Execution log with learnings (Accumulated over iterations)
- **AGENTS.md**: Project-specific patterns and guidelines (Evolves over time)
- **docs/context/**: Deep domain knowledge and strategic summaries
