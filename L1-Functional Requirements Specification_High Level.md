L1-FRS: Functional Requirements Specification (FRS) (High-Level)

1. Introduction
This document outlines the high-level functional requirements for the SafeLanes Rest Hours Submodule, ensuring maritime rest-hour compliance (MLC, STCW, OPA) in both vessel (offline) and office (online) environments. It specifies the primary capabilities, system interactions, data handling, and key validation rules that enable authorized crew and office staff to record, review, and analyze rest-hour data while preventing silent overwrites of compliance-critical fields. Low-level technical details, design decisions, and implementation specifics appear in separate L3-LLD documents.


2. Functional Requirements
2.1 User Role and Access Management
The system must provide clearly defined user roles: Vessel User, Vessel Admin, Vessel Super Admin, Office User, Office Admin, Office Super Admin, and External (read-only).
Each role has specific access limitations for viewing or editing rest-hour records, as follows:
Vessel Users can only view/edit their own logs.
Vessel Admins can edit or plan tasks for their department (including subordinate crew)..
Vessel Super Admins can access and manage all data on the assigned vessel.
Office Users and Office Admins can access aggregated vessel data. Per permissions granted in SAIL-app , they may have read-only or edit rights. By default, they only review data but do not edit.
Office Super Admin has full edit capabilities across all vessels.
External users have view-only access to vessel dashboards or analytics, as configured by SafeLanes RBAC.
The system must prevent unauthorized edits,.


2.2 Recording and Editing Daily Rest Hours
The system must allow each crew member to record daily rest-work patterns in half-hour increments for a single day record (48 blocks).
Users can create partial or complete entries. Each submission triggers immediate system checks for potential violations (MLC, STCW, OPA).
The system must enable designated Admins (vessel or office, depending on policy) to correct or finalize entries for other users, if permitted by RBAC.
Authorized roles must be able to edit historical logs). 
Office User, Office Admin or Office Super Admin must have read-only access to planning screens unless explicitly granted editing rights via SafeLanes RBAC settings.
2.3 Planning and Scheduling
Vessel Admin or Vessel Super Admin must be able to create "planned" tasks (fixed or variable) in a scheduling interface.
Crew members see these "planned" time blocks in grey. They can confirm or adjust them in their actual daily log, without losing the original plan data.
Office User, Office Admin or Office Super Admin must have read-only access to planning screens unless explicitly granted editing rights via SafeLanes RBAC settings.
2.4 Compliance Checks (MLC, STCW, OPA)
The system must apply configurable numeric thresholds for MLC, STCW, and OPA compliance. If OPA is disabled, OPA violation codes (7 & 8) are suppressed.
The system must automatically detect rest-hour violations whenever users save or modify records.
Predicted violations must be displayed to help users adjust schedules preemptively; actual violations remain authoritative for official logs.


2.5 Indefinite Retention and Audit Logging
All rest-hour entries, plan data, conflict overwrites, and system logs must be retained indefinitely unless the client's maritime policy directs otherwise.
A protected audit log must track each update to compliance-critical records. This log must remain accessible to authorized Admins or Super Admins for compliance reviews.
2.6 Basic Reporting 

Office dashboards must offer aggregated charting (violations by vessel, rank, or tasks) for multi-vessel oversight. Vessel-level dashboards must show daily or monthly summary for the assigned vessel.


3. System Interfaces
3.1 Integration with SafeLanes Identity Service
The RH microservice must integrate with the existing SAIL-app JWT-based authentication.


JWT claims must at least provide: subject (user), user role, vessel scope. 
3.2 Microfrontend Hosting with SAIL App
The system must function as a microfrontend integrated into the existing SAIL application shell.
Office users load the microfrontend from an office server, while the vessel has a local bundle served by the vessel server.

3.3 Database Connections
Each vessel must run a local relational database capable of supporting offline operations, while the office server hosts a corresponding database for aggregated data.

3.4 External Auditor Portal (Read-Only)
External auditors or third-party inspectors may be granted read-only access to a subset of vessel data.
Access is governed by SafeLanes RBAC, ensuring no data modification from external users.


4. Data Requirements
4.1 Rest-Hour Records
Store each crew member's daily entries in a single record with 48 half-hour blocks, capturing "work" (normal watch/day work) or "rest."
Each record must include: user ID, date, half-hour blocks, total rest/work hours, assigned violation codes, reason codes (optional comments), updatedAt timestamps (UTC), and local-time offset used.
Data for short intervals must remain flexible to meet maritime day-boundary rules (e.g., local midnight or offset).
4.2 Compliance Flags and Violations
Each daily record must contain system-generated violation fields if rest-hour thresholds are exceeded.
The system must store numeric codes referencing each type of regulatory breach (e.g., 1–6 for STCW/MLC, 7 & 8 for OPA). Predicted violations do not overwrite official results but are stored separately as warning indicators.
4.3 Planning and Variable Task Data
Each planned task includes: task name/description, start/end times (in half-hour increments), status ("Planned" or "Executed"), and assigned crew.
Variable tasks may attach optional documents (limit 5 MB per file) for additional details. 
4.4 Overwrite and Audit Logs
An overwrite log entry must capture old value, new value, timestamp, reason (if provided), and user ID/role that triggered the overwrite.
This log is immutable. The system must provide queries or an admin screen for authorized staff to review these logs.




5. Validation Rules
5.1 Hours and Day-Bound Enforcement
The system must not permit more than 24 total hours logged per calendar day. Any partial blocks exceeding 24 hours 
Each day's record is validated for MLC, STCW, and OPA constraints. If the combined work hours extend beyond allowable limits (e.g., over 14-hour daily maximum for STCW), the system sets violation codes.


5.2 Regulatory Ruleset Parameterization
Numeric thresholds for MLC, STCW, OPA must be configurable. If OPA is disabled, violation codes (7 & 8) must not appear.
The system must allow updates to these rules via versioned JSON or YAML. If a new version is detected, only new or updated records are checked against it automatically.
5.3 Historical Edits


The system must allow historical edits by authorized roles , then log them in the overwrite/audit logs for traceability.



End of Document  