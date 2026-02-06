# Canonical Raw Access Index (AI-Readable)

These RAW links MUST be used by AI agents. GitHub blob pages are NOT canonical.

---

## Startup Read Order (LOCKED)

1) Read THIS file: `REPO_ACCESS.md`
2) Read the handoff (authoritative resume point + instructions)
3) Read the manifest (required artifacts + required handoff sections)
4) Read ADRs (semantics)
5) Read Mermaid (canonical flow)
6) Read blueprints/schemas as needed for module order + sheet contracts

If any required artifact cannot be fetched, STOP.

---

## Authoritative Artifacts (RAW)

### Bootstrap
REPO_ACCESS (this file):
https://raw.githubusercontent.com/mdaniels5746/bw-ai-social-agent-v2/refs/heads/main/REPO_ACCESS.md

### Handoff (Resume authority)
Handoff:
https://raw.githubusercontent.com/mdaniels5746/bw-ai-social-agent-v2/refs/heads/main/handoff/AI_Social_AGENT_V2_Handoff.md

Manifest:
https://raw.githubusercontent.com/mdaniels5746/bw-ai-social-agent-v2/refs/heads/main/handoff/HANDOFF_MANIFEST.md

### Decisions (Semantics)
ADR-001 Status Semantics:
https://raw.githubusercontent.com/mdaniels5746/bw-ai-social-agent-v2/refs/heads/main/decisions/ADR-001-status-semantics.md

ADR-002 Repo Truth + Bootstrap + Mermaid-only:
https://raw.githubusercontent.com/mdaniels5746/bw-ai-social-agent-v2/refs/heads/main/decisions/ADR-002-truth-and-diagrams.md

### Diagrams (Enforced)
Weekly Planner V2 — Parent Ingestion (Mermaid):
https://raw.githubusercontent.com/mdaniels5746/bw-ai-social-agent-v2/refs/heads/main/diagrams/mermaid/weekly_planner_v2_parent_ingestion.mmd

### Blueprints (Often required for module order)
Weekly Planner V2 Blueprint:
https://raw.githubusercontent.com/mdaniels5746/bw-ai-social-agent-v2/refs/heads/main/blueprints/BW_V2_Weekly_Planner.blueprint.json

### Schemas (Often required for sheet contracts)
V2 Sheets Schema:
https://raw.githubusercontent.com/mdaniels5746/bw-ai-social-agent-v2/refs/heads/main/schemas/BW_AI_Social_Agent_V2_Sheets.json

---

## Rules (LOCKED)

- Resume point is authoritative ONLY in the handoff.
- Mermaid is the only enforced diagram format.
- draw.io is non-authoritative and must not be required.
- If a file is missing or unreadable, STOP and request the user to fix links or paste the artifact contents.
