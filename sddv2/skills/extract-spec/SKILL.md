---
name: extract-spec
description: Make the behavior preservation contract for a refactor or migration explicit and testable. Reads existing code and tests, interviews the user to classify current behavior, and produces a specification.md in the same shape as `requirements` — with each preserved FR backed by an existing test or a pin test added before the refactor begins.
version: 0.1.0
---

# Extract Spec

**Extract Spec** makes the behavior preservation contract for a refactor or migration explicit and testable. It reads existing code and tests, then collaborates with the user to classify current behavior, and produces a specification.md in the same shape as `requirements`.

It runs before `plan`, as a sibling to `requirements`, when the work preserves existing behavior.

Both this orchestrator and every subagent it launches follow the `language` skill for tone and vocabulary in all output, including replies to the user.

All SDD artifacts live in `.sdd/{feature}/`.

## What this skill does and does not guarantee

This skill makes the team's intent about behavior preservation explicit. It does not guarantee preservation — emergent behavior, implicit contracts with downstream consumers, and edge cases the test suite does not exercise can drift through any refactor.

The mitigation built into this skill: every preserved FR is exercised by an existing test, or by a pin test added as a task before refactor work begins. The spec records which is which; `plan` and `tasks` sequence the pin tests first.

## When to use

- The work preserves existing behavior (refactor, migration, dependency upgrade).
- The codebase has no current spec for the area being touched, or the spec is stale.

Greenfield features use `requirements`. Feature work that includes a refactor uses `extract-spec` for the preserved part and adds new FRs in the same spec via the "change me" path in the interview below.

## Process

### Step 1: Identify the area

Ask the user which code area is being refactored or migrated. Get the paths or module names. Explore with them to identify the boundary when the area is unclear.

### Step 2: Read the area

Read the code and its existing tests. Build a working understanding of:

- The public interfaces the area exposes (HTTP endpoints, CLI commands, event topics, library functions, etc.).
- The inputs each interface accepts.
- The outputs and side effects each interface produces.
- The error conditions each interface handles.
- The NFRs the existing system delivers (observed latency, throughput, durability — measure where you can).

### Step 3: Candidate-and-confirm interview

For each candidate behavior, run a two-question check with the user.

**Question 1: Is this intentional?**

Present the candidate in EARS form (see the `ears` skill) with the supporting evidence — the test or code excerpt that demonstrates it. Ask: *intentional, accidental, or change me?*

- **Intentional** → preserved FR in the spec. Go to Question 2.
- **Accidental** → known-issue note in the spec, or dropped if minor.
- **Change me** → new intended behavior in the spec, replacing the current one (this is "feature with refactor inside"). Treat as a greenfield FR and go to Question 2.

**Question 2: Does an existing test cover this behavior through its public channel?**

Look for a test whose When clause matches the FR's observable and whose Then clause asserts the same behavior.

- **Yes** → record the test path against the FR. The refactor inherits its safety net from the existing test.
- **No** → mark the FR as needing a pin test. `plan` and `tasks` will schedule the pin test as a task before any refactor work begins.

**For each candidate NFR:** Present the measured or stated current value. Ask: *preserve, tighten, or relax?* Record the chosen Target and Verification mode the same way `requirements` does.

Iterate until every observable behavior in the area is classified.

### Step 4: Capture motivation

Ask the user why the refactor or migration is happening. The motivation goes in the spec's Problem Statement. It guides `plan` later — the design has freedom to restructure as long as the spec's FRs and NFRs are preserved.

### Step 5: Write the specification

**Subagent prompt** (Task tool, `model: opus`):
> Think hard.
>
> Write the specification for {feature} at `.sdd/{feature}/specification.md`.
>
> Your job is to define the preserved behavior of the area being refactored, as a behavioral specification — what the system does today for users, operators, auditors, and reviewers, expressed independently of the current implementation. The specification is agnostic of implementation details: internal types, APIs, data structures, libraries, frameworks, and architectural patterns belong to the design phase.
>
> **Read:**
> - Specification template: `templates/specification.template.md`
> - EARS syntax reference: use the `ears` skill
> - Language standard: use the `language` skill
>
> **Context from extraction:**
> {paste the classified candidates here — intentional FRs with their test-coverage status, intentional NFRs, "change me" amendments, known-issue notes, and the refactor motivation}
>
> **Functional Requirements:**
> - Every FR is a single EARS statement (see the `ears` skill). The EARS sentence itself is the acceptance criterion.
> - Write in user/domain language. Keep internal types, APIs, and data structures for the design.
>
> **Non-Functional Requirements:** Capture the preserved or amended NFRs from the extraction. Each NFR states a measurable Target and a Verification mode (`app-instrumented`, `platform-observed`, or `architectural-only`). App-instrumented NFRs additionally name the metric and where it's read.
>
> **Acceptance Tests:**
> - Every FR is covered by at least one AT-XX whose Given/When/Then exercises the observable described by the FR's EARS statement.
> - Each AT-XX is exactly a Given/When/Then triple.
> - The channel (endpoint, command, topic, dashboard, etc.) lives inside the When clause.
> - **Existing-test annotation.** Where an existing test already covers an FR's observable, append `Existing test: {path}` to the AT. The implementer reuses that test as the AT's gate.
> - **Pin annotation.** Where no existing test covers an FR's observable, append `Pin test required` to the AT. `plan` and `tasks` will schedule the pin test as a task that lands before any refactor work begins; the implementer writes the test, runs it against the current code, and expects it to pass (the existing implementation already satisfies the FR).
>
> Save the document when done.

### Step 6: Review

Use the `review` skill for a Specification Review of `.sdd/{feature}/specification.md`.

### Step 7: Fix issues (if any)

If the review finds P0 or P1 issues, launch a subagent (Task tool, `model: opus`):

> Think hard.
>
> Fix the following issues in the specification at `.sdd/{feature}/specification.md`. Keep all other parts of the specification as they are.
>
> {paste review findings here}
>
> Save the document when done.

Repeat until the review passes.

## Handing off

The spec produced by `extract-spec` is identical in shape to one produced by `requirements`, with two additions on each AT: an existing-test path or a pin-test marker. `plan` reads it the same way. The design has freedom to restructure as long as the spec's FRs and NFRs are still satisfied after implementation.

`tasks` schedules every pin test as a task before any refactor task that touches the same area. `implement` writes the pin test, runs it against the current code, and expects it to pass — the existing implementation already satisfies the FR. Refactor tasks then run with the pin tests in place as the behavior gate.
