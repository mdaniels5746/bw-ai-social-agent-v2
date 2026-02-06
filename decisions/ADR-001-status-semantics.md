# ADR-001 — Status Semantics (Source vs Planner vs Content)

## Status
Accepted

## Context
The system uses a `status` field across multiple sheets/tables, but the meaning differs by surface.
This ADR exists to prevent semantic drift (e.g., confusing source eligibility with content lifecycle).

## Decision

### 1) Source Status (BW_SOURCES_SHOWS, BW_SOURCES_RELEASES, BW_SOURCES_EVENTS)
- Allowed values: `active`, `inactive` (or equivalent as defined by sheet enums)
- Meaning: Controls whether a source row is eligible to be ingested by Weekly Planner V2.
- Usage: Evaluated only in the source-filter stage before iteration.
- Source status MUST NOT propagate into BW_CONTENT_QUEUE content lifecycle.

### 2) Content Queue Status (BW_CONTENT_QUEUE)
BW_CONTENT_QUEUE uses editorial lifecycle semantics.

- Allowed values are defined by BW_ENUMS / handoff governance (examples):
  - `Idea`
  - `Needs Approval`
  - `Rejected`
  - `Approved`
  - `Scheduled`
  - `Posted`
  - `failed`

### 3) Parent Rows in BW_CONTENT_QUEUE
Parent rows are “containers” but still live inside BW_CONTENT_QUEUE.
Therefore parent rows MUST use the same lifecycle status semantics.

**Initial status for newly created parent rows MUST be:**

This is intentional and blueprint-aligned.

### 4) Rejected Regeneration Rule (Dependency)
If a child item is set to `Rejected`, regeneration MUST use the BW_CONTENT_QUEUE `notes` field as authoritative guidance for what to fix.

## Consequences
- Weekly Planner V2 creates parent rows with `status = Idea`.
- `open` is NOT a valid BW_CONTENT_QUEUE status unless explicitly added to enums (not currently allowed).
- Any change to status meaning requires a new ADR.

## References
- handoff/AI_Social_AGENT_V2_Handoff.md
- blueprints/BW_V2_Weekly_Planner.blueprint_*.json
