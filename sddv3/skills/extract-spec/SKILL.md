---
name: extract-spec
description: Make the behavior preservation contract for a refactor or migration explicit and testable. Reads existing code and tests, interviews the user to classify current behavior, and produces a specification in the same shape as `spec` — each preserved requirement backed by an existing test or a pin test added before the refactor begins.
version: 0.2.0
---

# Extract Spec

**Extract Spec** makes the behavior preservation contract for a refactor or migration explicit and testable. It reads existing code and tests, then collaborates with the user to classify current behavior, and produces a specification in the same shape as the one `spec` produces.

It runs in place of `spec`, when the work preserves existing behaviour rather than defining new behaviour; `design` (optional) and `breakdown` follow as usual.

Both this orchestrator and every subagent it spawns follow the `language` skill for tone and vocabulary in all output, including replies to the user. Invoking this skill authorizes the subagents below.

SDD artifacts are stored in SCS via the `scs` MCP server, not local files. Use the `artifacts` skill for the storage contract. Extract Spec writes the `specification` artifact on the concept named `{feature}` — the same artifact `spec` produces.

## What this skill does and does not guarantee

This skill makes the team's intent about behavior preservation explicit. It does not guarantee preservation — emergent behavior, implicit contracts with downstream consumers, and edge cases the test suite does not exercise can drift through any refactor.

The mitigation built into this skill: every preserved requirement is exercised by an existing test, or by a pin test added as a task before refactor work begins. The spec records which is which; `breakdown` sequences the pin tests first.

## When to use

- The work preserves existing behavior (refactor, migration, dependency upgrade).
- The codebase has no current spec for the area being touched, or the spec is stale.

Greenfield features use `spec`. Feature work that includes a refactor uses `extract-spec` for the preserved part and adds new requirements in the same spec via the "change me" path in the interview below.

## Process

### Step 1: Identify the area

Ask the user which code area is being refactored or migrated. Get the paths or module names. Explore with them, using available code search and file-read capabilities, to identify the boundary when the area is unclear.

### Step 2: Read the area

Read the code and its existing tests. Build a working understanding of:

- The public interfaces the area exposes (HTTP endpoints, CLI commands, event topics, library functions, etc.).
- The inputs each interface accepts.
- The outputs and side effects each interface produces.
- The error conditions each interface handles.
- The constraints the existing system delivers (observed latency, throughput, durability — measure where you can).

### Step 3: Candidate-and-confirm interview

For each candidate behavior, run a two-question check with the user.

**Question 1: Is this intentional?**

Present the candidate in EARS form (see the `ears` skill) with the supporting evidence — the test or code excerpt that demonstrates it. Ask: *intentional, accidental, or change me?*

- **Intentional** → preserved requirement in the spec. Go to Question 2.
- **Accidental** → known-issue note in the spec, or dropped if minor.
- **Change me** → new intended behavior in the spec, replacing the current one (this is "feature with refactor inside"). Treat as a greenfield requirement and go to Question 2.

**Question 2: Does an existing test cover this behavior through its public channel?**

Look for a test whose When clause matches the requirement's observable and whose Then clause asserts the same behavior.

- **Yes** → record the test path against the requirement. The refactor inherits its safety net from the existing test.
- **No** → mark the requirement as needing a pin test. `breakdown` will schedule the pin test as a task before any refactor work begins.

**For each candidate constraint:** Present the measured or stated current value. Ask: *preserve, tighten, or relax?* Record the chosen measurable target.

Iterate until every observable behavior in the area is classified.

### Step 4: Capture motivation

Ask the user why the refactor or migration is happening. The motivation goes in the spec's problem statement. It guides `design` later — the design has freedom to restructure as long as the spec's requirements are preserved.

### Step 5: Write the specification

Spawn a subagent to write the specification. Invoking this skill authorizes that subagent to use the `scs` tools.

**Subagent prompt** (use a high-capability reasoning model):
> Think hard.
>
> Write the specification for {feature} and save it as the `specification` artifact on the concept `{concept_name}` via the `scs` MCP server (create the concept, linked to the repository, if it does not exist).
>
> Your job is to define the preserved behavior of the area being refactored, as a behavioral specification — what the system does today for users, operators, auditors, and reviewers, expressed independently of the current implementation. The specification states the *what*, not the *how*: internal types, APIs, data structures, libraries, frameworks, and architectural patterns belong to the design phase.
>
> **Read:**
> - Storage contract: use the `artifacts` skill (how to read/write artifacts via `scs`)
> - EARS syntax reference: use the `ears` skill
> - Language standard: use the `language` skill
>
> **Context from extraction:**
> {paste the classified candidates here — intentional requirements with their test-coverage status, preserved or amended constraints, "change me" amendments, known-issue notes, and the refactor motivation}
>
> **Requirements:**
> - Every requirement is a single EARS statement (see the `ears` skill), written in user/domain language — a user- or consumer-facing outcome, not an implementation detail. Keep internal types, APIs, and data structures for the design.
> - Annotate each preserved requirement with its test backing:
>   - **Existing test:** where an existing test already covers the requirement's observable, append `Existing test: {path}`. The implementer reuses that test as the behavior gate.
>   - **Pin test required:** where no existing test covers it, append `Pin test required`. `breakdown` schedules the pin test as a task that lands before any refactor work begins; the implementer writes the test, runs it against the current code, and expects it to pass (the existing implementation already satisfies the requirement).
>
> **Constraints:** Capture the preserved or amended constraints from the extraction. Each states a measurable target (a specific threshold, not "fast" or "scalable").
>
> **Problem statement:** the refactor or migration motivation.
>
> Save it as the `specification` artifact on `{concept_name}` via `save_artifact_revision` when done (omit `base_revision_id` for the first revision).

### Step 6: Review

Use the `review` skill for a Specification Review of the `specification` artifact on the concept.

### Step 7: Fix issues (if any)

If the review finds P0 or P1 issues, spawn a subagent to fix them.

> Think hard.
>
> Fix the following issues in the `specification` artifact on the concept `{concept_name}` (read it via the `scs` MCP server; use the `artifacts` skill for read/write mechanics). Keep all other parts of the specification as they are.
>
> {paste review findings here}
>
> Save the updated specification as a new revision (pass the current revision's id as `base_revision_id`) when done.

Repeat until the review passes.

## Handing off

The spec produced by `extract-spec` is identical in shape to one produced by `spec`, with one addition on each preserved requirement: an existing-test path or a pin-test marker. `design` reads it the same way; the design has freedom to restructure as long as the spec's requirements are still satisfied after implementation.

`breakdown` schedules every pin test as a task before any refactor task that touches the same area. `implement` writes the pin test, runs it against the current code, and expects it to pass — the existing implementation already satisfies the requirement. Refactor tasks then run with the pin tests in place as the behavior gate.
