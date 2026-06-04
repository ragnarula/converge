---
name: roadmap
description: Break a too-large initiative into a sequence of vertical deliverables, each sized to one spec. Use this skill when research reveals more work than a single specification can hold, or when the user knows up-front the work is multi-spec. Produces a roadmap document that drives subsequent requirements/plan/tasks/implement loops, one deliverable at a time.
version: 0.2.1
---

# Roadmap

Use this skill when research has surfaced more work than a single
specification can hold, or when the user already knows the work is
multi-spec. The roadmap turns a large initiative into a sequence of
**vertical deliverables** — each one a user/operator outcome that ships
end-to-end on its own and fits one spec.

Both this orchestrator and every subagent it spawns follow the `language` skill for tone and vocabulary in all output, including replies to the user.

## When to invoke

- Triggered by `research`: research synthesis concludes the work
  exceeds one spec's scope (~1 day of implementation).
- Invoked directly: the user knows up-front the initiative is
  multi-spec and skips straight here after research.

If a roadmap already exists for the initiative, do NOT create a new
one — refine the existing roadmap instead.

## Practical Guidelines

### Project Structure

Roadmaps live at `.sdd/{initiative}/roadmap.md`, with each deliverable's
spec nested at `.sdd/{initiative}/{deliverable-slug}/specification.md`.
This co-locates an initiative's documents and keeps `.sdd/index.md`
showing initiatives at the top level.

### Templates

- Roadmap template: `templates/roadmap.template.md`

### Project Guidelines

Use the `handbook` skill to read and resolve project conventions before
writing the roadmap.

### Domain Skills

After reading the research synthesis, identify which domain skills apply
to the initiative as a whole. Load relevant skills and apply their
mindset to deliverable shaping and sequencing.

- **distributed-systems**, **low-level-systems**, **security**,
  **infrastructure**, **devops-sre**, **data-engineering**,
  **api-design** — same set the other workflow skills use.

## Process

You **MUST** read the research synthesis before drafting the roadmap.
You **MUST** understand project guidelines via the `handbook` skill.

### Creating a Roadmap

**Step 1: Build the Roadmap Brief**

Read `.sdd/{initiative}/research.md`. Distil into a compact brief the
writer can work from without opening the research file. Keep
under ~60 lines and **strip technical/implementation content** — the
brief feeds an outcomes-only roadmap.

```
## Roadmap Brief

**Problem:** {1 short paragraph in user/operator/business terms}

**Motivation:** {1 short paragraph: why now, what's at stake}

**Synthesised direction:** {the chosen direction from research, stated
as outcomes — not as architecture or technology}

**Scope axes:** {dimensions along which the work naturally splits —
user persona, business domain, surface area, risk tier. These inform
deliverable boundaries.}

**Constraints:** {hard constraints — compliance, deadlines, business
boundaries. Skip technical constraints; those belong in design.}

**Open questions:** {bullets}
```

Skip research's Observe / Orient / Diverge / Evaluate narrative and any
technical findings (existing patterns, integration points, prior art).
The roadmap is outcomes-only; technical context belongs in the design
phase, which will rediscover it.

**Step 2: Write the roadmap**

Spawn a subagent to write the roadmap. Invoking this skill authorizes that subagent.

**Subagent prompt** (use a high-capability reasoning model):
> Think hard.
>
> Write the roadmap for {INITIATIVE} at `.sdd/{initiative}/roadmap.md`.
>
> **Read:**
> - Roadmap template: `templates/roadmap.template.md`
> - Project conventions: use the `handbook` skill
>
> Do NOT read the research file — the brief below is your source of
> truth.
>
> **Roadmap Brief:**
> {paste brief from Step 1}
>
> **The roadmap is outcomes-only.** No architecture, no components,
> no libraries, no code, no file paths, no data shapes. If you find
> yourself describing *how* something works, stop — that belongs in
> the design phase. The roadmap captures problem, motivation, and a
> sequence of user/operator outcomes.
>
> **Problem and Motivation** are top-of-document and required.
> Problem states what's broken or missing in user/business terms.
> Motivation states why now — what changed, what's at stake.
>
> **Deliverables:**
> - Each deliverable is a **vertical outcome**: a user/operator can do
>   something after that they couldn't before. Not a layer ("build the
>   API"), not a tech component ("set up the queue"), not a phase
>   ("Phase 1 / Phase 2"). If a deliverable's value can only be
>   realised when paired with another, merge them.
> - Each deliverable must fit **one spec** (~1 day of implementation).
>   If a candidate is too big, split along a scope axis. If you cannot
>   split without losing the end-to-end property, surface this as an
>   open question — do not ship a roadmap with oversized deliverables.
> - **Outcome** field names the user/operator/business actor and what
>   they can do after. "We refactor X" is not an outcome; "operators
>   can replay failed webhooks" is.
> - **In scope / Out of scope** describe outcomes, not work. "Out of
>   scope: bulk replay" not "out of scope: the BulkReplayService".
> - Most deliverables should be **standalone**. If every deliverable
>   depends on D-01, D-01 is plumbing — reshape so D-01 ships an
>   outcome too.
> - Order deliverables by what each one **unlocks** — feedback,
>   learning, risk reduction, or dependent value. The sequencing
>   rationale explains *why* this order in outcome terms.
>
> **Output rules:**
> - Follow the template exactly. Skip optional sections that don't apply.
> - Keep the roadmap under 200 lines. If it grows past that, the
>   deliverables are too granular or the strategy is too detailed —
>   tighten.
> - Save the document. Never skip this.
>
> **Codebase exploration:** Trust the brief. Do NOT explore unless the
> brief is silent on a constraint that affects deliverable shaping —
> and even then, cap at 3 targeted reads.
>
> **Escalation:** If the brief is too ambiguous to shape deliverables,
> or if you cannot split the work without losing the end-to-end
> property, STOP and report what you have plus the specific blocker.

**Step 3: Review the roadmap**

Use the `review` skill to perform a **Roadmap Review** of the roadmap
at `.sdd/{initiative}/roadmap.md`.

**Step 4: Fix issues (if any)**

If the review finds P0 or P1 issues, spawn a subagent to fix them.

**Subagent prompt** (use a high-capability reasoning model):
> Think hard.
>
> Fix the following issues in the roadmap at
> `.sdd/{initiative}/roadmap.md`. If `.sdd/{initiative}/research.md`
> still exists (pre-approval refinement), use it as reference;
> otherwise work from the roadmap alone — research has already been
> retired.
>
> {paste review findings here}
>
> Save the document when done.

After the fix completes, re-run Step 3 (review). Repeat Steps
3–4 until the review passes.

**Step 5: Maintain the index**

Add the initiative to `.sdd/index.md` (create from
`requirements/templates/index.template.md` if it doesn't exist). The
index entry points at the roadmap. Each deliverable's spec, once
created, gets its own row indented under the initiative.

**Step 6: Retire the research file**

Once the roadmap is approved (review passes, user signs off), delete
`.sdd/{initiative}/research.md`. The roadmap is now the source of
truth; research was scaffolding for getting here. Git history
preserves the deleted file for anyone who needs to revisit the
exploration.

Confirm with the user before deleting if they have not previously
expressed standing approval for this workflow step.

Downstream consequence: the design phase will rediscover technical
context (existing patterns, integration points, prior art) via codebase exploration
when each deliverable's design is written. This is intentional — the
roadmap contract is outcomes-only.

### Refining a Roadmap

Refine when learning from a completed deliverable changes the shape of
what comes next.

1. Read existing roadmap and the research it links to.
2. Read the spec/design/implementation of any completed deliverables —
   what shipped and what was learned.
3. Re-evaluate remaining deliverables: still valuable? still vertical?
   still in the right order? Anything to add, drop, or merge?
4. Update the roadmap. Bump version in Change History.
5. Do NOT retroactively edit completed deliverable entries; they are
   the historical record.

## Interaction with other skills

- **research**: precedes roadmap. Research's synthesis is the input.
- **requirements**: each deliverable becomes one specification. The
  spec links back to the roadmap via a `Linked Roadmap` field.
- **express**: refuses to run if a roadmap exists. The user must pick
  a specific deliverable to express.
- **adr**: roadmap-level decisions (sequencing, what was deferred,
  what was deliberately cut from scope) sometimes warrant an ADR.
  Apply the same gate the `adr` skill uses — ADR only if meaningful
  alternatives were considered.
