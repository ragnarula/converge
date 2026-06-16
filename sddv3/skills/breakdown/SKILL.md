---
name: breakdown
description: Break a converged specification into a simple ordered task list for the problem-driven flow. Use this skill after converge has produced a specification and you want demoable, ordered implementation tasks. A simplified take on tasks — it reviews and fixes the breakdown autonomously, raising to the user only when a fix would conflict with the spec.
version: 0.1.0
---

# Breakdown

Both this orchestrator and every subagent it spawns follow the `language` skill for tone and vocabulary in all output, including replies to the user. Invoking this skill authorizes the subagents below.

Read the `specification` artifact for the concept from SCS (see the `artifacts` skill). In the problem-driven flow the spec already holds the requirements, design, and test plan. Break it into a `tasks` artifact and store it back in SCS.

## Step 1 — Write the breakdown

Spawn a subagent with a model appropriate for complex reasoning:

```
Think hard.

Break the specification below into tracer-bullet tasks. Each task is a thin vertical slice that is demoable or verifiable on its own — not a horizontal slice of one layer. Prefer many thin slices over few thick ones.

For each task, write:
- What to build — a short prose paragraph naming the end-to-end behaviour the slice delivers. No file paths or code; the implementer curates those at run time.
- Acceptance criteria — one or more Given/When/Then triples. Each is a test the implementer writes first.
- Blocked by — earlier tasks this depends on, or "None".
- Notes — 3–5 sentences of key decisions, edge cases, or fixture pointers.

Rules:
- Order tasks so dependencies point backwards: task N's "Blocked by" references only lower-numbered tasks. The set forms a flat ordered list and a DAG.
- Every requirement in the spec is addressed by at least one task. Every test named in the spec's test plan is produced by some task's work.
- Each task ends green on its own. Partial components are fine — the last task to touch a component completes it.

Follow the `language` skill for tone and vocabulary. Output only the task list.

Read the `specification` artifact for concept `{concept_name}` from SCS (see the `artifacts` skill) — that is the specification to break down.
```

Store the result as the `tasks` artifact on the concept (see the `artifacts` skill).

## Step 2 — Review

Spawn a review subagent with a model appropriate for complex reasoning:

```
Think hard.

Review the task breakdown below against the specification. Report findings as P0 (breaks a stated rule) or P1 (spirit not met); list each with the task it concerns.

Check:
- Every task is demoable or independently verifiable. Flag horizontal slices (one layer, no observable behaviour).
- Coverage: every requirement and every test in the spec's test plan is addressed by some task. List anything uncovered.
- Ordering: "Blocked by" references only earlier tasks; no cycles; each task can run once its blockers are done.
- Each acceptance criterion is a Given/When/Then triple.

Follow the `language` skill.

Read these from SCS for concept `{concept_name}` (see the `artifacts` skill): the `specification` artifact and the `tasks` artifact (the breakdown to review).
```

## Step 3 — Fix

If the review found no P0/P1 findings, you are done.

Otherwise spawn a fix subagent to apply the findings:

```
Think hard.

Apply the review findings below to the task breakdown. Change only the task breakdown — never the specification.

If a finding cannot be resolved without contradicting or going beyond the specification, do NOT change the breakdown to fit it. Instead report it as a SPEC CONFLICT, naming the finding and the specific part of the spec it clashes with.

Keep all other tasks as they are. Follow the `language` skill.

Read these from SCS for concept `{concept_name}` (see the `artifacts` skill): the `specification` artifact and the current `tasks` artifact (the breakdown to fix).

Findings:
{findings}
```

Save the fixed breakdown as a new revision of the `tasks` artifact, then re-run Step 2. Repeat until the review passes.

**Escalate only on spec conflict.** Apply ordinary fixes autonomously without involving the user. The single time you stop and raise to the user is when a fix reports a SPEC CONFLICT — present the conflict and let the user decide whether to amend the spec or drop the requirement before continuing.
