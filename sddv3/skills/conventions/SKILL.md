---
name: conventions
description: Discover and apply a repository's developer documentation and guidelines, wherever they live and whatever format they take. Use this whenever an SDD skill needs to resolve project conventions — producers apply them, reviewers check adherence against the same discovered set.
version: 0.1.0
---

# Conventions

A repository's guidelines live in different places and formats. This skill is the one discovery contract, so every step resolves the same conventions the same way.

## Where to look

Search these, and follow any pointer one file makes to another:

- `README`, `CONTRIBUTING`, `AGENTS.md`, `CLAUDE.md`.
- `docs/` — architecture and style docs, `CONTEXT.md`, `docs/adr/`.
- Machine configs that encode rules: `.editorconfig`, linter/formatter configs (eslint, prettier, ruff, gofmt, and the like), pre-commit hooks, CI workflows.

Markdown, rst, plain text, or config — all count.

## What to extract

Only the rules relevant to the work at hand: commenting, error handling, logging, naming, testing (where tests live and how to run them), commit format, project structure, pre-commit validation. This is an example list, use your context to decide what is relevant.

## Present or absent

Where a rule is documented, follow it. Where the repo is silent, treat the surrounding code and test suite as the convention of record.

## Two roles

- **Producers** (`design`, `breakdown`, `implement`) apply the discovered rules.
- **Reviewers** check adherence against the same discovery and flag each violation by severity.
