---
name: plan_v3
version: 0.2.5
description: Create and refine design documents for features using the SDD methodology. Use this skill when designing, creating designs, or refining designs. Produces structured design documents with architecture, components, test scenarios, and quality standards. Does NOT include task breakdown - use the tasks_v3 skill for that.
---

# Plan

**Plan** turns the specification into an architectural design expressed as a set of components. It runs after requirements and before tasks.

Both this orchestrator and every subagent it spawns follow the `language_v3` skill for tone and vocabulary in all output, including replies to the user.

All SDD artifacts live in the feature artifact directory. Standalone features use `.sdd/{feature}/`; roadmap deliverables use `.sdd/{initiative}/{deliverable-slug}/`. The `.sdd/index.md` is owned by `requirements` and `roadmap` — do not modify it from `plan`.

## Process

### Creating a Design

Copy `templates/design.template.md` to `{artifact_dir}/design.md` if it doesn't exist.

**Step 1: Write the design.** Spawn a subagent to write the design.

**Subagent prompt** (use a high-capability reasoning model):
> Think hard.
>
> Write the design document for {feature} at `{artifact_dir}/design.md`.
>
> Your job is to translate the specification's requirements into an architectural design, expressed as a **set of components**:
> - **Modified** — existing components whose behavior changes to serve a requirement.
> - **Added** — new components introduced because no existing component can serve the requirement.
> - **Used** — existing components leaned on unchanged.
>
> Every Modified or Added component traces back to at least one FR via its Rationale. Cut or fold-into-caller any component that doesn't earn its place. Every FR from the specification appears in at least one component's Rationale.
>
> **NFRs are met by architectural choices and observability** — not by adding components. App-instrumented NFRs add metrics, logs, or traces emitted by named components so the NFR is visible in production. Record architectural choices in the Architecture section; record observability in Instrumentation.
>
> Describe *what* and *why* at the component level. Keep *how* at the code level for the implementer. Another team member should be able to implement from the design without clarifying questions.
>
> **Read:**
> - Design template: `templates/design.template.md`
> - Specification: `{artifact_dir}/specification.md`
> - Research: `{artifact_dir}/research.md` (if it exists; absent for roadmap deliverables)
> - Project conventions: use the `handbook_v3` skill
> - EARS syntax reference: use the `ears_v3` skill (FRs are EARS sentences)
> - Interface design rules: use the `interface-design_v3` skill — components must be designed for testability (accept dependencies, return results, small surfaces)
> - Language standard: use the `language_v3` skill
> - Relevant domain skills (security, api-design, distributed-systems, data-engineering, devops-sre, infrastructure, low-level-systems) — load any that apply.
>
> **Component rules:**
> - Details: 5–10 lines of pseudo-code or type signatures. Keep implementation for the implementer.
> - Plumbing components (behaviour only meaningful via a caller) state in the Rationale which FR exercises them transitively.
> - App-instrumented NFRs drive entries in the Instrumentation section, naming the metric and the component that emits it. Platform-observed and architectural-only NFRs are recorded in Architecture or Risks alone — no component Rationale, no test.
>
> **API Design:** Describe operations and data shapes in prose (inputs, outputs, errors).
>
> **Feasibility Review:** List any design blocker. If the feature requires provisioning or imperative-script execution before an acceptance test can run, note which AT depends on which user-executed action.
>
> **Instrumentation:** Include when an NFR demands observability.
>
> Aim under 300 lines total. Use the template's sections as defined; omit any optional sections that don't apply.
>
> **Codebase exploration:** If research exists, trust its technical findings and cap exploration at 3 targeted file reads. If research is absent (roadmap deliverable), explore the codebase to discover existing patterns, integration points, and constraints; cap at 8 targeted file reads.
>
> **Escalation:** If the specification is too ambiguous to design fully, STOP and report what you completed and what needs clarification.

**Step 2: Review.** Use the `review_v3` skill to perform a **Design Review** of `{artifact_dir}/design.md`.

**Step 3: Fix issues (if any).** If the review finds P0 or P1 issues, spawn a subagent to fix them.

> Think hard.
>
> Fix the following issues in the design at `{artifact_dir}/design.md`, using `{artifact_dir}/specification.md` as reference. Research at `{artifact_dir}/research.md` may also exist; for roadmap deliverables it has been retired and should not be sought.
>
> Your job is to apply the review findings below. Keep all other parts of the design as they are.
>
> {paste review findings here}
>
> Save the document when done.

Repeat Steps 2–3 until the review passes.

### Refining a Design

Read the existing design and linked specification. Identify gaps, inconsistencies, or new requirements; explore the codebase for changed context. Update the design while maintaining the template structure. Verify every requirement still maps to a component.
