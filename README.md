# Beggars Whisky — AI Social Agent V2 (Project Repo)

**Authoritative baseline:** v1.15 handoff pack (2026-02-05).  
This repo is a **governance + enforcement layer** to keep the handoff, blueprint exports, and schemas consistent.

## Enforcement Rules (Repository)
- **Authority:** `/handoff/*.md`, `/blueprints/*.json`, `/schemas/*.json` are the source of truth.
- **No drift:** Any scenario change in Make requires updating:
  1) blueprint export JSON
  2) handoff markdown (UPDATE-IN-PLACE, with WHEN/WHY/FROM→TO)
  3) module inventory spreadsheet (if module set changes)
  4) mermaid diagram (mirrors the blueprint)
  
## Folder Guide
- `handoff/` — handoff markdown
- `blueprints/` — Make blueprint exports
- `schemas/` — Sheets schema exports / contracts
- `diagrams/mermaid/` — Mermaid flow diagrams (diffable)
- `decisions/` — ADRs (architecture decisions)

## Quick Start
1. Create a GitHub repo (private).
2. Upload this folder contents.
3. Add/commit changes only via pull requests (even solo).
