---
name: implement
version: 0.2.3
description: Implement SDD features task-by-task following the design document. Use this skill when implementing features or auto-implementing designs. One isolated work context per task, review at the end. Use the tasks skill for task breakdown.
---

# Implement

**Implement** delivers the tasks one at a time, with each task gated by its acceptance criteria. It runs after tasks and before merge.

Both this orchestrator and every delegated worker it uses follow the `language` skill for tone and vocabulary in all output, including replies to the user and commit messages.

All SDD artifacts live in the feature artifact directory. Standalone features use `.sdd/{feature}/`; roadmap deliverables use `.sdd/{initiative}/{deliverable-slug}/`.

## Orchestrator discipline

You are a coordinator. Keep your own context lean across all tasks:

- Your primary jobs are to extract task context, run each task in an isolated work context, track progress, and relay review findings.
- Prefer delegated work if the runtime supports it. If not, perform one task at a time directly with the same context discipline.
- Reading source files, running tests, and invoking linters or builds belongs inside the isolated work context for the current task.
- When reading SDD documents, extract only the section the current task needs.
- Run task work one at a time.

## Process

### Step 1: Implement each task

Read `{artifact_dir}/tasks.md` for the ordered task list. For each task in order, prepare an isolated work context:

1. Extract the single task from `{artifact_dir}/tasks.md` (Status, Blocked by, What to build, Acceptance criteria, Notes).
2. Extract from `{artifact_dir}/design.md` only the component sections this task touches.
3. Paste both into the prompt below. The design already incorporates the specification, so the specification itself stays out.

**Delegated-work prompt** (use an implementation-capable model):
> Think hard.
>
> Implement task {N} for {feature}.
>
> Your job is to deliver this tracer-bullet slice: write the AC tests, make them pass, and stop at the slice's boundary. Code that belongs to a later task stays in that task.
>
> **Task:**
> {paste the single task here}
>
> **Relevant design context:**
> {paste relevant design sections here}
>
> **Project guidelines:** Use the `handbook` skill.
>
> **Language standard:** Use the `language` skill for tone and vocabulary in all output, including commit messages.
>
> **Testing discipline:** Use the `tdd` skill. It owns red-green-refactor, the dependency-count heuristic, the existing-suite exploration rule, test code quality, and the anti-patterns. The `mocking` and `interface-design` skills load transitively from there.
>
> **SDD-specific testing constraints (in addition to `tdd`):**
> - **Scope:** apply TDD to application business logic and reusable IaC modules. For provisioning configs (root modules, env-specific stacks) and imperative scripts (migrations, runbooks), create and lint them in the task; defer execution to the user, since execution mutates real state.
> - **NFRs are exempt from TDD.** Architectural choices satisfy them; instrumentation observes them where possible. For an app-instrumented NFR, add the named metric or log. Platform-observed and architectural-only NFRs need no code.
> - If the task's ACs can only be exercised after the user applies an artefact (e.g., a queue must exist for the feature to receive messages), implement and lint the artefact, then pause and ask the user to apply or run it. Resume verification once they confirm.
> - After all tests pass, run the full test suite and include the output as evidence.
>
> **Other rules:**
> - Implement exactly what this task's ACs require. Move code that no test in this task exercises to the task that needs it.
> - One commit per task.
> - Check off completed test checkboxes in `{artifact_dir}/tasks.md` and update status to "Done".
> - Add comments only where logic isn't self-evident.
> - Keep SDD artifact identifiers (FR-XXX, NFR-XXX, AC-X, REQ-XXX) out of code and tests.
> - Every test asserts real behavior. Replace stubs with real assertions or remove them. If an external blocker forces a stub, record it in the task's Notes section and resolve before the final task.
> - Keep IaC and live-state interactions to lint and validate commands (`terraform validate`, `kubectl diff`, `shellcheck`, etc.). For live mutations (`terraform apply`, `kubectl apply`, `ansible-playbook`, database migrations), ask the user — they perform it, you wait.
>
> **Escalation:** If understanding requires code beyond what's provided, or the task is too large for remaining context, STOP and report what you completed and what remains.

### Step 2: Review

After all tasks are complete, use the `review` skill to perform an **Implementation Review**.

### Step 3: Fix issues (if any)

If the review finds P0 or P1 issues, fix them in an isolated work context. Prefer delegated work if the runtime supports it; otherwise perform the fix directly.

> Think hard.
>
> Fix the following issues in the implementation for {feature}.
>
> Your job is to apply the review findings below. Keep all other code and tests as they are.
>
> {paste review findings here}
>
> Use the `handbook` skill for project conventions. Commit each fix. Run tests and linting after each fix. Update checkboxes in `{artifact_dir}/tasks.md` as needed.
>
> **Escalation:** If a fix requires changes beyond the scope of the review findings, STOP and report what needs broader attention.

Repeat Steps 2–3 until the review passes.
