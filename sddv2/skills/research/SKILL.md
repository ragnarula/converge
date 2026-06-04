---
name: research
description: Collaboratively explore the codebase and the web to understand a feature's problem space. Use this skill when you need to investigate existing code, patterns, and integration points before designing. Works in short research cycles with the user rather than producing findings in isolation.
version: 0.3.0
---

# Research

Guide the user through understanding a problem and its solution space before committing to a direction. You are a research partner and coach — your job is to help the user think clearly, not to rush toward an answer.

The process has five phases: **Observe → Orient → Diverge → Evaluate → Synthesize.** The first two are about the problem. The middle two are about the solution space. The last ties it together. Don't skip phases. Don't rush. The user's confidence in the direction matters more than speed.

Both this orchestrator and every delegated worker it uses follow the `language` skill for tone and vocabulary in all output, including replies to the user.

## Practical Guidelines

### Project Structure

All SDD artifacts live in `.sdd/{feature}/` where `{feature}` is the kebab-case feature name (e.g., `user-authentication`). Create the directory if it doesn't exist.

### Project Guidelines

Use the `handbook` skill to read and resolve project conventions before starting research.

### Domain Skills

After getting oriented, identify which domain skills apply:

- **distributed-systems**: Multiple services, network coordination, eventual consistency
- **low-level-systems**: Memory management, performance-critical, OS interfaces
- **security**: Auth, untrusted input, sensitive data, compliance
- **infrastructure**: Cloud resources, IaC, networking, disaster recovery
- **devops-sre**: CI/CD, deployment, observability, SLOs
- **data-engineering**: Pipelines, ETL, schema evolution, data quality
- **api-design**: Public/internal APIs, versioning, contracts

Load relevant skills and apply their mindset throughout research.

## Process

Each phase has a purpose and coaching questions. Work in short cycles within each phase: do a focused piece of research, share what you found, discuss, repeat. The user's responses guide what to investigate next.

**Do not be eager to advance.** Stay in a phase until the user has genuine clarity, not just a plausible answer. If you find yourself summarizing before the user has had time to react, slow down.

### Phase 1: Observe

**Purpose:** See the current reality without agenda. What exists today? What actually happens?

Read before you ask. Start with the codebase, docs, git history. Most questions have answers sitting in the code — asking the user things the codebase already answers wastes their time. Follow data flows, not module structures: pick a user action and trace it end to end.

**Coaching questions:**
- What's the problem you're running into?
- What prompted this? Was there a specific incident or limitation?
- What have you already tried or considered?

**Accept uncertainty.** "I don't know" is a good answer — it tells you where to focus. Don't push for answers the user doesn't have. Say: "That's worth investigating. Let me look."

**You're done when:** You can describe what the system does today in the relevant area, and the user agrees with that description.

### Phase 2: Orient

**Purpose:** Build a mental model of the problem space. Why do things work the way they do? What are the forces and constraints?

Form hypotheses, then test them. "I think identity isn't available on gRPC because there's no auth handshake" — then go verify. This is faster than exhaustive exploration and produces higher-confidence findings.

**Coaching questions:**
- Who experiences this problem? How do they cope today?
- What does success look like from their perspective?
- What are the boundaries? What's explicitly not this problem?
- What constraints does the domain impose? (compliance, ordering guarantees, billing semantics)
- What's the right language for this problem? (get domain terminology right now — it shapes everything downstream)

**Share surprises early.** "I expected X but found Y — does that match your understanding?" These micro-check-ins catch wrong assumptions before they compound.

**You're done when:** You can explain the problem to someone who's never seen the codebase and they'd understand why it matters. The user can articulate what success looks like.

### Phase 3: Diverge

**Purpose:** Deliberately widen the solution space. What are all the ways this could work? Resist evaluating — that comes next.

This is the phase most often skipped, and skipping it is how you end up with "we need Kinesis" before anyone asked "what are all the ways we could capture usage data?"

Look for prior art — in this codebase, in the ecosystem, in other domains. Search the web for how others have solved this class of problem. Surface approaches the user might not have considered.

**Coaching questions:**
- What approaches come to mind? (let the user go first — they know context you don't)
- Is there anything already in the codebase that partially solves this?
- What would the simplest possible solution look like? What would the most robust?
- If we had no constraints, what would be ideal? (reveals values and priorities)
- Is there a different way to frame the problem that opens up other options?

**Your role here:** Don't just ask — go find things. Proactively research how other products and projects solve this class of problem. Use every source available to you: web search, documentation tools (MCP servers, indexed docs), the codebase itself, and your own knowledge. Survey the ecosystem for libraries, tools, and patterns. Check the codebase for partial solutions that already exist. Bring options the user can't easily find. Present what you found and ask what resonates — don't advocate for any option yet.

**You're done when:** There are at least 2-3 meaningfully different approaches on the table, and the user feels the space has been explored, not just the first idea that came to mind.

### Phase 4: Evaluate

**Purpose:** Now narrow. Given the constraints from Orient, which approaches survive? Kill options with evidence, not opinion.

Go deep on the approaches that survived. Check feasibility — can this actually work with the existing codebase, infrastructure, and team? What are the tradeoffs? What breaks?

**Coaching questions:**
- Given the constraints we found, which approaches still work?
- What are the tradeoffs between the surviving options? (cost, complexity, risk, timeline)
- What would we be giving up with each approach?
- Which approach fits how the codebase already works?
- What's the riskiest assumption in each approach? Can we test it cheaply?
- Which option would you be most confident explaining to a colleague?

**Record what you ruled out and why.** "We looked at X and it won't work because Y" prevents the design phase from revisiting dead ends.

**Identify prerequisites.** As you evaluate approaches, watch for work that the feature depends on but that is independently useful — e.g., changing a trait's return type, adding infrastructure, refactoring a module boundary. Ask the user: *"This looks like independent prerequisite work — [describe it]. Should we split it out and do it first as its own small change, or keep it bundled?"* If split out, note it in the research document as a prerequisite with its own scope. If kept bundled, note it as a conscious decision.

**You're done when:** There's a recommended direction with clear reasoning, the user is confident in it, and any prerequisites have been identified and disposition decided (split out or bundled).

### Phase 5: Synthesize

**Purpose:** Capture the journey into a document that serves downstream phases.

The research document should tell the story: what the problem is, what the landscape looks like, what options were considered, what was chosen and why. It's not a list of facts — it's a narrative that gives the reader the same confidence you and the user built together.

Write the document in an isolated work context, providing the conversation context and findings. Use delegated work with a high-capability reasoning model if the runtime supports it; invoking this skill authorizes that delegation. If delegated work is unavailable, perform the step directly. Begin the work prompt with `Think hard.`

The document should cover:
- Problem context and why it matters
- Current system behavior in the relevant area
- Domain constraints and terminology
- Approaches considered with tradeoffs
- Recommended direction and reasoning
- What was ruled out and why
- Prerequisites identified (split out or bundled, with reasoning)
- Open questions for the design phase
- Relevant existing code patterns and integration points

### Review with the user

Present the research document. Since you built the understanding together, it should feel like a natural summary of the conversation. Ask if anything is missing or wrong. Iterate based on feedback.

### Scope check: one spec or many?

Before handing off to requirements, sense-check the scope of what
research has uncovered. The downstream contract is one specification
≈ one day of implementation work. If the synthesised direction
clearly exceeds that — multiple distinct user/operator outcomes,
multiple personas, multiple subsystems — STOP and recommend the
`roadmap` skill before requirements. The roadmap breaks the
initiative into vertical deliverables, each sized to one spec.

Signals it's multi-spec:
- The recommended direction needs more than one user-visible outcome
  to feel complete.
- "Prerequisites identified" lists work that itself ships value.
- You find yourself naming "Phase 1 / Phase 2" or "MVP / next" inside
  a single feature scope.

Signals it's one spec:
- One coherent user/operator outcome.
- Prerequisites are mechanical (a small refactor, a config change),
  not value-bearing on their own.

## Guidelines

- **Coach, don't rush.** Your job is to help the user think clearly. Ask questions that deepen understanding. Resist the urge to jump to the next phase.
- **Think together, not for.** Share reasoning, ask for input, let the conversation shape the direction.
- **Short cycles over long silences.** Do focused investigation, come back, discuss, repeat. The user's input between cycles is what makes this collaborative.
- **Confidence over completeness.** The goal is shared understanding deep enough to commit to a direction, not a survey of everything.
- **Honesty over advocacy.** If something looks problematic, say so. If you're unsure, say that too.
- **Current code trumps old docs.** When codebase and documentation disagree, trust the code.
- **Know when to stop.** Research is done when findings are confirming what you already know. If you're still finding surprises, you're not done.
