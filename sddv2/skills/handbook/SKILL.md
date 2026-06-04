---
name: handbook
description: Read and resolve project-specific guidelines. Use this when an agent needs to understand and follow project conventions for error handling, logging, naming, testing, commits, and pre-commit validation.
version: 0.1.0
---

# Project Guidelines

Read and resolve project-specific guidelines so that all workflow phases follow consistent project conventions.

## When to Use This Skill

Use this skill when:
- Starting research for a feature
- Writing a specification
- Creating a design
- Breaking down tasks
- Implementing code
- Reviewing work

## The Guidelines File

Project guidelines are stored at `.sdd/handbook.md`.

This file can:
1. **Reference existing documentation** — list paths to docs, READMEs, or other files containing conventions
2. **Define inline guidelines** — specify conventions directly in the file

## Reading Process

### Step 1: Check for Guidelines

Check if `.sdd/handbook.md` exists.

If the file does not exist, skip to "When the Guide Does Not Exist" below.

### Step 2: Read the Guidelines File

If the file exists, read it thoroughly using the runtime's file-read capability.

### Step 3: Read All Referenced Documentation

The guidelines file may reference other files (paths to docs, READMEs, style guides, etc.). **You MUST read ALL referenced files.**

### Step 4: Extract Conventions

Extract every convention mentioned in the guide and referenced documentation. Follow whatever the guide says — don't filter or categorize beyond what the guide itself defines.

## When the Guide Does Not Exist

If `.sdd/handbook.md` does not exist, explore the codebase using the runtime's code search and file-read capabilities. Discover conventions for at minimum:

- **Error handling** — error types, propagation patterns, error content
- **Logging** — framework, log levels, structured logging
- **Naming** — files, classes, functions, variables
- **Testing** — test location, framework, assertion style, mocking patterns
- **Commits** — message format, conventional commits, scope conventions
- **Pre-commit validation** — linting, formatting, type checking, test commands to run before committing

Document what you find and recommend that a `.sdd/handbook.md` be created.

If guidelines are incomplete or ambiguous:

1. **Ask the user** to clarify conventions
2. **Document assumptions** you're making
3. **Note gaps** that should be filled in the guidelines
