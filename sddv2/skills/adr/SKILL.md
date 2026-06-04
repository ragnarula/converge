---
name: adr
description: Write an Architecture Decision Record after implementation, before merge. Captures key technical decisions from the SDD workflow so future readers understand what was decided, why, and when to revisit. Only use when meaningful alternatives were considered.
version: 0.1.1
---

# ADR

Write an Architecture Decision Record (ADR) capturing key technical decisions made during a feature's research and design. Run this after implementation, before merge.

Both this orchestrator and every delegated worker it uses follow the `language` skill for tone and vocabulary in all output.

## When to Write an ADR

Not every feature needs one. Write an ADR when:

- The research explored 2+ meaningfully different approaches
- A technology or infrastructure choice was made (database, messaging system, framework)
- The design creates a long-term constraint that future work must respect
- A significant tradeoff was accepted that future readers need to understand

Do NOT write an ADR for:
- Features with only one viable approach and no meaningful tradeoffs
- Bug fixes, refactors, or minor enhancements
- Decisions already well-documented in existing ADRs

**Ask the user** if you're unsure whether an ADR is warranted. If the answer to "will someone need to know why in 6 months?" is no, skip it.

## Practical Guidelines

### Project Structure and Paths

ADRs live alongside the feature's SDD artifacts in `.sdd/{feature}/` (or `.sdd/{initiative}/{deliverable-slug}/` for roadmap deliverables).

### ADR Numbering

If the project maintains a numbered ADR index (e.g., `docs/adr/`), follow that convention. Otherwise, store the ADR in the feature's `.sdd/` directory without a global number.

## Process

### Step 1: Identify decisions worth recording

Read the design at `{artifact_dir}/design.md`. Read research at `{artifact_dir}/research.md` if it exists — for roadmap deliverables it has been retired by the roadmap step, so fall back to `.sdd/{initiative}/roadmap.md` for context. Look for:

- Places where alternatives were considered and one was chosen over others
- Technology or infrastructure choices with tradeoffs
- Design constraints that future work must respect
- Assumptions that could become invalid

If nothing qualifies, tell the user and skip the ADR.

### Step 2: Write the ADR

Write the ADR in an isolated work context. Use delegated work if the runtime supports it; invoking this skill authorizes that delegation. If delegated work is unavailable, perform the step directly.

**Delegated-work prompt** (use a high-capability reasoning model):
> Think hard.
>
> Write an ADR for {feature} at `{artifact_dir}/adr.md`.
>
> **Read these files:**
> - Design: `{artifact_dir}/design.md`
> - Research: `{artifact_dir}/research.md` if it exists; otherwise `.sdd/{initiative}/roadmap.md` (for roadmap deliverables)
> - ADR template: templates/adr.template.md
>
> **Follow the template structure.** For each section:
> - **Context**: 2-3 sentences. Written for someone who wasn't in the room. Why was a choice needed?
> - **Decision Drivers**: The 2-4 criteria that actually tipped the balance, not every possible concern
> - **Options Considered**: For each option: what it is (one sentence), pros, cons. Pull from the research document's Diverge/Evaluate findings. Include options that were ruled out — this is the most valuable section for future readers.
> - **Decision**: One paragraph. What was chosen and why, linking back to the decision drivers.
> - **Consequences**: Honest positive and negative. What are we giving up?
> - **Revisit When**: Specific conditions that would invalidate this decision. Not vague ("if requirements change") — concrete ("if event volume exceeds 1M/day", "if we move off AWS"). Future readers check these to decide whether the ADR still holds.
>
> Keep the entire ADR to one page. If it's longer, you're including design detail that belongs in the design document.
>
> Save the document when done.

### Step 3: Review with the user

Present the ADR. Ask:
- Does this capture the real reason for the decision, or the post-hoc rationalization?
- Are the "Revisit When" conditions the right triggers?
- Is anything missing that a future reader would need?

Iterate based on feedback. Once the user approves, set status to Accepted and commit.
