## L3-FRS-WEBPRT: Functional Requirements Specification (FRS) for WEBPRT

### 1. Introduction
This document specifies the functional requirements for the Solarium Green Energy Web Portal (WEBPRT). The Web Portal serves two main roles:

1. Admin – full authority to manage system settings, user roles, lead data, commission payouts, and more.  
2. Key Account Manager (KAM) – territory-limited oversight with authority to manage leads and create or edit quotations.  

All functionality described here is focused specifically on the Web Portal component, aligning with the larger system’s requirements (reference L1-FRS sections 3.1–3.7) and certain client clarifications #1–#11 (see notes below). Where needed, requirements explicitly reference the corresponding L1-FRS subsection (e.g., “Implements L1-FRS §3.2 Lead Management”) to ensure traceability.

Clarification Numbering: The references to client clarifications (#1–#11) in this L3 document and (#1–#8) in L3-KD-WEBPRT do not necessarily align. Future revisions will unify or cross-reference these clarifications under a single consistent index.

Note: For clarifications #1–#11, please see the appended summaries or the relevant L3-KD-WEBPRT for details, ensuring consistent references across documentation.

### 2. Scope
• Covers how Admin and KAM users authenticate via email/password, manage leads, quotations, commissions, documents, and system configurations.  
• Excludes any direct functionality for CP (Channel Partner) or Customer apps (addressed in their respective L3-FRS documents).  
• Focuses on a scale of ~400–600 concurrent users, adhering to a pragmatic, poll-based data refresh approach (no real-time push notifications).  
• Incorporates final client clarifications on multi-select lead updates, draft quotations,  territory management, and more.  
• The design aims to follow basic accessibility guidelines (e.g., WCAG 2.1) wherever feasible, ensuring the portal is usable by a broad range of users, including those with assistive technologies.

### 3. Functional Requirements

#### 3.1 User & Session Management
Implements L1-FRS §3.1 (User Management & Authentication) for Admin and KAM via the Web Portal:
1. The Web Portal must allow Admin/KAM to log in using email/password (POST /api/v1/auth/login).  
2. The portal enforces a 30-minute inactivity timeout, after which the user's session is invalidated server-side. Although JWT tokens may have a longer overall lifespan for other roles, the Web Portal specifically treats any token beyond 30 minutes of inactivity as expired, requiring reauthentication.  
3. The portal must display appropriate UI elements based on the authenticated user’s role:  
   - Admin sees full system management features.  
   - KAM sees territory-limited features.  
4. The system shall deny access to any feature requiring higher privileges; if a KAM user attempts an Admin-only feature, the server returns 403 Forbidden.

Note: Currently, multi-factor authentication (MFA) is not implemented for Admin/KAM logins. If required in the future, MFA (e.g., TOTP or SMS-based) can be introduced without significant architectural changes.

#### 3.2 Master Data Management
Implements L1-FRS references to “master data” (part of §§3.2, 3.3 for BOM items, product catalogs, etc.):
1. Admin-only portal section for basic CRUD on panels, inverters, fees, and other relevant “master data” records.  
2. Simple CRUD with soft-delete is required; no formal version history (client clarification #3).  
While no formal version history is maintained, all create/update/delete actions are logged with timestamps and user IDs to provide a basic audit trail for compliance.  
3. Endpoint examples (Admin role only):  
   - GET /api/v1/masterdata?type=Panels  
   - POST /api/v1/masterdata (create)  
   - PATCH /api/v1/masterdata/{id} (update)  
   - DELETE /api/v1/masterdata/{id}?softDelete=1  

#### 3.3 Lead Management
Implements L1-FRS §3.2 (Lead Management) for Admin and KAM roles:
1. The portal shall retrieve a paged list of leads with filtering by status, date range, assigned CP or KAM, etc. (GET /api/v1/leads?offset=…&limit=…&status=…):  
   - No local browser caching is used; each refresh queries the backend (client clarification #9).  
2. Multi-Select Bulk Actions (client clarification #1) must be supported for up to 50 records at a time:  
   - Bulk “Update Status” or “Reassign CP” in a single action (PATCH /api/v1/leads/bulk).  
3. Single-lead updates remain available (PATCH /api/v1/leads/{leadId}/status).  
4. The portal must display a read-only chronological “Lead Timeline” reflecting changes in statuses, reassignments, and remarks (client clarification #8).  
   - Uses GET /api/v1/leads/{leadId}/timeline.  
5. Admin and KAM can override normal status progression if their role allows it (reference L1-FRS validations).  

All KAM actions (including lead updates, bulk operations, or reassignments) are strictly limited to leads within the KAM’s assigned territory. Any attempt to manipulate leads outside their territory shall result in a 403 Forbidden response from the backend.

At the current scale of ~400–600 concurrent users, poll-based refreshes with no local caching remain feasible. If usage significantly increases, partial or incremental data fetching may be reconsidered to manage server load.

6. CSV export of filtered leads must be provided for external analysis, per client clarification #7 (minimal reporting approach).

#### 3.4 Quotation Management
Implements L1-FRS §3.3 (Quotation Management):
1. Admin/KAM can create, edit, and share quotations for leads.  
2. The portal shall support saving quotations as “Draft” explicitly (client clarification #2) using:  
   - POST /api/v1/quotations (body includes "status": "Draft")  
   - Drafts remain stored indefinitely until promoted to “Created” or “Shared.”  
3. Once a quotation is finalized and “Shared” with a customer, it becomes immutable.  
4. Admin or KAM can override older quotations or create new ones but must maintain only one “Accepted” quotation per lead (per L1-FRS rules).  
5. Endpoint references (sample):  
   - POST /api/v1/quotations?leadId=…  (create draft or create final)  
   - PATCH /api/v1/quotations/{quoteId} (update details, share with Customer)  
   - GET /api/v1/quotations?leadId=… (fetch existing quotes)

#### 3.5 Commission & Payout
Implements L1-FRS §3.5 (Commission & Payout):
1. Commissions follow “Pending →  Paid” lifecycle (per L1-FRS).  
  
2. All commissions are handled by the Admin once the lead is “Executed.” The default lifecycle remains “Pending → Paid,” and the Admin may override or finalize amounts. If a future policy requires a KAM approval step, it can be introduced as an optional intermediate state without significant architecture changes.  
3. Admin can override amounts if business rules allow, then execute bulk “Paid” updates (e.g., marking multiple commissions as paid).  
4. Once “Paid,” a commission record is locked from further changes.  
5. The Web Portal must also capture UTR details or transaction reference from the Admin when marking “Paid.”

#### 3.6 Administrative Configuration
Implements L1-FRS references to system-level controls (e.g., §3.2, territory assignment) + client clarification #5 and #10:
1. Admin can assign or update a KAM’s territory from a static list (client clarification #5).  
2. Provide a “System Configuration” page (client clarification #10) for toggling certain disclaimers or feature flags in real time.  
   - Example toggles might be “Enable Additional Document Types,” “Enable Beta Dashboard,” etc.  
   - Implementation details may require storing these flags in a special config table (PATCH /api/v1/config).

#### 3.7 Document Viewing & Approval
Implements L1-FRS §3.4 (Document & KYC Management):
1. The portal must allow Admin/KAM to view or approve lead documents, KYC forms, etc., referencing the short-lived SAS token approach from the backend.  
2. Older versions of documents are hidden; only the latest version is displayed (client clarification #6).  
3. Admin can mark an uploaded file “Superseded” but never fully delete older versions from the system for compliance.  
Approved KYC documents are immutably locked and cannot be superseded per L1-FRS §3.4. If a document is already approved, the 'Supersede' action is disallowed, ensuring compliance with the immutability rule.  
4. Endpoints typically:  
   - GET /api/v1/leadDocuments?leadId=…  
   - GET /api/v1/documents/sas?docId=… (generates short-lived URL for download)  
   - PATCH /api/v1/documents/{docId}/supersede (mark older version)

#### 3.8 Reporting & Dashboards
Implements L1-FRS references to “oversight” (e.g., lead status tracking, commission analysis):
1. Provide minimal table listings with CSV export for advanced analysis externally (client clarification #7).  
2. Optionally, a simple aggregated view (e.g., total leads per status, total commissions pending vs. paid), not a complex chart-based solution by default.  
3. No real-time updates; the data refresh is triggered manually or by timed polling.  

Note: The “Cart & Acceptance” functionality primarily applies to the Customer App (CUSTAP). In the Web Portal, Admin or KAM users only monitor the acceptance status or override leads as needed, without direct involvement in the cart process itself.  

To align with measurable business outcomes, basic Key Performance Indicators (KPIs) such as lead conversion rates or average commission processing times may be included. However, these metrics remain optional at this stage. If needed, the portal can display summarized data or export it for advanced BI tools.

### 4. Component Interfaces
The Web Portal interacts exclusively with the Backend (BCKND) via RESTful JSON (HTTPS, JWT-enabled). It:
- Consumes the endpoints described above (e.g., /auth, /leads, /quotations, /commissions).  
- Obtains short-lived SAS tokens for file viewing or uploading from the backend.  
- Requires role-based access enforced server-side.  

No direct data-sharing with CPAPP or CUSTAP is done from WEBPRT; those channels also communicate only with the backend. Refer to L2-LLD-IC for cross-component interaction details.

### 5. Data Requirements
1. The Web Portal displays leads, quotations, and commissions from the central PostgreSQL database (via BCKND).  
2. No local data caching; each page or refresh action retrieves updated records from the backend.  
3. Form data (quotations, commission edits) must be validated locally for basic constraints before submission.  
4. Master data references (e.g., panel/inverter lists) are fetched on demand to ensure Admin edits are seen in near real time.  

5. All data (leads, quotations, commissions) is subject to a 7-year retention policy, consistent with overarching system constraints in the L1-HLD. No automated purge is currently implemented at the Web Portal layer, but administration can handle archiving or manual cleanup if needed.  

6. Typical response times under normal load (~400–600 concurrent users) should remain under two seconds for key listing or update operations. Load and stress testing shall validate that the system meets these performance goals at the specified scale.

### 6. Validation Rules
Although final authority resides in the backend, the WEBPRT must enforce or prompt the user for basic validations:
1. Mandatory fields (e.g., lead status remark ≥10 chars, next follow-up date ≤30 days in future). (Implements L1-FRS §3.2).  
2. Quotation lifecycle rules (cannot share a draft as final without key fields). (Implements L1-FRS §3.3).  
3. Commission transitions (Admin sets commissions as 'Paid'; a future optional KAM approval step, if introduced, must occur before final payout). (Implements L1-FRS §3.5).  
4. Document superseding (Admin can mark old doc as “Superseded,” but never a hard delete) unless the doc is already approved KYC. (Implements L1-FRS §3.4).

### 7. Sample Endpoints & Methods
Below is a non-exhaustive list of relevant endpoints used by the Web Portal; all calls are made via HTTPS with an Authorization Bearer token:

1. Authentication  
   • POST /api/v1/auth/login { email, password }  
2. Leads  
   • GET /api/v1/leads?offset=…&limit=…&status=…  
   • PATCH /api/v1/leads/{leadId}/status  
   • PATCH /api/v1/leads/bulk { leadIds: [], action: "UpdateStatus" }  
3. Quotations  
   • POST /api/v1/quotations?leadId=… (create draft or final)  
   • PATCH /api/v1/quotations/{quoteId} (update or share)  
4. Commissions  
   • PATCH /api/v1/commissions/{id} (mark single commission as “Paid”)  
   • PATCH /api/v1/commissions/bulkPay (mark multiple as “Paid”)  
5. Master Data  
   • GET /api/v1/masterdata?type=…  
   • POST /api/v1/masterdata (create)  
   • DELETE /api/v1/masterdata/{id}?softDelete=1  
6. Documents  
   • GET /api/v1/leadDocuments?leadId=…  
   • PATCH /api/v1/documents/{docId}/supersede  

Note: The endpoints /api/v1/leads/bulk, /api/v1/commissions/bulkPay, /api/v1/masterdata, and /api/v1/config are newly introduced for the Web Portal. These must be reflected in the L2-LLD-IC for inter-component alignment. Alternatively, existing endpoints can be extended to support bulk and config operations if that approach better fits the integration design.

### 8. References
- Implements overall functional requirements from L1-FRS §§3.1 (User Management), 3.2 (Lead Management), 3.3 (Quotation), 3.4 (Documents), 3.5 (Commissions), 3.6 (Cart & Acceptance – only Admin oversight part), 3.7 (Offline & Notification – poll-based model).  
- Conforms to cross-component interaction design in L2-LLD-IC and external integrations described in L2-LLD-INTG3P.  
- Key client clarifications #1–#11 have been integrated to finalize the Web Portal’s bulk actions, territory assignments, system configuration panel, minimal dashboards, and modern browser support.  

Note on L1-FRS Sections: This document references the official L1-FRS “Functional Requirements Specification” for alignment with user management, lead management, quotation flows, commissions, and offline strategies. Ensure that the correct L1-FRS version is used, matching these section numbers.

### 9. Error Handling
The Web Portal should handle both client-side and server-side errors gracefully:
1. For server-side 4xx or 5xx errors, the portal shall display a clear error message to the user, prompting a retry or further action.  
2. If a network timeout occurs, the Web Portal will prompt a refresh or retry.  
3. A standardized error response format (including status code and error message) must be followed for consistent handling in the UI.


This concludes the revised L3-FRS-WEBPRT specification, incorporating territory-based restrictions for KAM, clarifications on commission handling (aligned with L1-FRS), session management, performance expectations, error handling, and alignment with the 7-year data retention policy.