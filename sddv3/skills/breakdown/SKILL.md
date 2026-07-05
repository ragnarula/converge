---
name: breakdown
description: Break the available context into a simple ordered task list for the problem-driven flow. Use this skill after a problem is defined — and optionally after research, spec, and design — when you want demoable, ordered implementation tasks. A simplified take on tasks — it reviews and fixes the breakdown autonomously, raising to the user only when a fix would conflict with the source artifacts.
version: 0.1.0
---

# Breakdown

Both this orchestrator and every subagent it spawns follow the `language` skill for tone and vocabulary in all output, including replies to the user. Invoking this skill authorizes the subagents below.

Read whatever artifacts exist for the concept from SCS (see the `artifacts` skill): the `problem` (required), and the `research`, `specification`, and `design` if present. A missing optional artifact returns `artifact_has_no_saved_revision` — treat it as absent and work from what remains. The steps below adapt to what is available: requirements come from the `specification` (or the `problem` when there is no spec), and structure comes from the `design` when one exists. Break the available context into a `tasks` artifact and store it back in SCS.

## Step 1 — Write the breakdown

Spawn a subagent with a model appropriate for complex reasoning:

```
Think hard.

Break the available context below into tracer-bullet tasks. Each task is a thin vertical slice that is demoable or verifiable on its own — not a horizontal slice of one layer. Prefer many thin slices over few thick ones.

For each task, write:
- What to build — a short prose paragraph naming the end-to-end behaviour the slice delivers. No file paths or code; the implementer curates those at run time.
- Acceptance criteria — one or more Given/When/Then triples. Each is a test the implementer writes first.
- Blocked by — earlier tasks this depends on, or "None".
- Notes — 3–5 sentences of key decisions, edge cases, or fixture pointers.

Rules:
- Order tasks so dependencies point backwards: task N's "Blocked by" references only lower-numbered tasks. The set forms a flat ordered list and a DAG.
- Every requirement in the available context is addressed by at least one task. Where a design exists, every component it defines is delivered by some task, and its test strategy is carried out by the relevant tasks.
- If the specification marks a requirement as needing a pin test (a preserved behaviour with no existing test, from `extract-spec`), schedule that pin test as an early task that lands before any task refactoring the same area.
- Each task ends green on its own. Partial components are fine — the last task to touch a component completes it.
- Every task has at least one demoable acceptance criterion. The single exception is a contract change consumed by other code (e.g. a signature or schema update); flag any such task as a candidate for a separate prior PR.
- Context budget: each task, plus the code the implementer will curate around it, should fit comfortably in 30–40% of a context window. If a task looks heavy, split it.
- Use the `conventions` skill to apply the repository's guidelines for where tests live and how work is sliced and committed.

Follow the `language` skill for tone and vocabulary. Output only the task list.

Read the available artifacts for concept `{concept_name}` from SCS (see the `artifacts` skill) — the `problem`, and the `research`, `specification`, and `design` if present. That is the context to break down.
```

Store the result as the `tasks` artifact on the concept (see the `artifacts` skill).

## Step 2 — Review

Use the `review` skill for a Task Breakdown Review of the `tasks` artifact on the concept. It checks the breakdown against all the available context (problem, research, specification, design — whichever exist): demoability, requirement and design-component coverage, ordering, and per-task shape, loading the domain skills for slice soundness.

The review returns findings by severity. Treat P0 and P1 as the findings to fix in Step 3.

## Step 3 — Fix

If the review found no P0/P1 findings, you are done.

Otherwise spawn a fix subagent to apply the findings:

```
Think hard.

Apply the review findings below to the task breakdown. Change only the task breakdown — never the source artifacts (`problem`, `research`, `specification`, `design`).

If a finding cannot be resolved without contradicting or going beyond the available context (the `specification`, `design`, or `problem`), do NOT change the breakdown to fit it. Instead report it as a SOURCE CONFLICT, naming the finding and the specific artifact and part it clashes with.

Keep all other tasks as they are. Follow the `language` skill.

Read these from SCS for concept `{concept_name}` (see the `artifacts` skill): the available source artifacts (`problem`, and `research`, `specification`, `design` if present) and the current `tasks` artifact (the breakdown to fix).

Findings:
{findings}
```

Save the fixed breakdown as a new revision of the `tasks` artifact, then re-run Step 2. Repeat until the review passes.

**Escalate only on source conflict.** Apply ordinary fixes autonomously without involving the user. The single time you stop and raise to the user is when a fix reports a SOURCE CONFLICT — present the conflict, naming the artifact it clashes with, and let the user decide whether to amend that artifact or drop the requirement before continuing.
