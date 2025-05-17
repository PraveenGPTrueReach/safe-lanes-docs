## L3-NFRS-FE: Non-Functional Requirements Specification (NFRS) for FE

### 1. Introduction
This document <edit>specifies</edit> the non-functional requirements for the SafeLanes Rest Hours Front End (FE). The FE is an Angular 16 microfrontend capable of operating both online (office environment) and offline (vessel environment with a local fallback). It interacts with Nest.js back-end services (local or remote) to handle rest-hour data entry, <edit>task planning</edit>, and compliance checks. The requirements outlined here address performance, security, usability, reliability, compliance, and environmental aspects tailored to the stated project scale, <edit>avoiding overengineering</edit> or advanced solutions beyond current needs.

<add>Note: The "task planning" functionality mentioned here is not fully covered in the L1-OVERVIEW or L1-HLD documents. A formal update is recommended to align these L1 documents with the final scope.</add>

---

### 2. Performance Requirements
The FE must provide a responsive and smooth user experience across both vessel and office environments, ensuring basic performance targets without unnecessary complexity.

1. **Page Load & Rendering**  
   <delete>(per Clarification #1)</delete>  
   <edit>Maintain a Basic Page-Load Metric approach, ensuring that the initial load time does not exceed 3 seconds for at least 95% of page-load attempts. In extremely limited bandwidth conditions (e.g., ~256 kbps on vessels), some loading delays may occur, but the system should still strive for acceptable responsiveness wherever possible.</edit>

2. **CRUD Operations**  
   - <edit>Align with L1-NFRS performance standards of 2 seconds or less for typical create/read/update/delete operations in at least 95% of cases, and up to 4 seconds for heavier dashboards under moderate concurrency (~30 concurrent vessel users, ~100 office users).</edit>

3. **Scalability**  
   - Sufficient for ~100 concurrent office users and ~30 concurrent vessel-side users. The FE need only handle incremental growth (e.g., new modules or small expansions) without significant refactoring.

4. **Large Data Views**  
   - The FE employs simple pagination when users request extended date ranges or large volumes of data. <delete>No advanced client-side virtualization is mandated at this scale.</delete> <add>Avoiding complex client-side virtualization at the current scale is acceptable.</add>

5. **Offline Operation**  
   - The local fallback bundle on each vessel must load promptly (<edit>under 3 seconds</edit> in typical conditions). However, no complex in-FE queuing or background sync is required; an immediate error shows if the local Nest.js server is unavailable.

---

### 3. Security Requirements
The FE leverages SafeLanes’ existing authentication framework (JWT and short-lived offline keys) while following basic Angular security best practices. It does not store extensive data client-side or implement advanced cryptographic features.

1. **Authentication & Authorization**  
   - Rely on SafeLanes Identity Service for JWT-based auth in online mode. For offline mode, short-lived session keys (valid up to 14 days) are issued by the local vessel server.  
   - FE must properly handle role-based feature toggling (Vessel User, Vessel Admin, Office Admin, etc.), never exposing unauthorized actions in the UI.

2. **Data Protection**  
   - All FE calls to either vessel or office Nest.js back-ends use HTTPS/TLS 1.2 or higher.  
   <edit>Implementers should ensure a trusted certificate setup for local vessel servers to avoid browser security warnings, but the provisioning and renewal strategy is beyond the scope of this document.</edit>  
   - Sensitive forms and conflict resolutions must not leak data into browser logs, and major events are typically logged by the back-end for compliance.

3. **Front-End Hardening**  
   - Use Angular’s strict mode or default sanitization for user inputs to reduce XSS risk.  
   - <delete>Follow a modest Content Security Policy (CSP) that disables inline scripts while allowing hashed or external resources, but do not overcomplicate with advanced sanitization beyond Angular’s defaults (per Clarification #8).</delete> <add>Enforce a modest Content Security Policy that disallows inline scripts and leverages Angular's default sanitization, ensuring basic protection without adding undue complexity.</add>  
   - When local Nest.js or the office server is unreachable, the FE displays error prompts; no additional (re)try queue is maintained.

---

### 4. Usability Requirements
The FE must be straightforward for vessel crews, office staff, and occasional external/auditor users.

1. **Minimal User Training**  
   - Maintain a consistent layout (forms, day-level grids) across vessel and office contexts, ensuring quick familiarity with rest-hour entry.

2. **Accessibility**  
   - <edit>Follow a minimal accessibility subset: adequate color contrast, keyboard tab order, and basic ARIA attributes. No formal WCAG 2.1 conformance or advanced test suite is required at this stage.</edit>

3. **Responsive Design**  
   - Primary support for desktop/laptop use, with optional coverage for tablets if certain vessels use them. The layout should degrade gracefully without advanced mobile-optimized patterns.

4. **Version Mismatch & Clock Drift**  
   <delete>In alignment with L1-FRS §2.2</delete><add>In keeping with the overall system's policy,
   </add> the FE shall present a mismatch warning without fully blocking offline usage, ensuring that critical rest-hour data entry continues even in the event of version differences. <add>However, if a severe version incompatibility is detected (as determined by the SafeLanes team), the system may block usage to prevent data corruption, aligning with existing key decisions at the L1 or L2 level.</add>  
   <edit>Additionally, the FE should display a mild clock-drift warning if the local system clock is skewed, but not block entry for minor discrepancies.</edit>

---

### 5. Reliability Requirements
The FE serves as the user-facing component for logging hours, detecting conflicts, and providing read-only auditor screens. While most reliability aspects lie in the back-end or system layer, the FE must still handle certain scenarios gracefully.

1. **Offline Fallback**  
   - The FE loads locally on the vessel during offline periods. If unreachable or if local Nest.js is down, the user sees an error and must retry. No internal queuing is required.

2. **Conflict Resolution Prompts**  
   - If concurrency or offline conflicts are detected, the FE shows a clear pop-up to Admins or Super Admins for a day-level resolution. This ensures no silent overwrites of compliance-critical data.

3. **Error Handling & Recovery**  
   - Immediately display user-friendly errors if the FE cannot connect to local or office endpoints. Activity logs (e.g., last actions) are stored on the back-end, not the FE.

<add>4. Reliability Metrics  
   - Aim for at least 99.5% monthly availability of the FE. Operators should monitor accessibility (e.g., load failures or major UI errors) to ensure the system remains functional.  
   - In the event of major failures (e.g., repeated timeouts), administrators or designated personnel should restart or redeploy the local or office instances, following standard SafeLanes procedures.</add>

---

### 6. Compliance Requirements
The FE supports maritime rest-hour compliance by accurately capturing user inputs and displaying relevant warnings. Actual storage, audit logs, and final rule checks are handled by the back-end.

1. **Regulatory Awareness**  
   - Display compliance warnings (e.g., predicted violations) based on server calculations.  
   - For all overwrites of rest-hour data, the FE triggers conflict resolution if flagged as compliance-critical. The back-end logs final decisions in an immutable audit table.

2. **Auditor Access**  
   - Provide read-only dashboards for external/auditor roles without exposing editing or administrative menus.

---

### 7. Environmental Requirements
The FE must run reliably in the standard SafeLanes environment, bridging vessel servers with local fallback and the office server in an online setting.

1. **Hardware & Software**  
   - Typical Vessel Workstation: Modern OS (Windows 10/11 or a maintained Linux distro), Chrome or Edge (current–2).  
   - Angular 16 with minimal polyfills, tested on Chrome/Edge plus Safari ≥15 or ESR Firefox if needed.  
   - Local fallback served by Nginx or PM2 on the vessel’s Nest.js instance.

2. **Network Environment**  
   - Vessel side: ~256–512 kbps satellite link, possibly disconnected for up to 14 days. The FE’s local fallback prevents downtime in rest-hour entry.  
   - Office side: Standard broadband. Typical usage for ~100 concurrent users.

3. **Browser Baseline**  
   - Mandate modern browsers with minimal legacy polyfills. Officially disclaim older Internet Explorer versions or significantly outdated systems.

---

### 8. Dependencies and Requirements

<add>This section enumerates the relevant high-level documents and known constraints that influence this FE’s design. Where references to “L1-FRS” previously existed, they are replaced or removed if that document is not available. Some new features (like task planning) may require updates to the L1 documents for full alignment.</add>

1. **From L1-NFRS**  
   - Response time: Typical CRUD ≤2s, dashboards ≤4s, under moderate concurrency (~30 vessel, ~100 office).  
   - Security: TLS 1.2 or higher, role-based permissions.  
   - Offline usage with local fallback.

2. **From L1-OVERVIEW & L1-HLD**  
   - The solution includes rest-hour logging, compliance checks, offline operation, and synchronization.  
   - <add>Task planning is mentioned in lower-level documents but is not fully elaborated in L1-OVERVIEW or L1-HLD, so an update may be necessary to confirm final scope.</add>

3. **From L3-FRS-FE**  
   - Day-level rest-hour logging with half-hour blocks.  
   - Conflict resolution pop-ups for compliance-critical data.  
   - Minimal local data storage, with the majority stored in the Nest.js back-end.

<delete>4. Client Clarifications (#1, #2, #3, #4, #5, #7, #8, #10, #11, #12)</delete>

<add>4. Removal of Partial Upload / Retry Logic for Attachments  
   - Any references to partial-upload or re-try features for offline attachments are officially removed in this FE scope. Users must be online to upload attachments, ensuring consistency between documents.
</add>

<add>5. Notation on L2 Dependencies  
   - Applicable L2-level documents (if any) are not explicitly provided here. If L2 interactions or interface definitions exist, they should be cross-checked to ensure the FE meets any additional inter-service constraints.
</add>

---

*End of L3-NFRS-FE Specification*