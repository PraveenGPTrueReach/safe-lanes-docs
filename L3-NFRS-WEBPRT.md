## L3-NFRS-WEBPRT: Non-Functional Requirements Specification (NFRS) for WEBPRT

### 1. Introduction  
This document defines the non-functional requirements for the Solarium Web Portal (WEBPRT) component, which serves Admin and Key Account Manager (KAM) roles. It focuses on performance, security, usability, reliability, compliance, and environmental aspects specifically relevant to WEBPRT, ensuring it integrates seamlessly with the larger system while remaining aligned to the pragmatic scale (~400–600 total concurrent users system-wide, with a smaller subset expected for Admin/KAM usage). 

These requirements supplement the high-level standards defined in L1-NFRS and consider relevant client clarifications regarding concurrency distribution, DR strategy, accessibility depth, and other factors. All references to concurrency, performance goals, and DR/backup align with the single-region Azure approach currently in place.

Note that each sub-section below includes a brief introduction before listing specific requirements. Where clarifications or modifications to standard practices are made (e.g., storing JWT in sessionStorage), additional cautionary notes address potential risks and recommended mitigations.

---

## 2. Performance Requirements  
WEBPRT must maintain acceptable performance for Admin/KAM workflows, keeping in mind that these roles represent only a fraction (~10–15%) of the total 400–600 system-wide concurrency. Nevertheless, the portal should remain responsive and scalable to handle peak operations.

### 2.1 Response Times  
This sub-section defines the target response times for typical and peak usage scenarios, ensuring Admin/KAM can perform tasks quickly during normal and surge loads.  
- Under normal loads, 95% of page loads and data retrieval calls should complete in ≤2–3 seconds.  
- During peak loads or brief surges, response times may stretch to ≤5 seconds for critical Admin/KAM operations (e.g., lead or quotation lookups).  
- For heavy data exports (e.g., CSV of leads) or listing large tables (>5k records), the system may exceed these targets but should remain within a reasonable upper bound (~10 seconds).

### 2.2 Throughput & Concurrency  
This sub-section addresses the volume of concurrent users and bulk updates the Web Portal is expected to handle without major degradation.  
- WEBPRT is expected to handle a modest subset (e.g., ~60–90) of concurrent Admin and KAM users at peak, though it must be robust enough to scale if usage grows unexpectedly.  
- Bulk operations (e.g., updating up to 50 leads at once) should still finalize within a few seconds, assuming normal network and backend availability. The system logs batched updates to maintain consistency without locking out other users.

### 2.3 Scalability Approach  
This sub-section clarifies how the system can be scaled manually or automatically, maintaining adequate performance as usage patterns shift.  
- Horizontal or vertical scaling of the Azure Web App (or equivalent hosting) may be manually triggered by the operations team if heavier Admin/KAM usage is observed.  
- Any auto-scaling triggers can remain optional at this scale, consistent with the rest of the solution.  
- The portal itself (React + TypeScript SPA) has minimal server load beyond API calls; thus, backend throughput is usually the limiting factor.  
  
- Where feasible, define simple operational thresholds for alerting (e.g., 80% CPU usage sustained for 20 minutes). The operations team can decide whether to scale out or up based on these metrics.  


### 2.4 Performance Validation & Testing  
This sub-section outlines how performance goals (e.g., 2–3 seconds response times) will be measured and verified.  
- Conduct periodic load testing in a staging environment, simulating ~60–90 concurrent Admin/KAM sessions.  
- Use Azure Monitor / Application Insights to track metrics (avg. response times, error rates) under realistic scenarios.  
- Acceptable thresholds: 95% of requests ≤2–3 seconds under normal conditions, with concurrency aligned to expected usage.  
- Document test procedures and results in performance test reports to confirm compliance and identify optimization areas.

---

## 3. Security Requirements  
WEBPRT interfaces with the backend’s role-based authentication system to manage Admin/KAM sessions. It must ensure secure data handling and controlled access to privileged operations, while balancing practicality and the current scale (~400–600 total users overall).

### 3.1 Authentication & Authorization  
This sub-section addresses how user sessions are established and controlled, including token storage, role checks, and session timeout specifics.  
- Users log in via email/password, with a 30-minute inactivity session timeout enforced specifically for Admin/KAM sessions.  
- JWT tokens are stored in a secure manner (sessionStorage is used), and an attempted call with an expired token returns 401 Unauthorized, prompting re-authentication.  
- No mandatory multi-factor authentication (MFA) is imposed at present; however, the system design should permit optional MFA if future business needs change.  
  
- Because sessionStorage exposes tokens to potential XSS, implement robust Content Security Policies (CSP) and limit JavaScript injection risks. If future auditing deems sessionStorage insufficient, switch to HTTP-only cookies or an in-memory approach with short-lived tokens.  


### 3.2 Data Protection & Privacy  
This sub-section outlines how data in transit and at rest is secured, and how user privileges are enforced to protect sensitive assets.  
- All traffic from the SPA to the backend must occur over HTTPS (TLS 1.2+).  
- Sensitive fields (e.g., passwords) are never stored in the browser’s local storage; they are only submitted during login.  
- User roles and privileges are enforced by server-side checks, preventing KAM users from accessing Admin-only features. The SPA also conditionally hides UI elements to reduce accidental misuse.

### 3.3 Logging & Audit  
This sub-section details the requirements for logging privileged actions and retaining audit records for compliance.  
- Administrative and high-privilege actions (e.g., CP deactivation, lead status overrides, commission payouts) are logged with user ID, timestamp, and pre/post state for auditing.  
- Logs must be retained in alignment with the 7-year data retention policies defined at the solution level (L1-NFRS).  

---

## 4. Usability Requirements  
WEBPRT must provide a straightforward user experience for Admins and KAMs, respecting basic accessibility guidelines and consistent interface patterns. The following sub-sections detail how these goals translate into practical requirements.

### 4.1 User Interface Consistency  
This sub-section ensures a uniform look and feel across all screens, reducing user training needs and avoiding confusion.  
- The SPA layout (dashboards, sidebars, data grids) must be consistent across Admin and KAM views, with role-based feature exposure (Admin sees system-wide settings; KAM sees territory-limited data).  
- Common UI elements (e.g., date pickers, modals) should be styled uniformly to reduce training overhead.

### 4.2 Accessibility  
In this sub-section, we define the minimal accessibility standards and how they are verified.  
- Basic accessibility standards are required:  
  - Clear color contrast.  
  - Proper labeling of interactive controls.  
  - Keyboard navigation for critical paths.  
- Advanced accessibility (e.g., full WCAG 2.1 AA conformance) is not mandated at this time but can be incrementally introduced if usage demands.  
- To ensure compliance, at least partial WCAG checks (e.g., color contrast analyzers, screen-reader tests) must be performed quarterly or during major UI releases. Document any accessibility issues and track them until resolved.

### 4.3 Error Handling & Feedback  
This sub-section clarifies how the portal should guide users when errors occur, maintaining clarity in typical and outage scenarios.  
- The portal shall provide concise error messages when API calls fail or validations are not met, guiding the user to correct the issue.  
- For system-wide or network outages, a clear “Service Unavailable” screen is displayed, matching the single-region DR strategy.

---

## 5. Reliability Requirements  
WEBPRT’s reliability aligns with the overall single-region deployment model, tolerating occasional downtime with ~24-hour RTO and RPO in worst-case scenarios (full region outage or significant data failure). The sub-sections below address availability targets and DR considerations.

### 5.1 Availability & SLA  
This sub-section specifies the portal’s target uptime and the risks posed by a single-region DR approach.  
- Aiming for ~99.5% uptime on a monthly or annual basis, in line with the broader solution’s acceptance of single-region risk.  
- During planned maintenance, Admin/KAM users receive prior notification. The system may be temporarily unreachable.  
- A single 24-hour outage could represent ~0.3% downtime in a month, thus multiple prolonged outages can undermine 99.5% availability. This SLA is best-effort under the current DR strategy.

### 5.2 Disaster Recovery & Backups  
This sub-section explains how the portal behaves during major incidents and how data/configuration is restored.  
- The portal’s code is typically deployed as static files in Azure Storage or Azure App Service, alongside the shared backend hosting environment.  
- Daily backups of database and configuration settings ensure continuity if a restore is needed; the Web Portal typically shows “Service Unavailable” until the backend is restored.  
- No dedicated read-only fallback mode is implemented for partial outages; the portal simply becomes inaccessible if the backend or DB is down.  
- If achieving 99.5%+ uptime is critical, consider evaluating multi-region failover or a read-only fallback in future phases. Currently, the single-region approach and ~24-hour manual RTO are accepted trade-offs at this scale.

---

## 6. Compliance Requirements  
All compliance standards required at the solution level also apply to WEBPRT. The sub-sections below focus on portal-specific aspects (e.g., data retention, auditing) that must adhere to solution-level targets.

### 6.1 Data Retention & Auditing  
This sub-section ensures that lead timelines, user actions, and other logs remain available for the mandated 7-year retention.  
- Customer data (leads, KYC, etc.) is primarily stored in the backend. The portal simply displays or modifies it via REST APIs.  
- Audit logs, lead timelines, and user actions must persist for at least 7 years, consistent with L1-NFRS storage and logging policies.

### 6.2 Regulatory & Organizational Policies  
This sub-section clarifies that the portal cannot introduce any new data-sharing or compliance burdens beyond what is already established at the solution level.  
- The portal must not introduce additional tracking or data sharing beyond what is mandated at L1.  
- Any future legislative changes (e.g., data localization or new privacy regulations) will be handled by modifying backend data flows without requiring major UI changes.

---

## 7. Environmental Requirements  
WEBPRT is built using React + TypeScript and depends on the backend (Node.js or similar) environment on Azure. The sub-sections outline hosting details, browser compatibility, and integration with monitoring tools.

### 7.1 Hosting & Infrastructure  
This sub-section confirms how the portal files are hosted and aligned with the single-region architecture.  
- Hosted as static files in Azure Storage or Azure App Service, served via HTTPS.  
- Tied to the single-region architecture of the overall solution (with optional geo-redundant blob storage for user-uploaded files).

### 7.2 Browser Support  
This sub-section specifies which browsers are officially supported, balancing modern standards with minimal legacy overhead.  
- Must support recent versions of Chrome, Firefox, Safari, and Edge.  
- Basic functionality in older browsers (e.g., IE11) is not mandated; polyfills can be used if necessary for partial compatibility.

### 7.3 External Services & Tools  
This sub-section clarifies how the portal leverages third-party or Azure-centric tooling for monitoring and analytics.  
- The portal relies on Azure Monitor / Application Insights for performance analytics.  
- No real-time push mechanisms (WebSockets, SSE) are used; polling or user-triggered refresh is sufficient at this scale.

---

## 8. Dependencies and Requirements  
This section lists the higher-level documents and cross-cutting references that define or influence WEBPRT’s non-functional constraints. Each subsection briefly summarizes the relevant dependencies.

### 8.1 L1 (Solution-Level) Dependencies  
These are the overarching documents describing system-wide NFR, architecture, and key decisions that WEBPRT must comply with.  
- L1-NFRS: Defines overall system performance, security (e.g., encryption), reliability (e.g., 24h RTO), and compliance (7-year retention).  
- L1-HLD: Outlines the system’s single-region Azure deployment approach, modular monolith backend, and data flows on which the Web Portal depends.  
- L1-KD: Key technical decisions regarding concurrency models, single Azure region approach, and daily backup strategies.  
- L1-FRS: High-level functional features to which WEBPRT contributes (admin user management, lead/quotation oversight, etc.).

### 8.2 L2 (Cross-Cutting) Dependencies  
These documents detail inter-component interactions and third-party integrations that the Web Portal may indirectly invoke.  
- L2-LLD-IC: Inter-Component interaction details, specifying REST endpoints, roles, and authentication flows used by WEBPRT.  
- L2-LLD-INTG3P: Third-party service integrations (SMS, email) that the portal indirectly invokes via backend APIs.

### 8.3 Client Clarifications (#1–#7, etc.)  
Where references to concurrency distribution, optional MFA, or minimal accessibility are made, these clarifications guide the final design choices.  
- Concurrency distribution (Web Portal handling a smaller fraction of total load).  
- Optional MFA for Admin/KAM.  
- Basic accessibility coverage, no immediate deep WCAG conformance.  
- Same ~24-hour manual DR approach, no read-only fallback mode.  
- Manual “Save Draft” for partial quotations instead of auto-save.  
- Minimal code-splitting strategy (accept a single or few JS bundles).

**End of L3-NFRS-WEBPRT**  
This revised specification adds brief introductions for each sub-section, clarifies hosting as static files in Azure (rather than containerizing the SPA), and addresses the possibility that a single 24-hour outage can undermine a 99.5% uptime target. It also references partial WCAG checks, load-testing strategies, and robust CSP if sessionStorage is used for JWTs, ensuring alignment with the current solution scale (~400–600 total users) and established single-region constraints.  
