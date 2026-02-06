# HANDOFF_MANIFEST — BW AI Social Agent V2

## Purpose
This manifest defines the required authoritative artifacts and required handoff sections.
No handoff update is considered valid until all required items are present and consistent.

---

## Required Repo Artifacts (Must Exist)

### Bootstrap
- REPO_ACCESS.md (must be readable first)

### Handoff
- handoff/AI_Social_AGENT_V2_Handoff.md
- handoff/HANDOFF_MANIFEST.md

### Blueprints
- blueprints/BW_V2_Weekly_Planner.blueprint.json

### Schemas
- schemas/BW_AI_Social_Agent_V2_Sheets.json

### Diagrams (Enforced)
- diagrams/mermaid/weekly_planner_v2_parent_ingestion.mmd

### Decisions
- decisions/ADR-001-status-semantics.md
- decisions/ADR-002-truth-and-diagrams.md

---

## Required Handoff Sections (Must Exist)
The handoff must contain the following top-level headings:

- 0. Meta Rules (LOCKED)
- 1. Purpose of V2 (LOCKED)
- 2. Global Governance (LOCKED)
- Governance & Enforcement (LOCKED)
- 3. V2 Data Model (LOCKED)
- Phase 0 — Observability + Secrets + GCP Setup (COMPLETE)
- Phase 1 — Weekly Planner V2
- Phase 2 — Draft Generator V2
- Phase 3 — Scheduler/Posting V2
- Phase 4 — Video Workload (DISABLED)
- Phase 5 — Housekeeping + Archive
- Change Log

---

## Required Resume Marker (Must Exist)
The handoff must include an explicit Resume Point section containing:
- Phase
- Step
- Micro-step (if applicable)
- The last completed module name(s)
- The next action stated as a single micro-step

---

## Enforcement Notes
- Mermaid-only diagram enforcement is mandatory.
- draw.io is non-authoritative and must not be referenced as required.
- Any logic change requires WHEN/WHY/FROM→TO in the Change Log.
- If a blocker inserts steps, renumber downstream and record a Renumber Map in Change Log.

---
END MANIFEST
