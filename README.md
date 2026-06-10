# SDD v3 — Spec Driven Development for Agent Runtimes

An agent-runtime plugin for **problem-driven** development. Sharpen a vague problem into a well-posed definition, research it, converge on an approach and a specification, break that into tasks, and implement them — with review gates along the way.

Artifacts are stored in the **Simple Context Service (SCS)** — a remote, versioned store reached through the `scs` MCP server — not in local files. Each piece of work is an SCS *concept* holding its `problem`, `research`, `specification`, and `tasks` artifacts. Only ADRs remain local files. See the `artifacts` reference skill for the storage contract, including how to connect and sign in to the `scs` server.

## Workflow

```
problem-definition → problem-research → problem-converge → problem-tasks → implement (+ reviews)
```

- **problem-definition** — sharpen a vague pain point into a well-posed problem: what it is, who is affected, what success looks like.
- **problem-research** — decide the questions that must be answered, then answer them as a neutral, fully-explained description of the current state (not a recommendation).
- **problem-converge** — work with the user to weigh candidate approaches, pick a direction, and write the specification (requirements in EARS, design, test plan, docs).
- **problem-tasks** — break the spec into ordered, demoable tasks; reviews and fixes itself, raising to the user only when a fix would conflict with the spec.
- **implement** — one subagent per task, TDD against each task's acceptance criteria, then an implementation review.

Domain skills are applied throughout — at converge to validate candidate approaches, and inside reviews.

For a refactor or migration, use **extract-spec** to capture the behaviour-preservation contract as a specification instead of converging on a new direction.

## Workflow skills

Artifacts are addressed by SCS concept + kind. The concept is `{feature}` (kebab-case, e.g. `user-authentication`).

| Skill | Purpose | Artifact (SCS kind) |
|-------|---------|---------------------|
| `problem-definition` | Questioning interview → a well-posed problem (what, who, success, constraints, evidence) | `problem` |
| `problem-research` | Enumerate the askable questions (kill-shot / disconfirming first), then research them into a standalone description of current state | `research` |
| `problem-converge` | Weigh approaches with the user, pick a direction, write the specification (EARS requirements, design, test plan, docs) | `specification` |
| `extract-spec` | Sibling to `problem-converge` for refactors and migrations — captures preserved behaviour as a spec, each FR backed by an existing or pin test | `specification` |
| `problem-tasks` | Break the spec into ordered demoable tasks; auto-reviews and fixes, escalating only on spec conflict | `tasks` |
| `tasks` | Design-driven tracer-bullet task breakdown (the original, fuller breakdown) | `tasks` |
| `implement` | One subagent per task, TDD against the task's ACs, review at the end | commits + `tasks` revisions |
| `review` | Implementation review with P0–P3 severity | report |
| `adr` | Architecture Decision Record for key choices | local `adr.md` |

## Reference skills

Used by the workflow skills rather than invoked directly.

| Skill | Purpose |
|-------|---------|
| `ears` | EARS syntax patterns (ubiquitous, event-driven, state-driven, unwanted-behavior, optional-feature) |
| `interface-design` | Design components for testability (accept dependencies, return results, small surface) |
| `tdd` | Red-green-refactor with the dependency-count heuristic for test depth |
| `mocking` | Mock at system boundaries only, never internal modules |
| `language` | Active voice, simple English, no metaphors or slang — applied to docs and to interactive replies |
| `artifacts` | SCS storage contract — concept addressing, read/write/conflict mechanics, repository linking, auth preflight |

## Domain skills

Specialized lenses applied at convergence and in reviews:

`security` · `api-design` · `distributed-systems` · `data-engineering` · `devops-sre` · `infrastructure` · `low-level-systems`

They are used to validate candidate approaches during `problem-converge` (flag unsolvable, costly, or user-harming options) and to stress-test the specification and task breakdown for soundness and feasibility.

## Project conventions

Skills resolve project conventions by reading your **existing documentation** — README, CONTRIBUTING, `docs/`, `CONTEXT.md`, ADRs, and config files. There is no separate handbook to maintain; the skills look at what's already in the repo, and fall back to the existing code and test suite for what isn't written down.

The single most impactful thing you can do for output quality is document the conventions an agent would otherwise get wrong — test locations, error handling patterns, naming, commit format, how to run linters. Capture them where they naturally live (README, CONTRIBUTING, `docs/`):

- **Error handling** — error types, propagation patterns
- **Testing** — framework, file locations, fixtures, how to run the suite (`tdd` decides *how many* tests via the dependency-count heuristic; your docs and test suite cover *where* they live and *how* they're written)
- **Naming** — files, functions, modules
- **Pre-commit validation** — lint, format, type check commands
- **Project structure** — where new code goes

## Context Management

Some host runtimes can register hooks that prevent context exhaustion. The bundled Claude Code hooks (in `sddv3/hooks/`) are optional host integration, not part of the core contract:

- **sdd-statusline.js** — shows context usage in the status bar, writes metrics to a bridge file
- **sdd-context-monitor.js** — reads the bridge file after tool uses, warns the agent at 35% remaining, stops it at 25%

Register them by hand in your settings (a `statusLine` command and a `PostToolUse` hook pointing at the two scripts) if your host supports them.

## Runtime Portability

The workflow is defined in terms of capabilities, not one agent host's tool names. When a skill mentions one of these capability classes, use the equivalent tool in the current runtime:

| Capability | Meaning | Fallback if unavailable |
|------------|---------|-------------------------|
| File discovery | Find files and paths, such as globbing for `SKILL.md`, templates, hooks, or source files | Use shell commands such as `rg --files`, `find`, or the runtime's file search |
| File read/write | Read existing project docs; create or update local ADR files | Use normal filesystem tools available to the runtime |
| SCS artifact store | Read and write versioned artifacts via the `scs` MCP server (concept + kind addressing) | Required — the `artifacts` skill defines the contract and the connect/sign-in preflight |
| Codebase exploration | Search and read source, tests, docs, configs, and git history to understand current behavior | Use targeted shell searches and file reads; keep exploration bounded by the skill's stated read limits |
| Subagent | Run a writing, research, implementation, or review step in a spawned subagent | Required for steps that say to spawn a subagent. Invoking a workflow skill is explicit user authorization to spawn those subagents |
| High-capability reasoning model | A model suitable for problem framing, research synthesis, specification, and review work | Use the strongest available reasoning model |
| Implementation-capable model | A model suitable for coding tasks, tests, and local verification | Use the default coding model or strongest available implementation model |
| Host configuration update | Register optional statusline, hook, or settings integrations | If no config-update capability exists, print the exact manual configuration for the user |

Host-specific names in older prompts map as follows:

- `Explore tool` means codebase exploration.
- `Read` means file read.
- `Glob` means file discovery.
- `Task tool` means spawn a subagent.
- `model: opus`, `model: sonnet`, and `Ultrathink` mean the model classes above, not specific required model IDs.

If a preferred capability is missing, do not stop unless the skill explicitly requires a host integration. Continue with the fallback and record the limitation in the artifact or final report.

### Subagent Authorization

When the user invokes a workflow skill (`problem-definition`, `problem-research`, `problem-converge`, `extract-spec`, `problem-tasks`, `tasks`, `implement`, `review`, or `adr`), treat that invocation as an explicit request to spawn subagents for the skill's subagent steps. Do not ask for separate delegation permission unless the subagent would exceed the skill's normal scope or perform a live-state mutation that already requires user confirmation.

## Usage

```
Use problem-definition to frame the problem behind {pain point}
Use problem-research to research the open questions for {feature}
Use problem-converge to pick an approach and write the spec for {feature}
Use extract-spec to write a spec for a refactor or migration of {area}
Use problem-tasks to break {feature} into tasks
Use implement to build {feature}
Use review to review the implementation of {feature}
Use adr to record decisions for {feature}
```

## License

0BSD
