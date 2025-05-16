`
## L3-KD-BCKND: Component-Specific Key Decisions for BCKND

Below are the core technical decisions specifically affecting the Backend (BCKND). They consolidate clarifications from higher-level documents (L1-KD, L2-LLD) and the final client clarifications. Each decision includes references to its source and the specific plan for BCKND implementation.

---

### 1. Single “Modular Monolith” Architecture
(Ref: L1-KD, Client Clarification #1)

- Decision: Continue with a single containerized Node.js “modular monolith” for ~400–600 concurrent users. Defer splitting into separate services until usage or organizational demands clearly exceed current capacity.  
- BCKND Implementation:  
  - Maintain a single codebase structured into domain-focused modules (e.g., Leads, Quotation, Commissions).  
  - Enforce clear directory boundaries and modular design to ease a future extraction if scale or complexity grows.

---

### 2. Single-Region Deployment with Manual Failover
(Ref: L1-KD, Client Clarification #2)

- Decision: Host all BCKND workloads in one Azure region, using daily backups plus a manual restore procedure if a regional outage occurs. A second-region read replica or automated DR is postponed unless usage or SLA demands it.  
- BCKND Implementation:  
  - Retain daily database backups (Azure Database for PostgreSQL) and container images.  
  - Document manual failover steps; no automated multi-region or real-time replication is configured.

---

### 3. Last-Write-Wins with Basic Row-Version Checks
(Ref: L2-LLD-IC, Client Clarification #3)

- Decision: Operate with a last-write-wins approach overall, but insert a simple version/“updated_at” check on critical fields (e.g., commissions, final pricing) to detect concurrent overwrites.  
- BCKND Implementation:  
  - Add a lightweight version column (e.g., updated_at or revision_id) on leads, quotations, and commissions.  
  - Return HTTP 409 Conflict if the client’s submitted version differs from the latest in the database.  
  - Capture all changes in the audit trail for post-fact resolution if needed.

---

### 4. Online-Only Updates (No Offline Writes)
(Ref: L2-LLD-IC, Client Clarification #4)

- Decision: Disallow offline creation or modifications for Channel Partner and Customer apps, ensuring consistent real-time data integrity.  
- BCKND Implementation:  
  - Reject any attempted write if the client is offline; require an active network session for all data changes.  
  - Provide robust validation during online requests, with read-only caching permitted in the apps.

---

### 5. Full Data Refresh Instead of Delta Synchronization
(Ref: L2-LLD-IC, Client Clarification #5)

- Decision: Continue using full pagination-based retrieval (e.g., offset/limit) for leads, quotations, and other large data sets. Do not implement delta-based sync endpoints at this time.  
- BCKND Implementation:  
  - Maintain endpoints that return complete records in pages.  
  - Encourage front-end to request moderate page sizes (e.g., 25–50 records) to avoid heavy loads as data grows.

---

### 6. Synchronous PDF Generation in a Short-Lived Background Process
(Ref: L1-KD, Client Clarification #6)

- Decision: When generating PDF quotations or documents, spawn a child process or worker thread asynchronously but still within the same container. This prevents blocking the main process; a lightweight queue mechanism confirms completion to the caller.  
- BCKND Implementation:  
  - On a request to generate a PDF, create a short-lived task (e.g., Node.js child process or worker pool) that merges data and uploads to Azure Blob.  
  - Return a “processing” or “completed” status to the front-end. If large volumes arise, the queue can be scaled horizontally within the modular monolith.

---

### 7. Document Retention in Hot Storage
(Ref: L1-KD, Client Clarification #7)

- Decision: Store KYC files, lead documents, and PDF quotations in the Hot tier of Azure Blob for the entire 7-year retention. Tier transitions (Cool or Archive) may be configured later if storage costs become significant.  
- BCKND Implementation:  
  - BCKND keeps all document references in PostgreSQL, with no lifecycle rules enabled by default.  
  - Monitor monthly storage usage; add lifecycle policies to move rarely accessed files to a cheaper tier if it becomes necessary.

---

**End of Document**
```