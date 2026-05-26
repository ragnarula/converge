---
name: review
description: Review specifications, designs, and implementations for SDD features. Use this skill when reviewing specs, designs, or implementations. Produces structured review reports with severity-categorized findings.
version: 0.3.4
---

# Review

**Review** gates every SDD artifact — roadmap, specification, design, task breakdown, implementation — before it leaves its skill. It's invoked from each prior skill, not standalone.

Both this orchestrator and every reviewer subagent it launches follow the `language` skill for tone and vocabulary in all output, including review reports.

## Practical Guidelines

### Project Structure

All SDD artifacts live in `.sdd/{feature}/` where `{feature}` is the kebab-case feature name (e.g., `user-authentication`).

### Project Guidelines

Use the `handbook` skill to read and resolve project conventions before reviewing.

### Domain Skills

After exploring the codebase with the Explore tool and understanding the task, identify which domain skills apply:

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

**CRITICAL**: Use the Task tool to create a subagent for the review. The subagent reads files directly — do NOT paste full documents into the prompt.

### Severity Levels

All findings use: **P0** (explicit violation of stated requirement/guideline/contract), **P1** (spirit not met), **P2** (ambiguous/unclear), **P3** (gap worth noting). Report grouped by severity, P0 first. Recommend rejection if any P0.

---

### Roadmap Review

**Subagent prompt** (Task tool, `model: opus`):
> Ultrathink.
>
> Review the roadmap for {INITIATIVE}.
>
> Your job is to ensure the roadmap describes outcomes only and that every deliverable is independently shippable.
>
> **Read these files:**
> - Roadmap: `.sdd/{initiative}/roadmap.md`
> - Research: `.sdd/{initiative}/research.md` (if it still exists — retired after roadmap approval)
> - Project conventions: use the `handbook` skill
> - Language standard: use the `language` skill
>
> **Check for:**
> - **Outcomes only.** No architecture, components, libraries, code, file paths, or data shapes anywhere in the document. Flag any implementation detail. The roadmap's job is what users/operators can do after each step, not how it's built.
> - **Problem and Motivation** are present at the top. Problem in user/business terms (not technical). Motivation explains why now. Flag missing or generic versions.
> - Every deliverable is a vertical outcome — a user/operator can do something after that they couldn't before. Flag any deliverable that's a layer ("build the API"), a tech component ("set up the queue"), or a phase ("Phase 1").
> - Every deliverable's Outcome field names a specific user/operator/business actor and what they can do after. Flag generic outcomes ("improves the system", "enables future work").
> - In scope / Out of scope describe outcomes, not work items. Flag entries that name internal subsystems or services.
> - Every deliverable fits one spec (~1 day). Flag any that look multi-day or multi-subsystem — these need splitting along a scope axis.
> - Out of scope is present and load-bearing. Flag deliverables that don't state what's deliberately excluded.
> - Most deliverables are standalone. If every deliverable depends on D-01, D-01 is plumbing — flag for reshape.
> - Sequencing rationale explains *why* this order in outcome terms, not just lists the order. Flag missing or trivial rationale.
> - No leakage of research narrative (Observe/Orient/Diverge/Evaluate prose) — the roadmap is a forward plan, not a synthesis.
> - Roadmap is concise (under 200 lines). Flag if longer.
>
> **Severity:** P0=explicit violation, P1=implied discrepancy, P2=ambiguity, P3=consideration. Group by severity, P0 first. Reject if any P0.

---

### Specification Review

**Subagent prompt** (Task tool, `model: opus`):
> Ultrathink.
>
> Review the specification for {feature}.
>
> Your job is to ensure only buildable, verifiable, behavioral requirements reach the design phase. The spec is the solution definition, agnostic of implementation details.
>
> **Read these files:**
> - Specification: .sdd/{feature}/specification.md
> - Research: `.sdd/{feature}/research.md` (if it exists)
> - Project conventions: use the `handbook` skill
> - Language standard: use the `language` skill
> - EARS syntax reference: use the `ears` skill (needed for the FR check below)
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
> **Check for:**
> - Every requirement describes WHAT and WHY, never HOW
> - Requirements are behaviors, not solutions (no architecture, libraries, or patterns prescribed)
> - Requirements use user/domain language, not internal system terminology — "when content is uploaded to a store" not "when a KV write succeeds"; "the system records the change" not "a StorageEvent is emitted". If a requirement names internal types, APIs, or data structures, it belongs in the design, not the spec.
> - Scope fits a single iteration
> - No conflicts with existing functionality or project conventions
> - No vague/unmeasurable language ("fast", "secure", "user-friendly")
> - No technology choices or implementation assumptions embedded in requirements
> - If research exists, check that it was used correctly — requirements should reflect problem context and constraints from the research, not technical approaches or architecture that belong in the design
>
> **Functional Requirement checks (EARS):**
> - Every FR is a single EARS statement using one of the canonical patterns (see the `ears` skill). Flag any FR not in EARS shape.
> - Each FR has only a title and a `Statement:` field. Flag any FR carrying extra fields (e.g. `Verification:`, `Tested by:`) — those were removed.
> - Flag white-box FRs — statements whose verification requires reaching past the public interface into a third party or internal-only state. The FR must describe a user-experienceable outcome, or it isn't a requirement.
> - Flag any FR where no role (user, operator, auditor, reviewer) could tell whether it holds. Experienceability gate.
>
> **NFR checks:**
> - Each NFR has a measurable Target (specific threshold, not "fast" or "scalable") and a Verification mode (`app-instrumented`, `platform-observed`, or `architectural-only`). Flag any missing either.
> - App-instrumented NFRs additionally name the Observable (metric and where it's read). Flag any that don't.
> - NFRs do not appear in the Acceptance Tests section. Flag any that do.
> - Flag NFRs that prescribe implementation ("must use Redis"), invent Verification modes, or claim app-instrumented Verification for something inherently platform-observable.
>
> **Acceptance Test checks:**
> - Each AT is a Given/When/Then triple. No other fields. Flag any AT with extras (Verifies, Channel, Method, Test name) — they were removed.
> - Each AT's When clause names a concrete channel (endpoint, command, topic, dashboard, code-review surface). Flag When clauses that describe internal mechanics or abstract intent.
> - Each AT's Then is a concrete observable visible through that channel. Flag Then clauses that paraphrase the When ("when X is requested, then X was requested") or assert past the channel into internal state.
>
> **Coverage:**
> - Every FR's observable should appear in at least one AT's Then. List any FR whose observable is not covered by any AT.
> - If multiple FRs share the same observable outcome, expect them to share one AT — flag for consolidation if each has its own near-duplicate AT.
>
> **Domain feasibility:** The spec is problem-space — what the user needs, not how the system satisfies it. Missing solution-space concerns (auth mechanisms, retry policies, schema-evolution strategies, observability tooling, etc.) belong to the design and stay out of this review. Use each loaded domain skill to flag requirements the domain recognises as infeasible — physically impossible, internally contradictory, or in conflict with hard domain constraints. Name the conflict and the constraint it breaks.
>
> **Severity:** P0=explicit violation, P1=implied discrepancy, P2=ambiguity, P3=consideration. Group by severity, P0 first. Reject if any P0.

---

### Design Review

**Subagent prompt** (Task tool, `model: opus`):
> Ultrathink.
>
> Review the design for {feature}.
>
> Your job is to ensure the design is architecturally unambiguous, sound, and feasible — so an implementer can build from it without clarifying questions.
>
> **Read these files:**
> - Specification: .sdd/{feature}/specification.md
> - Design: .sdd/{feature}/design.md
> - Research: `.sdd/{feature}/research.md` (if it exists)
> - Project conventions: use the `handbook` skill
> - Language standard: use the `language` skill
> - EARS syntax reference: use the `ears` skill (FRs in the spec are EARS sentences)
>
> **Load relevant domain skills.** Scan the spec and design and load any that apply. Use them to check the design's **architectural soundness** and **feasibility** — see the Domain soundness check below.
> - **distributed-systems**: multiple services, network coordination, eventual consistency
> - **low-level-systems**: memory management, performance-critical paths, OS interfaces
> - **security**: auth, untrusted input, sensitive data, compliance
> - **infrastructure**: cloud resources, IaC, networking, disaster recovery
> - **devops-sre**: CI/CD, deployment, observability, SLOs
> - **data-engineering**: pipelines, ETL, schema evolution, data quality
> - **api-design**: public/internal APIs, versioning, contracts
>
> **Primary check — architectural unambiguity.** This is the most important thing the review enforces. An implementer reading the design should know exactly what components to build, modify, or use; what each component is responsible for; how the components fit together; and what each component's interface promises. Implementation details — *how* the inside of a component achieves its responsibility — are explicitly NOT the design's job; the design must leave them to the implementer. Flag both directions:
> - **Architectural ambiguity (P0):** component responsibilities that admit more than one plausible design; missing or vague interfaces; unclear data flow between components; unresolved choices the design must commit to; dependencies between components that are implied rather than stated.
> - **Over-specification (P1):** pseudo-code or type signatures that prescribe the *inside* of a component rather than its contract; design content an implementer should be free to choose at code-time. The 5–10 line Details cap exists to prevent this — flag any Details block that exceeds it or that reads like a function body rather than a contract.
>
> **Other checks:**
> - Every requirement addressed by design — no orphan requirements. Unchanged behavior covered by existing tests is not an orphan. Removed functionality needs its tests removed, not new tests added.
> - No gold-plating beyond requirements
> - Follows project conventions (error handling, logging, naming, architecture)
> - Failure cases handled at the architectural level (dependencies unavailable, invalid inputs, partial completion) — *what* the response is, not *how* it's implemented.
> - Risk assessment present with mitigations
> - Design is concise (under 300 lines)
> - If research exists, design should be grounded in its technical findings — existing patterns, integration points, and constraints identified in research should be reflected in the design. Flag designs that contradict or ignore research findings without justification.
>
> **Component Rationale checks:**
> - Every Modified and Added component has a Rationale that names a specific FR. Flag components whose Rationale is generic ("supports the feature") or absent.
> - Every FR from the specification is named in at least one component's Rationale. List any uncovered FR.
> - Plumbing components state their transitive coverage explicitly — they name the FR that exercises them through a caller. Flag components whose Rationale tries to assert standalone behaviour that just restates their implementation.
> - No TS-XX / ITS-XX / E2E-XX scenario blocks in the design. If present, the design is on the old template — flag for migration.
>
> **NFR coverage checks:**
> - Each app-instrumented NFR from the spec has a corresponding Instrumentation entry naming the metric and the component that emits it. Flag any uncovered.
> - Platform-observed NFRs are noted in Architecture or Risks, not assigned to a component, and not in Instrumentation. Flag misplacement.
> - Architectural-only NFRs are explained in Architecture (which choice satisfies them) without code or instrumentation. Flag if the design quietly invents tests or metrics for them.
>
> **Domain soundness and feasibility:** Apply each loaded domain skill's lens to the design.
> - **Architectural soundness:** Flag design choices the domain says are wrong or weak. Name the rule the design breaks.
> - **Feasibility:** Flag design choices the domain recognises as infeasible — physically impossible, in conflict with platform or library capabilities, or violating hard domain constraints. Name the constraint it breaks.
>
> **Severity:** P0=explicit violation, P1=implied discrepancy, P2=ambiguity, P3=consideration. Group by severity, P0 first. Reject if any P0.

---

### Task Breakdown Review

**Subagent prompt** (Task tool, `model: opus`):
> Ultrathink.
>
> Review the task breakdown for {feature}.
>
> Your job is to ensure every task is demoable on its own, every requirement is covered, and the ordering lets each task proceed once its blockers are done.
>
> **Read these files:**
> - Specification: `.sdd/{feature}/specification.md`
> - Design: `.sdd/{feature}/design.md`
> - Tasks: `.sdd/{feature}/tasks.md`
> - Project conventions: use the `handbook` skill
> - Language standard: use the `language` skill
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
> **Aggregate coverage:**
> - **Requirement coverage.** Every FR in the specification must be addressed by at least one task — the task's `What to build` prose and ACs together must exercise the FR's observable. Read all tasks together. List any uncovered FR as P0.
> - **Acceptance test coverage.** Every AT-XX in the spec's Acceptance Tests section must be exercised in aggregate by the task ACs. A single spec AT is typically split across multiple task ACs (each task is a slice). Flag any spec AT whose union of covering task ACs is incomplete.
> - Design-component coverage is NOT checked here — tasks don't enumerate components. It is verified at implementation review against the diff.
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

**Subagent prompt** (Task tool, `model: opus`):
> Ultrathink.
>
> Review the implementation of {feature}.
>
> Your job is to ensure the implementation matches the design, satisfies every spec AT, and contains no unfinished work.
>
> **Read these files:**
> - Tasks: `.sdd/{feature}/tasks.md`
> - Design: `.sdd/{feature}/design.md`
> - Specification: `.sdd/{feature}/specification.md` (needed for FR coverage check below)
> - Language standard: use the `language` skill
>
> **Steps:**
> 1. Read `.sdd/{feature}/specification.md` and list every FR and AT-XX. For each FR, confirm at least one task's `What to build` and ACs address it. Any FR not delivered is P0.
> 2. Run `git diff main...HEAD` to scope the change.
> 3. Verify each Modified or Added component from the design appears in the diff.
> 4. Check the final state matches the design's component contracts and interfaces. Mid-stream tasks may implement only the slice their ACs need; partial implementation in earlier tasks is fine if a later task completes it.
> 5. For each AT-XX in the spec's Acceptance Tests, find a concrete test in the diff that exercises the AT's When and asserts its Then. A test that would still pass if the satisfying implementation were deleted is P0. Spot-check 2–3 ATs by mentally deleting the implementation.
> 6. NFRs: for each app-instrumented NFR, confirm the named metric or log is in the diff. Platform-observed and architectural-only NFRs need no diff evidence. Don't expect tests for NFRs.
> 7. Stubs and dead code: search for `skip`, `todo`, `pending`, `pass` in test functions, placeholder assertions, unused imports, unused functions, commented-out code. Flag each unless the task's Notes section records it as a known external blocker.
> 8. Test code quality: tests follow the same engineering standards as production code. Flag duplicated arrange blocks, copy-pasted assertions that differ only in inputs, inline fixtures that should be shared, test names that don't state the behaviour, and ad-hoc mocks where a project fixture exists.
> 9. IaC and live-state: provisioning configs and imperative scripts (root IaC modules, env stacks, migrations, runbooks) should be linted in the diff, not executed by the implement subagent. If an AT passed against an applied environment, confirm via tasks.md or commit messages that the user applied it. Reusable IaC modules carry acceptance tests via plan-time or policy assertions.
> 10. SDD leakage: search for `FR-`, `NFR-`, `AT-`, `REQ-` in code, comments, docstrings, or test names.
> 11. Verify project conventions via the `handbook` skill: error handling, logging, naming, test structure, commit format.
> 12. Run tests, linters, and build.
>
> **Severity:** P0=explicit violation, P1=implied discrepancy, P2=ambiguity, P3=consideration. Group by severity, P0 first. Reject if any P0.
>
> **Escalation:** If the diff is too large to review in one pass, review the most critical files first and report what you covered vs. what remains.
