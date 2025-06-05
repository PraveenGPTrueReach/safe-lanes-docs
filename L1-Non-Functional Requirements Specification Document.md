## L1-NFRS: Non-Functional Requirements Specification (NFRS) (High-Level)

## 1. Introduction

This document enumerates the overarching non-functional requirements for the SafeLanes Rest Hours Submodule. It addresses performance targets, security standards, usability considerations, reliability expectations, compliance obligations, and the operating environment. These high-level requirements ensure the system meets maritime rest-hour regulations (MLC, STCW, OPA) without overengineering, aligning with moderate concurrency (up to ~30 vessel users, ~100 office users).

---

## 2. Performance Requirements

The system must provide efficient response times, and reasonable scalability for current usage levels:

### 2.1 Response Time & Throughput

- Typical CRUD Operations (e.g., entering/editing rest-hour logs) must respond within 2 seconds for 95% of requests under normal office or vessel load.  
- Dashboard or analytics screens should load within 4 seconds for 95% of requests, given moderate concurrency.  
- Local vessel operations, performed offline against a local database, are expected to remain well under these thresholds due to reduced latency.

### 2.2 Scalability

- The system must provide basic monitoring of CPU/memory usage on the office server to allow manual scale-up if user counts exceed ~100 office users.  
- Vessel-side servers (up to ~30 concurrent crew) remain autonomous and are sized to handle local rest-hour entry and departmental usage.

---

## 3. Security Requirements

These requirements ensure appropriate measures for data confidentiality and integrity, including offline scenarios:

### 3.1 Authentication & Authorization

- JWT-based authentication, integrated with SafeLanes identity service, governs user sessions.  
- Multi-factor authentication (MFA) is not mandated for this release. Password-based credentials from authorized personnel suffice.

### 3.2 Role-Based Access Controls (RBAC)

- SafeLanes roles (Vessel User, Vessel Admin, etc.) must be strictly enforced, preventing unauthorized edits of compliance-critical fields.  
- External auditor accounts are read-only, allowing limited dashboard or record viewing as configured by SafeLanes RBAC.

---

## 4. Usability Requirements

The solution must remain straightforward for vessel and office personnel of varying technical expertise:

### 4.1 User Interface & Accessibility

- The user interface must present clear forms, tables, and dashboards with adequate color contrast. Full WCAG 2.1 conformance is not required at this time.  
- An English-only interface is acceptable. Localization, if needed, should be minimal and not mandatory for initial deployment.  
- The primary SAIL application layout, plus the integrated microfrontend, must be intuitive for vessel crews with minimal training.

### 4.2 Offline UI

- a locally bundled UI must seamlessly serve the same features, ensuring no downtime in rest-hour recording.  
- The system should display version mismatch warnings if the local UI bundle lags too far behind the back-end.  
  For major incompatibilities, usage may be blocked until an update is performed; however, the specifics are subject to client policy.

---

## 5. Compliance Requirements

Maritime rest-hour compliance is central to this solution, necessitating indefinite record-keeping and robust auditing:

### 5.1 Regulatory & Audit

- MLC, STCW, and (optionally) OPA regulations govern the solution's rest-hour checks. The system must store or display violation codes accordingly.  
- All overwrites of compliance-critical data must log before-and-after values in an immutable audit record.  
- Data must be retained indefinitely (or follow client-specific maritime mandates) to satisfy potential flag-state or port-state inspections.

---

## 6. Environmental Requirements

### 6.1 Hardware & Software

- Office Server: ~16 GB RAM, 4 CPU cores minimum, capable of running a Node-based server application and an SQL database, plus a process supervisor and reverse proxy suitable for the deployment.  
- Vessel Server: ~8 GB RAM, 2 CPU cores as a baseline for offline operations with a local relational database installed.  
- Deployments are script-based, without requiring containerization.

### 6.2 Network Environment

- The vessel must initiate outbound connections over secure satellite or internet links for data sync; no inbound firewall changes are assumed.  
- TLS certificates for the vessel and office must be maintained and rotated periodically to preserve security.

---

*End of High-Level NFRS*  
