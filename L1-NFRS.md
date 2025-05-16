## L1-NFRS: Non-Functional Requirements Specification (NFRS) (High-Level)

### 1. Introduction
This section introduces the overall non-functional guidelines defining how the Solarium Green Energy software solution must perform, remain secure, and comply with organizational policies, within the scale of ~400 concurrent users as normal and up to ~600 at peak load. It also clarifies the poll-based (“pull-driven”) nature of in-app alerts, rather than any real-time server-initiated push mechanism.

This document outlines the high-level non-functional requirements for the Solarium Green Energy software solution. These requirements guide the system’s performance, security, usability, reliability, compliance, and environmental standards. They align with the current project scope of supporting approximately 400–600 concurrent users in a single-region deployment on Microsoft Azure.

All references to “push notifications” in related documents are  interpreted as poll-based notifications or user-initiated syncs, rather than real-time server-initiated pushes.  
Likewise, any mention of “delta sync” or partial synchronization in other documents should be understood as a full refresh only; no partial/delta-based sync is currently supported.

---

### 2. Performance Requirements
This section describes constraints regarding system responsiveness, throughput, concurrency, and the approach to scaling resources. It ensures the solution remains stable under normal and peak usage while providing acceptable user experiences for common tasks.

1. **Response Time & Throughput**  
   - The system must respond to 95% of requests within 2–3 seconds under normal load (up to ~400 concurrent sessions).  
   -  Under peak conditions (400–600 concurrent users), response times may extend up to 5 seconds.  
   - The system should handle short bursts of increased load without compromising core functionality.  
   Large file uploads (up to 10 MB) are exempted from the 2–3 second target, as they naturally require more time. Likewise, compute-intensive tasks (e.g., generating printable PDFs) may exceed this target on occasion but should remain reasonable (e.g., ≤10 seconds 95th percentile). It is strongly recommended to provide a clear client-side progress indicator or streaming approach for these uploads.

2. **Concurrency & Capacity**  
   - The system is tested  for stable performance at up to 400 concurrent users (normal) and can sustain up to ~600 for peaks.  
   - All critical database operations (e.g., lead status changes, quotation creation) must complete reliably under this load.

3. **Scalability Approach**  
   - Manual (“tier-based”) scaling is acceptable: operations staff may increase Azure Web App resources as usage grows.  
   - However, to mitigate sudden traffic spikes, basic monitoring alerts (e.g., high CPU or queue thresholds) should notify operations staff in real time, enabling proactive scaling actions.  
   Basic use of Azure Application Insights or equivalent is recommended for real-time performance metrics collection, ensuring ongoing compliance with the defined response times.  
   Auto-scaling rules (e.g., CPU/queue-based) are optional but can be adopted to reduce risk of SLA breaches during off-hour surges.

---

### 3. Security Requirements
This section outlines the expected controls for authentication, data protection, and auditing. It ensures each user role (Admin, KAM, CP, Customer) interacts safely with the system while meeting basic industry norms for confidentiality, integrity, and availability.

1. **Authentication & Authorization**  
   - Admin/KAM users must log in with email and password, adhering to standard password complexity and must face lockout after 5 failed attempts for 15 minutes.  
   -  CP and Customer logins also use a 15-minute lockout after 5 failed OTP or credential attempts, ensuring consistent security posture across all roles.  
   - CP and Customer logins use phone-based OTP with a short validity window.  
   - JWT-based tokens protect API calls; Admin/KAM sessions expire after 30 minutes of inactivity. CP and Customer tokens auto-refresh approximately every 24 hours, with a hard logout after 30 days of inactivity, aligning with the CP/Customer app workflows.  
   - While not strictly mandatory at this scale, implementing multi-factor authentication (MFA) for Admin/KAM logins is  encouraged to enhance security around privileged actions.

2. **Data Protection**  
   - All data in transit must be encrypted with TLS 1.2+; all data at rest must use AES-256-based encryption in Azure Blob and Azure Database for PostgreSQL.  
   -  Enforcing phone/email uniqueness across roles remains mandatory; only Admin can override in rare exceptions.

3. **Auditing & Logging**  
   - Logs are retained for up to seven years in alignment with the retention policies for leads and documents, ensuring thorough long-term traceability for compliance or forensic analysis.  
   - The system must log all write operations and authentication events.  
   - Lead timelines record all status changes and overrides to provide a robust audit trail.  
   A multi-tier approach for log retention (e.g., hot storage for recent months and cold/Archive tiers for older logs) is recommended to manage storage costs over the full seven-year period.

4. **Minimal Privileged Access**  
   - Admin and KAM have broader privileges but must be mindful of data changes and status overrides.  
   - Reassigning leads or overriding statuses is restricted to Admin/KAM, with each action logged.

---

### 4. Usability Requirements
This section ensures that user-facing interfaces for the CP app, Customer app, and the Web Portal all provide consistent design, adequate accessibility, and clear feedback to guide users in normal and offline usage scenarios.

1. **Interface Design & Consistency**  
   - The solution follows standard UI/UX best practices, ensuring consistent navigation bars, icons, and layout.  
   - The mobile apps provide clear offline/online indicators, but for the Channel Partner app, offline usage remains read-only with no new leads or updates allowed when offline.

2. **Accessibility**  
   - Basic accessibility considerations, such as readable fonts, contrasting colors, and legible labels, are required.  
   - No advanced accessibility guidelines (e.g., WCAG certification) are mandated at this scale.  
   However, performing partial WCAG 2.1 checks (e.g., color contrast, keyboard navigability) and incorporating them into QA workflows is recommended to broaden usability.

3. **Error Handling & Feedback**  
   - Each user-facing application must show clear error messages when uploads fail or lead status changes are invalid.  
   - Customer and CP apps must guide users back to valid states after network disruptions or synchronization delays.

---

### 5. Reliability Requirements
This section addresses system availability targets and disaster recovery provisions, ensuring minimal downtime and recoverability within agreed limits. It also clarifies how backups and failover strategies are handled for a single-region deployment.

1. **System Availability**  
   - The target uptime is ~99.5% on an annual or monthly basis, sufficient for a single-region Azure deployment.  
   - Planned maintenance windows are permissible, notified to users in advance.  
   Note that a single 24-hour outage could significantly impact this target; the organization accepts this risk given manual DR procedures.

2. **Backup & Disaster Recovery**  
   - Daily full backups of the PostgreSQL database are required.  
   - A manual restore strategy is acceptable, with an RTO of ~24 hours and an RPO of ~24 hours.  
   -  Azure Blob storage (optionally configured as geo-redundant) for safeguarding documents and generated PDFs.  
   Point-in-time recovery (PITR) may reduce data loss below 24 hours, but 24 hours remains the official upper bound expectation for RPO at this scale.  
   Periodic disaster recovery drills are recommended to confirm that the 24-hour RTO/RPO is consistently achievable.

3. **Fault Tolerance**  
   - Core operations (e.g., reading leads, creating quotations) must handle transient network errors gracefully.  
   - Persistent or major outages are addressed through manual failover to restored backups within the 24-hour RTO constraint.

---

### 6. Compliance Requirements
This section defines how data locality, retention intervals, and relevant legal guidelines must be observed. It ensures the solution meets organizational and basic regulatory standards within the chosen Azure region.

1. **Data Residency**  
   - All  compute resources run in a single Azure region, with optional Azure Blob redundancy (LRS or GRS) for additional safety. No additional cross-border storage constraints apply by default.

2. **Data Retention**  
   - KYC and lead documents remain in primary blob storage for up to seven years and may be manually purged thereafter in accordance with organizational policy; no automated purge process is currently in place.  
   - Historical versions of updated documents are retained but not exposed to end users, except for auditing or legal inquiries.

3. **Legal & Industry Standards**  
   - Basic compliance with local data protection norms, encryption at rest, and minimal data sharing only with authorized third parties (SMS/email gateways).

---

### 7. Environmental Requirements
This section clarifies the infrastructure, technology stack, and operating conditions in which the system runs, ensuring alignment with existing Azure-based hosting plans and minimal overhead.

1. **Hosting & Infrastructure**  
   - Hosted on Azure Web App (container or service-based) for the backend, with React-based SPA for the Web Portal and React Native apps for CP and Customer.  
   - Appropriate Azure App Service tiers are selected based on concurrency, with an option to scale up or out if load approaches next thresholds.

2. **Software Stack**  
   - Backend: Node.js or similar server framework organized as a modular monolith.  
   - Database: Azure Database for PostgreSQL Flexible Server.  
   - Storage: Azure Blob for KYC documents, lead files, and quotation PDFs.

3. **Networking & Access**  
   - Internet-accessible endpoints are protected by TLS 1.2+; no advanced VNet or on-prem integration is required.  
   - Admin and KAM access the web portal via modern browsers, and CP/Customer apps are installed from respective app stores.

4. **Scalability Beyond 600 Users**  
   - If concurrency approaches or exceeds ~600 sessions in production, system owners may manually scale up the Azure App Service or the database tier.  
   Real-time alerts (e.g., from Azure Application Insights) help identify emerging load issues; staff can then intervene or preemptively add resources.

5. Poll-Based Notification Load  
   - All references to “push” notifications in the solution are delivered through poll-based or user-initiated sync.  
   - Teams should avoid excessively short polling intervals to minimize unintended traffic spikes. Evaluating an adaptive approach or a hybrid model for critical updates can further reduce overhead.

*End of L1-NFRS (High-Level)*