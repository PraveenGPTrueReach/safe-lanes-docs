## L3-NFRS-BE: Non-Functional Requirements Specification (NFRS) for BE

## 1. Introduction

This document specifies the non-functional requirements for the Back-End (BE) component—the Nest.js + MySQL service supporting SafeLanes' Rest Hours solution. It outlines performance, security, usability, reliability, compliance, and environmental constraints pertinent to the Back-End at both the vessel (autonomous) and office (online) sites. The scope of these requirements focuses on current usage levels (up to ~30 concurrent vessel users, ~100 office users) and moderate data transaction rates, aligning with the clarifications provided by the client.

## 2. Dependencies and Requirements

This BE component inherits and builds on the following high-level documents and clarifications:

- L1-NFRS: High-level Non-Functional Requirements.  
- L1-HLD: High-Level Design, describing overall architecture (vessel/offline + office/online).  
- L1-KD: Key Technical Decisions, e.g., single-record-per-day schema, conflict resolution approach.  
- L3-FRS-BE: Functional Requirements Specification for the Back-End (details REST endpoints, data model).  
- L3-KD-BE: Key Technical Decisions for the Back-End at component level (e.g., UTC offset handling).  
- L3-WF-BE: Workflow Details for how the BE handles authentication, data submission, conflict resolution.  
- Client Clarifications #1–#13: Guidance on scaling, concurrency, auditing measures, etc.

These dependencies shape the non-functional criteria below.

## 3. Performance Requirements

The Back-End must handle typical CRUD operations for daily rest-hour logs, tasks, and synchronization with acceptable response times under moderate concurrency.

### 3.1 Response Times

- Under normal load (~30 vessel users, ~100 office users), 95% of standard CRUD requests (create/update rest-hour records, tasks) should complete within 2 seconds.  
- More complex or aggregated endpoints (dashboards, simple analytics) should generally respond within 4 seconds for 95% of requests.  

### 3.2 Throughput & Concurrency

- Typical concurrency remains modest (dozens of concurrent sessions at the office, up to 30 crew members on each vessel).  
- The Back-End is expected to handle short data bursts (sync sessions) gracefully without exhausting server resources.

### 3.3 Load & Stress Testing

- Per Clarification #1, performance testing will be primarily manual on a staging environment with nominal concurrency.  
- The system should at least demonstrate that it meets the stated response-time metrics under typical usage scenarios (30–100 concurrent users).  
- The development team may, at its discretion, use lightweight tools (e.g., JMeter) for spot testing, but full automated stress-tests in CI/CD are not mandatory.

### 3.4 Scalability Approach

- Per Clarification #2, the office Back-End will rely on manual scaling (hardware upgrades—CPU/RAM) if usage exceeds ~100 concurrent office users or consistently high resource usage is detected (80%+ for an extended period).  
- The vessel Back-End does not require auto-scaling—8 GB RAM, 2 CPU cores are typically sufficient. Basic OS-level monitoring is advised.

## 4. Security Requirements

Security in this context addresses both online (office) and offline (vessel) modes, ensuring role-based access, data protection, and minimal risk of unauthorized overwrites.

### 4.1 Authentication & Authorization

- The Back-End validates tokens via SafeLanes Identity Service (JWT)   
- External or read-only users must be strictly prevented from modifying any data. The Back-End rejects unauthorized write attempts with 403 Forbidden.

### 4.2 Audit Logging Integrity

All overwrites for data (rest-hour logs), as well as any other fields, are recorded in an immutable audit log. At the current scale (up to ~30 vessel users, ~100 office users), explicit tamper-evident mechanisms (e.g., cryptographic hashing) are not mandated. Existing database ACLs are deemed sufficient to restrict direct edits. However, future regulatory or business requirements may warrant a more robust, cryptographically verifiable audit trail.

## 5. Usability Requirements

While the Back-End primarily exposes RESTful APIs rather than a user interface, there remain relevant usability considerations:

### 5.1 API Documentation & Consistency

- REST endpoints must be documented clearly (e.g., via OpenAPI or internal references) to ensure predictable usage by vessel and office front-ends.  
- Error responses should maintain consistent HTTP status codes and JSON structures.

### 5.2 Minimal Learning Curve for Admin Operators

- Vessel or office system administrators must be able to configure the BE easily (e.g., environment variables, property files).  
- Documentation on manual scaling, backup/restore, and log auditing must be concise and accessible.

## 6. Reliability Requirements

Reliability covers uptime, and failure handling. Vessels operate offline by design, so reliability focuses on stable local usage.

### 6.1 Uptime Targets

- Office Back-End targets ~99.5% monthly availability (per L1-NFRS). Vessel servers are "best effort," with manual restarts if needed.  
- Single instance architecture at the office (per Clarification #10). Accept a few hours downtime in case of hardware failure, with data restored from daily backups.

### 6.2 Conflict Resolution

All data follows a last-write-wins approach with no additional admin-driven merges or approvals. Overwritten records remain in the audit log as a historical record of changes, acknowledging that "silent overwrites" may occur.

## 7. Compliance Requirements

The BE must adhere to maritime regulations (MLC, STCW, OPA if enabled) and store data indefinitely for potential audits or inspections.

### 7.1 Regulatory Alignment

- Retain all rest-hour data for indefinite periods, supporting potential legal or port-state inquiries.  
- Enforcement of roles, read-only external audits, and archived logs meet minimal maritime compliance standards (per L1-NFRS, L1-FRS).

### 7.2 Data Privacy

- Per Clarification #13, no additional data-privacy or GDPR obligations are enforced. All personal data remains in identifiable form for regulatory traceability.

## 8. Environmental Requirements

### 8.1 Hardware & Software

- Office Server: ~16 GB RAM, 4 CPU cores, running Nest.js + MySQL (with PM2, Nginx). Manual scaling only if usage consistently remains near resource limits.  
- Vessel Server: ~8 GB RAM, 2 CPU cores. Linux or Windows OS, hosting Nest.js + MySQL, packaged with minimal overhead. Basic OS-level monitoring (per Clarification #6).

### 8.2 Attachments & Storage

- Attachments (≤5 MB) are stored indefinitely on the main server file system (per Clarification #11). Disk expansion or housekeeping is performed manually as usage grows.

---

All non-functional aspects here align with L1/L2 materials, L3-FRS-BE, L3-KD-BE, L3-WF-BE, and the client clarifications.  
