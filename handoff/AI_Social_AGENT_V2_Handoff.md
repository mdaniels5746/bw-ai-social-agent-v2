# Beggars Whisky — AI Social Agent V2 Handoff (MASTER PLAN)

**Document:** AI_Social_AGENT_V2_Handoff_v1.15  
**Status:** Phase 0 (Design + Build + Test Gate COMPLETE) · Phase 1 (Design complete · Build IN PROGRESS)
**Date:** 2026-02-05

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

---

- **Handoff update documentation rule:** Any design/process change must be documented with **WHEN** [phase/step/micro-step] and **WHY** [reason], and the **Change Log** must include **FROM → TO** details.

- Beginner-safe micro-steps only
- Manual execution only during development
- No jumping between phases or major steps
- Strict JSON discipline for AI outputs
- Credit conservation enforced
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
- **ISO week stamp:** `formatDate(now; "GGGG-[W]WW")`
- **Date formatting & parsing:** Use Date and time functions (e.g., `formatDate`, `parseDate`, `addDays`).
- **Filters:** A filter is bound to the connection **between modules**. If a connected module is deleted/replaced, the filter must be recreated.

### Scenario Build Rule Update (LOCKED)
- Do **not** create throwaway placeholder modules to “hold a filter.” If a filter needs an anchor, add the **real downstream module type** and leave it minimally configured until ready.


## 3. Status Semantics (LOCKED)

`BW_CONTENT_QUEUE.status` meanings:
- **Idea** – Parent row exists; requires first-pass generation
- **Needs approval** – Draft(s) generated; awaiting human approval
- **Rejected** – Human rejected; regeneration required
- **Approved** – Approved for scheduling
- **Scheduled** – Sent to scheduler
- **Posted** – Successfully posted
- **failed** – Scheduling/posting failed; manual intervention required

`Drafted` is deprecated and must not be used.

---

## 4. V2 Workbook Strategy (LOCKED)

### 4.1 Workbook
**Name:** `BW_AI_Social_Agent_V2`

V2 uses a separate Google Sheet workbook. V1 sheets remain untouched.

### 4.2 Tabs (LOCKED)
- **BW_CONTENT_QUEUE** (core; parent + child rows)
- **BW_CONTENT_QUEUE_ARCHIVE** (archive; built in Phase 5)
- **BW_SOURCES_SHOWS**
- **BW_SOURCES_EVENTS**
- **BW_SOURCES_RELEASES**
- **BW_SETTINGS**
- **BW_PLANNER_SETTINGS** (planner constants; cadence/windows/approval/hashtags/forbidden phrases)
- **BW_ENUMS**
- **BW_LOGS** (reserved; non-authoritative)

### 4.3 Read / Write Permissions (LOCKED)

### 4.4 Sheet UX Rules (LOCKED)
- Freeze **row 1** (header row) immediately after adding headers on every tab.
- Header names are authoritative; automations rely on them.

**BW_CONTENT_QUEUE**
- Weekly Planner V2: read + write (parents only)
- Draft Generator V2: read + write (children + parent status)
- Scheduler (future): lifecycle fields only
- Video Workload (future): child asset fields only

**Source Tabs**
- Weekly Planner V2: read-only
- All other scenarios: no access

**Settings / Enums**
- Read by automation
- Written by humans only

---

## 5. Core Identifiers (LOCKED)

### 5.1 `week_key`
`v2|<source_type>|<source_id>|<YYYY-Www>`  
- ISO week (Monday start)
- Used only for Weekly Planner deduplication

### 5.2 `post_id`
`BW2-<source_type>-<source_id>-<YYYYMMDD>`  
- Generated once at parent creation
- Stable forever
- Shared by parent + children

### 5.3 Row Linkage
- `row_id`: unique per row
- `row_type`: `parent | child`
- `parent_row_id`: empty for parent; required for child
- One-level hierarchy only (no grandchildren)

---

## 6. End-to-End System Flow (CURRENT PLAN)

1) **Weekly Planner V2** reads sources and creates **parent rows** in `BW_CONTENT_QUEUE` (status `Idea`).  
2) **Draft Generator V2** fans out **child rows** per platform for each parent row, writes drafts, and sets parent to `Needs approval`.  
3) Human reviews and sets child/parent statuses to `Approved` or `Rejected`.  
4) **Scheduler V2** pushes approved child rows to **ContentStudio** and updates statuses to `Scheduled` / `Posted` / `failed`.  
5) **Housekeeping V2** archives posted/failed rows older than threshold to `BW_CONTENT_QUEUE_ARCHIVE`.  
6) **Firestore** stores run logs and posting receipts with TTL retention.  
7) Future: **Video Workload** runs when enabled for `requires_video=true` and blocks scheduling until complete.

---

# PHASES (MASTER PLAN)

## Phase 0 — Firestore Observability Layer (Cloud Run + Firestore) (NEW)

**Purpose:** Provide reliable, queryable operational logging and posting receipts from day one, with TTL retention.

### Step 0.1 — Design (COMPLETE)
**Decisions locked:**
- Hosting: **Cloud Run** in same GCP project as Firestore
- Firestore retention: `runs` = 90 days, `posting_receipts` = 180 days
- TTL field name: `expireAt`
- Logging level switch: `BW_SETTINGS.logging_level` = `minimal|normal|verbose`
- Do **not** store full generated copy in Firestore logs (store references + metadata only)

**Additional decisions locked (this session):**
- GCP Project ID: `bw-social-agent-v2`
- Gateway auth header: `Authorization: Bearer <BW_V2_SECRET>`
- Shared secret key name (Make Data Store record id): `BW_V2_SECRET` [stored in `BW_V2_SECRETS`]

### Secrets Management (Authoritative)

- **Make Storage:** Data Store `BW_V2_SECRETS`
- **Data Structure:** `BW_V2_SECRETS_KV`
  - `key` (Text, required)
  - `value` (Text, required)
- **Secret Record:**
  - Record ID: `BW_V2_SECRET`
  - `key`: `BW_V2_SECRET`
  - `value`: Bearer secret string
- **Access Pattern:** Use *Data store → Get a record* with key `BW_V2_SECRET`
- **Restrictions:**
  - Secrets must never be hardcoded
  - Secrets must never be stored in Sheets

- RUN model: **one RUN per parent scenario execution**; children do not create RUNs
- RUN outcomes: `success | partial_success | failed | canceled`
- RUN rollups include `items_rejected` (count only)
- `run_id` is generated by the gateway and returned from `POST /runs/start`
- Gateway sets canonical timestamps; Make references returned values
- `platform` is the **exact placement channel** enum (e.g., `instagram_reel`, `youtube_shorts`) to support future video enablement without schema changes

**Artifacts to define:**
1) **GCP Project ID**: `bw-social-agent-v2`
2) **Firestore (Native)** enabled
3) **Cloud Run service**: `bw-v2-logger-gw`
4) Service account: `bw-v2-logger-sa` (least privilege)
5) Collections + schemas (see below)
6) Gateway endpoints and request/response contracts

### Step 0.2 — Build (COMPLETE)
**Build order:**
1) Create new GCP project + enable billing
2) Enable Firestore (Native)
3) Create service account `bw-v2-logger-sa` and grant least privilege for Firestore writes
4) Deploy Cloud Run service `bw-v2-logger-gw`
5) Configure TTL policies on:
   - `runs.expireAt`
   - `posting_receipts.expireAt`
6) Add Make scenario modules (Phase 1/2/3/5) to call gateway (no schedules)

#### Step 0.2 Build — Execution Log (2026-02-03)
**Goal:** Stand up the clean V2 GCP foundation (new project + Firestore + required APIs). No schedules.

**Micro-step 0.2.0 — Decommission legacy V1 project (COMPLETED)**
- Verified active legacy project:
  - Project ID: `bw-social-agent`
  - Project number: `338379609909`
- Action taken: **Project shut down** (delete scheduled)
  - Console message: Project "BW-Social-Agent" shut down and scheduled to be deleted after **Mar 5, 2026**

**Micro-step 0.2.1 — Create V2 GCP project (COMPLETED)**
- Project name: `bw-social-agent-v2`
- Project ID: `bw-social-agent-v2`
- Project number: `1008197508830`
- Billing: **enabled** (required for Cloud Run)

**Micro-step 0.2.2 — Create Firestore database (COMPLETED)**
- Database ID: `(default)`
- Edition: Standard
- Mode: Firestore Native
- Security rules preset: Restrictive
- Location: Region `us-central1 (Iowa)`
- Creation time (UTC-6): Feb 3, 2026, 4:41:33 PM
- Scheduled backups: Disabled
- Encryption: Google-managed

**Micro-step 0.2.3 — Enable required APIs (COMPLETED)**
- Google Cloud Firestore API: enabled
- Cloud Run Admin API: enabled (billing enabled as prerequisite)
- Cloud Logging API: already enabled
- Cloud Build API: enabled

**Micro-step 0.2.4 — Create service account + Firestore least-privilege (COMPLETED)**  
- Cloud Shell opened (project context confirmed: `bw-social-agent-v2`).  
- Created service account:
  - Name: `bw-v2-logger-sa`
  - Email: `bw-v2-logger-sa@bw-social-agent-v2.iam.gserviceaccount.com`
- Granted Firestore write permissions [Native mode uses Datastore IAM]:
  - Role: `roles/datastore.user`
  - Binding verified via `gcloud projects get-iam-policy` [filtered on the service account].

**Micro-step 0.2.5 — Deploy Cloud Run gateway skeleton (COMPLETED)**  
- Created Cloud Run service:
  - Service name: `bw-v2-logger-gw`
  - Region: `us-central1`
  - Runtime identity: `bw-v2-logger-sa@bw-social-agent-v2.iam.gserviceaccount.com`
  - Public access: `--allow-unauthenticated` [temporary; auth enforced at app layer via Bearer secret]
- Source deploy prerequisites:
  - Artifact Registry repository auto-created: `cloud-run-source-deploy` [region: `us-central1`]
- Gateway code scaffold created in Cloud Shell folder `~/bw-v2-logger-gw`:
  - `main.py` [Flask app + `/health`]
  - `requirements.txt` [Flask, gunicorn, google-cloud-firestore]
  - `Dockerfile` [gunicorn entrypoint]
- Deployment validated:
  - Service URL: `https://bw-v2-logger-gw-t45xvarx7q-uc.a.run.app`
  - Health check: `GET /health` returned `{"ok":true,"project":"bw-social-agent-v2"}`

**Micro-step 0.2.6 — Enable Bearer auth + implement `POST /runs/start` (COMPLETED)**  
- Implemented gateway auth [LOCKED contract]:
  - Header: `Authorization: Bearer <BW_V2_SECRET>`
  - Secret provided via Cloud Run env var: `BW_V2_SECRET` [value stored externally; do not write to repo/handoff]
- Set env var on Cloud Run:
  - `BW_V2_SECRET` [generated via `openssl rand -hex 32` and saved locally]
- Implemented endpoint:
  - `POST /runs/start`
  - Validates required fields: `scenario_parent`, `environment`
  - Generates `run_id` [doc id] and writes to Firestore `runs/{run_id}` with:
    - `run_id`, `scenario_parent`, `environment`, `started_at` [timestamp], `run_context` [object]
- Live validation [Cloud Run]:
  - Request succeeded and returned:
    - `{"run_id":"run_cc74b04d2033","started_at":"2026-02-04T00:18:47.829Z"}`
  - Confirms end-to-end: Cloud Run → Firestore write with SA identity + Bearer auth.

**Micro-step 0.2.7 — Implement `PATCH /runs/end` + rotate secret if exposed (COMPLETED)**  
- Updated `main.py` to add endpoint:
  - `PATCH /runs/end`
  - Required fields: `run_id`, `outcome`
  - Optional: `rollup` (object), `error_summary` (string)
- Gateway behavior:
  - Fetches `runs/{run_id}` and reads `started_at`
  - Sets `ended_at` (gateway UTC timestamp)
  - Computes `duration_ms` = `(ended_at - started_at) * 1000` (integer)
  - Writes: `ended_at`, `duration_ms`, `outcome`, `rollup`, `error_summary?`
  - Writes TTL: `expireAt = ended_at + 90 days`
- Deployed updated service to Cloud Run (source deploy).
- Live validation:
  - `PATCH /runs/end` succeeded for an existing run id and returned:
    - `{"ok":true,"run_id":"run_cc74b04d2033","duration_ms":<int>}`
- **Security incident handling (procedure validated):**
  - If `BW_V2_SECRET` is pasted/shared anywhere, treat it as compromised.
  - Rotate immediately by generating a new secret and updating Cloud Run env var `BW_V2_SECRET`.
  - Validate rotation by confirming the old secret returns `401` and the new secret returns `200`.

**Micro-step 0.2.8 — Validate `runs` TTL field `expireAt` (90 days) (COMPLETED)**  
- Created a new run via `POST /runs/start`, then closed via `PATCH /runs/end`.
- Read back the Firestore document using Firestore REST API + `gcloud auth print-access-token`.
- Confirmed fields present on the live doc:
  - `started_at`, `ended_at`, `duration_ms`, `outcome`, `rollup`
  - **`expireAt`** set to **ended_at + 90 days** (exact timestamp offset)
- Note: TTL deletion is handled by Firestore TTL and may not delete immediately.

**Micro-step 0.2.9 — Implement `POST /posting_receipts` + validate TTL (180 days) (COMPLETED)**  
- Updated `main.py` to add endpoint:
  - `POST /posting_receipts`
  - Required fields: `post_id`, `platform`, `status`
  - Optional pass-through fields:
    - `channel`, `external_post_id`, `scheduled_for`, `posted_at`, `url`, `error_summary`, `meta` (object)
- Gateway behavior:
  - Creates `receipt_id` (doc id) like `rcpt_<12hex>`
  - Sets `created_at` (gateway UTC timestamp)
  - Writes TTL: `expireAt = created_at + 180 days`
  - Writes document to: `posting_receipts/{receipt_id}`
- Live validation:
  - Created a `posting_receipts` doc and read it back via Firestore REST API.
  - Confirmed:
    - `created_at` present
    - **`expireAt`** set to **created_at + 180 days**

**Micro-step 0.2.X — Cloud Run URL canonicalization (COMPLETED)**  
- Service name: `bw-v2-logger-gw`
- Region: `us-central1`
- Canonical Service URL (record + use in Make):  
  - `https://bw-v2-logger-gw-1008197508830.us-central1.run.app`
- Earlier deploy URL (historical; do not rely on it going forward):  
  - `https://bw-v2-logger-gw-t45xvarx7q-uc.a.run.app`

### Step 0.3 — Test Gate (COMPLETE)
**Validated behaviors (live Cloud Run):**
- Public liveness:
  - `GET /health` returns HTTP `200` without auth.
- Auth enforcement:
  - Missing Authorization on `POST /runs/start` → HTTP `401`
  - Missing Authorization on `POST /posting_receipts` → HTTP `401`
- Input validation:
  - Missing required fields on `POST /runs/start` (e.g., omit `environment`) → HTTP `400`
  - Missing required fields on `POST /posting_receipts` (e.g., omit `status`) → HTTP `400`
- Not-found behavior:
  - Unknown `run_id` on `PATCH /runs/end` → HTTP `404`
- End-to-end Firestore writes:
  - `POST /runs/start` writes `runs/{run_id}` and returns `run_id`, `started_at` (UTC `Z` format).
  - `PATCH /runs/end` updates-in-place and writes: `ended_at`, `duration_ms`, `outcome`, `rollup`, `expireAt` (+90d).
  - `POST /posting_receipts` writes: `created_at`, `expireAt` (+180d) and returns `receipt_id`.

**Hardening decision (LOCKED for now):**
- Keep Cloud Run `--allow-unauthenticated` for Make simplicity.
- Security boundary is enforced at app layer via `Authorization: Bearer <BW_V2_SECRET>`.
- If/when Make is ready for Google-authenticated calls, revisit ingress policy as an optional upgrade.

**TTL timing note (LOCKED):**
- Firestore TTL deletions are not immediate; deletions can take time after `expireAt` passes.

### Phase 0 Firestore Collections (LOCKED)
#### Collection: `runs`
Document fields (minimum, V2):
- `run_id` (string) — **generated by gateway** (also used as doc id or stored as field)
- `scenario_parent` (string) — e.g. `weekly_planner_parent`, `draft_generator_parent`, `scheduler_parent`, `housekeeping_parent`
- `environment` (string) — e.g. `prod` (even during manual runs)
- `started_at` (timestamp) — **gateway timestamp**
- `ended_at` (timestamp, optional until closed) — **gateway timestamp**
- `duration_ms` (number, optional until closed)
- `outcome` (string): `success | partial_success | failed | canceled`
- `error_summary` (string, optional) — short summary only
- `rollup` (map):
  - `items_total` (number)
  - `items_succeeded` (number)
  - `items_failed` (number)
  - `items_rejected` (number) — **count only**
  - `child_scenarios_triggered` (number)
- `run_context` (map, optional) — small JSON for debugging (batch id, date range, manual/auto, etc.)
- `version` (string, optional) — handoff/system version
- `expireAt` (timestamp): `ended_at + 90 days`


#### Collection: `posting_receipts`
Document fields (minimum, V2):
- `receipt_id` (string) — generated by gateway (also used as doc id or stored as field)
- `run_id` (string) — parent RUN id
- `post_id` (string)
- `platform` (string) — **exact placement channel** (e.g., `instagram_feed`, `instagram_reel`, `facebook_feed`, `facebook_reel`, `tiktok`, `youtube_shorts`)
- `child_scenario` (string) — e.g. `draft_generator_child`, `scheduler_child`
- `status` (string): `succeeded | failed`
- `attempt` (number) — 1-based attempt counter
- `notes` (string, optional) — short human-readable note (no large payloads)
- `timestamps` (map):
  - `created_at` (timestamp) — **gateway timestamp**
  - `scheduled_at` (timestamp, optional) — gateway timestamp (when applicable)
  - `posted_at` (timestamp, optional) — gateway timestamp (when applicable)
- `payload` (map, optional) — small structured metadata (e.g., `draft_id`, `external_post_id`, etc.)
- `failure_reason` (string, optional)
- `expireAt` (timestamp): `timestamps.created_at + 180 days`


#### Collection: `video_jobs` (DESIGNED-IN, DISABLED)
- Defined in Phase 4 when video is enabled; do not build now.

### Phase 0 Cloud Run Gateway (LOCKED)
Service: `bw-v2-logger-gw`  
Auth: `Authorization: Bearer <BW_V2_SECRET>` (secret stored in Make Data Store `BW_V2_SECRETS`) + server-side IAM to Firestore

Endpoints (minimum):
- `POST /runs/start`
- `PATCH /runs/end`
- `POST /posting_receipts`

### Gateway Endpoint Contracts (LOCKED)

#### `POST /runs/start`
Request (Make → Gateway):
```json
{"scenario_parent":"draft_generator_parent","environment":"prod","run_context":{"batch_id":"2026-W06","source":"manual_run"}}
```
Response (Gateway → Make):
```json
{"run_id":"run_01HZXABC123","started_at":"2026-02-03T14:22:11.123Z"}
```

#### `PATCH /runs/end`
Request:
```json
{"run_id":"run_01HZXABC123","outcome":"partial_success","rollup":{"items_total":24,"items_succeeded":22,"items_failed":1,"items_rejected":1,"child_scenarios_triggered":24},"error_summary":"1 child failed: posting_receipt write timeout"}
```
Response:
```json
{"ok":true}
```

#### `POST /posting_receipts`
Request:
```json
{"run_id":"run_01HZXABC123","post_id":"BW2-SHOW-000123-20260203","platform":"instagram_reel","child_scenario":"draft_generator_child","status":"succeeded","attempt":1,"notes":"Generated copy + hashtags","payload":{"draft_id":"draft_abc123"}}
```
Response:
```json
{"receipt_id":"receipt_01HZYXYZ789","ok":true}
```

---

## Phase 1 — Weekly Planner V2

**Status:** Step 1.1 Design COMPLETE · Step 1.2 Build PENDING · Step 1.3 Test Gate PENDING  
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

Modules (Make):
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

**Goal:** Weekly batch planner that reads sources (Shows first), creates **idempotent parent rows** in `BW_CONTENT_QUEUE`, then later fans out children (Draft Generator handles child generation).

**Current Weekly Planner scenario modules (by name):**
1) **Set Run Context**
2) **Read Planner Settings**
3) **Aggregate Planner Settings**
4) **Initialize Planner Settings Map**
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

**Validated behaviors (Shows slice):**
- Iterator yields 1 bundle per active show
- Parent creation happens only when no parent exists for the same show + week_key
- Re-run is idempotent (2nd run does not create a new parent row)
- Duplicate parent rows created during initial mis-mapping were cleaned up manually

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
| — | Filter: Shows — status = active | Route filter | — | status (I) == "active" |
| 9 | Iterate Active Shows | Iterator | per-show bundle | Configured and validated |
| 10 | Set Current Show Context | Tools — Set variable | current_show | Stores show bundle |
| 11 | Set Current Show ID | Tools — Set variable | current_show_id | Maps row_id (A) from Read Shows Source |
| 12 | Set Current Show Date | Tools — Set variable | current_show_date | Maps event_date (B) from Read Shows Source |
| 14 | Compute Week Key | Tools — Set variable | week_key | `v2|shows|<id>|<GGGG-Www>` |
| 15 | Search Existing Parent Row | Google Sheets — Search rows | 0+ parent rows | Filters by row_type=parent AND source_row_id AND week_key |
| — | Filter: Parent Not Found | Route filter | — | Total bundles == 0 |
| 17 | Create Parent Row — Shows | Google Sheets — Add a row | new parent row | Creates BW_CONTENT_QUEUE parent row (Idea) |


### Step 1.3 — Test Gate (PENDING)
Must prove:
- Parent rows created correctly
- Deduplication via `week_key` works
- Re-runs are idempotent
- No child rows created
- Firestore run logging works (start/end, counts, outcome)

Planned schedule (DISABLED until explicitly enabled):
- Weekly Planner runs **Sunday 1:00 AM**

---

## Phase 2 — Draft Generator V2 (PLANNED)

**Purpose:** Create/update child rows per platform from parent rows, generate copy via OpenAI, enforce strict JSON outputs, update statuses for human approval.

Planned cadence (DISABLED until enabled): **Weekly batch** (Sunday after Weekly Planner; e.g., 1:10 AM).

### Step 2.1 — Design (NOT STARTED)
### Rejection & Regeneration Rules (LOCKED)

When a **parent or child post** is set to `Rejected`, the Draft Generator **must**:

- Read the human-entered **`notes` (Notes/Feedback) field** associated with the rejection
- Treat this field as **authoritative regeneration guidance**
- Explicitly incorporate the feedback into the next OpenAI prompt

Examples of valid regeneration feedback:
- "Needs to be punchier"
- "Did not include correct hashtags"
- "Logo placement missing"
- "Too long for platform"
- "Tone is off-brand"

Regeneration behavior:
- If `notes` includes asset requirements (e.g., logo/overlay), Draft Generator must set the child fields that flag asset needs (future video/asset workflow) but must NOT execute video workflow while disabled.

- Regeneration **updates existing child rows in place** (does not create duplicates)
- `regen_count` is incremented on each regeneration
- Parent status remains `Rejected` until all regenerated children are approved
- Regeneration never creates new parents

Failure to apply feedback is considered a **Draft Generator defect**.


Locked constraints:
- OpenAI API usage only in this phase
- Child fan-out only; never create parents
- Regeneration updates children in place by (parent_row_id + platform)
- `requires_video` designed-in but disabled

### Step 2.2 — Build (NOT STARTED)
Scenario name: **`BW_V2__DraftGenerator`** (planned)

Planned module inventory (names reserved now; details locked to future design):
1) Module 1 — Logger: runs/start
2) Module 2 — Load Settings (BW_SETTINGS)
3) Module 3 — Search Parents Needing Drafts (BW_CONTENT_QUEUE) (row_type=parent, status in Idea/Rejected; read `notes` for regen guidance when Rejected)
4) Module 4 — Iterator (Parents)
5) Module 5 — Parse platform_targets (Tools/JSON)
6) Module 6 — Iterator (Platforms)
7) Module 7 — Check Existing Child (BW_CONTENT_QUEUE) (parent_row_id + platform)
8) Module 8 — Build OpenAI Prompt (Text/JSON) (MUST incorporate `notes` when status=Rejected)
9) Module 9 — OpenAI: Create Chat Completion (OpenAI API)
10) Module 10 — Validate Strict JSON Output (JSON module + router fail path)
11) Module 11 — Upsert Child Row (BW_CONTENT_QUEUE) (create/update child in place; increment `regen_count` on regeneration)
12) Module 12 — Update Parent Status → Needs approval (BW_CONTENT_QUEUE)
13) Module 13 — Router: requires_video? (DISABLED path)
14) Module 14 — Logger: runs/end

### Step 2.3 — Test Gate (NOT STARTED)
Must prove:
- Children created/updated correctly
- Parent status transitions correct
- Regeneration updates child in place using `notes` as corrective guidance
- Strict JSON validation enforcement
- Firestore logs and counts correct
- Video path remains disabled

---

## Phase 3 — Scheduler / Posting V2 (PLANNED)

**Purpose:** Push approved child rows into ContentStudio and record receipts; update statuses.

### Step 3.1 — Design (NOT STARTED)
Constraints:
- No AI
- Reads Approved children; writes lifecycle fields only
- Writes posting receipts to Firestore via gateway

### Step 3.2 — Build (NOT STARTED)
Scenario name: **`BW_V2__Scheduler`** (planned)

Planned modules (names reserved now):
1) Logger: runs/start
2) Load Settings
3) Search Approved Children (BW_CONTENT_QUEUE) (row_type=child, status=Approved)
4) Iterator (Children)
5) Router (platform)
6) ContentStudio: Create/Schedule Post (integration method TBD)
7) Write posting_receipt (Cloud Run gateway)
8) Update Child Status → Scheduled / failed
9) Logger: runs/end

### Step 3.3 — Test Gate (NOT STARTED)
Must prove:
- Correct filtering and posting flow
- Receipts written
- Status updates correct
- Failure handling safe

Planned schedule (DISABLED until enabled): TBD

---

## Phase 4 — Video Workload V2 (DESIGNED-IN, DISABLED)

Purpose: When enabled, handle video asset generation/rendering with Desktop FFmpeg workflow and gate scheduling until complete.

Scenario name: **`BW_V2__VideoWorkload`** (planned)

Key rules:
- Triggered only when `requires_video=true`
- Disabled/hidden until explicitly approved
- Uses Firestore `video_jobs` collection (planned)

Steps:
- Step 4.1 Design (NOT STARTED)
- Step 4.2 Build (DISABLED)
- Step 4.3 Test Gate (DISABLED)

---

## Phase 5 — Housekeeping & Archival V2 (PLANNED)

Purpose: Keep primary queue small by archiving old Posted/failed items.

### Step 5.1 — Design (PLANNED)
Decisions locked:
- Archive tab: `BW_CONTENT_QUEUE_ARCHIVE` (same workbook)
- Archive threshold: **90 days**
- Manual-run first; schedule later

### Step 5.2 — Build (NOT STARTED)
Scenario name: **`BW_V2__Housekeeping`** (planned)

Planned modules (names reserved now):
1) Logger: runs/start
2) Load Settings (archive threshold)
3) Search Rows Eligible for Archive (BW_CONTENT_QUEUE) (status in Posted/failed and older than threshold)
4) Iterator (Rows)
5) Append Row to Archive (BW_CONTENT_QUEUE_ARCHIVE)
6) Delete Row from Primary Queue (BW_CONTENT_QUEUE) (default action)
7) Logger: runs/end

### Step 5.3 — Test Gate (NOT STARTED)
Must prove:
- Correct rows moved
- No loss (count matches)
- Archive is append-only
- Firestore logs correct

Planned schedule (DISABLED until enabled):
- Monthly (1st @ 2:00 AM)

---

## 7. Planned Scheduling Summary (DISABLED until explicitly enabled)
- Weekly Planner: Sunday 1:00 AM
- Draft Generator: Weekly batch Sunday ~1:10 AM
- Scheduler: TBD after Phase 3 design
- Housekeeping: Monthly (1st @ 2:00 AM)

---

## 8. Resume Point (AUTHORITATIVE)

**Current resume point:**  
> Phase 1 — Weekly Planner V2 → Step 1.2 Build (IN PROGRESS)

**Completed (since v1.14):**
- Configured `Iterate Active Shows` (Iterator) and validated 1 bundle per active show.
- Added show context variables:
  - `current_show` (full show row bundle)
  - `current_show_id` (row_id)
  - `current_show_date` (event_date)
- Implemented parent idempotency for Shows:
  - Compute composite `week_key` = `v2|shows|<source_row_id>|<GGGG-Www>`
  - Search for existing parent row in `BW_CONTENT_QUEUE`
  - Filter `Parent Not Found` (Total bundles == 0)
  - Create parent row (Shows) only when not found
  - Validated idempotency (2nd run does not create a new row)
  - Cleaned up duplicates created during earlier mis-mapping

**Next micro-step:**  
> Phase 1 → Step 1.2 Build — Micro-step 1.2.3-Y: Build Releases ingestion skeleton (Read Releases Source → Filter active → Iterate Active Releases), then apply the same parent idempotency pattern using the normalized entity contract.

---

## 9. Change Log
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
  - TO: Composite `week_key` + parent existence check + `Parent Not Found` filter + controlled parent creation in `BW_CONTENT_QUEUE`.
  - FROM: Assumed Make function `concat()` available.
  - TO: Documented current Make UI: string concatenation via adjacency or `&` operator; `concat()` not present in picker.
  - FROM: Placeholder module used to anchor filter (risk: deleting module removes filter).
  - TO: Governance rule: do not use throwaway placeholder modules; anchor filters using the real downstream module.
- **Artifacts added:**
  - Weekly Planner module inventory table (in handoff + spreadsheet) for fast reference and update discipline.

### v1.13 — 2026-02-04
- **WHEN:** Phase 1 → Step 1.2 (Build) → Micro-step 1.2.0 (Pre-flight) — secret storage setup.
- **WHY:** Make.com current UI does **not** support reliable global custom org variables; Data Store is the durable supported mechanism.
- **FROM → TO:**
  - FROM: Store `BW_V2_SECRET` as a Make global/org variable.
  - TO: Store `BW_V2_SECRET` as a **Make Data Store record** in `BW_V2_SECRETS` [structure `BW_V2_SECRETS_KV`].
- **Implementation note:** UI grid writes were unreliable; record creation was verified via a Make module write (Data store → Create a record) using record id `BW_V2_SECRET`.
- Added governance rule requiring all future handoff updates to document **WHEN** and **WHY**, and log **FROM → TO** changes.

### v1.12 — 2026-02-03
- Phase 0 Step 0.2 Build completed end-to-end:
  - `bw-v2-logger-sa` created and bound to `roles/datastore.user` for Firestore Native writes.
  - Cloud Run service `bw-v2-logger-gw` deployed (source deploy) in `us-central1` with runtime SA.
  - Implemented gateway endpoints: `/health`, `POST /runs/start`, `PATCH /runs/end`, `POST /posting_receipts`.
  - Implemented Bearer auth via `BW_V2_SECRET` (stored externally only) and validated secret rotation workflow.
  - Confirmed TTL fields on live docs:
    - `runs.expireAt = ended_at + 90 days`
    - `posting_receipts.expireAt = created_at + 180 days`
- Phase 0 Step 0.3 Test Gate completed:
  - Validated HTTP behaviors: 200/401/400/404 as expected.
  - Locked decision to keep `--allow-unauthenticated` and enforce security at app layer.
- Resume point advanced to Phase 1 Step 1.2 Build.

### v1.9 — 2026-02-03
- Phase 0 Step 0.2 Build updated: Micro-steps 0.2.4–0.2.6 completed.
- Created service account `bw-v2-logger-sa` and granted `roles/datastore.user` [least-privilege Firestore writes].
- Deployed Cloud Run gateway `bw-v2-logger-gw` [region: `us-central1`] using `bw-v2-logger-sa` as the runtime identity.
- Artifact Registry repo `cloud-run-source-deploy` auto-created in `us-central1` for source deploys.
- Gateway scaffold implemented and validated:
  - `GET /health` live check passed.
  - Bearer auth enabled via Cloud Run env var `BW_V2_SECRET` [value stored externally; not recorded here].
  - Implemented `POST /runs/start` and validated Firestore write with live curl test [returned `run_cc74b04d2033`].
- Updated authoritative resume point to Micro-step 0.2.7 (`PATCH /runs/end`).

### v1.8 — 2026-02-03
- Phase 0 Step 0.2 Build moved to IN PROGRESS.
- Decommissioned legacy V1 project `bw-social-agent` (shut down; deletion scheduled after Mar 5, 2026).
- Created V2 GCP project `bw-social-agent-v2` (project number `1008197508830`) and enabled billing.
- Created Firestore database `(default)` (Standard Edition, Native mode, Restrictive rules preset, Region `us-central1`).
- Enabled required APIs: Google Cloud Firestore API, Cloud Run Admin API, Cloud Build API (Cloud Logging API already enabled).
- Updated authoritative resume point to Micro-step 0.2.4.

### v1.7 — 2026-02-03
- Phase 0 Step 0.1 Design marked COMPLETE.
- Locked GCP Project ID: `bw-social-agent-v2`.
- Locked gateway auth: `Authorization: Bearer <BW_V2_SECRET>`; locked Make secret key: `BW_V2_SECRET` stored in Data Store `BW_V2_SECRETS`.
- Locked RUN model (parent-only), RUN outcomes (`success|partial_success|failed|canceled`), and required rollups (including `items_rejected` count).
- Updated Firestore schemas for `runs` and `posting_receipts` to match V2 design + TTL.
- Added locked gateway endpoint contracts for `POST /runs/start`, `PATCH /runs/end`, and `POST /posting_receipts`.

### v1.5 — 2026-02-03
- Updated Draft Generator regeneration requirements: `notes` is the authoritative regen brief when status=Rejected; module inventory now explicitly reads/incorporates notes and increments `regen_count` on regeneration.

### v1.4 — 2026-02-03
- Updated handoff into full V2 master plan across all phases
- Added Phase 0 Firestore Observability Layer (Cloud Run + Firestore)
- Added TTL retention rules (runs 90d, receipts 180d) using `expireAt`
- Added Phase 5 Housekeeping/Archival plan (archive tab + monthly schedule disabled)
- Added planned scheduling model (weekly batch) with schedules disabled by default
- Updated resume point to Phase 0 Step 0.1 Design
