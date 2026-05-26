---
name: tasks
version: 0.3.0
description: Break a design into demoable tracer-bullet tasks. Use this skill when creating task breakdowns. Each task delivers a vertical slice that is verifiable on its own, with Given/When/Then acceptance criteria and explicit dependencies. Reviews check that the task set, taken together, covers every requirement and every design component, in a valid order.
---

# Tasks

**Tasks** breaks the design into demoable tracer-bullet tasks. It runs after plan and before implement.

Both this orchestrator and every subagent it launches follow the `language` skill for tone and vocabulary in all output, including replies to the user.

All SDD artifacts live in `.sdd/{feature}/`.

## Process

### Task Breakdown

Launch a subagent to write the task breakdown.

**Subagent prompt:**
> Create the task breakdown for {feature} at `.sdd/{feature}/tasks.md`.
>
> Your job is to break the design into **tracer-bullet tasks**. Each task is a thin vertical slice that delivers concrete value on its own — demoable or verifiable in isolation, not a horizontal slice of one layer. Prefer many thin slices over few thick ones.
>
> **Read:**
> - Task template: `templates/tasks.template.md`
> - Design: `.sdd/{feature}/design.md`
> - Specification: `.sdd/{feature}/specification.md`
> - Project conventions: use the `handbook` skill
> - Language standard: use the `language` skill
>
> **For each task, write:**
> - **What to build** — a short prose paragraph describing the end-to-end behavior the task delivers. Use plain prose to name *what* the slice does. Keep file paths and code snippets out of this paragraph; the implementation agent will curate them at run time.
> - **Acceptance criteria** — one or more Given/When/Then triples that, when green, mean this slice is complete. The AC is the slice's contract; the implementer writes a test for each AC first (TDD red).
> - **Blocked by** — explicit references to prior tasks this task depends on, or "None".
> - **Notes** — 3–5 sentences of key decisions, edge cases, or fixture pointers.
>
> The task captures intent and acceptance contract only. The implementation agent does the rest at run time: read the design to identify touched components, write one test per AC, then use the dependency-count heuristic to judge additional coverage. The handbook covers where tests live and how to write them; depth is the implementer's call.
>
> **Breakdown rules:**
> - Every task is demoable or verifiable on its own. If you can state what an outside observer would see after the task, the slice is vertical. Reshape slices that fail this test.
> - Order tasks so dependencies point backwards: task N's `Blocked by` references only tasks numbered < N. Each task ends green when complete.
> - Aggregate coverage: every FR in the spec is addressed by at least one task's `What to build` and ACs. Design-component coverage is verified at implementation review (against the diff).
> - Tasks form a flat ordered list.
> - Every task has at least one demoable AC. The single exception is a contract change consumed by other code (e.g., trait signature update); flag any such task as a candidate for a separate prior PR.
> - Vertical slicing means partial components are expected — a component's full design may be realised across multiple tasks. The final task that touches a component completes it.
>
> **Context budget:** Each task plus what the implementation agent will curate around it should fit comfortably in 30–40% of a context window. If a task looks heavy, split it.
>
> **Escalation:** If the design is too large or ambiguous to break down fully, STOP and report what you completed and what needs clarification.
>
> Save the document when done.

### Review

Use the `review` skill for a **Task Breakdown Review** of `.sdd/{feature}/tasks.md`.

### Fix issues (if any)

If the review finds P0 or P1 issues, launch a subagent:

> Fix the following issues in the task breakdown at `.sdd/{feature}/tasks.md`, using `.sdd/{feature}/design.md` and `.sdd/{feature}/specification.md` as reference.
>
> Your job is to apply the review findings below. Keep all other parts of the task breakdown as they are.
>
> {paste review findings here}
>
> Save the document when done.

Repeat until the review passes.
