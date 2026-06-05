---
name: ears_v3
description: EARS (Easy Approach to Requirements Syntax) reference. Use this when writing or reviewing requirements that must be stated in EARS syntax — the five canonical patterns, how to combine them, and the "shall" verb convention.
version: 0.1.0
---

# EARS — Easy Approach to Requirements Syntax

Five canonical patterns for stating a requirement as a single sentence. Pick the pattern that fits the behavior; combinations are allowed.

## Patterns

**Ubiquitous** — always active.
> *The system shall {capability}.*
>
> Example: *The system shall encrypt user passwords at rest.*

**Event-driven** — triggered by a discrete event.
> *When {trigger}, the system shall {response}.*
>
> Example: *When a user marks a notification as read, the system shall set `read_at` to the time of the request.*

**State-driven** — active while a condition holds.
> *While {state}, the system shall {behavior}.*
>
> Example: *While a payment is in flight, the system shall reject duplicate submissions for the same cart.*

**Unwanted behavior** — response to a fault or invalid input.
> *If {condition}, then the system shall {response}.*
>
> Example: *If a user attempts to mark a notification they do not own as read, then the system shall reject the request and leave `read_at` unchanged.*

**Optional feature** — applies only when a feature is included.
> *Where {feature is included}, the system shall {behavior}.*
>
> Example: *Where SSO is enabled, the system shall redirect unauthenticated users to the identity provider.*

## Combinations

Combine patterns when the behavior needs more than one qualifier.

> *While {state}, when {trigger}, the system shall {response}.*
>
> Example: *While a feature flag is enabled, when a user opts in, the system shall enroll them in the new flow.*

## Voice

Always "the system shall" — not "should", "must", "will", or "may". The "shall" verb is what makes the requirement reviewable as EARS.
