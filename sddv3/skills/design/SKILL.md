---
name: design
description: Weigh candidate approaches with the user, pick a direction, and produce the best possible design for the job at a depth that matches the risk. Use this skill after a problem is defined (and optionally after explore and spec), when you want to converge on HOW to build something. Sharpens the user's thinking and stress-tests the choice rather than deciding for them, then writes the design. Optional — a user may skip it and let breakdown work from whatever exists.
version: 0.1.0
---

# Design

**Design** produces the _how_: it converges with the user on an approach, then writes the best design for the job at a depth that matches the risk. It runs after a problem is defined, and optionally after `explore` and `spec`, before `breakdown`. It is an **optional** step — the user may judge a design unnecessary and let `breakdown` work from whatever exists.

Both this orchestrator and every subagent it spawns follow the `language` skill for tone and vocabulary in all output, including replies to the user. Invoking this skill authorizes the subagents below.

Read from SCS for the concept (see the `artifacts` skill):

- `problem` — the problem definition. Required.
- `research` — optional; a missing artifact returns `artifact_has_no_saved_revision`. Treat that as "no research".
- `specification` — optional; the agreed requirements and constraints. If it exists, treat its requirements as the criteria the design must satisfy. If it does not, establish the criteria and constraints with me before diverging.

# Step 1 — Converge on an approach

You are helping me converge on how to build this. You may have a problem definition, research, and a specification. Do not recommend a solution yet. Your role is to sharpen my thinking, not replace it.

Work through these phases in order. Stop at each phase that asks for my input.

## Phase 1 — Elicit my instinct and the technical risks I see.

Before you say anything substantive, ask me two things: where my instinct points — what direction I'm leaning toward and why — and what technical risks I already see, the parts I expect to be hard, fragile, or uncertain. Then stop and wait for my answer. Do not offer your own view first — I want my thinking on record before yours can anchor it. If I name a direction, treat it as one hypothesis to test later, NOT as the target to satisfy. Carry the risks I name into the diverge and interrogate phases, and into how deep the design goes.

## Phase 2 — Diverge (enumerate, don't decide).

Now lay out the candidate approaches the problem, any research, and any spec support — including any I proposed and any I missed. For each: how it works, what it trades off, and which finding or constraint makes it more or less viable.

Use the relevant domain skills (`security`, `api-design`, `distributed-systems`, `data-engineering`, `devops-sre`, `infrastructure`, `low-level-systems`) to validate each option. Point out any clear problems they have, particularly if they're unsolvable, have an impact on resource usage or costs, or could have a negative impact on the user.

Do not rank them and do not recommend one. You should be able to argue for several in different directions; you should not have a preferred answer in mind. The goal is to widen the menu, not narrow it.

## Phase 3 — Interrogate my choice.

After I pick, switch to adversary. Give me the strongest case that my choice is wrong, the conditions under which it fails, and what I'd have to believe for it to be right. Then surface the questions I still need to answer to be confident — ask them, don't answer them for me.

## Constraints throughout:

Don't walk me to a predetermined answer with leading questions. If you're asking questions, they should be genuinely open.
The selection is mine. Your jobs are enumerating the option space, finding gaps in my thinking, and stress-testing my decision — not making it.
Keep your own preferences out of phases 1–2.

# Step 2 — Write the design

The technical risks I named in Phase 1, and what the interrogation in Phase 3 surfaced, together mark where this choice is risky, likely to change, or still unresolved. Let that set how much design to do and where to go deep — deepen those parts, and keep the rest light. Read the `interface-design` skill for how to keep components testable (accept dependencies, return results, small surfaces), and use the `conventions` skill to apply the repository's guidelines for architecture, error handling, naming, and structure.

Always present, at any depth:

- The components and their boundaries, and for each boundary, what change it keeps local. Draw boundaries where the requirements are most likely to change, so a likely change stays inside one component.
- The key decisions made and why. Pin down decisions the implementer should not have to relitigate, but avoid implementation details and don't constrain the implementer on things they should own.
- The key technical risks — the ones I raised in Phase 1 and the ones the interrogation surfaced — and, for each, how the design addresses it and where it goes deep. This record is what a later review checks the design's depth against.

Deepen only where the interrogation surfaced risk or likely change — for those parts, work out the data flow, interfaces, sequencing, and failure modes. Keep the rest light.

Add the test strategy: use dependencies as a heuristic. More foundational components get more unit tests; end-user and consumer-facing outcomes are covered by smoke/e2e/qa tests at the system boundary.

Aim for a max of 400 lines; expand only the parts risk justified.

Store the document as the `design` artifact on the concept in SCS (see the `artifacts` skill).

# Step 3 — Review

Use the `review` skill for a Design Review of the `design` artifact on the concept.

Let the user know if any P0 or P1 findings surface. Save any fixes as a new revision of the `design` artifact, then re-run the review until it passes.
