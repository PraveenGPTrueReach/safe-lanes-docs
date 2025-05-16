
## L3-IB-WEBPRT: Component Implementation Backlog for WEBPRT

### 1. Introduction
This document provides a granular list of implementation tasks (user stories and technical tasks) for the Solarium Web Portal (WEBPRT). These items are derived from the L3-FRS-WEBPRT (Functional Requirements), L3-LLD-WEBPRT (Low-Level Design), L3-KD-WEBPRT (Key Decisions), and associated client clarifications. Each backlog item includes references, acceptance criteria, dependencies, and an effort estimate (in relative points or small/medium/large) to guide development and ensure traceability.

The scope focuses on pragmatic solutions for the current ~400–600 concurrent user scale, with minimal overengineering. All advanced or future-scope details are deferred unless explicitly required.

---

### 2. Implementation Backlog

#### IB-WEBPRT-001: Admin & KAM Login Screen
• Description  
  Implement the login screen and underlying authentication logic for Admin/KAM users. This includes the email/password form, error handling, and session initiation.  

• Technical Requirements  
  – Use POST /api/v1/auth/login as specified in L3-FRS-WEBPRT §3.1.  
  – Store JWT in session storage (per L3-KD-WEBPRT #3) and enforce a 30-minute inactivity timeout.  
  – Display separate success flows or error messages based on server responses (401, 403, 5xx).  

• Acceptance Criteria  
  1. Admin/KAM can successfully authenticate and see role-appropriate UI.  
  2. Session expires after 30 minutes of inactivity, requiring re-login.  
  3. Inactive or expired tokens show “Session expired” and redirect users to the login page.  

• References  
  – L3-FRS-WEBPRT §3.1  
  – L3-LLD-WEBPRT §§3.1, 6.1 (security design)  
  – L3-KD-WEBPRT (single SPA approach, short-lived tokens)  

• Dependencies  
  – Backend authentication endpoints (BCKND).  

• Priority / Effort  
  – Priority: High (core security and access).  
  – Effort: 3 points (small-medium).

---

#### IB-WEBPRT-002: Role-Based Navigation & UI Controls
• Description  
  Create a unified SPA with conditional rendering based on user role (Admin vs. KAM). Hide or disable Admin-only features for KAM.  

• Technical Requirements  
  – Implement route guards in React that check user.role before granting access (L3-LLD-WEBPRT §§3.1, 3.4.1).  
  – Provide a shared navigation component that conditionally shows Admin menus.  

• Acceptance Criteria  
  1. UI menu changes dynamically for Admin vs. KAM roles.  
  2. KAM is denied access to Admin screens or endpoints (403 returned if forced).  
  3. Basic tests confirm correct gating in place.  

• References  
  – L3-FRS-WEBPRT §3.1 (role-based visibility), L3-KD-WEBPRT #1 & #8.  
  – L3-LLD-WEBPRT §§3.1, 3.3 (module descriptions).  

• Dependencies  
  – IB-WEBPRT-001 (Authentication).  

• Priority / Effort  
  – Priority: High (foundation for role-based features).  
  – Effort: 3 points (small-medium).

---

#### IB-WEBPRT-003: Session Timeout & Token Handling
• Description  
  Implement front-end logic to track inactivity and invoke re-authentication after 30 minutes.  

• Technical Requirements  
  – Track user activity (mouse movement, clicks, etc.) and reset a timer.  
  – On timeout, clear session storage and route to /login.  
  – Catch 401 or 403 responses from the backend and handle forced logout.  

• Acceptance Criteria  
  1. Users are logged out upon 30 minutes of inactivity (strict).  
  2. On receiving 401 from any API call, the portal automatically logs out and prompts a re-login.  

• References  
  – L3-FRS-WEBPRT §3.1 (session management).  
  – L3-LLD-WEBPRT §§6.1 (security) & 3.4.1 (route guard flow).  

• Dependencies  
  – IB-WEBPRT-001 (JWT login) and IB-WEBPRT-002 (role gating).  

• Priority / Effort  
  – Priority: High.  
  – Effort: 2 points (small).

---

#### IB-WEBPRT-004: Master Data CRUD (Admin-Only)
• Description  
  Build the Admin-only screens and forms for creating, updating, and soft-deleting master data records (panels, inverters, fees, etc.).  

• Technical Requirements  
  – Implement GET/POST/PATCH/DELETE calls to /api/v1/masterdata (L3-FRS-WEBPRT §3.2).  
  – Provide a basic data grid + detail form.  
  – Log create/update/delete events with timestamps (basic audit).  

• Acceptance Criteria  
  1. Admin can add or edit master data items.  
  2. Soft-deleted items no longer appear in the main list but remain in the audit logs.  
  3. KAM cannot access Master Data screens.  

• References  
  – L3-FRS-WEBPRT §3.2.  
  – L3-LLD-WEBPRT §§3.1, 3.5 (services, MasterData module).  

• Dependencies  
  – IB-WEBPRT-002 (Admin gating).  

• Priority / Effort  
  – Priority: Medium.  
  – Effort: 3 points (small-medium).

---

#### IB-WEBPRT-005: Territory-Based Lead Visibility & Reassignment
• Description  
  Implement lead list filtering based on KAM’s assigned territory. Admin can reassign territories or update a KAM’s assigned geographic region.  

• Technical Requirements  
  – Query leads using GET /api/v1/leads?territory=KAM_TERRITORY or rely on server-side filtering (L3-FRS-WEBPRT §3.3).  
  – Implement immediate revoke of leads if territory is changed (Client Clarification #2).  

• Acceptance Criteria  
  1. KAM only sees leads in their assigned territory.  
  2. Updating a KAM’s territory instantly hides leads from the old region.  
  3. Admin can view all leads and freely reassign them.  

• References  
  – L3-FRS-WEBPRT §3.3 (Lead Management), L3-KD-WEBPRT #2.  
  – L3-LLD-WEBPRT §§3.2 (LeadService code).  

• Dependencies  
  – IB-WEBPRT-002 (role-based gating).  
  – Admin’s territory assignment screen.  

• Priority / Effort  
  – Priority: High.  
  – Effort: 5 points (medium).

---

#### IB-WEBPRT-006: Bulk Lead Actions (Status Update & CP Reassignment)
• Description  
  Enable multi-select of up to 50 leads for bulk operations: “Update Status” or “Reassign CP.”  

• Technical Requirements  
  – UI to select multiple leads (checkbox row selection) up to 50.  
  – PATCH /api/v1/leads/bulk with action type (UpdateStatus or Reassign).  
  – Provide success/failure feedback for each item.  

• Acceptance Criteria  
  1. Bulk actions successfully update or reassign up to 50 leads at once.  
  2. Show partial success if some leads fail.  
  3. Conform to territory restrictions (403 if out of KAM scope).  

• References  
  – L3-FRS-WEBPRT §3.3 (client clarification #1—bulk updates).  
  – L3-KD-WEBPRT #2, #4 (last-write-wins concurrency).  

• Dependencies  
  – Basic lead listing done, territory logic in place.  

• Priority / Effort  
  – Priority: Medium.  
  – Effort: 4 points (medium).

---

#### IB-WEBPRT-007: Lead Timeline & CSV Export
• Description  
  Present read-only timeline for each lead, capturing status changes, remarks, and reassignments. Also implement CSV export for leads.  

• Technical Requirements  
  – Lead timeline: GET /api/v1/leads/{leadId}/timeline (L3-FRS-WEBPRT §3.3).  
  – CSV export: Must reflect current filters (status/date range) and not rely on local data caching.  

• Acceptance Criteria  
  1. The timeline UI displays all historical changes with timestamps.  
  2. Admin/KAM can export the currently filtered lead list to CSV (including a “Download CSV” button).  
  3. Large CSV downloads handle up to the limit specified by the backend.  

• References  
  – L3-FRS-WEBPRT §3.3 (timeline, client clarification #8), §3.8 (CSV export).  
  – L3-LLD-WEBPRT §§3.1, 7 (reporting & dashboards).  

• Dependencies  
  – Basic lead listing retrieval.  

• Priority / Effort  
  – Priority: Medium.  
  – Effort: 4 points (medium).

---

#### IB-WEBPRT-008: Quotation Draft & Finalization
• Description  
  Implement the ability to create and save draft quotations, then finalize (share) them with customers.  

• Technical Requirements  
  – POST /api/v1/quotations with "status":"Draft" to store indefinitely.  
  – PATCH /api/v1/quotations/{quoteId} to finalize (“Shared”).  
  – Only one “Accepted” quotation per lead; apply server validations.  

• Acceptance Criteria  
  1. KAM/Admin can create a draft, then finalize it without re-entering data.  
  2. Once a quotation is “Shared,” it is locked from further edits.  
  3. Proper error or warning if a second “Accepted” quote is attempted.  

• References  
  – L3-FRS-WEBPRT §3.4 (Quotation Management).  
  – L3-KD-WEBPRT #6 (domain logic in backend).  

• Dependencies  
  – Lead data must exist; territory checks for KAM.  

• Priority / Effort  
  – Priority: Medium.  
  – Effort: 5 points (medium).

---

#### IB-WEBPRT-009: Commission Marking & Overrides
• Description  
  Provide an Admin-only interface to view commissions, override amounts, and mark them “Paid.” Once paid, no “undo” is allowed (Client Clarification #3).  

• Technical Requirements  
  – PATCH /api/v1/commissions/bulkPay for multi-record updates.  
  – Data grid showing “Pending” or “Paid” status, plus override entry for new amounts.  
  – “Strict No Undo” once paid: must create a separate adjusting record if needed.  

• Acceptance Criteria  
  1. Admin can override commission amounts before paying.  
  2. Bulk pay multiple “Pending” commissions at once.  
  3. System disallows reverting “Paid” status.  

• References  
  – L3-FRS-WEBPRT §3.5 (Commission & Payout).  
  – L3-KD-WEBPRT #3 (strict “No Undo”).  

• Dependencies  
  – IB-WEBPRT-002 (Admin gating).  

• Priority / Effort  
  – Priority: Medium.  
  – Effort: 4 points (medium).

---

#### IB-WEBPRT-010: System Configuration Panel & Poll-Based Reload
• Description  
  Build an Admin page to toggle system “feature flags” or disclaimers. Implement poll-based reload so existing sessions pick up changes ~every 5–10 minutes.  

• Technical Requirements  
  – A config table in the backend (PATCH /api/v1/config).  
  – Portal fetches updated flags on an interval (clarification #1).  
  – UI updates features accordingly without forcing immediate reload on all sessions.  

• Acceptance Criteria  
  1. Admin can toggle flags (e.g., “Enable Beta Dashboard”).  
  2. Active Admin/KAM sessions see updated config within 10 minutes.  
  3. No disruption to in-progress tasks.  

• References  
  – L3-FRS-WEBPRT §3.6 (client clarification #5, #10).  
  – Client Clarification #1 (real-time config toggle).  
  – L3-KD-WEBPRT #2 (poll-based approach).  

• Dependencies  
  – IB-WEBPRT-002 (role gating).  

• Priority / Effort  
  – Priority: Medium.  
  – Effort: 4 points (medium).

---

#### IB-WEBPRT-011: Document Viewing & KYC Approvals
• Description  
  Implement a module for Admin/KAM to view, approve, or supersede lead documents (except for locked KYC).  

• Technical Requirements  
  – GET /api/v1/leadDocuments?leadId=...  
  – GET /api/v1/documents/sas?docId=... to retrieve short-lived SAS URL.  
  – PATCH /api/v1/documents/{docId}/supersede (if not locked).  

• Acceptance Criteria  
  1. Documents are displayed with their current “approved” or “superseded” status.  
  2. Approved KYC docs cannot be superseded or deleted.  
  3. Admin sees all, KAM sees only territory leads (403 otherwise).  

• References  
  – L3-FRS-WEBPRT §3.7.  
  – L3-LLD-WEBPRT §§3.6, 7 (DocumentsService).  

• Dependencies  
  – Basic lead retrieval & territory logic.  

• Priority / Effort  
  – Priority: Medium.  
  – Effort: 5 points (medium).

---

#### IB-WEBPRT-012: Dashboard & Minimal Reporting
• Description  
  Provide a minimal dashboard summarizing leads per status, commissions pending vs. paid, and basic CSV exports for further analysis.  

• Technical Requirements  
  – Summaries from existing endpoints (e.g., leads in status counts, commission totals).  
  – No complex charts by default; simple table or numeric displays.  

• Acceptance Criteria  
  1. Admin/KAM see counts of leads by status, total pending commissions.  
  2. CSV export button for each table if needed.  
  3. Poll-based or on-demand refresh.  

• References  
  – L3-FRS-WEBPRT §3.8 (Reporting & Dashboards).  
  – L3-LLD-WEBPRT §§4.2, 7 (UI & aggregated data).  

• Dependencies  
  – Lead, commission retrieval.  

• Priority / Effort  
  – Priority: Low.  
  – Effort: 3 points (small-medium).

---

#### IB-WEBPRT-013: Basic Validation & Error Handling
• Description  
  Enforce essential UI validations (e.g., required fields, date ranges) and gracefully handle server errors (4xx, 5xx).  

• Technical Requirements  
  – Show descriptive error toasts when server returns 400/403/500.  
  – Validate mandatory fields for lead updates, quotations (draft to final), commissions, etc.  

• Acceptance Criteria  
  1. User is prompted on incomplete fields or invalid data before submission.  
  2. Server-side failures produce a clear error toast or modal.  
  3. No unhandled rejections in console logs during normal usage.  

• References  
  – L3-FRS-WEBPRT §§3.6 (validations), 9 (error handling).  
  – L3-LLD-WEBPRT §§7 (error handling strategy).  

• Dependencies  
  – All forms & modules hooking into backend.  

• Priority / Effort  
  – Priority: Medium.  
  – Effort: 2 points (small).

---

#### IB-WEBPRT-014: Performance & Scalability Considerations
• Description  
  Ensure the Web Portal remains responsive under ~60–90 concurrent Admin/KAM roles (subset of the total 400–600).  

• Technical Requirements  
  – Minimize unnecessary re-renders or large in-memory data sets.  
  – Use pagination (offset/limit) for large lead/quotation lists.  
  – Basic load testing with ~80–100 concurrent test users in QA environment.  

• Acceptance Criteria  
  1. 95% of key pages load within 2–3 seconds at normal usage.  
  2. Bulk operations or CSV exports remain under ~10 seconds for large sets.  
  3. No major performance degradation with ~80 concurrent Admin/KAM sessions.  

• References  
  – L3-NFRS-WEBPRT §2 (Performance).  
  – L3-LLD-WEBPRT §§3.4, 5 (performance strategies, polling).  

• Dependencies  
  – All major features implemented.  

• Priority / Effort  
  – Priority: Medium.  
  – Effort: Ongoing tasks (medium).

---

#### IB-WEBPRT-015: Audit & Logging of Admin Actions
• Description  
  Record critical Admin actions (e.g., territory changes, commissions paid, master data edits) for compliance with the 7-year retention.  

• Technical Requirements  
  – On each Admin workflow, log the action in the timeline or a dedicated audit endpoint.  
  – Show minimal logs in UI if needed (or rely on backend queries).  

• Acceptance Criteria  
  1. Timestamps, user ID, and action details are captured.  
  2. Logs are persisted per the 7-year data requirement (viewable or exportable if needed).  

• References  
  – L3-FRS-WEBPRT §3.2, 3.3, 3.5 for mention of basic audit trails.  
  – L3-KD-WEBPRT #4 (last-write-wins plus logs).  

• Dependencies  
  – Existence of the backend logging infrastructure.  

• Priority / Effort  
  – Priority: Medium.  
  – Effort: 3 points (small-medium).

---

### 3. Conclusion

These 15 backlog items collectively address the core functionalities and design elements of the Solarium Web Portal (WEBPRT). Each item references the relevant sections in L3-FRS-WEBPRT, L3-LLD-WEBPRT, L3-KD-WEBPRT, and client clarifications, ensuring traceability. By fulfilling these items in sequence, the development team will deliver a fully functional Admin/KAM web portal aligned with the scale, performance, and security requirements defined for ~400–600 concurrent users (with ~10–15% for Admin/KAM usage).

All acceptance criteria must be validated in the staging environment, with partial load testing for concurrency and security checks before final production deployment. Additional tasks or refinements can be appended if new clarifications arise or scale requirements change.
```
