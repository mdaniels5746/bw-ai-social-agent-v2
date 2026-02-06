# ADR-001 — Status Semantics (Source vs Planner vs Content)

## Status
Accepted

## Context
The system uses the column name `status` in multiple sheets with different meanings.
This ADR exists to prevent semantic drift and incorrect mappings.

## Decision

### 1. Source Status (BW_SOURCES_SHOWS, BW_SOURCES_RELEASES)
- Allowed values: `active`, `inactive`
- Meaning: Controls whether a source row is ingested into the Weekly Planner.
- Usage: Evaluated only in pre-iteration filters.
- Source status never propagates beyond ingestion.

### 2. Content Queue Status (BW_CONTENT_QUEUE)
- Allowed values: `Idea`, `Needs Approval`, `Approved`, `Rejected`, etc.
- Meaning: Editorial lifecycle of content items.
- Usage: Draft Generator, regeneration, human approval, posting.

### Parent Rows in BW_CONTENT_QUEUE
- Parent rows are containers, but they still live in BW_CONTENT_QUEUE.
- Parent rows MUST use the Content Queue status lifecycle.
- Initial status for newly created parent rows is:

This matches the v1.15 Weekly Planner blueprint and is intentional.

### 3. Planner Container State
- There is no separate planner-status column.
- Planner state is inferred from:
- Parent row existence
- Child rows
- Content Queue status progression

## Consequences
- Parent rows are created with `status = Idea`.
- `open` is NOT a valid status in BW_CONTENT_QUEUE.
- Any future change to status semantics requires a new ADR.

## References
- BW_V2_Weekly_Planner.blueprint_v1.15.json
- AI_Social_AGENT_V2_Handoff_v1.15.md
