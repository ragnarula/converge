# SDD v3 — Spec Driven Development for Agent Runtimes

An agent-runtime plugin for **problem-driven** development. Sharpen a vague problem into a well-posed definition, research it, specify what it must do, design how to build it, break that into tasks, and implement them — with review gates along the way. Research, spec, and design are each optional: take the steps the risk justifies and skip the rest.

Artifacts are stored in the **Simple Context Service (SCS)** — a remote, versioned store reached through the `scs` MCP server — not in local files. Each piece of work is an SCS *concept* holding its `problem`, `research`, `specification`, and `tasks` artifacts. Only ADRs remain local files. See the `artifacts` reference skill for the storage contract, including how to connect and sign in to the `scs` server.

## Workflow

```
frame → explore → spec → design → breakdown → implement (+ reviews)
         └── research, spec, and design are each optional ──┘
```

- **frame** — sharpen a vague pain point into a well-posed problem: what it is, who is affected, what success looks like.
- **explore** — decide the questions that must be answered, then answer them as a neutral, fully-explained description of the current state (not a recommendation). Optional.
- **spec** — pin down the *what*: the requirements in EARS, constraints, and docs, independent of any solution. Grills the user until the scope is unambiguous. Optional.
- **design** — converge on the *how*: weigh candidate approaches with the user, pick a direction, and design it to a depth that matches the risk. Optional.
- **breakdown** — break whatever context exists (problem, research, spec, design) into ordered, demoable tasks; reviews and fixes itself, raising to the user only when a fix would conflict with a source artifact.
- **implement** — one subagent per task, TDD against each task's acceptance criteria, then an implementation review.

Each step asks the user where they feel the risk is before proceeding, and lets that answer set how much to do — how hard to pin down requirements, how deep to design, what to sequence first. Domain skills are applied throughout — at design to validate candidate approaches, and inside reviews.

For a refactor or migration, use **extract-spec** to capture the behaviour-preservation contract as a specification instead of specifying and designing a new direction.

## Workflow skills

Artifacts are addressed by SCS concept + kind. The concept is `{feature}` (kebab-case, e.g. `user-authentication`).

| Skill | Purpose | Artifact (SCS kind) |
|-------|---------|---------------------|
| `frame` | Questioning interview → a well-posed problem (what, who, success, constraints, evidence) | `problem` |
| `explore` | Enumerate the askable questions (kill-shot / disconfirming first), then research them into a standalone description of current state | `research` |
| `spec` | Pin down the *what* with the user: requirements in EARS, constraints, docs — independent of any solution; grills until scope is unambiguous | `specification` |
| `design` | Converge on the *how*: weigh approaches, pick a direction, design it to a depth that matches the risk | `design` |
| `extract-spec` | Sibling to `spec`/`design` for refactors and migrations — captures preserved behaviour as a spec, each FR backed by an existing or pin test | `specification` |
| `breakdown` | Break whatever context exists into ordered demoable tasks; auto-reviews and fixes, escalating only on source conflict | `tasks` |
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
| `conventions` | Discover and apply the repo's guidelines wherever they live — producers apply them, reviewers verify adherence |

## Domain skills

Specialized lenses applied at design and in reviews:

`security` · `api-design` · `distributed-systems` · `data-engineering` · `devops-sre` · `infrastructure` · `low-level-systems`

They are used to validate candidate approaches during `design` (flag unsolvable, costly, or user-harming options) and to stress-test the specification, design, and task breakdown for soundness and feasibility.

## Project conventions

Skills resolve project conventions through the `conventions` reference skill, which reads your **existing documentation** — README, CONTRIBUTING, `docs/`, `CONTEXT.md`, ADRs, and config files — wherever and in whatever format it lives. There is no separate handbook to maintain; producers apply what they find, reviewers check adherence, and both fall back to the existing code and test suite for what isn't written down.

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

When the user invokes a workflow skill (`frame`, `explore`, `spec`, `design`, `extract-spec`, `breakdown`, `implement`, `review`, or `adr`), treat that invocation as an explicit request to spawn subagents for the skill's subagent steps. Do not ask for separate delegation permission unless the subagent would exceed the skill's normal scope or perform a live-state mutation that already requires user confirmation.

## Usage

```
Use frame to frame the problem behind {pain point}
Use explore to research the open questions for {feature}
Use spec to write the specification for {feature}
Use design to pick an approach and design {feature}
Use extract-spec to write a spec for a refactor or migration of {area}
Use breakdown to break {feature} into tasks
Use implement to build {feature}
Use review to review the implementation of {feature}
Use adr to record decisions for {feature}
```

## License

0BSD
