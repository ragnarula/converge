---
name: requirements
description: Create and refine specifications for features using the SDD methodology. Use this skill when writing, creating, or refining specs and specifications. Conducts discovery interviews and produces structured specification documents.
version: 0.3.3
---

# Requirements

**Requirements** turns an idea into a behavioral specification. It runs after research (if any) and before plan.

## Practical Guidelines

### Project Structure

All SDD artifacts live in `.sdd/{feature}/` where `{feature}` is the kebab-case feature name (e.g., `user-authentication`).

### Templates

- Specification template: `templates/specification.template.md`
- Feature index template: `templates/index.template.md`

### Project Guidelines

Use the `handbook` skill to read and resolve project conventions. These conventions affect how specifications are structured.

### Domain Skills

After exploring the codebase with the Explore tool and understanding the task, identify which domain skills apply:

- **distributed-systems**: Multiple services, network coordination, eventual consistency
- **low-level-systems**: Memory management, performance-critical, OS interfaces
- **security**: Auth, untrusted input, sensitive data, compliance
- **infrastructure**: Cloud resources, IaC, networking, disaster recovery
- **devops-sre**: CI/CD, deployment, observability, SLOs
- **data-engineering**: Pipelines, ETL, schema evolution, data quality
- **api-design**: Public/internal APIs, versioning, contracts

Load relevant skills and apply their mindset and practices throughout specification, design, and review phases.

## Process

You **MUST** explore the codebase using the Explore tool before doing **ANY** of the below.
You **MUST** understand project guidelines by using the `handbook` skill

### Creating a Specification

Do this when a user asks to create a specification.

**Roadmap check:** If the parent directory contains a `roadmap.md`, the spec is a roadmap deliverable. Confirm which D-XX with the user, read its Outcome and In/Out scope from the roadmap, and add a `**Linked Roadmap:** .sdd/{initiative}/roadmap.md (D-XX)` field to the spec. Update the deliverable's Spec status as it moves Drafting → Approved → Implemented.

1. **Create a feature branch** from main named `feature/<feature-name>` (e.g., `feature/user-authentication`). For roadmap deliverables, name the branch `feature/<initiative>-<deliverable-slug>`.
2. **Create the spec folder.** Standalone: `.sdd/{feature}/`. Roadmap deliverable: `.sdd/{initiative}/{deliverable-slug}/`.
3. **Copy** `templates/specification.template.md` to the new folder's `specification.md` if it doesn't already exist.
4. **Maintain the index:**
   - If `.sdd/index.md` doesn't exist, create it from `templates/index.template.md`
   - Add a row for the new feature (newest entries at top, ordered by date). Roadmap deliverables nest under their initiative row.
   - Update the status as the feature progresses through Draft → Approved → Implemented

### Specifying

Your **GOAL** is to complete all parts of the specification template for the feature.

**Scope:** Each spec is ~1 day of implementation. Suggest splitting a larger feature into multiple specs during the interview.

**Template guidance:**
- Follow the template structure as defined in templates/specification.template.md
- Sections marked "optional" or "if needed" can be omitted entirely if not applicable
- Do NOT add new sections that aren't in the template

#### Process

**Step 1: Read existing context**

For a **standalone feature**, check if `.sdd/{feature}/research.md` exists. If it does, extract what informs behavioral requirements: user and operator needs, what the system does and why, scope-shaping constraints, feasibility evidence. Technical approaches, architecture, data models, and code patterns belong to the design.

For a **roadmap deliverable**, research has been retired by the roadmap step. Read the deliverable's entry in `.sdd/{initiative}/roadmap.md` — Outcome, In/Out scope, and Depends-on define the bounds; Problem and Motivation give context. No research file exists.

**Step 2: Discovery Interview**

Interview the user until you can fill every template section unambiguously. If research exists, focus on gaps it didn't cover. Don't ask about template sections directly — ask about the problem, users, and goals.

Cover:
- Problem and motivation; who feels it and how they cope today.
- Success criteria — what observable change tells them it's working.
- Boundaries — what's explicitly out of scope.
- Edge cases — what could go wrong.
- For each behavior: the channel through which someone experiences the outcome (API, CLI, UI, event, dashboard, code review surface) and the role of that someone (end user, operator, auditor, reviewer).

**Probe vague answers** — reject "fast", "secure", "user-friendly" without measurable criteria.

**Experienceability gate** — if no role can tell whether a candidate requirement is met, it isn't a requirement. Reshape or drop it during the interview.

NFRs are optional — only include them when there are genuine, measurable quality constraints.

**Step 3: Write the Specification**

Once you have enough information to fill out every section unambiguously, use the Task tool to launch a subagent that writes the specification. Do NOT write it yourself.

**Subagent prompt:**
> Write the specification for {feature} at .sdd/{feature}/specification.md.
>
> Your job is to define the solution as a behavioral specification — what the system does for users, operators, auditors, and reviewers. The specification is agnostic of implementation details: internal types, APIs, data structures, libraries, frameworks, and architectural patterns belong to the design phase.
>
> **Read:**
> - Specification template: templates/specification.template.md
> - Research findings: `.sdd/{feature}/research.md` (if it exists)
> - EARS syntax reference: use the `ears` skill
>
> **Context from discovery interview:**
> {paste the interview findings here}
>
> **Template guidance:**
> - Use the template's sections exactly as defined; omit any that are marked optional and don't apply.
>
> **Functional Requirements:**
> - Every FR is a single EARS statement (see the `ears` skill for patterns and combinations). The EARS sentence itself is the acceptance criterion.
> - Write in user/domain language. Keep internal types, APIs, and data structures for the design.
> - Each FR describes a user-experienceable outcome verifiable through a public interface. Reshape or drop candidates that can only be verified by reaching into internal state or a third party.
> - Each FR has a named role (user, operator, auditor, reviewer) who can tell whether it holds. Reshape or drop candidates that fail this experienceability gate.
>
> **Non-Functional Requirements:** Include NFRs only when there are genuine, measurable quality constraints. Each NFR states a measurable Target and a Verification mode (`app-instrumented`, `platform-observed`, or `architectural-only`). App-instrumented NFRs additionally name the metric and where it's read. NFRs feed the design's Architecture and Instrumentation sections exclusively.
>
> **Acceptance Tests:**
> - Every FR is covered by at least one AT-XX whose Given/When/Then exercises the observable described by the FR's EARS statement.
> - Each AT-XX is exactly a Given/When/Then triple.
> - The channel (endpoint, command, topic, dashboard, etc.) lives inside the When clause.
> - Collapse multiple FRs into one AT when they share an observable outcome.
>
> Write the complete specification in one pass. Fill in every section fully. Save the document when done.

**Step 4: Review the Specification**

Use the `review` skill to perform a **Specification Review** of the specification at .sdd/{feature}/specification.md.

**Step 5: Fix issues (if any)**

If the review finds P0 or P1 issues, use the Task tool to launch a subagent to fix them. Do NOT fix them yourself.

**Subagent prompt:**
> Fix the following issues in the specification at .sdd/{feature}/specification.md, using the template at templates/specification.template.md as reference.
>
> Your job is to apply the review findings below. Keep all other parts of the specification as they are.
>
> {paste review findings here}
>
> Save the document when done.

After the fix subagent completes, re-run Step 4 (review). Repeat Steps 4-5 until the review passes.

### Refining a Specification

When asked to refine a specification:
1. Read existing specification thoroughly
2. Identify gaps, inconsistencies, or new requirements
3. Use the Explore tool to search the codebase for changed context or new patterns
4. Ask stakeholder about changed priorities or constraints
5. Update the specification while maintaining template structure
6. Verify all requirements are still testable and measurable
