---
name: spec
description: Turn a defined problem into the best possible specification — the requirements, constraints, and documentation for the job, independent of any solution. Use this skill after frame (and optionally explore) when you want to pin down WHAT must be true before deciding HOW. Sharpens the user's requirements rather than choosing a solution, then writes the specification. Optional — a user may skip straight to design or breakdown when a spec adds nothing.
version: 0.1.0
---

# Spec

**Spec** produces the *what*: the requirements, constraints, and documentation for the job, expressed independently of any solution. It runs after `frame` (and optionally `explore`), and before `design`. It is an **optional** step — the user may judge a spec unnecessary and go straight to `design` or `breakdown`.

Both this orchestrator and every subagent it spawns follow the `language` skill for tone and vocabulary in all output, including replies to the user. Invoking this skill authorizes the subagents below.

Read the `problem` artifact for the concept from SCS (see the `artifacts` skill). Also read the `research` artifact if it exists — research is optional, so a missing artifact returns `artifact_has_no_saved_revision`; treat that as "no research" and continue. These are the `{problem}` and (optional) `{research}` referred to below.

# Step 1 — Sharpen the requirements

You are helping me pin down what a solution must achieve, before anyone picks how to build it. Do not propose or design a solution — that is the `design` skill's job. Keep the requirements solution-independent.

Work through these phases in order. Stop at each phase that asks for my input.

## Phase 1 — Elicit my criteria.

Ask me what I want a solution to achieve — the outcomes that define "good" here, my constraints and priorities, and who the users and consumers are. Then stop and wait. Do not offer requirements of your own first — I want my thinking on record before yours can anchor it.

## Phase 2 — Grill me until the scope is unambiguous.

Interrogate the scope until it is clear and unambiguous. This is a loop, not a single question. Ask me where the scope is unclear: where this work starts and stops, which requirements could be read more than one way, and what is in or out of scope. When I answer, look at what my answer leaves open — new ambiguities it introduces, edge cases it does not cover, boundaries I named only vaguely — and question those too. Keep going, one sharp question at a time, until you can find no remaining ambiguity in the scope. Do not move on while any part of the scope could still be read two ways. Do not decide for me what is in scope — surface each ambiguity and make me resolve it.

## Phase 3 — Draft and validate the requirements.

Write the key requirements in EARS format (see the `ears` skill). These are user- or consumer-facing outcomes only, not implementation details. Present them to me, then stop and wait. Revise until I confirm the requirements are right — everything downstream builds on them, so do not move on until they hold.

# Step 2 — Write the specification

Once the requirements are agreed, write the specification. Confirm the details below are clear; ask me if any are not.

- Requirements — the agreed requirements in EARS format. User- or consumer-facing outcomes only, not implementation details.
- Constraints — the constraints and priorities that bound any solution, measurable where they set a limit. Not implementation choices.
- Documentation — any docs that should be updated or created. Consider both users and developers.

Aim for a max of 400 lines. The specification states the *what*; leave the *how* to `design`.

Store the document as the `specification` artifact on the concept in SCS (see the `artifacts` skill).

# Step 3 — Review

Use the `review` skill for a Specification Review of the `specification` artifact on the concept.

Let the user know if any P0 or P1 findings surface. Save any fixes as a new revision of the `specification` artifact, then re-run the review until it passes.
