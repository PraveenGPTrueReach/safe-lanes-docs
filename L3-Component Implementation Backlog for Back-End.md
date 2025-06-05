## L3-IB-BE: Component Implementation Backlog for BE

This document provides a granular list of implementation items for the Back-End (Nest.js \+ MySQL) of the SafeLanes Rest Hours solution. Each backlog item references key requirements from L3-FRS-BE (functional), L3-TSD (technology constraints), L3-LLD-BE (low-level design), L3-NFRS-BE (non-functional), and L3-KD-BE (key decisions), alongside acceptance criteria, dependencies, priority, and estimates. These items are intended as atomic tasks or user stories that can be directly assigned to developers.

---

### 1\. Project Initialization & Configuration

#### 1.1 Nest.js Project Setup

• Description: Create the Nest.js project scaffold with recommended module structure (e.g., RestHoursModule, TasksModule, AuthModule, SyncModule). Configure TypeORM (or equivalent) for MySQL connectivity.  
• Technical Requirements:

- Use environment variables to switch between vessel (offline) and office (online) modes (L3-LLD-BE §2).  
- Include initial database connection config.  
- Additionally, ensure alignment with L3-TSD for overall technology constraints (e.g., Nest.js version, MySQL version, and no Docker usage).  
  • Acceptance Criteria:  
- Nest.js app compiles and runs with the selected build scripts.  
- Docker is not used; PM2 \+ Nginx approach is documented.  
  • Traceability: L3-FRS-BE §1 (Intro), L3-LLD-BE §6 (Database Module), L3-KD-BE (“Project Structure Simplification”).  
  • Dependencies: None.  
  • Effort Estimate: 2 points  
  • Priority: High

#### 1.2 Environment Configuration for Vessel vs. Office

• Description: Implement environment-based config to differentiate vessel (offline) from office (online) settings (DB\_HOST, mode=‘vessel’ vs. ‘office’).  
• Technical Requirements:

- Distinct .env files or environment variables for local vessel deployment vs. office server.  
- • Acceptance Criteria:  
- The application must run seamlessly in both modes, verifying relevant feature toggles.  
- Documented instructions for administrators to switch or set environment variables.  
  • Traceability: L3-FRS-BE §2.2, L3-LLD-BE §12 (Deployment), Clarification \#2 (Offline behavior).  
  • Dependencies: Item 1.1 completed (project scaffold).  
  • Effort Estimate: 1 point  
  • Priority: High

---

### 2\. Data Model & Database Entities

#### 2.1 Create Core Entities (RestHours, Tasks, AuditLog, OfflineSessions)

• Description: Define TypeORM entities/tables for daily rest-hour records (48 blocks), tasks (single attachment path), audit logs,.  
• Technical Requirements:

- Store daily rest-hour blocks as short enum strings (e.g., “D,” “W,” “A,” with any unrecognized entry treated as rest) instead of integers (Clarification \#3).  
- Add fields: isDeleted (soft-delete), updatedAt, vesselId columns, and rule references for compliance checks.  
- Single column for tasks.attachmentPath.  
  • Acceptance Criteria:  
- DB schema matches the design: resthours, tasks, audit\_log,, rulesets.  
- Migrations or schema scripts are available for both vessel and office.  
  • Traceability:  
- L3-FRS-BE §4 (Data Requirements),  
- L3-LLD-BE §5 (ER diagram),  
- L3-NFRS-BE §4 (Security & Data Integrity),  
- L3-KD-BE (“Daily Record Structure,” “Single Attachment”).  
  • Dependencies: Item 1.1 completed.  
  • Effort Estimate: 3 points  
  • Priority: High

#### 2.2 Implement Rule Sets & Versions Table

• Description: Implement the “rule\_sets” table for MLC/STCW/OPA parameters, with versioning.  
• Technical Requirements:

- For each ruleType (MLC, STCW, OPA), store thresholds in JSON.  
- No retroactive re-check of older data (forward-only rules).  
  • Acceptance Criteria:  
- Ability to insert new rule set entries, referencing version, publishedAt.  
- Vessel can download changes in future tasks (Sync logic, see item 7.1).  
  • Traceability: L3-FRS-BE §2.5, L3-LLD-BE §6.3, L3-KD-BE (“Forward-Only Rule Updates”).  
  • Dependencies: Item 2.1 completed.  
  • Effort Estimate: 2 points  
  • Priority: Medium

---

### 3\. Authentication & Access Control

#### 3.1 JWT & Role Validation (Online)

• Description: Implement standard JWT-based role validation  
• Technical Requirements:

- Validate tokens from SafeLanes Identity Service.  
- Enforce roles: VesselUser, VesselAdmin, OfficeUser, OfficeAdmin, ExternalReadOnly, etc.  
  • Acceptance Criteria:  
- All incoming requests with a valid JWT are authorized if the correct claims/roles are present.  
- Unauthorized or invalid token requests must return 401/403 errors.  
  • Traceability: L3-FRS-BE §2.1, L3-LLD-BE §3 (AuthService), L3-NFRS-BE §4.1.  
  • Dependencies: Items 1.1, 1.2 completed.  
  • Effort Estimate: 2 points  
  • Priority: High

---

### 4\. Daily Rest-Hour Records (CRUD & Compliance)

#### 4.1 Create/Update Daily Rest-Hours

• Description: Implement endpoints: POST /resthours, PUT /resthours/:id to create or update a single daily record containing 48 half-hour blocks.  
• Technical Requirements:

- Entire day is replaced on update (no partial merges), referencing updatedAt.  
- Validate the presence of all 48 blocks (each as short enum strings).  
- For each change, insert an audit\_log record capturing oldValue/newValue (if any).  
  • Acceptance Criteria:  
- A new daily record can be created, storing block\_1..block\_48 with default or user-provided states.  
- System rejects incomplete or invalid requests (e.g., missing crewId/date).  
- Updates overwrite the entire day’s record if updatedAt is more recent.  
  • Traceability: L3-FRS-BE §2.3, §4.1, L3-LLD-BE §3, L3-KD-BE (“Day-Level Edit,” “Single Last-Write-Wins”), Clarification \#2.  
  • Dependencies: Items 2.1, 3.1.  
  • Effort Estimate: 3 points  
  • Priority: High

#### 4.2 Compliance Check Execution

• Description: On each rest-hour create/update, run compliance checks (MLC, STCW, OPA if enabled).  
• Technical Requirements:

- Load the latest rule\_set from DB (vessel or office).  
- If thresholds are exceeded, populate violationCodes in the resthours entity (e.g., \[“MLC-1,” “STCW-2”\]).  
- If OPA is disabled, skip OPA checks.  
  • Acceptance Criteria:  
- Proper violationCodes are added whenever a record fails the rule sets.  
- System logs date/time of the check for auditing.  
  • Traceability: L3-FRS-BE §2.5, L3-KD-BE (“Forward-Only Rule Updates”).  
  • Dependencies: Items 2.2, 4.1.  
  • Effort Estimate: 5 points (rule logic \+ tests)  
  • Priority: High

#### 4.3 Soft-Delete Daily Records

• Description: Implement DELETE /resthours/:id as a soft-delete (isDeleted \= true).  
• Technical Requirements:

- Data is never physically removed; indefinite retention is mandatory.  
- Overwrite the record’s isDeleted field, update updatedAt, and log changes in audit\_log.  
  • Acceptance Criteria:  
- Soft-deleted daily records remain in the database for reporting or future reference.  
- The system can differentiate active vs. isDeleted records in queries, as needed.  
  • Traceability: L3-FRS-BE §2.3, §2.8, L3-LLD-BE §5.2, L3-NFRS-BE §6.3.  
  • Dependencies: Items 4.1, 3.1.  
  • Effort Estimate: 1 point  
  • Priority: Medium

---

### 5\. Task Management (CRUD & Attachments)

#### 5.1 Create/Update Tasks

• Description: Implement endpoints: POST /tasks, PUT /tasks/:id for planning tasks with one optional attachment path.  
• Technical Requirements:

- last-write-wins for all  fields (startTime, endTime, assignedCrew).  
  • Acceptance Criteria:  
- Successfully create a new task, storing vesselId, assignedCrew, start/end times.  
- Update merges if updatedAt is more recent.  
  • Traceability: L3-FRS-BE §2.4, L3-LLD-BE §3 (TasksModule), Clarification \#4 (Single attachment).  
  • Dependencies: Item 2.1, 3.1.  
  • Effort Estimate: 2 points  
  • Priority: Medium

#### 5.2 Soft-Delete Tasks

• Description: Implement DELETE /tasks/:id via isDeleted \= true.  
• Technical Requirements:

- Indefinite retention: do not physically remove records.  
- Document the reason for deletion in the audit\_log.  
  • Acceptance Criteria:  
- The user can “delete” tasks, but the data remains in the DB for possible future retrieval.  
  • Traceability: L3-FRS-BE §2.4, §2.8.  
  • Dependencies: Items 5.1, 2.1.  
  • Effort Estimate: 1 point  
  • Priority: Medium

#### 5.3 Attachment Handling (≤5 MB)

• Description: Implement file upload/download endpoints that store a single local file path in tasks.attachmentPath.  
• Technical Requirements:

- On vessel, store attachments locally; optionally defer or automatically upload once stable connectivity is available (per the selected approach).  
- Ensure the file path is protected by role-based checks.  
  • Acceptance Criteria:  
- A user can upload up to 5 MB file in a single request (or local deferred if offline).  
- The BE saves the path in tasks.attachmentPath.  
- If a new file overwrites an existing path, the oldValue/newValue are logged in audit\_log.  
  • Traceability: L3-FRS-BE §4.5, Clarification \#5 (Deferred upload for up to 5 MB).  
  • Dependencies: Items 5.1, environment config for local storage path.  
  • Effort Estimate: 3 points  
  • Priority: Medium

---

### 6\. Conflict Handling within the same premise & Audit Logging

#### 6.1 Simple Last-Write-Wins Logic

• Description: Implement conflict resolution so that if a record’s updatedAt in DB is older than the incoming record’s updatedAt, the incoming update overwrites.

• Technical Requirements:

- Overwritten version is appended to audit\_log with oldValue & newValue.  
- No partial merges by half-hour block; entire daily record is replaced in a single transaction.

• Traceability: L3-FRS-BE §2.6, L3-LLD-BE §6.1, L3-KD-BE (“Conflict Resolution for Compliance-Critical Data”).  
• Dependencies: DB entities, daily record endpoints.  
• Effort Estimate: 3 points  
• Priority: High

#### 6.2 Immutable Audit Log

• Description: Implement the audit\_log table and automatic writes whenever a resthour, task, is overwritten or changed.  
• Technical Requirements:

- No direct DELETE or UPDATE on audit\_log.  
- Each audit entry: referenceTable, referenceId, oldValue, newValue, userId, vesselId, timestamp.  
  • Acceptance Criteria:  
- For each update or soft-delete, an audit entry is created in the same DB transaction.  
- Querying the audit\_log shows full history of changes, enabling indefinite retention.  
  • Traceability: L3-FRS-BE §2.8, §4.4, L3-NFRS-BE §4.3, L3-LLD-BE §5 (AuditLoggingModule).  
  • Dependencies: Items 2.1, 4.1 (CRUD), 5.1.  
  • Effort Estimate: 3 points  
  • Priority: High

### 7\. Non-Functional & Cross-Cutting

#### 7.1 Logging & Error Handling

• Description: Implement a unified logger (e.g., Nest.js logger) and exception filters to handle 4xx/5xx errors.  
• Technical Requirements:

- Standardize log levels (info, warn, error).  
- On vessel, logs stored locally (rotating if needed).

• Acceptance Criteria:

- All CRUD and sync operations produce meaningful logs for debugging.  
- 4xx/5xx are consistently structured with JSON responses.  
  • Traceability: L3-NFRS-BE §6 (Reliability), L3-LLD-BE §11.  
  • Dependencies: Basic app structure.  
  • Effort Estimate: 2 points  
  • Priority: Medium

#### 7.2 Unit & Integration Testing

• Description: Implement unit tests for controllers/services and integration tests for DB writes, auditing, offline/online transitions.  
• Technical Requirements:

- Achieve coverage of CRUD endpoints, compliance checks, and log creation.  
- Use a test DB or in-memory DB approach for local development.  
  • Acceptance Criteria:  
- At least 70-80% coverage for critical modules (rest hours, tasks, sync).  
- Clear pass/fail reporting in CI logs.  
  • Traceability: L3-NFRS-BE §§3.3-3.4 (Performance & Testing).  
  • Dependencies: Implementation of CRUD and sync modules.  
  • Effort Estimate: 5 points  
  • Priority: High

---

### 8\. Priority & Sequence Overview

Below is a simplified sequence for planning:

1. Project Initialization (Items 1.1, 1.2)  
2. Data Model Entities (Items 2.1, 2.2)  
3. Auth & Access Control (Items 3.1, 3.2)  
4. Daily Rest-Hours CRUD \+ Compliance (Items 4.1, 4.2, 4.3)  
5. Task Management (Items 5.1, 5.2, 5.3)  
6. Conflict & Audit Logging (Items 6.1, 6.2)  
7. Logging & Error Handling (Item 9.1)  
8. Testing (Item 9.2)

Implementation order may be adjusted slightly based on team priorities and cross-dependencies.

---

### Final Remarks

This L3-IB-BE backlog breaks down the Back-End’s functional requirements (L3-FRS-BE) and design specifications (L3-LLD-BE, L3-KD-BE, L3-NFRS-BE) into tangible tasks. Each item is deliberately scoped to match the solution’s current scale (up to \~30 vessel users, \~100 office users), with simple last-write-wins conflict resolution and no mandatory Admin intervention for compliance data overrides (per client Clarification \#1). The tasks collectively ensure the Nest.js \+ MySQL BE is ready for incremental vessel-office deployment, indefinite data retention, and an immutable audit trail.  
