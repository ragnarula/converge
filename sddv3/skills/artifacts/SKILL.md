---
name: artifacts
description: Storage contract for SDD artifacts. SDD artifacts (research, specification, design, tasks, roadmap) live in the Simple Context Service (SCS) via the `scs` MCP server, not in local files. Use this when any SDD skill needs to read or write an artifact — it owns concept addressing, the read/write/conflict mechanics, repository linking, and the auth preflight.
version: 0.1.0
---

# Artifacts — SCS storage contract

SDD artifacts are stored in the **Simple Context Service (SCS)**, a remote
store reached through the `scs` MCP server. They are **not** kept in local
`.sdd/` files. Every SDD skill that reads or writes an artifact follows the
contract below.

ADRs stay **local** and are out of scope for SCS — `adr.md` in the feature area,
or the project's numbered `docs/adr/`.

## Preflight — availability and sign-in

Before the first artifact operation in a session:

1. Confirm the `scs` MCP server is connected. If its tools are not available,
   tell the user to add it and stop:
   `claude mcp add --transport http scs https://scs-mcp.delicate-feather-44ab.workers.dev/mcp`
   then authenticate via `/mcp`.
2. Call `get_signed_in_account`. If any artifact call returns
   `error_code: "sign_in_required"`, tell the user to authenticate the `scs`
   server via `/mcp` and stop until they confirm.

Subagents inherit this: a skill that authorizes a subagent also authorizes that
subagent to call the `scs` tools. The subagent is given the concept name and the
artifact kind, and uses the same read/write mechanics below.

## Addressing — concepts and kinds

Each artifact is addressed by a **concept name** plus an **artifact kind**.

**Concept name** encodes the feature/initiative identity:

| Work shape | Concept name |
|------------|--------------|
| Standalone feature | `{feature}` (kebab-case, e.g. `user-authentication`) |
| Roadmap initiative | `{initiative}` (holds the `roadmap` artifact) |
| Roadmap deliverable | `{initiative}/{deliverable-slug}` |

**Artifact kind.** SCS supports these kinds: `problem`, `research`, `specification`,
`design`, `tasks`, `adr`, `roadmap`. The current skills write these:

| Skill | Concept | Artifact kind |
|-------|---------|---------------|
| `frame` | feature | `problem` |
| `explore` | feature | `research` |
| `converge` / `extract-spec` | feature | `specification` |
| `breakdown` | feature | `tasks` |

`design`, `roadmap`, and `adr` are not produced in this flow (`adr` stays local).

## Concept creation and repository linking

The first artifact for a feature requires a concept. Before the first write:

1. Resolve the repository URL from the current git remote
   (`git remote get-url origin`, or the configured remote).
2. Call `list_concepts` with that `repository_url` to see whether a concept
   already exists for this work. Concept listing returns names only.
3. If the concept does not exist, call `create_concept` with `name` set to the
   concept name above and `repository_urls` set to the resolved remote (omit
   `repository_urls` when the repo has no remote). Linking lets later
   `list_concepts repository_url=…` calls find the work.

The roadmap initiative concept and each deliverable concept are separate
concepts; create each on first write.

## Reading an artifact

Call `get_artifact` with `concept_name` and `artifact_kind` (omit `revision_id`
for the latest). The result's `structuredContent` holds:

- `body` — the artifact text.
- `revision.revision_id` — the current revision identity (keep it for the next save).
- `revision.revision_number`, `is_current`.

`error_code: "artifact_has_no_saved_revision"` means the artifact does not exist
yet — treat it as "not written" (e.g. research is absent for a roadmap
deliverable). `error_code: "concept_not_found_for_signed_in_account"` means the
concept has not been created.

## Writing an artifact

Call `save_artifact_revision` with `concept_name`, `artifact_kind`, and `body`.
Each save appends a new immutable revision — it never overwrites history.

- **First revision:** omit `base_revision_id`.
- **Updating an existing artifact:** first `get_artifact` to read the current
  `revision.revision_id`, then save with `base_revision_id` set to it. This lets
  SCS reject a save over a newer revision.
- **Conflict:** a save returning `error_code: "conflict"` includes
  `current_revision.revision_id`. Re-read the current artifact, merge your
  change, and save again with that revision id as `base_revision_id`.

Bodies are capped at 1 MiB (1,048,576 UTF-8 bytes). Keep artifacts within the
size guidance each skill states (designs under ~300 lines, roadmaps under ~200,
etc.).

## Why versioned, not files

SCS keeps every revision immutable and readable by identity, so there is no
"delete" — superseding an artifact means saving a new revision, and "retiring"
one (e.g. research after a roadmap is approved) means downstream skills simply
stop reading it. Older revisions remain in history via `list_artifact_revisions`.
