# Beggars Whisky — AI Social Agent V2 Handoff (MASTER PLAN)

**Document:** AI_Social_AGENT_V2_Handoff_v1.18  
**Status:** Phase 0 (Design + Build + Test Gate COMPLETE) · Phase 1 (Design complete · Build IN PROGRESS)
**Date:** 2026-02-06

> **Single Source of Truth:** This document + the matching handoff ZIP are authoritative for V2.  
> **Update-not-append rule:** This handoff is maintained as a *current-state master plan*; sections are updated as decisions evolve.

---

## 0. Meta Rules (LOCKED)

### 0.1 Phase Structure Rule (LOCKED)
Every V2 phase MUST follow this structure:

**Phase N — <Module Name>**
- **Step N.1 — Design** (definition only)
- **Step N.2 — Build** (implementation + inline validation)
- **Step N.3 — Test Gate** (end-to-end verification)

A phase is **complete only after Step N.3 passes**.

### 0.2 Design-First Requirement (LOCKED)
No build begins until:
1) design is explicitly locked, and  
2) the handoff has been updated and confirmed.

### 0.3 Mandatory Handoff Prompt Rule (LOCKED)
Before crossing any major boundary, prompt:
> "Do you want to start or confirm the handoff update before proceeding?"

**Major boundaries include:**
- Step N.1 → Step N.2
- Step N.2 → Step N.3
- Phase N → Phase N+1

### 0.4 Scheduling Rule (LOCKED)
All scenarios are built **Run-on-demand** by default. Schedules are **disabled** until explicitly approved later.

---

## 1. Purpose of V2 (LOCKED)

V2 is a clean redesign (not a patch on V1) designed to:
- Normalize workflows around a **single parent → multi-child (platform) fan-out** model
- Improve content quality by using the **OpenAI API** (Draft Generator only)
- Support future **video workflows** without refactoring
- Provide reliable **observability and retention** via Firestore logging + TTL
- Enforce strict phase gates with test gates

Planned stack:
- **Make.com** (orchestration)
- **ContentStudio** (scheduling/posting)
- **OpenAI API** (draft generation)
- **Cloud Run** (Firestore logging gateway)
- **Firestore** (operational logging + receipts; TTL retention)
- Future video: **Desktop FFmpeg workflow** (designed-in, disabled)

---

## 2. Global Governance (LOCKED)

### AI Access Rule (Enforced)

All AI agents MUST bootstrap from REPO_ACCESS.md.
If unavailable, execution MUST STOP.

A `HANDOFF_MANIFEST` file will be maintained to define required sections and required repo artifacts; updates must satisfy the manifest before commit.

---

- **Handoff update documentation rule:** Any design/process change must be documented with **WHEN** [phase/step/micro-step] and **WHY** [reason], and the **Change Log** must include **FROM → TO** details.

- Beginner-safe micro-steps only
- Manual execution only during development
- No jumping between phases or major steps
- Strict JSON discipline for AI outputs
- Credit conservation enforced
- All Make module Names and Notes must be provided as copy-paste blocks (LOCKED)
- Sheets and schemas may be recreated in V2 by explicit design

---

## Governance & Enforcement (LOCKED)

This repository is the authoritative governance and enforcement layer for Beggars Whisky — AI Social Agent V2.

### Single Source of Truth
The following artifacts are authoritative and must remain consistent at all times:

- `/handoff/*.md` — Canonical system documentation (UPDATE-IN-PLACE only)
- `/blueprints/*.json` — Make scenario exports
- `/schemas/*.json` — Google Sheets schema contracts
- `/diagrams/mermaid/*.mmd` — Canonical, diffable flow diagrams
- `/decisions/ADR-*.md` — Architecture Decision Records

### Enforcement Rules
- Any logic change requires updating **all applicable artifacts** above.
- Mermaid diagrams must exactly match the Make blueprint and module names.
- ADRs are required for any semantic or lifecycle decision (e.g., status meaning).
- Chat history is **not** an authoritative source.

### Status Semantics
Status meanings are defined in:
- `decisions/ADR-001-status-semantics.md`

Any deviation requires a new ADR.

---

### Make UI — Functions & Operators Reference (CURRENT UI)
- **String concatenation:** Use **adjacent tokens** or the **`&` operator** (General → Operators). `concat()` is **not present** in the current picker UI.
- **ISO week stamp:** `formatDate(now; "GGGG-'W'WW")` (ISO week; Monday start). Use as the basis for `week_key`.

---

## 3. V2 Data Model (LOCKED)

### 3.1 Google Sheets (V2 Workbook)
Workbook: `BW_AI_Social_Agent_V2`

Core tabs:
- `BW_SETTINGS`
- `BW_PLANNER_SETTINGS`
- `BW_ENUMS`
- `BW_SOURCES_SHOWS`
- `BW_SOURCES_EVENTS`
- `BW_SOURCES_RELEASES`
- `BW_CONTENT_QUEUE`

### 3.2 BW_CONTENT_QUEUE — Entity Field Contract (LOCKED)
- Parent rows only created by Weekly Planner.
- Child rows are created later by Draft Generator.
- Parent/child linkage uses:
  - `row_type` = `parent` or `child`
  - `parent_row_id` only on child rows
- `week_key` is the idempotency anchor:
  - `v2|<source_type>|<source_row_id>|<GGGG-Www>`

### 3.3 Status Semantics (LOCKED)
- Source sheets:
  - `active` = eligible for planning
- `BW_CONTENT_QUEUE` lifecycle:
  - `Idea` (initial parent + newly generated child)
  - `Needs Approval`
  - `Rejected` (must regenerate using `notes`)
  - `Approved`
  - `Scheduled`
  - `Posted`
  - `failed`

---

## Phase 0 — Observability + Secrets + GCP Setup (COMPLETE)

[UNCHANGED: Phase 0 content preserved exactly as previously documented in repo handoff.]

---

## Phase 1 — Weekly Planner V2

**Status:** Step 1.1 Design COMPLETE · Step 1.2 Build IN PROGRESS · Step 1.3 Test Gate PENDING  
**Dependency:** Phase 0 must complete before Phase 1 Step 1.2 begins.

### Step 1.1 — Design (COMPLETE)

#### Step 1.1A — Expected Behavior (LOCKED)
- Manual run only (scheduling disabled until later)
- Reads from source tabs (Shows, Events, Releases)
- Creates **parent rows only** in `BW_CONTENT_QUEUE`
- Uses `week_key` for deduplication
- Idempotent and safe to re-run

Explicitly does NOT:
- Generate copy
- Create child rows
- Call OpenAI
- Schedule or post content

#### Step 1.1B — Scenario + Module Inventory (LOCKED)
Scenario name: **`BW_V2__WeeklyPlanner`**

Modules (Design-level abstraction; build uses route-specific modules):
1) **Module 1 — Set Run Context** (run variables, timestamps)
2) **Module 2 — Load Settings (BW_SETTINGS)** (read-only)
3) **Module 3 — Load Enums (BW_ENUMS)** (read-only)
4) **Module 4 — Get Shows (BW_SOURCES_SHOWS)**
5) **Module 5 — Get Events (BW_SOURCES_EVENTS)**
6) **Module 6 — Get Releases (BW_SOURCES_RELEASES)**
7) **Module 7 — Iterator (Unified Sources)** (normalize fields)
8) **Module 8 — Compute `week_key`**
9) **Module 9 — Search Existing Parents (BW_CONTENT_QUEUE)** (row_type=parent + source_id + week_key)
10) **Module 10 — Router (Parent Exists?)**
11) **Module 11 — Create Parent Row (BW_CONTENT_QUEUE)** (writes parents only)
12) **Module 12 — End**

**Logging modules (added in Build once Phase 0 is complete):**
- `Logger: runs/start` at beginning
- `Logger: runs/end` at end (include counts and outcome)

#### Step 1.1C — Module-to-Module Data Contracts (LOCKED)
- Modules 4–6 output source rows only
- Module 7 guarantees normalized output; downstream is source-agnostic
- Module 8 guarantees stable `week_key`
- Module 9 only checks existence; downstream does not depend on parent contents
- Module 11 may only write parents and may not update existing rows

### Step 1.2 — Build (IN PROGRESS)

**Goal:** Weekly batch planner that reads sources (Shows + Releases implemented first), creates **idempotent parent rows** in `BW_CONTENT_QUEUE`, then later fans out children (Draft Generator handles child generation).

**Current Weekly Planner scenario modules (by name):**
1) **Set Run Context**
2) **Read Planner Settings**
3) **Aggregate Planner Settings**
4) **Initialize Planner Settings Map**

**Shows route (built + validated):**
5) **Read Shows Source** (Google Sheets → Search rows)
6) **Filter: Shows — status = active**
7) **Iterate Active Shows** (Flow Control → Iterator) — CONFIGURED + VALIDATED
8) **Set Current Show Context** (Tools → Set variable) — `current_show`
9) **Set Current Show ID** (Tools → Set variable) — `current_show_id`
10) **Set Current Show Date** (Tools → Set variable) — `current_show_date`
11) **Compute Week Key** (Tools → Set variable) — `week_key = v2|shows|<source_row_id>|<GGGG-Www>`
12) **Search Existing Parent Row** (Google Sheets → Search rows) — `BW_CONTENT_QUEUE` (row_type=parent AND source_row_id AND week_key)
13) **Filter: Parent Not Found** — total bundles == 0
14) **Create Parent Row — Shows** (Google Sheets → Add a row) — creates `parent` row with `Idea` status

**Releases route (built + validated):**
15) **Read Releases Source** (Google Sheets → Search rows)
16) **Filter: Releases — status = active**
17) **Iterate Active Releases** (Flow Control → Iterator) — CONFIGURED + VALIDATED
18) **Set Current Release Context** (Tools → Set variable) — `current_release`
19) **Set Current Release ID** (Tools → Set variable) — `current_release_id`
20) **Set Current Release Date** (Tools → Set variable) — `current_release_date`
21) **Compute Week Key — Releases** (Tools → Set variable) — `week_key = v2|releases|<source_row_id>|<GGGG-Www>`
22) **Search Existing Parent Row — Releases** (Google Sheets → Search rows) — `BW_CONTENT_QUEUE` (row_type=parent AND source_row_id AND week_key)
23) **Filter: Parent Not Found — Releases** — total bundles == 0
24) **Create Parent Row — Releases** (Google Sheets → Add a row) — creates `parent` row with `Idea` status

**Validated behaviors (Shows + Releases slices):**
- Iterator yields 1 bundle per active show
- Parent creation happens only when no parent exists for the same show + week_key
- Re-run is idempotent (2nd run does not create a new parent row)
- Duplicate parent rows created during initial mis-mapping were cleaned up manually

- Iterator yields 1 bundle per active release
- Release parent creation happens only when no parent exists for the same release + week_key
- Re-run is idempotent for releases (2nd run does not create a new release parent row)

**Execution discoveries captured (locked):**
- Make UI does not expose `concat()` in the functions picker; use adjacency or `&`.
- Filters are lost if you delete the module connection they live on; avoid throwaway placeholders.

#### Weekly Planner V2 — Module Inventory (Reference)
A full, updatable inventory is also provided as: `BW_V2_WeeklyPlanner_ModuleInventory_v1.15.xlsx`

| Make ID | Module Name | App/Type | Key Output(s) | Notes |
|---:|---|---|---|---|
| 1 | Set Run Context | Tools — Set variable | run_context | Sets per-run identifiers (locked pattern) |
| 2 | Read Planner Settings | Google Sheets — Search rows | settings rows | Reads `BW_SETTINGS_PLANNER` |
| 3 | Aggregate Planner Settings | Aggregator | settings array | Aggregates settings bundles |
| 5 | Initialize Planner Settings Map | Tools — Set variable | planner_settings | Normalized key/value map |
| 8 | Read Shows Source | Google Sheets — Search rows | show rows | Reads `BW_SOURCES_SHOWS` |
| — | Filter: Shows — status = active | Route filter | — | status == "active" |
| 9 | Iterate Active Shows | Iterator | per-show bundle | Configured and validated |
| 10 | Set Current Show Context | Tools — Set variable | current_show | Stores show bundle |
| 11 | Set Current Show ID | Tools — Set variable | current_show_id | Maps row_id (A) from Read Shows Source |
| 12 | Set Current Show Date | Tools — Set variable | current_show_date | Maps event_date (B) from Read Shows Source |
| 14 | Compute Week Key | Tools — Set variable | week_key | `v2|shows|<id>|<GGGG-Www>` |
| 15 | Search Existing Parent Row | Google Sheets — Search rows | 0+ parent rows | Filters by row_type=parent AND source_row_id AND week_key |
| — | Filter: Parent Not Found | Route filter | — | Total bundles == 0 |
| 17 | Create Parent Row — Shows | Google Sheets — Add a row | new parent row | Creates BW_CONTENT_QUEUE parent row (Idea) |
| 18 | Read Releases Source | Google Sheets — Search rows | release rows | Reads `BW_SOURCES_RELEASES` |
| — | Filter: Releases — status = active | Route filter | — | status == "active" |
| 19 | Iterate Active Releases | Iterator | per-release bundle | Configured and validated |
| 20 | Set Current Release Context | Tools — Set variable | current_release | Stores release bundle |
| 21 | Set Current Release ID | Tools — Set variable | current_release_id | Maps row_id (A) from Read Releases Source |
| 22 | Set Current Release Date | Tools — Set variable | current_release_date | Maps release_date (B) from Read Releases Source |
| 23 | Compute Week Key — Releases | Tools — Set variable | week_key | `v2|releases|<id>|<GGGG-Www>` |
| 24 | Search Existing Parent Row — Releases | Google Sheets — Search rows | 0+ parent rows | Filters by row_type=parent AND source_row_id AND week_key |
| — | Filter: Parent Not Found — Releases | Route filter | — | Total bundles == 0 |
| 27 | Create Parent Row — Releases | Google Sheets — Add a row | new parent row | Creates BW_CONTENT_QUEUE parent row (Idea) |

#### Resume Point (AUTHORITATIVE)
**Phase 1 → Step 1.2 — Build**
- Shows and Releases parent creation paths are built and validated through:
  - **Create Parent Row — Shows**
  - **Create Parent Row — Releases**
- Next micro-step begins immediately **downstream of Create Parent Row — Releases** per the blueprint.

### Step 1.3 — Test Gate (PENDING)
Must prove:
- Parent rows created correctly
- Deduplication via `week_key` works
- Re-runs are idempotent
- No child rows created
- Firestore logging shows correct run start/end data (once logging enabled in this scenario)
- All failures are surfaced in run logs (Phase 0 gateway)

---

## Phase 2 — Draft Generator V2 (DESIGN LOCKED · BUILD PENDING)

[UNCHANGED: Phase 2 content preserved exactly as previously documented in repo handoff.]

---

## Phase 3 — Scheduler/Posting V2 (DESIGN LOCKED · BUILD PENDING)

[UNCHANGED: Phase 3 content preserved exactly as previously documented in repo handoff.]

---

## Phase 4 — Video Workload (DESIGNED-IN · DISABLED)

[UNCHANGED: Phase 4 content preserved exactly as previously documented in repo handoff.]

---

## Phase 5 — Housekeeping + Archive (DESIGN LOCKED · BUILD PENDING)

[UNCHANGED: Phase 5 content preserved exactly as previously documented in repo handoff.]

---

## 9. Change Log
### v1.18 — Phase 1 Step 1.2 Updated to Match Blueprint (Releases Parent Complete) + Manifest Rule
**WHEN:** Phase 1 → Step 1.2 (Build) — post-validation of Create Parent Row — Releases
**WHY:** Align handoff current-state with the latest Weekly Planner blueprint and prevent resume-point drift.
**FROM → TO:**
- FROM: Step 1.2 documented only the Shows parent slice; Releases slice absent; resume implied at Releases ingestion start.
- TO: Step 1.2 documents Shows + Releases parent slices through parent creation; explicit Resume Point added downstream of Create Parent Row — Releases.
- FROM: No manifest concept for completeness checks.
- TO: Added HANDOFF_MANIFEST requirement (to be created) for required sections/artifacts verification.

### v1.17 — Mermaid-Only Diagram Enforcement
**WHEN:** Phase 1 / Step 1.2 (governance alignment)
**WHY:** Remove draw.io dependency to eliminate tooling friction while retaining enforceable, diffable diagrams.
**FROM → TO:**
- FROM: Mermaid + draw.io both enforced
- TO: Mermaid enforced; draw.io optional/non-enforced

### v1.16 — Governance Enforcement Added
**WHEN:** Phase 1 / Step 1.2 (mid-step pause)
**WHY:** Prevent semantic drift and undocumented logic changes.
**FROM → TO:**
- FROM: Handoff + blueprint only
- TO: Handoff + blueprint + schemas + Mermaid + ADR enforcement

### v1.15 — 2026-02-05
- **WHEN:** Phase 1 → Step 1.2 (Build) → Micro-steps 1.2.3-I through 1.2.3-X.
- **WHY:** Complete the Shows “parent row” slice end-to-end with idempotency and document Make UI realities discovered during execution.
- **FROM → TO:**
  - FROM: `Iterate Active Shows` module exists but is not configured.
  - TO: `Iterate Active Shows` configured and validated (1 bundle per active show).
  - FROM: No stable per-show context variables.
  - TO: `current_show`, `current_show_id`, `current_show_date` variables created and validated.
  - FROM: No V2 parent idempotency enforcement for Weekly Planner.
  - TO: Composite `week_key` computed and used to enforce idempotency.
  - FROM: Parent row creation created duplicates.
  - TO: Parent idempotency validated; duplicates cleaned.

[END OF DOCUMENT]
