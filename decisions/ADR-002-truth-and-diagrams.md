# ADR-002 — Repo Truth Model, Bootstrap Access, and Diagram Enforcement

## Status
Accepted

## Context
We require a deterministic “single truth” model so new chats and agents can resume work without relying on memory.
Historically, drift occurred due to:
- outdated resume points in handoff vs actual blueprint state
- ambiguous diagram sources
- tooling dependencies (draw.io) creating read/parsing failures

## Decision

### 1) Canonical Truth Sources (Repo)
The following are authoritative and must remain consistent:

- `handoff/AI_Social_AGENT_V2_Handoff.md`
- `blueprints/*.json`
- `schemas/*.json`
- `diagrams/mermaid/*.mmd`
- `decisions/ADR-*.md`

Chat history is not authoritative.

### 2) Bootstrap Rule (REPO_ACCESS.md)
New chats/agents MUST read `REPO_ACCESS.md` first.
This file defines RAW URLs and authoritative file mappings.

If REPO_ACCESS.md is unavailable or unreadable, execution MUST STOP.

### 3) Resume Point Authority
The authoritative “where we resume” marker exists ONLY in the handoff:

- `handoff/AI_Social_AGENT_V2_Handoff.md`

Diagrams describe flow but do not define resume progress.

### 4) Mermaid-Only Diagram Enforcement
Mermaid is the only enforced diagram format.

- Canonical flow diagrams live in `diagrams/mermaid/`.
- Any enforced flow diagram MUST match the Make blueprint and module names.
- draw.io is explicitly NON-AUTHORITATIVE and is not required.

## Consequences
- Any logic change requires updating:
  - blueprint JSON (if Make changed)
  - handoff (UPDATE-IN-PLACE with WHEN/WHY/FROM→TO)
  - mermaid diagram (if flow changed)
  - ADRs (if semantics changed)

- draw.io files may exist but must not be required for enforcement or resume.

## References
- REPO_ACCESS.md
- handoff/AI_Social_AGENT_V2_Handoff.md
- diagrams/mermaid/*.mmd
