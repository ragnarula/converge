---
name: tdd
description: Test-driven development for SDD — red-green-refactor with the dependency-count heuristic for test depth. Use this when implementing a task.
version: 0.1.0
---

# TDD

## Principle

You can't test everything. Test the contracts callers depend on, not the shape of the code that delivers them. Most code is covered by being on the path of an AC test; only some warrants a dedicated test.

Good tests describe what the system does through its public interface. They survive refactors. Bad tests are coupled to implementation — they mock internal collaborators, assert call counts, or verify state through internal access. If a refactor that doesn't change behavior breaks the test, the test was wrong.

See the `interface-design` skill for designing components to be testable, and the `mocking` skill for where to use test doubles.

## The Loop

For each task:

### 1. Red — write AC tests first

The task's Acceptance criteria are the gate. Write one test per AC, exercising the When clause and asserting the Then. Run them. They must fail. If a test passes before any implementation, it's asserting something already true — fix it.

Each AC test asserts through the surface the When names, not internal state.

### 2. Green — minimum code to pass

Write only enough code to make the failing tests pass. Skip methods, branches, and error paths that belong to other tasks' ACs. If you find code that no test in this task exercises, move it to the task that does.

### 3. Refactor

Once green: remove duplication, deepen modules where complexity warrants it, clarify names. Get back to green after each change. Refactor only from green.

## Test depth — dependency-count heuristic

Beyond AC tests, you decide how much additional testing each touched component warrants. The dependency-count heuristic is the signal.

Read the design (the `design` artifact on the concept in SCS, or the design context the implement orchestrator pasted into the task prompt) for the components this task touches and their dependant counts:

- Modified components: `Dependants:` field.
- Added components: `Consumers:` field.
- Used components: `Used by:` field.

More dependants → invest more in tests. A component used widely has a bigger blast radius if it breaks. A leaf with few or no dependants is usually covered by the AC tests that exercise it transitively.

This is a heuristic, not a threshold. Weigh the cost of a bug against the cost of writing and maintaining the test. Project docs and the existing suite show where tests live and how to write them, not how many — that call is yours.

## Where and how to write tests

1. Look for existing project docs (README, CONTRIBUTING, docs/) covering test file locations, fixtures, assertion style, framework, runner, and naming.
2. Explore the existing test suite for what the docs leave implicit — fixtures and helpers for the dependencies you're touching, arrangement patterns, parameterisation conventions.

Reuse what exists. Use real code paths; mock only at system boundaries (see the `mocking` skill).

## Test code quality

Treat tests as production code:

- Extract a helper if the same arrange block appears in three tests.
- Parameterise tests that differ only in input.
- Name tests after the behavior under test.
- Reuse shared fixtures and helpers.

## Anti-patterns

- **Horizontal slicing.** Writing all tests before any implementation produces tests for imagined behavior. One test → one implementation → next.
- **Tautologies.** Then clauses that restate the When ("when X is requested, then X was requested"). Delete and rewrite.
- **Plumbing tests.** Standalone tests for components that only exist to be called by a tested component. They are covered transitively. If a plumbing test seems necessary, the AC test isn't reaching the real path — fix the AC test.
- **Stubs and dead code.** Replace stubs (`skip`, `todo`, `pending`, empty test body, `assert True`) with real assertions or remove them. If an external blocker forces a stub, record it in the task's Notes section and resolve before the final task.
