## L3-FRS-BCKND: Functional Requirements Specification (FRS) for BCKND

This document describes the specific functionalities and behaviors the Backend (BCKND) component of the Solarium Green Energy solution must implement to meet the system’s business requirements. It references the high-level requirements defined in L1-FRS and incorporates client clarifications pertinent to backend operations.

---

### Introduction

The BCKND is a containerized Node.js “modular monolith” responsible for all business logic, data persistence, file management, and integration with external services (SMS, email, and Azure Blob). It provides secure REST APIs to the Channel Partner App (CPAPP), Customer App (CUSTAP), and Web Portal (WEBPRT), enforcing authorization based on user roles (Admin, KAM, CP, Customer).

---

### Scope

• Implement all server-side logic to manage leads, quotations, documents, commissions, and user/authentication flows described in L1-FRS.  
• Expose REST endpoints for other components to consume, adhering to the functional requirements outlined below.  
• Ensure data integrity, audit trails, concurrency control, and minimal offline usage per the current scale of ~400–600 concurrent users.  
• Integrate with third-party services (MSG91, SendGrid, Azure Blob) for OTP delivery, email dispatch, and secure file storage.

---

### 1. BCKND Functional Requirements

Below are the primary functionalities the BCKND must offer, mapped to the relevant sections of L1-FRS.

#### 1.1. User Management & Authentication (Implements L1-FRS §3.1)

1. The BCKND shall provide endpoints for user login, logout, and role-based access checks:  
   - POST /api/v1/auth/login (supports both phone+OTP for CP/Customer, and email+password for Admin/KAM).  
   - BCKND issues short-lived JWT tokens (typically ~24 hours) with role claims.  
   - Users must re-authenticate when the token expires or is invalid.  

2. The BCKND shall enforce phone-number uniqueness for CP/Customer roles (unless overridden by an Admin), returning HTTP 409 if a duplicate is attempted.  

3. The BCKND shall lock OTP attempts (for CP/Customer) after five failed tries for 15 minutes (reference: Client Clarification #17 for 2-retry approach on external SMS calls).

4. For Admin/KAM password resets, provide a time-bound reset link via SendGrid (reference: Client Clarification #16).  
   - POST /api/v1/auth/requestPasswordReset  
   - PATCH /api/v1/auth/completePasswordReset  
   - Single-use token valid for X minutes; expired or invalid tokens return 401.

#### 1.2. Lead Management (Implements L1-FRS §3.2)

1. The BCKND shall expose endpoints to create, retrieve, update, and delete leads:
   - POST /api/v1/leads → create a new lead (online only).  
   - GET /api/v1/leads → list/filter leads by status, assigned user, etc.  
   - PATCH /api/v1/leads/{id} → update fields like status, remarks, next follow-up date.  

2. The BCKND shall maintain a semi-flexible status transition model:
   - For forward progress (e.g., New → In Discussion → Physical Meeting → Customer Accepted → Won), no special override is required.  
   - Backward or “unusual jump” transitions (e.g., Won → In Discussion) require an override parameter from Admin/KAM, and all changes are logged in the audit trail (Client Clarification #4).

3. The BCKND shall allow reassigning leads to different Channel Partners if the “stateCode” matches (Client Clarification #8).  
   - PATCH /api/v1/leads/{id}/reassign?cpId=XYZ  
   - Enforce territory-based or stateCode-based checks; override permitted for Admin if mismatched.

4. The BCKND shall recognize phone/email duplicates in leads but not forcibly block them (Client Clarification #10). Admin can manually merge or close duplicates.

5. The BCKND shall provide an optional daily scheduled job to notify CP/KAM of overdue next-follow-up leads via email or minimal SMS (Client Clarification #11).

#### 1.3. Quotation Management (Implements L1-FRS §3.3)

1. The BCKND shall implement endpoints to create, update, share, and accept quotations:
   - POST /api/v1/quotations (CP must finalize in one go; Admin/KAM can save drafts).  
   - PATCH /api/v1/quotations/{id}/share  
   - PATCH /api/v1/quotations/{id}/accept  

2. Commission Calculation: The final lumpsum commission is derived using a percentage-based formula:  
   - commission = systemPrice × margin% (Client Clarification #5).  
   - This margin% is stored in the quotation record.

3. Subsidy Calculations: Admin-manageable subsidy slabs in a table to be applied automatically in the wizard process (Client Clarification #6).

4. If a quote is accepted by the customer, the BCKND updates the associated lead to “Customer Accepted.” A CP or Admin must then finalize the lead as “Won.”

5. PDF Generation:  
   - On finalizing a quotation, the BCKND spawns a child process or worker to generate the PDF and upload it to Azure Blob.  
   - If generation fails, return an HTTP 500 or 409 error. The user can retry manually (Client Clarification #13).

#### 1.4. Document & KYC Management (Implements L1-FRS §3.4)

1. The BCKND shall provide secure file upload and retrieval:  
   - POST /api/v1/kycDocuments → request for a SAS token, then PUT to Azure Blob.  
   - GET /api/v1/documents/sas?docId=XYZ → retrieve a short-lived read token for download.  

2. Up to seven lead-level documents (each ≤10 MB), stored in Azure Blob (Hot tier) with references in PostgreSQL.  

3. KYC Re-Uploads: If a document is “Rejected,” the system allows unlimited re-uploads (Client Clarification #15).  

4. Document Retention & Deletion:  
   - Retain documents for seven years by default.  
   - Admin can delete wrongful or accidental uploads anytime, with entries logged in the audit timeline (Client Clarification #3).  
   - A scheduled job or manual script shall purge or archive documents older than seven years.

#### 1.5. Commission & Payout (Implements L1-FRS §3.5)

1. The BCKND shall derive commissions from the final accepted quotation’s margin% calculation.  
2. A single lumpsum approach: Pending → Approved → Paid.  
3. Endpoints:  
   - GET /api/v1/commissions?cpId=XXX  
   - PATCH /api/v1/commissions/{id}/approve  
   - PATCH /api/v1/commissions/{id}/markPaid (requires Admin role).  

4. Audit each status change, storing important details (payment date, UTR info).

#### 1.6. Customer Service Requests & Quotation Acceptance (Implements L1-FRS §3.6)

1. The BCKND shall handle multi-service cart submissions from the Customer app, creating a New lead record with associated service details.  
2. A “Shared” quotation can be accepted by the customer, automatically switching the lead to “Customer Accepted.”  
3. No real-time push notifications: the BCKND relies on poll-based or daily scheduled notifications for major updates (e.g., acceptance events, KYC rejections).

#### 1.7. Offline & Notification Requirements (Implements L1-FRS §3.7)

1. The BCKND shall reject all create/update calls if the client is offline (in practice, the mobile app is responsible for only sending requests while online).  
2. The BCKND shall rely on poll-based or manual sync to inform CPAPP/CUSTAP/WEB of new data.  
3. For scheduled notifications:  
   - Daily or periodic tasks can check overdue leads or pending KYC and send minimal SMS/email via external APIs.  
   - Strict limit: 2 quick retries for SMS/email sending if the external service fails (Client Clarification #17).

---

### 2. Component Interfaces

The BCKND interfaces with the following:

1. **Frontend Clients (CPAPP, CUSTAP, WEBPRT)**  
   - Consumes all public endpoints under /api/v1.  
   - Must provide valid JWT in Authorization header (Bearer <token>).  
   - Must handle concurrency conflicts, 400/401/403/409 responses as described.

2. **MSG91 (SMS OTP) and SendGrid (Email)**  
   - BCKND calls external APIs using environment variables for credentials (Client Clarification #18).  
   - Retries the request up to 2 times on 5xx or timeout.

3. **Azure Blob Storage**  
   - BCKND generates short-lived SAS tokens for upload/download.  
   - Tracks references in DB for each file (e.g., docType, docId, container path).

---

### 3. Data Requirements

1. **Database Structure**  
   - Use Azure Database for PostgreSQL for leads, quotations, commissions, users, documents metadata.  
   - Store incremental numeric IDs with prefixes (e.g., LEAD-1001) for leads (Client Clarification #9).  

2. **Audit Trail**  
   - Maintain a single comprehensive “audit_timeline” table logging CREATE/UPDATE/DELETE events (Client Clarification #2).  
   - For concurrency or override, store override=true plus user role in the same record.  

3. **Concurrency & Versioning**  
   - Last-write-wins approach with an “updated_at” or revision_id column. Return HTTP 409 if mismatch unless “override=true” from Admin/KAM (Client Clarification #1).  

4. **Master Data**  
   - Store minimal sets: Panels, Inverters, Fees, GST, Subsidy, plus a “Custom Fee” table for flexible additions (Client Clarification #7).  

5. **CSV Import**  
   - POST /api/v1/leads/import → all-or-nothing approach.  
   - The response must return a bulk error list for row-by-row invalid data (Client Clarification #12).  

---

### 4. Validation Rules

1. **HTTP Status**  
   - 200/201 for success  
   - 400 for invalid data (e.g., missing required fields, file size > 10 MB)  
   - 401 for invalid/expired token  
   - 403 for unauthorized role access  
   - 409 for concurrency mismatch or conflict (e.g., lead locked, version mismatch)  

2. **Lead Status Updates**  
   - Routine forward transitions do not require override.  
   - Backward or unusual transitions require override=true from Admin/KAM.  

3. **Quotation Acceptance**  
   - Patch calls with override are restricted to Admin/KAM if lead is locked or final.  

4. **Commission Calculations**  
   - Confirm margin% is within a valid range (0%–100%).  
   - If the system price is zero, return 400 or a domain-specific 422 error.  

5. **Files**  
   - Return 400 if file type not in [PDF, JPG, PNG], or if size > 10 MB.  
   - Return 409 if the lead already has 7 documents and the user attempts an 8th upload.

---

### 5. References

- **L1-FRS**: High-Level Functional Requirements (Sections 3.1 – 3.7).
- **L1-HLD**: High-Level Design referencing the modular monolith approach in Node.js.  
- **L3-KD-BCKND**: Backend-specific key decisions (e.g., concurrency override, single-region, daily scheduled tasks).  
- **Clarifications #1 – #18**: Including concurrency override, audit logging, territory matching, 2-retry model for external services, user password reset approach, etc.

---

By implementing the above functionalities and respecting the defined data/validation rules, the BCKND fulfills the requirements of L1-FRS for the current scale of ~400–600 concurrent users, ensuring a pragmatic, maintainable architecture with moderate extensibility.