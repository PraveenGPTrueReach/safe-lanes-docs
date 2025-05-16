## L3-IB-BCKND: Component Implementation Backlog for BCKND

Note: For additional technology constraints and standards, see L3-TSD. All references to concurrency handling in this backlog now align with a last-write-wins model, reserving HTTP 409 only for explicit business-rule conflicts (e.g., resource limits, locked leads), not for generic concurrency mismatches.

This document provides a granular list of implementation items for the Backend (BCKND) component, derived from the Functional Requirements (L3-FRS-BCKND) and Low-Level Design (L3-LLD-BCKND). Each backlog item includes a concise description, relevant technical details, acceptance criteria, references (traceability), dependencies, estimated effort, and priority indicators. The items are grouped by functional areas as defined in L3-FRS-BCKND.

---

## 1. Introduction

The BCKND is a containerized Node.js “modular monolith” responsible for:
- Managing all server-side logic for user authentication, lead/quotation workflows, commissions, and file management (KYC/attachments).  
- Integrating with external services, such as MSG91 for SMS, SendGrid for email, and Azure Blob for file storage.  
- Enforcing security, concurrency rules (primarily last-write-wins with no 409 for concurrency), and auditing changes.

This backlog lists atomic tasks aligned with the current scale (~400–600 concurrent users). Each item references the functional requirements (L3-FRS-BCKND) and design decisions (L3-LLD-BCKND). Where applicable, clarifications and key decisions from L3-KD-BCKND and the consolidated client clarifications are noted.

---

## 2. Implementation Backlog

### 2.1. User Management & Authentication

#### 2.1.1 Implement Phone+OTP Login
- Description: Create endpoint for phone-based OTP login (CP/Customer). Integrate with MSG91 to send OTP and validate against user’s phone.  
- Technical Requirements:  
  - Use environment variables for MSG91 API credentials (L3-LLD-BCKND §3.1, Integration Module).  
  - Limit OTP attempts to 5; lock further attempts for 15 minutes (L3-FRS-BCKND §1.1.3).  
  - Generate JWT (2 hours short-lived, auto-refresh up to ~24 hours for consistency).  
- Acceptance Criteria:  
  - Endpoint: POST /api/v1/auth/login with phone param.  
  - On success, returns HTTP 200 + JWT. On lock or invalid OTP, returns 401/403 with correct error message.  
  - Audit log entry when login is successful or locked (L3-LLD-BCKND §3.6).  
- Traceability: L3-FRS-BCKND §1.1, L3-LLD-BCKND §3.1.  
- Dependencies: MSG91 integration configured; basic user table with unique phone for role=“Customer”.  
- Effort: 5 points  
- Priority: High

#### 2.1.2 Implement Email+Password Login
- Description: Create endpoint for Admin/KAM to log in via email/password. Support password reset flows with SendGrid emails.  
- Technical Requirements:  
  - Endpoint: POST /api/v1/auth/login (detect login method by request body).  
  - Maintain hashed passwords in DB.  
    
  - Standardize JWT as 2-hour token + auto-refresh up to 24 hours for Admin/KAM sessions.  
    
  - Provide “requestPasswordReset” and “completePasswordReset” with a time-bound reset token (L3-FRS-BCKND §1.1.4).  
- Acceptance Criteria:  
  - Correctly enforce password policy (if any).  
  - Send password reset link via SendGrid with limited validity.  
  - Return 401 for invalid/expired tokens during reset.  
- Traceability: L3-FRS-BCKND §1.1, L3-KD-BCKND #1.  
- Dependencies: SendGrid integration, user table with email+password columns.  
- Effort: 5 points  
- Priority: High


#### 2.1.5 (Optional) Multi-Factor Authentication for Admin/KAM
- Description: Provide an optional additional factor (e.g., TOTP or SMS OTP) for Admin/KAM logins to enhance security, especially for handling sensitive KYC data.
- Technical Requirements:
  - Configure a 2FA mechanism that can be toggled on/off via environment variables or application settings.
  - If enabled, after successful email+password, prompt for a second factor.
- Acceptance Criteria:
  - When 2FA is on, Admin/KAM must pass the second factor to obtain a JWT.
  - Return 401 if the second factor is invalid or expired.
- Traceability: L3-FRS-BCKND (security considerations), L3-TSD for potential libraries.
- Dependencies: Email or SMS integration, user record storing 2FA seeds if TOTP.
- Effort: 5 points
- Priority: Low


#### 2.1.3 Enforce Phone Number Uniqueness for Customers
- Description: On registration or profile update for role=“Customer,” ensure phone is unique. Return 409 if violated (unless Admin override). This constraint applies only to the 'Customer' role, not CP.  
- Technical Requirements:  
  - Database constraint or index + application logic.  
  - Admin override flag to bypass uniqueness if required.  
  - CP phone uniqueness is not enforced by default based on final clarifications.  
- Acceptance Criteria:  
  - Attempting to use a taken phone for a new “Customer” returns 409 by default.  
  - Admin requests with override=true can bypass.  
- Traceability: L3-FRS-BCKND §1.1 (revised for Customer-only phone uniqueness).  
- Dependencies: Existing user table structure.  
- Effort: 2 points  
- Priority: Medium

#### 2.1.4 Token Invalidation & Role Check
- Description: Implement “blocked_token” check in each authorized endpoint. If blocked=true for a user, return 403.  
- Technical Requirements:  
  - BCKND verifies the JWT but also queries user record for `blocked_token`.  
  - Ensure role-based access is verified on every request (L3-LLD-BCKND §3.1, Auth Module).  
- Acceptance Criteria:  
  - Setting a user’s `blocked_token`=true denies further API access.  
  - Return HTTP 403 with a meaningful error.  
  - Correct roles are enforced (Admin vs. CP vs. KAM vs. Customer).  
- Traceability: L3-FRS-BCKND §1.1, L3-LLD-BCKND §6.  
- Dependencies: Basic user table with `blocked_token` field.  
- Effort: 3 points  
- Priority: Medium

---

### 2.2. Lead Management

#### 2.2.1 Create Lead Endpoint
- Description: Implement POST /api/v1/leads to create a new lead record (status=“New,” assigned to CP).  
- Technical Requirements:  
  - Validate required fields (customer name, phone, etc.).  
  - Enforce online-only creation (L3-FRS-BCKND §1.2.1).  
  - Live or placeholder territory check for assignment.  
- Acceptance Criteria:  
  - On success, returns 201 + lead ID.  
  - Audit log capturing creation event.  
  - If offline or invalid data, return error code (400 or 403).  
- Traceability: L3-FRS-BCKND §1.2, L3-LLD-BCKND §3.2.  
- Dependencies: Auth working; lead DB schema.  
- Effort: 4 points  
- Priority: High

#### 2.2.2 Update Lead Status (Forward & Override)
- Description: Implement PATCH /api/v1/leads/{id} to update lead status or fields (remarks, follow-up date). Support an override param for backward/unusual transitions.  
- Technical Requirements:  
  - Standard forward moves (New → In Discussion → Physical Meeting → Customer Accepted → Won) do not need override.  
  - Admin or KAM can override with override=true for backward transitions, logging the override reason.  
    
  Use last-write-wins for concurrency; only return 409 for locked lead or other business-rule conflicts (e.g., lead is in a terminal state).  
- Acceptance Criteria:  
  - Audit log includes old_status, new_status, override if used.  
  - Return 200 on success, or an appropriate error if the lead is locked or no override is provided.  
- Traceability: L3-FRS-BCKND §1.2.  
- Dependencies: Database for leads, minimal concurrency checks.  
- Effort: 4 points  
- Priority: High

#### 2.2.3 Reassign Lead to Another CP
- Description: Implement PATCH /api/v1/leads/{id}/reassign?cpId=XYZ to change Channel Partner. Possibly check territory or stateCode before reassign.  
- Technical Requirements:  
  - If mismatch in territory, Admin can supply override.  
  - Log reassign event in the audit trail.  
- Acceptance Criteria:  
  - CP changes reflect in leads table.  
  - If override is required and not provided, return 403 or 409 (as a business conflict).  
- Traceability: L3-FRS-BCKND §1.2.3, Clarification #8 if territory reassign is relevant.  
- Dependencies: CP user table, territory logic.  
- Effort: 3 points  
- Priority: Medium





---

### 2.3. Quotation Management

#### 2.3.1 Create & Update Quotation
- Description: Build endpoints for POST /api/v1/quotations and PATCH /api/v1/quotations/{id}. CP must finalize in one request; Admin/KAM can save partial drafts.  
- Technical Requirements:  
  - Quotation includes systemPrice, margin%, references leadId.  
  - For partial/draft flow, store status=“Draft” for Admin/KAM. CP has only final submission.  
    
  Apply last-write-wins for concurrency; 409 only if a locked or final quotation is forcibly altered without override.  
- Acceptance Criteria:  
  - 201 Created on successful new quotation.  
  - Audit log for creation/update events.  
- Traceability: L3-FRS-BCKND §1.3.1, L3-LLD-BCKND §3.3.  
- Dependencies: lead must exist, no version-based concurrency unless specifically locked.  
- Effort: 4 points  
- Priority: High

#### 2.3.2 Share & Accept Quotation
- Description:  
  - PATCH /api/v1/quotations/{id}/share: sets status=“Shared,” notifies customer.  
  - PATCH /api/v1/quotations/{id}/accept: sets status=“Accepted,” triggers lead status=“Customer Accepted.”  
- Technical Requirements:  
  - If a quotation is “Accepted,” lead auto-updates to “Customer Accepted.”  
  - Return 409 if the quote was already accepted or lead is locked (business-rule conflict).  
- Acceptance Criteria:  
  - Shared → Customer sees it in CUSTAP.  
  - Accepted → lead is updated, event logged.  
  - If override is needed (Admin forcibly accepting?), log with reason.  
- Traceability: L3-FRS-BCKND §1.3.1.  
- Dependencies: CP or Admin role to share, customer or admin to accept.  
- Effort: 4 points  
- Priority: High

#### 2.3.3 PDF Generation & Upload
- Description: Implement background process for finalizing a quotation, generating a PDF, and uploading to Azure Blob.  
- Technical Requirements:  
  - Use child_process or worker thread (2–3 concurrency).  
  - If generation fails, return HTTP 500 (server error), not 409.  
  - Store the PDF blob path in the quotations table.  
- Acceptance Criteria:  
  - Automatic or manual re-try if PDF generation fails.  
  - Upload to Hot tier in Azure Blob.  
  - Confirm reference in DB is valid.  
- Traceability: L3-FRS-BCKND §1.3.5, L3-LLD-BCKND §3.4.  
- Dependencies: Azure Blob credentials, new container or existing container.  
- Effort: 6 points  
- Priority: Medium

---

### 2.4. Document & KYC Management

#### 2.4.1 KYC Document Upload with SAS
- Description: Provide endpoint (POST /api/v1/kycDocuments) returning a short-lived SAS token for the client to upload to Blob.  
- Technical Requirements:  
  - Exclude this from the lead’s “7 documents max” limit (KYC is separate).  
  - Overwrite in place if doc is “Rejected” and re-uploaded. Keep a single record, referencing Clarification #15 for unlimited re-uploads if rejected.  
- Acceptance Criteria:  
  - Return 403 if user is not authorized.  
  - 409 if file size > 10 MB or type not in [PDF, JPG, PNG].  
  - Rejected doc re-upload overwrites the existing reference.  
- Traceability: L3-FRS-BCKND §1.4.1–1.4.3, Clarification #15.  
- Dependencies: Azure Blob integration, doc metadata table.  
- Effort: 5 points  
- Priority: High

#### 2.4.2 Lead Document Upload (Max 7)
- Description: Implement POST /api/v1/leadDocuments to get SAS for uploading lead-level files. Enforce a 7-file limit per lead.  
- Technical Requirements:  
  - Count existing doc references for the lead. If 7 reached, return 409.  
  - Must handle concurrency (e.g., if two files are uploading simultaneously).  
- Acceptance Criteria:  
  - Return short-lived SAS token.  
  - If limit exceeded, return 409 with “max doc limit” message.  
  - File references stored in DB with docType, leadId.  
- Traceability: L3-FRS-BCKND §1.4.1–1.4.2, L3-LLD-BCKND §3.4.  
- Dependencies: lead existence, doc metadata table.  
- Effort: 4 points  
- Priority: High

#### 2.4.3 Document Retention & Deletion
- Description: Provide logic to keep docs for 7 years. Admin can delete accidental/wrongful uploads.  
- Technical Requirements:  
  - On deletion request, remove DB reference, call Blob delete, log in audit.  
  - Optionally implement a cron or script to purge old docs.  
- Acceptance Criteria:  
  - Admin-only deletion endpoint.  
  - Deletion logs in the audit.  
  - Confirm file removed from Blob.  
- Traceability: L3-FRS-BCKND §1.4.4, L3-LLD-BCKND §7.  
- Dependencies: Azure Blob, lead or doc references.  
- Effort: 4 points  
- Priority: Medium

---

### 2.5. Commission & Payout

#### 2.5.1 Commission Calculation & Record Creation
- Description: When a lead is moved to “Executed,” if a final accepted quotation is present, create a commission record using Commission=systemPrice * margin%.  
- Technical Requirements:  
  - Commission module references final quotation ID.  
  - Single lumpsum approach: Pending → Approved → Paid.  
- Acceptance Criteria:  
  - Commission record is created automatically upon “Executed.”  
  - 0% or 100% margin edge cases yield domain error if unexpected.  
  - Audit logs the creation with relevant details.  
- Traceability: L3-FRS-BCKND §1.5.1.  
- Dependencies: lead status=“Executed,” final quotation ID.  
- Effort: 5 points  
- Priority: High

#### 2.5.2 Approve & Pay Commission
- Description: Implement endpoints: PATCH /api/v1/commissions/{id}/approve (Admin/KAM) and PATCH /api/v1/commissions/{id}/markPaid (Admin).  
- Technical Requirements:  
  - Validate role-based permissions.  
  - Once “Paid,” no further changes allowed unless override=Admin.  
  - Manual Overwrite approach if CP is reassigned mid-process; refer to relevant clarifications or territory logic (#8) for correct approach.  
- Acceptance Criteria:  
  - Approve sets status=“Approved,” pay sets status=“Paid.”  
  - Commission cannot be partially paid.  
  - Audit log captures date, user, UTR info.  
- Traceability: L3-FRS-BCKND §1.5, Clarification #8 (territory reassign if relevant).  
- Dependencies: Commission table, minimal concurrency checks for final status changes.  
- Effort: 4 points  
- Priority: Medium

---

### 2.6. Customer Service Requests & Quotation Acceptance

#### 2.6.1 Multi-Service Cart Submission
- Description: From CUSTAP, create a new lead with multiple requested services. Each service can lead to a separate line item in the lead or a sub-record.  
- Technical Requirements:  
  - Map submitted services into a lead-level structure.  
  - Potential expansions for partial quotes, but currently store them as line items.  
- Acceptance Criteria:  
  - POST /api/v1/leads or /api/v1/serviceRequests → new lead with all details.  
  - Audit log creation.  
- Traceability: L3-FRS-BCKND §1.6.1.  
- Dependencies: lead table updated for multi-service (extra columns or child table).  
- Effort: 3 points  
- Priority: Medium

#### 2.6.2 Quotation Acceptance Flow via Customer App
- Description: Implement logic to let the customer accept a “Shared” quotation from the CUSTAP, automatically setting lead=“Customer Accepted.”  
- Technical Requirements:  
  - Checks if the quote is in status=“Shared.”  
  - Immediately update lead status.  
  - No real-time push, poll-based or daily notifications suffice.  
- Acceptance Criteria:  
  - PATCH /api/v1/quotations/{id}/accept with role=“Customer.”  
  - Lead transitions to “Customer Accepted.”  
  - 409 if quote is locked or not shareable.  
- Traceability: L3-FRS-BCKND §1.6.2.  
- Dependencies: Quotation is “Shared,” lead must not be final/won.  
- Effort: 3 points  
- Priority: High

---

### 2.7. Offline & Notification Requirements

#### 2.7.1 Reject Writes if Offline
- Description: Ensure the BCKND does not accept offline updates (the mobile apps must only call APIs when online).  
- Technical Requirements:  
  - Optionally detect if the front-end passes an “offlineFlag”? Or simply rely on network connectivity.  
  - Return 400 if “offlineFlag” is present or handle it on front-end.  
- Acceptance Criteria:  
  - No direct acceptance of offline requests.  
  - Documented that front-ends do not attempt offline writes.  
- Traceability: L3-FRS-BCKND §1.7.1.  
- Dependencies: Mobile app logic.  
- Effort: 1 point  
- Priority: Low



---

## 3. General Technical & Design Tasks

### 3.1 Database Schema Implementations
- Description: Create necessary tables (users, leads, quotations, commissions, documents, audit_log, etc.) in Azure PostgreSQL.  
- Technical Requirements:  
  - Use ID sequences with optional prefixes (LEAD-1000, QUOTE-1000).  
  - “updated_at” columns are used for logging but do not enforce concurrency 409 (last-write-wins, except for locked resources).  
- Acceptance Criteria:  
  - DB migrations or scripts versioned in source code.  
  - Verified local/staging environment.  
- Traceability: L3-LLD-BCKND §3.3.  
- Dependencies: None (foundation).  
- Effort: 5 points  
- Priority: High

### 3.2 Audit Logging Implementation
- Description: Implement a single, comprehensive audit_log table storing old/new data for leads, quotations, KYC, etc.  
- Technical Requirements:  
  - On create/update/delete, store the relevant JSON snapshot.  
  - Each service method calls AuditLogger.  
- Acceptance Criteria:  
  - Logs appear for critical changes, capturing userId, timestamp, oldValue, newValue.  
- Traceability: L3-FRS-BCKND §3, L3-LLD-BCKND §3.6.  
- Dependencies: DB schema in place.  
- Effort: 4 points  
- Priority: Medium

### 3.3 Rate Limiting & Basic Security
- Description: Enforce simple rate limiting (e.g., 100 requests/min/IP) on login or file upload endpoints to prevent brute force.  
- Technical Requirements:  
  - Return 429 if rate is exceeded.  
  - Could use a library (e.g., express-rate-limit).  
- Acceptance Criteria:  
  - Observed 429 if threshold is hit.  
  - Security logs or counters reset after a short period.  
- Traceability: L3-NFRS-BCKND §4.4.  
- Dependencies: Infrastructure or Node library choice.  
- Effort: 3 points  
- Priority: Medium

### 3.4 Worker Threads for PDF Generation
- Description: Use Node’s worker threads or child_process for generating PDF to avoid blocking the main event loop.  
- Technical Requirements:  
  - Limit concurrency to 2–3 workers.  
  - Provide fallback or queue if all workers are busy.  
- Acceptance Criteria:  
  - PDF generation tested with multiple concurrent requests.  
  - No significant slowdown of main API.  
- Traceability: L3-LLD-BCKND §3.4, L3-KD-BCKND #6.  
- Dependencies: Quotation PDF module.  
- Effort: 5 points  
- Priority: Medium

### 3.5 Deployment & Configuration
- Description: Containerize the BCKND, set environment variables for DB, MSG91, SendGrid, etc. Deploy to Azure Web App.  
- Technical Requirements:  
  - Single region (manual DR).  
  - Ensure TLS is enforced.  
- Acceptance Criteria:  
  - Running container on staging, then production with correct environment config.  
  - Basic health checks pass.  
- Traceability: L3-LLD-BCKND §3.5, L3-KD-BCKND #2.  
- Dependencies: Dockerfile, Azure subscription.  
- Effort: 3 points  
- Priority: Medium

---

## 4. Effort & Priority Summary

Below is a quick summary (subject to refinement in project planning):

- High-priority (HP) items:  
  - User login endpoints (OTP/email), create/update leads, create quotations, doc upload limitations, commission calculation (on “Executed”), DB schema.  
- Medium-priority (MP) items:  
  - Lead reassign, PDF generation worker threads, , doc retention logic, rate limiting, partial concurrency checks for locked resources.  
- Low-priority (LP) items:  
  - , offline writes rejection, , optional MFA for Admin/KAM.

All items align with a pragmatic approach for ~400–600 concurrent users. No microservices or complex distributed solutions are proposed, as the current scale and design decisions (L3-KD-BCKND) favor a single modular monolith in a single region, with moderate concurrency and manual DR.

---

## 5. Conclusion

This backlog details atomic implementation tasks for the BCKND, referencing functional requirements, low-level design, and updated clarifications that unify concurrency handling to last-write-wins. Items beyond the original scope, such as lead-merge or automated overdue follow-up notifications, are removed or deferred. The solution remains suitable for the stated concurrency level (400–600) while permitting moderate future growth without excessive complexity.