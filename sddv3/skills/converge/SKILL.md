---
name: converge
description: Given a defined problem and its research, work with the user to converge on an approach and produce a specification. Use this skill after frame and explore, when the user wants to weigh candidate approaches, pick a direction, and turn it into a spec. Sharpens the user's thinking rather than choosing for them, then writes the specification.
version: 0.1.0
---

# Converge

Both this orchestrator and every subagent it spawns follow the `language` skill for tone and vocabulary in all output, including replies to the user. Invoking this skill authorizes the subagents below.

Read the `problem` and `research` artifacts for the concept from SCS (see the `artifacts` skill). These are the `{problem}` and `{research}` referred to below — load them before Step 1.

# Step 1 - Converge

You are helping me converge on an approach to a problem. You have a problem definition and the research I've gathered. Do not propose or recommend a solution yet. Your role is to sharpen my thinking, not replace it.

Work through these phases in order. Do not skip ahead, and stop at each phase that asks for my input.

## Phase 1 — Elicit my read of the research.

Before you say anything substantive, ask me: what's my read of the research? What stood out, what surprised me, does anything shift the original framing? Then stop and wait for my answer. Do not offer your own interpretation first — I want my thinking on record before yours can anchor it.

## Phase 2 — Elicit my criteria.

Once I've responded, ask what I want a solution to achieve — the criteria for "good" here, my constraints and priorities, and whether I'm already leaning toward an approach. Then stop and wait. If I name a preferred approach, treat it as one hypothesis to test later, NOT as the target to satisfy.

## Phase 3 — Diverge (enumerate, don't decide).

Now lay out the candidate approaches the research supports — including any I proposed and any I missed. For each: how it works, what it trades off, and which research finding or constraint makes it more or less viable.

Use the relevant domain skills (`security`, `api-design`, `distributed-systems`, `data-engineering`, `devops-sre`, `infrastructure`, `low-level-systems`) to validate each option. Point out any clear problems they have, particularly if they're unsolvable, have an impact on resource usage or costs, or could have a negative impact on the user.

Do not rank them and do not recommend one. You should be able to argue for several in different directions; you should not have a preferred answer in mind. The goal is to widen the menu, not narrow it.

## Phase 4 — Interrogate my choice.

After I pick, switch to adversary. Give me the strongest case that my choice is wrong, the conditions under which it fails, and what I'd have to believe for it to be right. Then surface the questions I still need to answer to be confident — ask them, don't answer them for me.

## Constraints throughout:

Don't walk me to a predetermined answer with leading questions. If you're asking questions, they should be genuinely open.
The selection is mine. Your jobs are enumerating the option space, finding gaps in my thinking, and stress-testing my decision — not making it.
Keep your own preferences out of phases 1–3.

# Step 2 - Create a specification

Write a draft specification for the chosen direction. To do it, the following details need to be clear. If they're not, ask the user to clarify.

- Requirements - The key requirements written in EARS format (see the `ears` skill). These should be user or consumer facing outcomes only, not implementation details.
- Design - The solution design, broken down into components. Enough detail to express the design, but avoid implementation details. Pin down decisions made, but don't constrain the implementer on things they should own.
- Test Plan - Testing strategy for the work. What unit/smoke/integration/e2e/qa etc tests need to be added. Use dependencies as a heuristig. More foundational components should have more unit tests. End user and consumer facing outcomes should have smoke/e2e/qa tests at the system boundary.
- Documentation - Any docs that should be updated or created. Consider both users and developers.

Aim for a max of 600 lines.

Store the document as the `specification` artifact on the concept in SCS (see the `artifacts` skill). Then fan out relevant domain review subagents using the domain skills above; have each read the `specification` artifact for concept `{concept_name}` from SCS rather than receiving it in the prompt. Let the user know if any major issues surface, and save any fixes as a new revision of the `specification` artifact.
