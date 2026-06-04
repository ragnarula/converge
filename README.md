# SDD v2 — Spec Driven Development for Agent Runtimes

An agent-runtime plugin for structured feature development. Turn an idea into EARS requirements, an architectural design, demoable tracer-bullet tasks, and an implementation — with a review gate at every step.

## Workflow

```
roadmap (optional) → research (optional) → requirements | extract-spec → plan → tasks → implement → review → adr
```

Use `requirements` for greenfield work, `extract-spec` for refactors and migrations.

Each step is a skill you can invoke independently or chain via `express`.

## Workflow skills

| Skill | Purpose | Artifact |
|-------|---------|----------|
| `research` | Guided problem exploration (Observe → Orient → Diverge → Evaluate → Synthesize) | `.sdd/{feature}/research.md` |
| `roadmap` | Break a too-large initiative into vertical deliverables, each sized to one spec | `.sdd/{initiative}/roadmap.md` |
| `requirements` | Discovery interview → behavioral specification in EARS, with Given/When/Then acceptance tests | `.sdd/{feature}/specification.md` |
| `extract-spec` | Sibling to `requirements` for refactors and migrations. Reads existing code, interviews the user, produces the same spec shape with each preserved FR backed by an existing test or a pin test scheduled before refactor work begins | `.sdd/{feature}/specification.md` |
| `plan` | Architectural design as a set of components (Modified / Added / Used) traceable to FRs | `.sdd/{feature}/design.md` |
| `tasks` | Demoable tracer-bullet tasks with prose `What to build`, Given/When/Then ACs, and explicit `Blocked by` | `.sdd/{feature}/tasks.md` |
| `implement` | One isolated work context per task, TDD against the task's ACs, review at the end | commits |
| `review` | Roadmap / spec / design / task / implementation review with P0–P3 severity | report |
| `adr` | Architecture Decision Record for key choices | `.sdd/{feature}/adr.md` |
| `express` | Chains requirements → plan → tasks → implement end-to-end | all of the above |
| `setup` | Registers context monitoring hooks + discovers project conventions | `.sdd/handbook.md` |
| `handbook` | Reads and resolves project conventions for all skills | — |

## Reference skills

Loaded by the workflow skills rather than invoked directly.

| Skill | Purpose | Loaded by |
|-------|---------|-----------|
| `ears` | EARS syntax patterns (ubiquitous, event-driven, state-driven, unwanted-behavior, optional-feature) | `requirements`, `review` |
| `interface-design` | Design components for testability (accept dependencies, return results, small surface) | `plan` |
| `tdd` | Red-green-refactor with the dependency-count heuristic for test depth | `implement` |
| `mocking` | Mock at system boundaries only, never internal modules | `tdd` |
| `language` | Active voice, simple English, no metaphors or slang — applied to docs and to interactive replies | every workflow skill |

## The Handbook

The single most impactful thing you can do for output quality is write a good `.sdd/handbook.md`. Every skill reads it. Every delegated worker follows it. Without it, agents guess at conventions and get things wrong — test locations, error handling patterns, naming, commit format, how to run linters.

Run the `setup` skill to auto-discover conventions from your codebase, then refine the result. A good handbook covers:

- **Error handling** — error types, propagation patterns
- **Testing** — framework, file locations, fixtures, how to run the suite (`tdd` decides *how many* tests via the dependency-count heuristic; the handbook covers *where* they live and *how* they're written)
- **Naming** — files, functions, modules
- **Pre-commit validation** — lint, format, type check commands
- **Project structure** — where new code goes

The handbook doesn't need to be exhaustive — it needs to capture what an agent would get wrong without it.

## Domain skills

Specialized review lenses loaded when relevant:

`security` · `api-design` · `distributed-systems` · `data-engineering` · `devops-sre` · `infrastructure` · `low-level-systems`

Each review stage uses these differently:

- **Spec review** — feasibility only (flag requirements the domain says are physically impossible or contradictory).
- **Design review** — architectural soundness *and* feasibility (flag wrong/weak design choices and infeasible choices).
- **Task breakdown review** — slice soundness (flag slice shapes or orderings the domain says create real problems).

## Context Management

Some host runtimes can register hooks that prevent context exhaustion. The included Claude Code hooks are optional host integration, not part of the core SDD contract:

- **sdd-statusline.js** — shows context usage in the status bar, writes metrics to a bridge file
- **sdd-context-monitor.js** — reads the bridge file after tool uses, warns the agent at 35% remaining, stops it at 25%

Run the `setup` skill once per project to discover project guidelines. If the host supports the included hooks, `setup` can also register them.

## Runtime Portability

The SDD workflow is defined in terms of capabilities, not one agent host's tool names. When a skill mentions one of these capability classes, use the equivalent tool in the current runtime:

Skills use `{artifact_dir}` for the resolved SDD artifact directory. Standalone features resolve to `.sdd/{feature}/`; roadmap deliverables resolve to `.sdd/{initiative}/{deliverable-slug}/`.

| Capability | Meaning | Fallback if unavailable |
|------------|---------|-------------------------|
| File discovery | Find files and paths, such as globbing for `SKILL.md`, templates, hooks, or source files | Use shell commands such as `rg --files`, `find`, or the runtime's file search |
| File read/write | Read templates and artifacts; create or update `.sdd/**` files | Use normal filesystem tools available to the runtime |
| Codebase exploration | Search and read source, tests, docs, configs, and git history to understand current behavior | Use targeted shell searches and file reads; keep exploration bounded by the skill's stated read limits |
| Delegated work | Run a writing, implementation, or review step in an isolated context | Prefer subagents when available. If unavailable, the current agent performs the step directly while preserving the same inputs, outputs, and review gate |
| High-capability reasoning model | A model suitable for specification, design, roadmap, ADR, and review work | Use the strongest available reasoning model |
| Implementation-capable model | A model suitable for coding tasks, tests, and local verification | Use the default coding model or strongest available implementation model |
| Host configuration update | Register optional statusline, hook, or settings integrations | If no config-update capability exists, print the exact manual configuration for the user |

Host-specific names in older prompts map as follows:

- `Explore tool` means codebase exploration.
- `Read` means file read.
- `Glob` means file discovery.
- `Task tool` or `subagent` means delegated work.
- `model: opus`, `model: sonnet`, and `Ultrathink` mean the model classes above, not specific required model IDs.
- `update-config` means host configuration update.

If a preferred capability is missing, do not stop unless the skill explicitly requires a host integration. Continue with the fallback and record the limitation in the artifact or final report.

## Usage

```
Use roadmap to break {initiative} into deliverables
Use research to explore the problem space for {feature}
Use requirements to write a spec for {feature}
Use extract-spec to write a spec for a refactor or migration of {area}
Use plan to design {feature}
Use tasks to break down {feature}
Use implement to build {feature}
Use review to review {feature}
Use adr to record decisions for {feature}
Use express to run the full workflow for {feature}
```

## License

0BSD
