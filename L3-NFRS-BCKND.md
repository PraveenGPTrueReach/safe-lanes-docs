
## L3-NFRS-BCKND: Non-Functional Requirements Specification (NFRS) for BCKND

### 1. Introduction
This document defines the non-functional requirements for the Backend (BCKND) component of the Solarium Green Energy solution. The BCKND is a containerized Node.js “modular monolith,” responsible for all server-side operations—business logic, data persistence, file management, and integration with external services. It must support ~400 to ~600 concurrent users under typical demand, with short-term bursts allowed by manual scaling in Azure. The requirements specify how the BCKND should perform in terms of speed, scalability, security, reliability, and compliance to meet the project objectives without overengineering.

### 2. Dependencies and Requirements
The BCKND inherits core standards from the higher-level (L1) Non-Functional Requirements (NFRS) and related cross-cutting documents. Key dependencies affecting this component include:
- L1-NFRS: Overall system performance, security, data retention, and compliance expectations.  
- L1-KD: Key decisions for concurrency (last-write-wins) and single-region deployment.  
- L3-KD-BCKND: Specific backend technical decisions (e.g., concurrency override, daily backups, manual DR).  
- Client Clarifications:  
  - #1: Limit parallel heavy tasks (e.g., PDF generation) to a small in-process worker pool.  
  - #2: Target ~30 requests per second (RPS) sustained and ~60 RPS bursts.  
  - #3: Separate long-retention audit logs from short-term operational logs.  
  - #4: Perform quarterly backup-restore (DR) drills.  
  - #5: Use environment variables for secrets, rotate them manually.  
  - #6: Implement simple rate limiting (e.g., 100 requests/min/IP).  
  - #7: Allow scheduled maintenance windows with planned downtime.  
  - #8: Basic usage threshold alerts (CPU, Memory, Error Rate).  
  - #9: JWT tokens use HS256 with a shared secret, rotated quarterly.  
  - #10: Microsoft-managed encryption keys for at-rest data.  
  - #11: Enforce a minimum 30-second polling interval for pseudo-push to avoid high load.

These dependencies define core obligations the BCKND must satisfy to remain aligned with the overall solution goals.

---

## 3. Performance Requirements
### 3.1 Responsiveness and Concurrency
• The BCKND shall uphold an average response time of 2–3 seconds for 95% of standard API calls when up to ~400 concurrent sessions are active. During peak loads (400–600 concurrent sessions), response times may extend up to ~5 seconds.  
• PDF generation, large file uploads, or complex queries may exceed these targets but should remain within ~10 seconds, with clear status or progress indications returned to client applications.

### 3.2 Throughput
• The BCKND must sustain ~30 requests per second (RPS) under normal conditions and handle bursts up to ~60 RPS. This capacity aligns with ~600 peak concurrent users each performing a few minimal operations per minute.  
• Non-critical bulk imports or large file operations should be rate-limited or off-peak scheduled to avoid degrading overall throughput.

### 3.3 Heavy Task Handling
• The BCKND shall limit simultaneous heavy operations (e.g., PDF processes) by maintaining a small in-process worker pool (2–3 parallel tasks). Additional requests are queued until a worker is free, preventing excessive CPU/IO contention.

### 3.4 Polling Intervals
• To prevent load spikes from frequent “pseudo-push” polling, the BCKND shall reject polling requests if they arrive more frequently than once every 30 seconds (HTTP 429 Too Many Requests), or implement a backoff instruction to the client.

### 3.5 Basic Monitoring and Alerting
• Basic usage threshold alerts will be configured (e.g., CPU >80%, memory >80%, error rate >5%). Advanced latency-based alerting at p95 or p99 is optional but not mandatory at this scale.

---

## 4. Security Requirements
### 4.1 Authentication and Authorization
• BCKND shall issue JWT tokens using HS256 with a strong shared secret, rotated quarterly.  
• The system must authenticate each incoming request via a valid token, enforcing role-based permissions (Admin, KAM, CP, Customer).

### 4.2 Data Protection
• All data in transit is protected via TLS 1.2+; requiring HTTPS for all external communications.  
• At-rest encryption in the database (Azure Database for PostgreSQL) and Blob Storage uses Microsoft-managed AES-256 keys. If future compliance demands arise, customer-managed keys may be adopted.

### 4.3 Secrets Management
• Environment variables in Azure App Service or container configurations store SMS, email, and database credentials.  
• Secrets shall be rotated or updated manually at least every 6–12 months (or sooner if compromised).

### 4.4 Rate Limiting and Abuse Prevention
• The BCKND shall enforce local throttling (e.g., 100 requests/min/IP) for sensitive endpoints (login, file uploads) to mitigate brute force or abuse scenarios, returning HTTP 429 if exceeded.  
• OTP attempts for phone-based logins are locked after 5 failures for 15 minutes.

---

## 5. Usability Requirements
Because the BCKND is a server-side API with no direct user interface, its principal usability concerns relate to consistent and transparent error messaging for client applications.  

### 5.1 Consistent Error Handling
• The BCKND shall provide coherent HTTP status codes (e.g., 400 for validation, 401/403 for unauthorized, 409 for concurrency conflicts).  
• Error payloads must include clear messages for easy debugging by the frontend teams (CPAPP, CUSTAP, WEBPRT).

### 5.2 API Documentation
• While direct UI accessibility is not a factor, the BCKND should offer up-to-date API documentation (e.g., OpenAPI spec) for internal developers, clarifying endpoint contracts, request/response schemas, and typical error codes.

---

## 6. Reliability Requirements
### 6.1 Availability and Uptime
• The BCKND targets ~99.5% uptime on an annual or monthly basis, allowing for occasional single-region outages.  
• Routine maintenance windows (monthly or quarterly) are acceptable, providing prior user notification. A short downtime in these periods is permissible.

### 6.2 Backup and Disaster Recovery
• Daily database backups and Blob data snapshots must be maintained. The Recovery Time Objective (RTO) is ~24 hours, and the Recovery Point Objective (RPO) is ~24 hours.  
• Quarterly DR drills shall validate end-to-end restore procedures (database + container configurations). Ephemeral data (such as in-process PDF tasks) may not be fully recoverable.

### 6.3 Resilience for Transient Faults
• The BCKND shall gracefully handle transient faults (e.g., external API timeouts, short database hiccups) by retrying or returning consistent error codes.  
• If a short-lived outage occurs, the BCKND recovers gracefully once dependencies (DB, Blob, SMS/email services) are restored.

### 6.4 Logging and Audit Trails
• The BCKND shall store compliance-critical write/audit logs for seven years, in line with system requirements.  
• High-volume operational logs (warnings, debug) should rotate after 30–90 days to avoid excessive costs.  
• The system must differentiate critical events (lead status changes, doc uploads, overwrites) and maintain them in the long-term audit trail.

---

## 7. Compliance Requirements
### 7.1 Regulatory and Organizational Policies
• The BCKND must follow encryption at rest and in transit guidelines established at the solution level (TLS 1.2+ and AES-256).  
• Access to personal data (KYC documents, lead records) must respect the principle of minimal privilege, enforced by role-based checks.

### 7.2 Data Retention
• All relevant write/audit logs remain stored for up to 7 years; operational logs rotate sooner for cost control.  
• No additional cross-border or multi-region data storage requirements are mandated at this time.

### 7.3 Legal Considerations
• The BCKND may store personally identifiable information (PII) in Microsoft Azure region. Further expansions (e.g., second region or advanced compliance modules) can be revisited if the business scale or local regulations demand it.

---

## 8. Environmental Requirements
### 8.1 Hosting Environment
• The BCKND is containerized and deployed to a single Azure Web App instance (or equivalent container service). This approach is sufficient for ~400–600 concurrent users.  
• Manual (“tier-based”) scaling is permissible if usage approaches the upper end of current concurrency forecasts.

### 8.2 Software Stack
• Node.js runtime with a modular monolith architecture, connected to Azure Database for PostgreSQL.  
• Azure Blob is used for document storage (KYC, PDF quotes).  
• Worker-pool concurrency for CPU-intensive tasks ensures stable performance at the stated scale.

### 8.3 Network Requirements
• Exposed only via HTTPS endpoints.  
• Minimal egress to third-party providers (MSG91 for SMS, SendGrid for email).  
• No advanced traffic shaping or VNet integration is required at the current concurrency level.

---

## 9. Conclusion
The BCKND’s non-functional requirements summarized here ensure responsive performance, robust security, reliable operation, and compliance with retention policies. These guidelines accommodate the solution’s current concurrency range (400–600) with headroom for minor surges, without imposing excessive overhead. Implementation of rate limiting, worker-pool constraints for PDF generation, environment-based secret management, and consistent backup/DR practices each play essential roles in maintaining stable operations at this scale.

By adhering to these requirements, the BCKND will deliver a secure, performant, and maintainable backend for Solarium Green Energy’s lead management and quotation workflows, meeting project objectives and stakeholder expectations.
```
