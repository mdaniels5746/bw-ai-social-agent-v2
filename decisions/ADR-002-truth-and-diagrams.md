# ADR-001 — Truth Sources & Diagram Enforcement

**Status:** Accepted  
**Date:** 2026-02-05

## Context
We maintain V2 automation logic across Make scenarios, Google Sheets schemas, and handoff documentation.

## Decision
GitHub repo is the governance source. Mermaid + draw.io diagrams must mirror the exported Make blueprint(s).
Changes require updating blueprint export + handoff + diagrams together.

## Consequences
- Reduced drift / misinterpretation.
- Easier audits and rollbacks.
- Requires discipline: no “quick fixes” in Make without updating files.
