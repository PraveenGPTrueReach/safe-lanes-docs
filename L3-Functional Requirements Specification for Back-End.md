## L3-FRS-BE: Functional Requirements Specification (FRS) for BE

This document details the specific functions and behaviors that the Back-End (Nest.js \+ MySQL) component must implement to fulfill the SafeLanes Rest Hours business requirements as outlined in higher-level documentation. It focuses on the services, data processing, and interfaces that the Back-End must provide to successfully support both vessel (offline) and office (online) operations.

---

### 1\. Introduction

The Back-End is responsible for:

- Storing and managing rest-hour data (48 half-hour blocks) for each crew member per day.  
- Implementing business rules for maritime compliance checks (MLC, STCW, and OPA if enabled).  
- Handling offline operations on vessels (local MySQL DB,).  
- Providing endpoints for creating, reading, updating, records,.  
- Maintaining an audit log of overwritten data.  
- Ensuring indefinite retention of records in a single MySQL schema with a vesselId column.  
- Storing any planning-task attachments on the file system, with references in the database.

All requirements trace to the L1-FRS (high-level) documents, referencing sections (e.g., “Implements L1-FRS §2.3”) where applicable.

---

### 2\. Functional Requirements

#### 2.1 Role Verification and Access Control

• Implements L1-FRS §2.1  
• The BE must validate user roles (e.g., VesselUser, OfficeAdmin) via JWT tokens.  
• Must only allow write operations if the user has proper role/permission, enforcing SafeLanes role-based rules.  

#### 2.2 Offline Operations

• Implements L1-FRS §2.2  
• The BE must support full offline functionality on vessel side:  
– Persist new or updated daily rest-hour records in the local MySQL.  
– Confirm compliance checks locally.  
• The BE must always validate users against the locally available SAIL-app’s identity service by obtaining standard JWTs.

#### 2.3 Recording and Editing Daily Rest Hours

• Implements L1-FRS §2.3  
• Provide endpoints for creating, reading, updating, and deleting daily rest-hour records:  
– Example:  
– POST /resthours  
– GET /resthours/:id  
– PUT /resthours/:id  
– DELETE /resthours/:id (soft-delete, see §2.8 for retention)  
• Each record corresponds to a single day per crew member, storing data in 48 discrete columns (block\_1 … block\_48) plus metadata fields (crewId, date, vesselId, updatedAt, etc.).  
• Must apply compliance checks (see §2.5) upon every creation or update, tagging violations as needed.

#### 2.4 Planning and Scheduling Tasks

• Implements L1-FRS §2.4  
• Provide endpoints to record or update planned tasks—for instance:  
– POST /tasks  
– GET /tasks/:id  
– PUT /tasks/:id  
– DELETE /tasks/:id (soft-delete or mark inactive, see §2.8)  
• Each task must reference a vesselId, assigned crew, start/end times (in half-hour increments), and an optional attachment path if a file is uploaded.  
• Attachments are stored on the local file system and referenced by the DB record (e.g., tasks.attachmentPath).  
• If multiple office or vessel users edit the same task, the system applies last-write-wins for all fields based on updatedAt.

#### 2.5 Compliance Checks (MLC, STCW, OPA)

• Implements L1-FRS §2.5  
• For each updated or newly inserted daily record, the BE must run compliance checks.  
• Rules are versioned and stored in a DB table (“rule\_set” or similar). Vessels sync new rule-set entries from office as needed.  
• The BE must attach violation codes to the daily record if thresholds are exceeded (e.g., block\_1…block\_48 show too many “work” segments).  
• If OPA is disabled, the BE omits OPA checks and flags.

#### 2.6 Conflict Handling

• Implements L1-FRS §2.6  
• All data (including compliance-critical fields) follows a simple last-write-wins approach. The record’s updatedAt is compared upon sync or multi-user edits.  
– If the updatedAt in the DB differs from what the client last saw, the client’s request overwrites the record if the client’s updatedAt is more recent.  
– The overwritten version is written to an audit (log) table for indefinite retention.  
• No additional Admin-driven merges are required. The system always treats the most recent updatedAt as canonical.

#### 2.7 Indefinite Retention and Audit Logging

• Implements L1-FRS §2.8  
• The BE must never discard daily rest-hour data or the associated audit logs.  
• All overwrites (including sync merges) must append an entry to an immutable audit table with these fields:  
– referenceId (the rest-hour record or task ID)  
– oldValue, newValue (JSON or text describing changes)  
– timestamp, userId, vesselId  
• The system does not allow editing or physically removing entries from the audit log. If there is a correction, it is written as a new row referencing the old row.  
• The system always treats the latest updatedAt as final for all data fields.  
• Where “delete” endpoints exist, they must perform a soft-delete (isDeleted flag) to preserve the record, fulfilling the indefinite retention mandate.

---

### 3\. Component Interfaces

#### 3.1 Daily Rest-Hour Records

• POST /resthours  
– Creates a new daily record (48 blocks, compliance checks).  
• GET /resthours/:id  
– Retrieves a single daily record by ID, with violation codes if present.  
• PUT /resthours/:id  
– Updates a daily record, re-checking compliance.  
• DELETE /resthours/:id  
– Perform a soft-delete by marking isDeleted \= true, retaining the record for compliance and historical audit.

#### 3.2 Tasks (Planning and Scheduling)

• POST /tasks  
– Creates a task record, including optional references to local file attachments.  
• GET /tasks/:id  
– Returns a single task record.  
• PUT /tasks/:id  
– Updates the task timing/crew assignment; last-write-wins applies for non-critical fields.  
• DELETE /tasks/:id  
– Marks the task as deleted (e.g., setting isDeleted \= true) but keeps it in the database to satisfy indefinite retention.

#### 3.3 Authentication & Session Management

• The BE uses SafeLanes Identity Service for JWT validation.

---

### 4\. Data Requirements

#### 4.1 Daily Rest-Hour Schema

• One table per environment (vessel or office) with columns:  
– id (PK), vesselId (FK to identify which vessel), crewId, date (YYYY-MM-DD),  
– block\_1 … block\_48 (varchar or integer representing rest/work/other states),  
– violationCodes (text or JSON if multiple codes apply),  
– updatedAt (timestamp in UTC), createdAt (timestamp),  
– localOffset (string, e.g. “+03:00”),  
– isDeleted (boolean, default false), used to mark a record as soft-deleted.

#### 4.2 Task Schema

• One table with columns:  
– id (PK), vesselId, name/description, startTime, endTime, assignedCrew, updatedAt, createdAt, attachmentPath (nullable).  
– Possibly an isDeleted or status column to mark tasks as inactive.  
– isDeleted (boolean, default false) for soft-deletion.

#### 4.3 Rule Sets and Versions

• A “rule\_sets” table to store MLC/STCW/OPA parameters (e.g., max daily work hours).  
• version (int), ruleType (string), thresholds (JSON), publishedAt.  
• The vessel environment downloads updated rows for new or modified rules. During record creation or updates, the system references the newest matching rule set in the local DB.

#### 4.4 Audit Log Schema

• One table (e.g., “audit\_log”) containing:  
– logId (PK), referenceTable (e.g., “resthours”), referenceId, oldValue (text/JSON), newValue (text/JSON), userId, updatedAt, vesselId.  
• No delete or update actions permitted on this table (immutable).  
• Overwrites appear as new rows,.

#### 4.5 Attachments

• Physical storage on the file system, with a tasks.attachmentPath pointer.  
• Up to 5 MB per file. Sync is chunked or managed separately if connection is slow.  
• The back-end must provide endpoint(s) to upload or download these files, protected by proper role checking.

---

### 5\. Validation Rules

#### 5.1 Data Acceptance

• Implements L1-FRS §2.3, §2.5  
• The BE must reject invalid or incomplete daily rest-hour requests, such as:  
– Overlapping half-hour blocks that exceed 24 hours in a day.  
– Missing crewId or date.  
– If the request does not contain all required blocks (1–48), the system sets default values or rejects the request.

#### 5.2 Compliance Checking

• On each submission or update:  
– The BE reads the latest rule set from local DB.  
– Summarizes total rest vs. work hours and checks thresholds (MLC, STCW, OPA).  
– If violations are found, store them in violationCodes, e.g. \[“STCW-1”, “MLC-2”\].  
– For OPA, skip or include checks depending on whether OPA is enabled.

#### 5.3 Conflict Resolution for Compliance-Critical Data

• If a client tries to update a record whose updatedAt no longer matches what the client provided, the newer updatedAt prevails for all fields.   
• No partial merges or block-by-block conflict resolution.

#### 5.4 Indefinite Retention

• Implements L1-FRS §2.8  
• The system never purges historical daily rest-hour records or tasks.  
• All modifications produce an audit entry, preserving previous versions. Delete operations become soft-deletes, ensuring no actual data is lost.

---

### Final Remarks

This L3-FRS-BE document defines the Nest.js \+ MySQL Back-End requirements for the SafeLanes Rest Hours solution while ensuring alignment with the higher-level documents:

- Use discrete columns for 48 half-hour blocks, storing updatedAt.  
- Provide endpoints that enable both vessel offline operation and office aggregation.  
- Maintain a robust audit log with indefinite retention.  
- Keep compliance checks straightforward, run on each record update.  
- Store rule sets in the database for easy offline replication.  
- Use a simple file system approach for attachments, referencing them in the DB.  
- Where deletes exist, apply soft-delete rather than physical removal to satisfy indefinite retention.

These features align with the project’s current scale (30 crew per vessel, up to \~100 office users, offline periods up to 14 days) while ensuring indefinite data retention.  
