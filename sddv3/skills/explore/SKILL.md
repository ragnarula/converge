---
name: explore
description: Given a problem definition, research the information needed to frame it. Use this skill after a problem is defined and before solution work — it decides the key questions that must be answered, then answers them. First enumerate and tag the askable space (kill-shot and disconfirming questions first), then research the chosen questions.
version: 0.1.0
---

# Explore

Both this orchestrator and every subagent it spawns follow the `language` skill for tone and vocabulary in all output, including replies to the user. Invoking this skill authorizes the subagents below.

Read the `problem` artifact for the concept from SCS (see the `artifacts` skill). Create a temporary local research document as a scratchpad; the final research lands in SCS.

The output is a **description of the current state** — what is true, with evidence — that answers the chosen questions. It is NOT a recommendation and does NOT pick a direction or propose a solution. Choosing an approach is the `design` skill's job; research that recommends pre-empts it and biases the decision.

## What makes a good research question

A research question is the unit this skill is built on. Both the generation step and the final document are only as good as the questions. A good research question is:

- **Answerable** — you can name the kind of evidence that settles it: a measurement, a lookup, a piece of prior art, or a recorded decision. If nothing could answer it, it is not a research question.
- **Specific and bounded** — narrow enough that the answer is a concrete finding, not a survey. *What is the p99 write latency of X at 1k writes/sec?* — not *How does X perform?*
- **One question** — asks a single thing. Split *Does it scale and is it secure?* into two questions.
- **Decision-relevant** — the answer changes a choice the user has to make. If you cannot name two different actions for two different answers, drop it.
- **Neutral** — it does not presume its own answer or smuggle in a solution. *What consistency guarantees does X give?* — not *X is eventually consistent, right?*
- **Shaped** — you can picture the form of the answer before you have it (a number, a yes/no with conditions, a list of prior art), which is how you know it is answerable.

Spawn a subagent with a model appropriate for complex reasoning with the following prompt:

```
You are helping frame a problem before any solution work begins. You will be given a problem definition.
First, assess whether the problem is well-posed enough to research. A problem is well-posed only if it states what would count as a solution (success criteria). If it doesn't, do NOT generate research questions — instead, say so and ask the 1–2 questions needed to sharpen the frame. Generating research questions toward an undefined target manufactures false progress.
If it is well-posed, generate research questions. Each question must be: answerable (you can name the evidence that settles it), specific and bounded (the answer is a concrete finding, not a survey), a single question (not a compound), decision-relevant (the answer changes a choice), neutral (it does not presume its answer or smuggle in a solution), and shaped (you can picture the form of the answer before you have it). For each one provide:

- Type — empirical (answered by measuring/testing), factual (answered by looking up / prior art), or judgment (no lookup answer; resolved by a decision or tradeoff).
- Leverage — pivotal (the answer selects between fundamentally different approaches, or kills one), tuning (adjusts a chosen approach), or inert (interesting but moves no decision).
- Why it matters — name what the user would do differently depending on the answer. If you can't name two distinct actions for two distinct answers, drop the question.

Then highlight separately:

- Kill-shot questions — high-leverage questions where one possible answer would invalidate the whole direction. These should be answered first and cheaply.
- Disconfirming questions — questions that test whether the stated problem is the real problem, which the user may be motivated not to ask.

Do not rank the full list by importance — leverage and uncertainty depend on context you don't have. Your job is to enumerate the askable space and tag it, so the user can decide what's worth pursuing.

Aim for 3-6 relevant questions.

Follow the `language` skill for tone and vocabulary.

Read the `problem` artifact for concept `{concept_name}` from SCS (see the `artifacts` skill) — that is the problem definition to work from.
```

While the subagent is working, ask the user what questions they believe should be answered before proceeding.

When the subagent completes, ask the user if they want to remove or can already answer any of the agent's suggested questions.

Save all the chosen questions to the temporary research document.

Spawn subagents with a model appropriate for search in batches of 3 to answer the questions posed, each with this prompt:

```
Answer the research question below for the given problem. Report only what is true and the evidence for it — describe the current state, facts, prior art, measurements, and tradeoffs.

Do NOT recommend a solution, pick a direction, or say which approach is best — that decision is made later, not here. State tradeoffs neutrally; if the evidence is mixed or you couldn't find an answer, say so. Cite sources or files for each claim.

Anything you surface that was not already in the problem definition — a system behaviour, a constraint, a library's limits, a number, a piece of prior art, a term of art — must be captured with enough context to stand on its own: explain what it is, where it came from, and why it bears on the question. A later reader who only has this document, not your search results, should understand each finding without looking anything up. Follow the `language` skill for tone and vocabulary.

Keep your answer to this one question tight: as long as it needs to be to stand on its own, and no longer. Most answers fit in a few hundred words; let a genuinely complex question run longer. Report the findings, not the search you ran to get them.

Read the `problem` artifact for concept `{concept_name}` from SCS (see the `artifacts` skill) for the problem definition.

Question:
{question}
```

Append each answer to the research document as it completes.

## Write the research document

The appended answers are raw subagent output. They are the input to this step, not the output. Before storing, turn them into one standalone document a reader can pick up cold — someone who has the problem definition but never saw the research. The document must read well, not like a stitched-together transcript.

**Keep the structure of the questions.** Each chosen question is a section with the question as its heading. A reader scanning the headings sees the askable space the research covered. Do not collapse the questions into a single essay, and do not reorder them into a recommendation.

**Capture every finding so it stands on its own.** The value of this artifact is the information discovered during research — facts that were not in the problem definition. For each one, explain what it is, where it came from (cite the source or file), and why it matters to the question. Define terms of art and give the background a reader needs. Do not leave a finding as a bare reference or a claim that only makes sense if you saw the search results. Do not restate the problem.

**Deduplicate facts across questions.** A fact often answers more than one question. State it in full once, under the question it best belongs to, and refer back to it from the others rather than repeating it. The reader should never read the same finding twice.

**Editorialize for readability.** Cut filler, merge overlapping sentences, resolve contradictions between answers (or note them), and order the findings within each section so they build. The goal is a document that is easy to read, not exhaustive to read.

**Apply the `language` skill** to the whole document — active voice, simple English, no metaphors or slang.

**Length is per question, not per document.** Keep each question's section tight: as long as it needs to be to stand alone, and no longer. Most sections are a few hundred words; let a genuinely complex question run longer. The document as a whole has no length cap — a thorough research effort with many questions is allowed to be long, as long as no single section is padded.

Store the document as the `research` artifact on the concept in SCS (see the `artifacts` skill).
