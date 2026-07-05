---
name: review
description: Review specifications, designs, task breakdowns, and implementations for SDD features. Use this skill when reviewing specs, designs, task breakdowns, or implementations. Produces structured review reports with severity-categorized findings.
version: 0.3.4
---

# Review

**Review** gates every SDD artifact — specification, design, task breakdown, implementation — before it leaves its skill. It's invoked from each prior skill, not standalone.

Both this orchestrator and every reviewer subagent it spawns follow the `language` skill for tone and vocabulary in all output, including review reports.

## Practical Guidelines

### Artifact Storage

SDD artifacts are stored in SCS via the `scs` MCP server, not local files. Use the `artifacts` skill for the storage contract. Review reads whatever artifacts exist (problem, research, specification, design, tasks) on the concept named `{feature}`. Review reads only — it does not save artifacts; findings are returned to the calling skill.

### Project Guidelines

Use the `conventions` skill to discover the repository's guidelines, and check the artifact under review adheres to them.

### Domain Skills

After codebase exploration and understanding the task, identify which domain skills apply:

- **distributed-systems**: Multiple services, network coordination, eventual consistency
- **low-level-systems**: Memory management, performance-critical, OS interfaces
- **security**: Auth, untrusted input, sensitive data, compliance
- **infrastructure**: Cloud resources, IaC, networking, disaster recovery
- **devops-sre**: CI/CD, deployment, observability, SLOs
- **data-engineering**: Pipelines, ETL, schema evolution, data quality
- **api-design**: Public/internal APIs, versioning, contracts

Load relevant skills and apply their mindset and practices throughout review.

## Process

Identify the review type requested, then follow ONLY that type's section below. Each section has preparation steps and a subagent prompt template.

**CRITICAL**: Spawn a subagent for each review. Invoking this skill authorizes that subagent. The reviewer reads files directly — do NOT paste full documents into the prompt.

### Severity Levels

All findings use: **P0** (explicit violation of stated requirement/guideline/contract), **P1** (spirit not met), **P2** (ambiguous/unclear), **P3** (gap worth noting). Report grouped by severity, P0 first. Recommend rejection if any P0.

---

### Specification Review

**Subagent prompt** (use a high-capability reasoning model):
> Think hard.
>
> Review the specification for {feature}.
>
> Your job is to ensure the spec states an unambiguous, verifiable, solution-independent *what* — so design and breakdown build on a clear, agreed target. The spec defines outcomes, not implementation.
>
> **Read whatever exists** for the concept (via the `scs` MCP server; use the `artifacts` skill for the read mechanics). The spec is required; the rest are optional, and a missing one returns `artifact_has_no_saved_revision` — treat it as absent and skip the checks that depend on it:
> - Specification: the `specification` artifact on `{concept_name}` — the artifact under review.
> - Problem: the `problem` artifact on `{concept_name}`.
> - Research: the `research` artifact on `{concept_name}` (if it exists).
> - Project conventions: use the `conventions` skill to discover the repository's guidelines.
> - Language standard: use the `language` skill. EARS reference: use the `ears` skill.
>
> **Load relevant domain skills.** Scan the spec and load any that apply. Use them solely for the Domain feasibility check below — flagging infeasible or contradictory requirements, NOT missing solution-space concerns.
> - **distributed-systems**: multiple services, network coordination, eventual consistency
> - **low-level-systems**: memory management, performance-critical paths, OS interfaces
> - **security**: auth, untrusted input, sensitive data, compliance
> - **infrastructure**: cloud resources, IaC, networking, disaster recovery
> - **devops-sre**: CI/CD, deployment, observability, SLOs
> - **data-engineering**: pipelines, ETL, schema evolution, data quality
> - **api-design**: public/internal APIs, versioning, contracts
>
> **Check 1 — Scope unambiguity (primary).** The spec should have been grilled until its scope is clear. Read every requirement and the scope boundaries as an adversary hunting for a second reading. Flag:
> - Any requirement that could be read more than one way (P0 if it would send design or implementation in the wrong direction, else P2).
> - Any boundary left vague — where the work starts and stops, what is in or out of scope — that a designer or implementer would have to guess at (P0).
> - Edge cases the requirements leave unresolved where the right outcome is not obvious (P1/P2).
>
> **Check 2 — EARS well-formedness.** Every requirement is a single EARS statement using one of the canonical patterns (see the `ears` skill). Flag any requirement not in EARS shape.
>
> **Check 3 — WHAT, not HOW.** Requirements describe user- or consumer-facing outcomes in user/domain language — never architecture, libraries, patterns, internal types, or data structures. "When content is uploaded to a store" not "when a KV write succeeds"; "the system records the change" not "a StorageEvent is emitted". Flag any requirement that prescribes or assumes a solution — it belongs in the design.
>
> **Check 4 — Verifiable and experienceable.** Every requirement is something a role — user, operator, auditor, reviewer — could tell holds from outside the system. Flag white-box requirements whose truth can only be checked by reaching into internal state. Flag vague or unmeasurable language ("fast", "secure", "user-friendly") with no measure that would decide whether it holds.
>
> **Check 5 — Alignment with the available context.** Adapt to what exists:
> - **Against the problem:** the requirements together deliver the framed problem's definition of success for the affected users. Flag requirements that drift from the problem, and any success criterion in the problem that no requirement covers.
> - **If research exists:** requirements reflect the problem context and constraints the research found — not the technical approaches or architecture it surfaced, which belong to the design. Flag research constraints ignored, and solution-space detail pulled in from research.
> - No conflicts with existing functionality or project conventions.
>
> **Check 6 — Domain feasibility.** The spec is problem-space — what the user needs, not how the system satisfies it. Missing solution-space concerns (auth mechanisms, retry policies, schema-evolution strategies, observability tooling, etc.) belong to the design and stay out of this review. Use each loaded domain skill to flag requirements the domain recognises as infeasible — physically impossible, internally contradictory, or in conflict with a hard domain constraint. Name the conflict and the constraint it breaks.
>
> **Also:** the spec is concise (under 400 lines) and states only the *what* — flag any design or implementation content that has leaked in.
>
> **Severity:** P0=explicit violation, P1=implied discrepancy, P2=ambiguity, P3=consideration. Group by severity, P0 first. Reject if any P0.

---

### Design Review

**Subagent prompt** (use a high-capability reasoning model):
> Think hard.
>
> Review the design for {feature}.
>
> Your job is to ensure the design is unambiguous, matched in depth to its risk, holds the properties of good design, and aligns with everything the team has decided so far — so an implementer can build from it with confidence.
>
> **Read whatever exists** for the concept (via the `scs` MCP server; use the `artifacts` skill for the read mechanics). The design is required; the rest are optional, and a missing one returns `artifact_has_no_saved_revision` — treat it as absent and skip the checks that depend on it:
> - Design: the `design` artifact on `{concept_name}` — the artifact under review.
> - Specification: the `specification` artifact on `{concept_name}` (if it exists).
> - Research: the `research` artifact on `{concept_name}` (if it exists).
> - Problem: the `problem` artifact on `{concept_name}`.
> - Project conventions: use the `conventions` skill to discover the repository's guidelines.
> - Language standard: use the `language` skill. EARS reference: use the `ears` skill (spec requirements are EARS sentences).
>
> **Load relevant domain skills.** Scan the design and any spec and load any that apply. Use them for the Domain soundness check below.
> - **distributed-systems**: multiple services, network coordination, eventual consistency
> - **low-level-systems**: memory management, performance-critical paths, OS interfaces
> - **security**: auth, untrusted input, sensitive data, compliance
> - **infrastructure**: cloud resources, IaC, networking, disaster recovery
> - **devops-sre**: CI/CD, deployment, observability, SLOs
> - **data-engineering**: pipelines, ETL, schema evolution, data quality
> - **api-design**: public/internal APIs, versioning, contracts
>
> **Check 1 — Unambiguity (primary).** An implementer reading the design should know exactly what components to build, modify, or use; what each is responsible for; how they fit together; and what each interface promises — without needing to ask a clarifying question. Flag both directions:
> - **Ambiguity (P0):** responsibilities that admit more than one plausible design; missing or vague interfaces; unclear data flow between components; unresolved choices the design should have committed to; dependencies implied rather than stated.
> - **Over-specification (P1):** content that dictates the *inside* of a component — pseudo-code, function bodies, field-by-field layouts — rather than its contract. That is the implementer's to own.
>
> **Check 2 — Depth aligned to risk.** Depth is earned by risk, not spread evenly. Read the risks the design records (and any the spec or research imply):
> - Flag any risky, uncertain, or likely-to-change area left shallow — boundaries, data flow, interfaces, or failure modes not worked out where the risk called for it (P0 if it blocks the implementer, else P1).
> - Flag over-investment where risk was low — speculative flexibility, extension points, or abstraction layers the recorded risks do not justify (P1).
> - Flag a risk the design records but never addresses in the design itself (P1).
>
> **Check 3 — Properties of good design.**
> - **Modularity:** components have high cohesion and low coupling. Flag a component doing several unrelated things, or components so entangled they cannot change independently.
> - **Boundaries on change lines:** each boundary states what change it keeps local, and boundaries fall where the requirements are most likely to change. Flag boundaries drawn by layer or by processing step instead of by likely change.
> - **Small, clear interfaces:** each component's contract is narrow and explicit. Flag wide, leaky, or under-described interfaces.
> - **Decisions at the right altitude:** key decisions are pinned with a reason; nothing the implementer should own is over-constrained. Flag both an unmade key decision and an over-constrained detail.
> - **No gold-plating:** nothing designed beyond what the context calls for.
>
> **Check 4 — Alignment with the available context.** The design must serve what the team has already decided. Adapt to what exists:
> - **If a spec exists:** every requirement is addressed by the design — no orphan requirements — and nothing in the design contradicts a requirement, constraint, or stated scope. List any requirement no component serves (P0) and any design choice that breaks a requirement (P0).
> - **If research exists:** the design reflects its findings — existing patterns, integration points, and constraints. Flag designs that contradict or ignore research findings without justification (P1).
> - **Against the problem:** the design actually solves the framed problem for the affected users. Flag drift from the problem (P1).
> - Follows project conventions (error handling, logging, naming, architecture). Handles failure cases at the architectural level (dependencies unavailable, invalid inputs, partial completion) — *what* the response is, not *how* it is coded.
>
> **Check 5 — Domain soundness and feasibility.** Apply each loaded domain skill's lens to the design.
> - **Soundness:** flag design choices the domain says are wrong or weak. Name the rule the design breaks.
> - **Feasibility:** flag choices the domain recognises as infeasible — physically impossible, in conflict with platform or library capabilities, or violating a hard domain constraint. Name the constraint it breaks.
>
> **Also:** the design is concise (under 400 lines). Flag if longer.
>
> **Severity:** P0=explicit violation, P1=implied discrepancy, P2=ambiguity, P3=consideration. Group by severity, P0 first. Reject if any P0.

---

### Task Breakdown Review

**Subagent prompt** (use a high-capability reasoning model):
> Think hard.
>
> Review the task breakdown for {feature}.
>
> Your job is to ensure every task is demoable on its own, every requirement and design component is covered, and the ordering lets each task proceed once its blockers are done.
>
> **Read whatever exists** for the concept (via the `scs` MCP server; use the `artifacts` skill). The tasks are required; the rest are optional, and a missing one returns `artifact_has_no_saved_revision` — treat it as absent and skip the checks that depend on it:
> - Tasks: the `tasks` artifact on `{concept_name}` — the breakdown under review.
> - Specification: the `specification` artifact on `{concept_name}` (if it exists).
> - Design: the `design` artifact on `{concept_name}` (if it exists).
> - Problem: the `problem` artifact on `{concept_name}`.
> - Project conventions: use the `conventions` skill to discover the repository's guidelines.
> - Language standard: use the `language` skill.
>
> **Load relevant domain skills.** Scan the spec, design, and task breakdown and load any that apply. Use them for the Slice soundness check below.
> - **distributed-systems**: multiple services, network coordination, eventual consistency
> - **low-level-systems**: memory management, performance-critical paths, OS interfaces
> - **security**: auth, untrusted input, sensitive data, compliance
> - **infrastructure**: cloud resources, IaC, networking, disaster recovery
> - **devops-sre**: CI/CD, deployment, observability, SLOs
> - **data-engineering**: pipelines, ETL, schema evolution, data quality
> - **api-design**: public/internal APIs, versioning, contracts
>
> **Per-task gate — demoable.** Every task must deliver a slice that is demoable or independently verifiable on completion. After the task lands, an outside observer should be able to point at something they see differently. Flag any task that fails this test — it's a horizontal slice (one layer only, no observable behaviour) and needs reshaping. This is the tracer-bullet test; the breakdown is only valid if every task passes it.
>
> **Aggregate coverage against the available context:**
> - **Requirement coverage.** Every requirement in the available context — the spec's requirements, or the problem's success criteria when there is no spec — must be addressed by at least one task. The task's `What to build` prose and ACs together must exercise it. Read all tasks together. List any uncovered requirement as P0.
> - **Design-component coverage.** Where a design exists, every component it defines must be delivered by some task. List any uncovered component as P0.
> - The task ACs are the acceptance tests; there is no separate acceptance-test section to cross-check.
>
> **Ordering:**
> - Each task's `Blocked by` must reference only earlier-numbered tasks, or "None". Flag forward references.
> - The dependency graph formed by `Blocked by` must be a DAG. Flag any cycle.
> - Each task must produce code that compiles and passes tests independently — i.e. a task can run when its blockers are done. Flag tasks that implicitly depend on a later task's work.
>
> **Per-task checks:**
> - **What to build is prose.** No file paths and no code snippets inside the paragraph (decision-encoding artefacts from a prototype excepted). Flag bullet lists, checkbox lists, or pseudo-code in this section.
> - **Acceptance criteria are Given/When/Then.** Each AC names preconditions, an action, and an observable result. Flag ACs missing a clause or written in prose.
> - **Notes are 3–5 sentences of prose** — no code blocks. Flag overruns.
> - **No legacy IDs or fields.** Flag any `TS-XX`, `ITS-XX`, `E2E-XX`, `Satisfies: FR-XX`, `Acceptance gate: AT-XX`, `Files to read/modify`, or explicit `Tests:` section — all removed.
> - **No phases or grouping.** Tasks are a flat ordered list. Flag any phase headings.
> - **No "add tests" tasks.** Testing happens inside each task, not afterwards.
> - **No overlapping tasks** modifying the same component without explicit dependency.
>
> **Slice soundness:** Apply each loaded domain skill's lens to the task breakdown. Flag slice shapes or orderings that introduce real domain problems. Name the slice and the domain rule it breaks.
>
> **Severity:** P0=explicit violation, P1=implied discrepancy, P2=ambiguity, P3=consideration. Group by severity, P0 first. Reject if any P0.

---

### Implementation Review

**Subagent prompt** (use a high-capability reasoning model):
> Think hard.
>
> Review the implementation of {feature}.
>
> Your job is to ensure the implementation matches the design, satisfies every requirement and every task's acceptance criteria, and contains no unfinished work.
>
> **Read whatever exists** for the concept (via the `scs` MCP server; use the `artifacts` skill). A missing optional artifact returns `artifact_has_no_saved_revision` — treat it as absent and skip the checks that depend on it:
> - Tasks: the `tasks` artifact on `{concept_name}`.
> - Design: the `design` artifact on `{concept_name}` (if it exists).
> - Specification: the `specification` artifact on `{concept_name}` (if it exists — for the requirement coverage check).
> - Problem: the `problem` artifact on `{concept_name}`.
> - Language standard: use the `language` skill.
>
> **Steps:**
> 1. Set the coverage target: every requirement in the spec if it exists, otherwise the problem's success criteria; plus every task's acceptance criteria (Given/When/Then) from the `tasks` artifact. For each requirement, confirm at least one task's `What to build` and ACs deliver it. Any requirement not delivered is P0.
> 2. Determine the implementation diff base from the current branch's upstream, merge base, or the branch recorded when the feature was created. If no base is clear, ask the user. Run the equivalent of `git diff {base}...HEAD` to scope the change.
> 3. If a design exists, verify each Modified or Added component it defines appears in the diff.
> 4. If a design exists, check the final state matches its component contracts and interfaces. Mid-stream tasks may implement only the slice their ACs need; partial implementation in earlier tasks is fine if a later task completes it.
> 5. For each task's acceptance criteria, find a concrete test in the diff that exercises the Given/When and asserts the Then. A test that would still pass if the satisfying implementation were deleted is P0. Spot-check 2–3 ACs by mentally deleting the implementation.
> 6. Measurable constraints: if the spec or design states a measurable constraint (performance, security threshold, and the like), confirm the diff carries the evidence it calls for where the system can produce it — a metric, log, or check. A constraint satisfied by an architectural choice needs no code. Don't expect unit tests for these.
> 7. Stubs and dead code: search for `skip`, `todo`, `pending`, `pass` in test functions, placeholder assertions, unused imports, unused functions, commented-out code. Flag each unless the task's Notes section records it as a known external blocker.
> 8. Test code quality: tests follow the same engineering standards as production code. Flag duplicated arrange blocks, copy-pasted assertions that differ only in inputs, inline fixtures that should be shared, test names that don't state the behaviour, and ad-hoc mocks where a project fixture exists.
> 9. IaC and live-state: provisioning configs and imperative scripts (root IaC modules, env stacks, migrations, runbooks) should be linted in the diff, not executed by the implement worker. If a task's acceptance test passed against an applied environment, confirm via the `tasks` artifact or commit messages that the user applied it. Reusable IaC modules carry acceptance tests via plan-time or policy assertions.
> 10. SDD leakage: search for `FR-`, `NFR-`, `AT-`, `REQ-` in code, comments, docstrings, or test names.
> 11. Verify adherence to the repository's guidelines (use the `conventions` skill): error handling, logging, naming, test structure, commit format.
> 12. Run tests, linters, and build.
>
> **Severity:** P0=explicit violation, P1=implied discrepancy, P2=ambiguity, P3=consideration. Group by severity, P0 first. Reject if any P0.
>
> **Escalation:** If the diff is too large to review in one pass, review the most critical files first and report what you covered vs. what remains.
