# RALPH: The Concentric Core of Agentic Workflows

> "Complex systems are built from simple ones, not the other way around. Precision in intent engineering correlates directly to precision in execution."

This document defines the **Ralph Loop** methodology as applied to the **Nutanix VDI Agentic Optimization** project. It serves as the foundational guide for both human architects and autonomous agents.

---

## 1. The Methodology: "The Ralph Loop"

The Ralph Loop is a three-phase engineering lifecycle designed to maintain high-fidelity context across a 1M token window.

### Phase 1: Intent Engineering (The "Why")
**Artifact:** `.ralph/NARRATIVE.md`
- **Objective:** Inject the business narrative, strategic goals, and "system scars" into the agent's memory.
- **Protocol:** Socratic interviewing to reach the fundamental business driver (The 5 Whys).

### Phase 2: First Principles Thinking (The "What")
**Artifact:** `.ralph/REQUIREMENTS.md`
- **Objective:** Deconstruct the narrative into **Atoms**: simple, immutable, non-decomposable facts.
- **Protocol:** Every requirement must be measurable and atomic. FRs follow the `[Actor] shall [Action] so that [Value]` format.

### Phase 3: Agentic Execution (The "How")
**Artifact:** `.ralph/prd.json`
- **Objective:** Translate Atoms into a phased technical roadmap with **Structural Fidelity**.
- **Protocol:** Adhere to the **Atomic Task Rule**. Tasks must be small enough to be verifiable by a focused set of tests.

---

## 2. The Artifact Stack

| Artifact                 | Responsibility                | Purpose                                                        |
| :----------------------- | :---------------------------- | :------------------------------------------------------------- |
| `README.md`              | Master Manifesto              | High-level overview and methodology guide.                     |
| `.ralph/PRESALE.md`      | Methodology Protocol          | Instructions for the Intent and Requirements phases.           |
| `.ralph/PROMPT.md`       | Agent System Prompt           | Core operational instructions for task implementation.         |
| `.ralph/NARRATIVE.md`    | (generated) Strategic Context | The business story and success metrics (KPIs).                 |
| `.ralph/REQUIREMENTS.md` | (generated) Source of Truth   | The list of atomic functional and non-functional requirements. |
| `.ralph/prd.json`        | (generated) Technical Roadmap | The granular, phased backlog for the Executioner agents.       |

---

## 3. Prompting the Ralph Loop: Real-World Examples

Use these exact prompting patterns to drive the agent through the concentric engineering layers.

### Phase 1: Intent Engineering (The "Why")
**Goal:** Build the Strategic Narrative and identify the business KPIs.
> **Prompt:** *"I want to bootstrap a project for Nutanix VDI optimization. The primary drivers are cost-optimization via automated curfews and improving user experience for StoreFront stability. Erik Harold (Director) is our stakeholder. We are migrating from VMware to Nutanix, and eventually to Azure. Socratic interview me to build the NARRATIVE.md."*

### Phase 2: Requirements Engineering (The "What")
**Goal:** Deconstruct the Narrative into Atomic Functional and Non-Functional Requirements.
> **Prompt:** *"Based on our NARRATIVE.md, deconstruct the requirements into REQUIREMENTS.md. We need a 1-hour idle curfew logic, Geo 9-to-5 regional awareness, and a UUID diagnostic bridge for ServiceNow. Ensure every requirement is an 'Atom'—simple, immutable, and measurable."*

### Phase 3: Agentic Execution (The "How")
**Goal:** Translate Requirements into a granular, phased roadmap with Structural Fidelity.
> **Prompt:** *"Generate the prd.json based on REQUIREMENTS.md. You MUST strictly mirror the schema and structural fidelity of prd.example.json. Use numeric priorities, separate concerns with prefixes (SEC-, NTNX-, CTX-), and ensure every task is small enough to be verifiable by a focused set of tests."*

### Phase 4: Implementation & The Loop
**Goal:** Execute the tasks while continuously refining the requirement "Atoms."
> **Prompt:** *"Read prd.json and pick up Task SEC-001. Before you start, check if any new 'Atoms' are discovered during implementation (e.g., API rate limits or specific log schemas). If so, update REQUIREMENTS.md immediately as per the Ralph Loop protocol in PROMPT.md."*

---

> "The LLM is the new virtual machine, your context is the stack of the VM. If the stack is garbage, the execution is garbage."
