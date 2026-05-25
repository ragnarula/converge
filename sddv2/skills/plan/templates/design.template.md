# Design: {Feature Name}

**Version:** 1.0
**Date:** YYYY-MM-DD
**Status:** Draft | Under Review | Approved | Implemented
**Linked Specification** `.sdd/{feature}/specification.md`

---

# Design Document

---

## Architecture Overview

### Current Architecture Context
- {How this fits into the existing system.}

### Proposed Architecture
- {Diagram or description.}
- {Key patterns and rationale.}
- {Sequence diagram if it earns its place.}

### Technology Decisions
- {Choices and justification.}

### Quality Attributes
- {Scalability approach.}
- {Maintainability considerations.}

---

## API Design (optional)
- {Public interface definitions — operations, inputs, outputs, errors.}
- {Data shapes in prose or simple schemas. No language syntax.}
- {Versioning approach.}

---

## Components

### Modified

#### {Component name}
- **Change:** {Current behaviour → new behaviour. The delta only.}
- **Dependants:** {Callers that must change.}
- **Kind:** {Function | Struct | Module | Crate | Database | ...}
- **Details:** {5–10 lines of pseudo-code or type signatures.}
- **Rationale:** {One short paragraph. Name the FR this serves and
  how this component contributes. If it's plumbing, say which FR
  exercises it transitively.}

### Added

#### {Component name}
- **Responsibility:** {One sentence.}
- **Consumers:** {Who calls it.}
- **Location:** {Path or module.}
- **Kind:** {...}
- **Details:** {5–10 lines max.}
- **Rationale:** {One short paragraph linking it to FR.}

### Used

#### {Component name}
- **Location:** {Path.}
- **Provides:** {What we depend on.}
- **Used by:** {Modified/Added components that lean on it.}
- **Rationale:** {One line — why it's reached for here. Name a fixture
  if there's a reusable test sink or helper.}

---

## Feasibility Review

- {Design blockers, provisioning dependencies, or AT-XX setup gaps that gate the implementation. Name which FR each blocker affects.}

---

## Risks and Dependencies

- {Technical risks and mitigations.}
- {External dependencies.}
- {Assumptions worth recording.}

---

## Documentation

- {Dev docs to write or update.}
- {API docs to write or update.}
- {READMEs to touch.}

---

## Instrumentation (optional)

- {Metric/log/trace and the component that owns it.}

---

## Appendix

### Glossary
- **Term:** Definition.

### References
- {Links.}

### Change History
| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | YYYY-MM-DD | {Name} | Initial design. |

---
