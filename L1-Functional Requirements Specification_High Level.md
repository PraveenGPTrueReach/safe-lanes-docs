## L1-FRS: Functional Requirements Specification (FRS) (High-Level)

---

## 1. Introduction
This document outlines the high-level functional requirements for the SafeLanes Rest Hours Submodule, ensuring maritime rest-hour compliance (MLC, STCW, OPA) in both vessel (offline) and office (online) environments. It specifies the primary capabilities, system interactions, data handling, and key validation rules that enable authorized crew and office staff to record, review, and analyze rest-hour data while preventing silent overwrites of compliance-critical fields. Low-level technical details, design decisions, and implementation specifics appear in separate L3-LLD documents.

---

## 2. Functional Requirements

### 2.1 User Role and Access Management
- The system must provide clearly defined user roles: Vessel User, Vessel Admin, Vessel Super Admin, Office User, Office Admin, Office Super Admin, and External (read-only).  
- Each role has specific access limitations for viewing or editing rest-hour records, as follows:  
  - Vessel Users can only view/edit their own logs.  
  - Vessel Admins can edit or plan tasks for their department (including subordinate crew). 
    By default, Vessel Admins have the authority to resolve rest-hour conflicts for compliance-critical fields, consistent with the system policy that an Admin or Super Admin may finalize overwrites.  
  - Vessel Super Admins can access and manage all data on the assigned vessel.  
  - Office Users and Office Admins can access aggregated vessel data. Per client policy, they may have read-only or edit rights. By default, they only review data but do not edit.  
  - Office Super Admin has full edit capabilities across all vessels.  
  - External users have view-only access to vessel dashboards or analytics, as configured by SafeLanes RBAC.  
- The system must prevent unauthorized edits, granting conflict-resolution privileges only to those roles allowed to overwrite compliance-critical records.

### 2.2 Offline Operation and Local Fallback
- Vessels must operate fully offline for up to 14 consecutive days. During this period, users can log rest hours and manage tasks on the local server.  
- Short-lived offline session keys (valid up to 14 days) must allow continued access when central identity services are unreachable. If exceeded, a local override by the Vessel Master (Super Admin) or Admin is required to renew offline credentials.  
- A local UI fallback must be served from the vessel’s local server if internet connectivity is unavailable, ensuring the interface remains accessible for compliance data entry.  
- The system must present a warning if a version mismatch is detected between the local UI fallback and the back-end, but it should not fully block usage while offline. Users are advised to update or confirm an override to maintain data integrity.  

### 2.3 Recording and Editing Daily Rest Hours
- The system must allow each crew member to record daily rest-work patterns in half-hour increments for a single day record (48 blocks).  
- Users can create partial or complete entries. Each submission triggers immediate system checks for potential violations (MLC, STCW, OPA).  
- The system must enable designated Admins (vessel or office, depending on policy) to correct or finalize entries for other users, if permitted by RBAC.  
- Authorized roles must be able to edit historical logs within a configurable open period (default 14 days). Past that window, an Admin override is required, and the event must be logged.

### 2.4 Planning and Scheduling
- Vessel Admin or Vessel Super Admin must be able to create “planned” tasks (fixed or variable) in a scheduling interface.  
- Crew members see these “planned” time blocks in grey. They can confirm or adjust them in their actual daily log, without losing the original plan data.  
- Office Admin or Office Super Admin must have read-only access to planning screens unless explicitly granted editing rights via SafeLanes RBAC settings.

### 2.5 Compliance Checks (MLC, STCW, OPA)
- The system must apply configurable numeric thresholds for MLC, STCW, and OPA compliance. If OPA is disabled, OPA violation codes (7 & 8) are suppressed.  
- The system must automatically detect rest-hour violations whenever users save or modify records.  
- Predicted violations must be displayed to help users adjust schedules preemptively; actual violations remain authoritative for official logs.

### 2.6 Conflict Resolution
- The system must detect concurrent edits of the same daily rest-hour record from different sources (vessel vs. office, multiple vessel admins, etc.).  
- Non-critical fields (e.g., comments or optional metadata) default to a last-write-wins approach.  
- Compliance-critical fields require an authorized user to confirm any overwriting at the daily level, typically involving “keep vessel data” or “keep office data” upon conflict. A consolidated daily view may be provided to review changes before final confirmation.  
- Overwritten versions must be preserved in a read-only audit log for indefinite retention, ensuring no silent loss of compliance data.

### 2.7 Synchronization and Data Merge
- The vessel must synchronize new or updated records with the office server on a scheduled or on-demand basis whenever connectivity is available.  
- Sync operations must transfer all relevant data fields (rest-hour records, NC/violation flags, planned tasks, etc.) using incremental timestamps to detect updates.  
- Conflicts identified during sync must be flagged. If compliance-critical, an Admin must review and finalize the data using the conflict-resolution procedure.  
- All data synchronization between the vessel and the office must occur via a secure transmission channel (e.g., HTTPS over TLS) to protect sensitive rest-hour data in transit.

### 2.8 Indefinite Retention and Audit Logging
- All rest-hour entries, plan data, conflict overwrites, and system logs must be retained indefinitely unless the client’s maritime policy directs otherwise.  
- A protected audit log must track each update to compliance-critical records. This log must remain accessible to authorized Admins or Super Admins for compliance reviews.

### 2.9 Basic Reporting and Export
- The system must provide a minimal “Export to CSV/PDF” mechanism for daily or monthly rest-hour logs, enabling official documents for auditor reviews.  
- Office dashboards must offer aggregated charting (violations by vessel, rank, or tasks) for multi-vessel oversight. Vessel-level dashboards must show daily or monthly summary for the assigned vessel.

---

## 3. System Interfaces

### 3.1 Integration with SafeLanes Identity Service
- The system must integrate with the existing SafeLanes JWT-based authentication.  
- When offline, the vessel server must issue local session keys, validated against stored role/identity claims. Once online, these must be re-synchronized with the central identity service.  
- JWT claims must at least provide: subject (user), user role, vessel scope, and expiration. Future expansions (office scope, permissions arrays) may be added as needed, but not mandated at this release scale.

### 3.2 Microfrontend Hosting with SAIL App
- The system must function as a microfrontend integrated into the existing SAIL application shell.  
- Office users load the microfrontend from an office server, while the vessel has a local fallback bundle served by the vessel server.  
- If the host environment does not support the same microfrontend framework/version, a minimal fallback approach (e.g., a web component wrapper or iframe) may be used to ensure compatibility without specifying a particular major version.

### 3.3 Database Connections
- Each vessel must run a local relational database capable of supporting offline operations, while the office server hosts a corresponding database for aggregated data.  
- The system must allow optional encryption at rest or whole-disk encryption, guided by client policy, without prescribing a specific technology.

### 3.4 External Auditor Portal (Read-Only)
- External auditors or third-party inspectors may be granted read-only access to a subset of vessel data.  
- Access is governed by SafeLanes RBAC, ensuring no data modification from external users.

---

## 4. Data Requirements

### 4.1 Rest-Hour Records
- Store each crew member’s daily entries in a single record with 48 half-hour blocks, capturing “work” (normal watch/day work) or “rest.”  
- Each record must include: user ID, date, half-hour blocks, total rest/work hours, assigned violation codes, reason codes (optional comments), updatedAt timestamps (UTC), and local-time offset used.  
- Data for short intervals must remain flexible to meet maritime day-boundary rules (e.g., local midnight or offset).

### 4.2 Compliance Flags and Violations
- Each daily record must contain system-generated violation fields if rest-hour thresholds are exceeded.  
- The system must store numeric codes referencing each type of regulatory breach (e.g., 1–6 for STCW/MLC, 7 & 8 for OPA). Predicted violations do not overwrite official results but are stored separately as warning indicators.

### 4.3 Planning and Variable Task Data
- Each planned task includes: task name/description, start/end times (in half-hour increments), status (“Planned” or “Executed”), and assigned crew.  
- Variable tasks may attach optional documents (limit 5 MB per file) for additional details. If local bandwidth is restricted, the system must handle partial uploads or re-try logic.

### 4.4 Overwrite and Audit Logs
- An overwrite log entry must capture old value, new value, timestamp, reason (if provided), and user ID/role that triggered the overwrite.  
- This log is immutable. The system must provide queries or an admin screen for authorized staff to review these logs.

### 4.5 User, Role, and Session Data
- Store each user’s identity, role, primary vessel assignment, last login, and (if offline) short-lived session key data with explicit expiration to enforce security.  
- Mandatory fields include user ID, role, assigned vessel scope, account status (active/inactive), and offline session key expiry date.

---

## 5. Validation Rules

### 5.1 Hours and Day-Bound Enforcement
- The system must not permit more than 24 total hours logged per calendar day. Any partial blocks exceeding 24 hours or overlapping with existing entries must be blocked or flagged as invalid.  
- Each day’s record is validated for MLC, STCW, and OPA constraints. If the combined work hours extend beyond allowable limits (e.g., over 14-hour daily maximum for STCW), the system sets violation codes.

### 5.2 Conflict Detection
- During saves, the system must compare the stored record’s updatedAt timestamp to detect changes. If a mismatch is found for compliance-critical fields, a conflict resolution prompt must appear.  
- The conflict resolution interface must allow a daily-level approach, requiring the user to keep either vessel data or office data for compliance-critical fields. This ensures no partial, block-by-block merges are needed.

### 5.3 Offline Session Key Validity
- When offline, if the user’s token is expired but within the 14-day limit, the system issues or extends a short-lived key. Beyond 14 days, the system must refuse access unless a Vessel Master (Super Admin) or Admin override occurs.  
- Systems must record any manual override events in a secure audit record.

### 5.4 Regulatory Ruleset Parameterization
- Numeric thresholds for MLC, STCW, OPA must be configurable. If OPA is disabled, violation codes (7 & 8) must not appear.  
- The system must allow updates to these rules via versioned JSON or YAML. If a new version is detected, only new or updated records are checked against it automatically.

### 5.5 Historical Edits
- Editing records older than the current open period (14 days by default) must require an Admin override.  
- The system must allow finalizing these changes, then log them in the overwrite/audit logs for traceability.

---

*End of Document*