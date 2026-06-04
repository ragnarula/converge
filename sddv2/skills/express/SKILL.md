---
name: express
description: Run the full SDD workflow end-to-end after research is prepared. Chains requirements, plan, tasks, and implement without stopping unless user input is required. Use this skill when the user wants to go from research straight through to implementation in one shot.
version: 0.3.1
---

# Express

Run the full SDD workflow end-to-end. Assumes research has already been completed. Chains requirements, plan, tasks, and implement sequentially, continuing automatically unless user input is required.

## Prerequisites

Either:
- a standalone feature with research at `.sdd/{feature}/research.md`, or
- a chosen deliverable from a roadmap at `.sdd/{initiative}/roadmap.md`.

If neither exists, tell the user to run the `research` skill first and stop.

**Roadmap check:** If a roadmap covers this work, refuse to run on the whole initiative. Ask the user which deliverable (D-XX) to express, then run express scoped to that single deliverable's artifact directory. Express handles one feature at a time, never a sequence.

## Orchestrator Discipline

You are a coordinator across the full workflow. Keep your context lean:
- Do NOT read source code, test files, or SDD documents yourself except when a skill step explicitly requires it
- Each skill you invoke manages its own isolated work contexts. Prefer delegated work where available; otherwise let the skill run directly with the same context discipline
- Your only jobs: invoke skills in sequence, relay user input when needed, report results

## Process

**CRITICAL**: Execute ONE step at a time. Never run multiple delegated-work steps in a single message. Wait for each step to complete before starting the next.

**CRITICAL**: Do NOT stop between steps unless user input is genuinely required (e.g., the `requirements` skill needs a discovery interview answer, or a review finds P0 issues that need user decision). If a step completes successfully, move immediately to the next step.

### Step 1: Requirements

Use the `requirements` skill to create the specification for {feature}.

The requirements skill conducts a discovery interview. Answer what you can from the research findings. When the skill requires input only the user can provide, pause and ask the user. Once the user responds, continue.

### Step 2: Plan

Use the `plan` skill to create the design for {feature}.

Continue automatically once the specification from Step 1 is complete.

### Step 3: Tasks

Use the `tasks` skill to create the task breakdown for {feature}.

Continue automatically once the design from Step 2 is complete.

### Step 4: Implement

Use the `implement` skill to implement {feature}.

Continue automatically once the task breakdown from Step 3 is complete.

### Completion

When all steps are done, report:
- What was implemented
- How many tasks were completed
- Any review findings that were resolved along the way
- Any outstanding P2/P3 observations from reviews
