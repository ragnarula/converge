---
name: problem-research_v3
description: Given a problem definition, research the information needed to frame it. Use this skill after a problem is defined and before solution work — it decides the key questions that must be answered, then answers them. First enumerate and tag the askable space (kill-shot and disconfirming questions first), then research the chosen questions.
version: 0.1.0
---

# Problem Research

Both this orchestrator and every subagent it spawns follow the `language_v3` skill for tone and vocabulary in all output, including replies to the user. Invoking this skill authorizes the subagents below.

Read the `problem` artifact for the concept from SCS (see the `artifacts_v3` skill). Create a temporary local research document as a scratchpad; the final research lands in SCS.

The output is a **description of the current state** — what is true, with evidence — that answers the chosen questions. It is NOT a recommendation and does NOT pick a direction or propose a solution. Choosing an approach is the `problem-converge_v3` skill's job; research that recommends pre-empts it and biases the decision.

Spawn a subagent with a model appropriate for complex reasosning with the following prompt:

```
You are helping frame a problem before any solution work begins. You will be given a problem definition.
First, assess whether the problem is well-posed enough to research. A problem is well-posed only if it states what would count as a solution (success criteria). If it doesn't, do NOT generate research questions — instead, say so and ask the 1–2 questions needed to sharpen the frame. Generating research questions toward an undefined target manufactures false progress.
If it is well-posed, generate research questions, and for each one provide:

- Type — empirical (answered by measuring/testing), factual (answered by looking up / prior art), or judgment (no lookup answer; resolved by a decision or tradeoff).
- Leverage — pivotal (the answer selects between fundamentally different approaches, or kills one), tuning (adjusts a chosen approach), or inert (interesting but moves no decision).
- Why it matters — name what the user would do differently depending on the answer. If you can't name two distinct actions for two distinct answers, drop the question.

Then highlight separately:

- Kill-shot questions — high-leverage questions where one possible answer would invalidate the whole direction. These should be answered first and cheaply.
- Disconfirming questions — questions that test whether the stated problem is the real problem, which the user may be motivated not to ask.

Do not rank the full list by importance — leverage and uncertainty depend on context you don't have. Your job is to enumerate the askable space and tag it, so the user can decide what's worth pursuing.

Aim for 3-6 relevant questions.

Follow the `language_v3` skill for tone and vocabulary.

Read the `problem` artifact for concept `{concept_name}` from SCS (see the `artifacts_v3` skill) — that is the problem definition to work from.
```

While the subagent is working, ask the user what questions they believe should be answered before proceeding.

When the subagent completes, ask the user if they want to remove or can already answer any of the agent's suggested questions.

Save all the chosen questions to the temporary research document.

Spawn subagents with a model appropriate for search in batches of 3 to answer the questions posed, each with this prompt:

```
Answer the research question below for the given problem. Report only what is true and the evidence for it — describe the current state, facts, prior art, measurements, and tradeoffs.

Do NOT recommend a solution, pick a direction, or say which approach is best — that decision is made later, not here. State tradeoffs neutrally; if the evidence is mixed or you couldn't find an answer, say so. Cite sources or files for each claim.

Anything you surface that was not already in the problem definition — a system behaviour, a constraint, a library's limits, a number, a piece of prior art, a term of art — must be captured with enough context to stand on its own: explain what it is, where it came from, and why it bears on the question. A later reader who only has this document, not your search results, should understand each finding without looking anything up. Follow the `language_v3` skill for tone and vocabulary.

Read the `problem` artifact for concept `{concept_name}` from SCS (see the `artifacts_v3` skill) for the problem definition.

Question:
{question}
```

Append each answer to the research document as it completes.

## Capture findings with full context

The value of this artifact is the information discovered during research — facts that were not in the problem definition. Every such finding must be captured so it stands on its own: explain what it is, where it came from (cite the source or file), and why it matters to the question. Define terms of art and give the background a reader needs. Do not leave a finding as a bare reference or a claim that only makes sense if you saw the search results.

Do not restate the problem — assume the reader has the problem definition. The job here is to make the newly captured knowledge fully understandable on its own.

The appended answers are raw subagent output. Before storing, synthesize them into one coherent document — deduplicate overlapping findings, order them sensibly, and resolve contradictions between answers (or note them). Edit the whole document to follow the `language_v3` skill for tone and vocabulary; do not leave it as a stitched-together transcript.

Store the document as the `research` artifact on the concept in SCS (see the `artifacts_v3` skill). Try to limit it to 600 lines.
