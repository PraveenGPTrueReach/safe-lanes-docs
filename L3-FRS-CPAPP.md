## L3-FRS-CPAPP: Functional Requirements Specification (FRS) for CPAPP

This revised document incorporates and highlights changes requested based on the recent client suggestions and validation results. All insertions, deletions, or edits are marked with “&lt;add&gt; &lt;/add&gt;”, “&lt;delete&gt; &lt;/delete&gt;”, or “&lt;edit&gt; &lt;/edit&gt;” tags.

---

### 1. Introduction
This document provides a detailed specification of the functional requirements for the Channel Partner Mobile App (CPAPP) within the Solarium Green Energy solution, ensuring alignment with L1-FRS and the inter-component design in L2-LLD-IC. It addresses the ~400–600 concurrent user scale while keeping the architecture and workflows pragmatic.

The CPAPP is a React Native mobile application used by Channel Partners to:  
- Create and manage leads.  
- Generate and share quotations with customers.  
- Upload KYC/lead documents on behalf of customers.  
- Track commissions and payouts.

CPAPP interacts exclusively with the Backend (BCKND) via HTTPS RESTful APIs, storing limited offline data locally in read-only mode.

---

### 2. Scope
- Implements L1-FRS 3.1 (User Management & Authentication), 3.2 (Lead Management), 3.3 (Quotation Management), 3.4 (Document & KYC Management), 3.5 (Commission & Payout), and 3.7 (Offline & Notification Requirements) for the CP role.  
- Incorporates client clarifications #1–#9 (see Section 8), explicitly enumerating how CPAPP handles concurrency, file uploads, caching, multi-device logins, etc.  
- Excludes functionalities handled by other components (Customer App, Admin/KAM Web Portal, or the backend’s internal logic).

Note on SAS Token Validity: This specification aligns with the L1-HLD approach of ~5-minute short-lived SAS tokens, superseding any references to longer durations in other documents.

---

### 3. Functional Requirements

#### 3.1 User Management & Authentication
1. Implements L1-FRS 3.1 for CP role:  
   - CP logs in via phone-based OTP (POST /api/v1/auth/login). OTP attempts are limited to five, after which a 15-minute lockout is enforced.  
   - Each OTP is valid for 2 minutes, after which it expires.  
   - CPAPP stores the JWT in a secure device-level storage mechanism (e.g., react-native-keychain) for up to 24 hours.  
   -   
   - A separate refresh-token endpoint is not provided, and auto-refresh is not implemented, so re-login is required after the 24-hour token expires. This final approach overrides any references in L1-HLD to auto-refresh.

2. Multiple Device Sessions (Clarification #8):  
   - Permits concurrent logins across multiple devices for the same CP.  
   - Last-write-wins applies if updates are made from different devices simultaneously (see Section 6.3).

#### 3.2 Lead Management
1. **Create New Lead** (ref: L1-FRS 3.2):  
   - CP can create leads only while online via POST /api/v1/leads, passing required fields (e.g., customer name, phone).  
   - Offline creation is disallowed.  
   If the user attempts to create or update a lead while offline, the app displays “No internet connection. Please go online to proceed.”

2. **Update Lead Status**:  
   - CP can update lead status (PATCH /api/v1/leads/{leadId}/status).  
   - The app enforces mandatory remarks or next follow-up date as defined by the backend.  
   - If Admin or KAM has changed the same lead, last-write-wins remains in effect (no conflict prompt).

3. **View Leads Offline**:  
   - CPAPP caches lead lists from the backend to allow read-only offline viewing.  
   - New or updated leads require an active internet connection.

4. **Finalize “Customer Accepted” to “Won”**:  
   - Once a customer accepts a shared quotation, the lead moves to “Customer Accepted.”  
   - The CP must finalize it as “Won” to confirm project readiness, enabling subsequent processes (e.g., commission payout).

5. **Lead Search & Filtering**  
   - CPAPP shall provide a simple search/filter mechanism (by customer name or phone) for leads.  
   - Online searching filters data directly from the server; offline mode filters only locally cached results and warns users that results may be incomplete or outdated.

#### 3.3 Quotation Management
1. **Generate Quotation** (ref: L1-FRS 3.3):  
   - CP sends minimal inputs (e.g., system kW, roof type) to the backend via POST /api/v1/quotations.  
   - CPAPP displays the returned pricing details (no local financial logic).

2. **Share Quotation**:  
   - CP can mark a quotation as “Shared” with the customer (PATCH /api/v1/quotations/{quoteId}/share).  
   - Once shared, the customer can accept or reject it. If accepted, the system automatically updates the lead’s status to “Customer Accepted.”

3. **Rejected Quotation Handling**:  
   - If the customer rejects the quotation, the backend sets the quotation status to “Rejected.”  As per the official L1-FRS status matrix, the lead typically remains “In Discussion” or moves to another valid terminal/interest-lost status (e.g., “Not Interested” or “Not Responding”) as configured server-side.  
   - During the next poll-based sync, CPAPP fetches the updated status. The CP can modify or create a new quotation if needed.

4. **View Quotation Offline**:  
   - Quotations are stored in the read-only local cache.  
   - Any new or updated quote requires an online request.

#### 3.4 Document & KYC Management
1. **KYC/Lead Document Upload** (ref: L1-FRS 3.4):  
   - CP uses a two-step approach:  
     a) POST /api/v1/leadDocuments or /api/v1/kycDocuments to request a short-lived (~5 minutes) SAS URL.  
     b) PUT the file to Azure Blob using the received SAS URL.  
   - Single-attempt upload: if upload fails or times out, the user is prompted to retry manually.

2. **File Size Constraint & Compression** (Clarification #9):  
   - Each file ≤ 10 MB.  
   - Each lead can have a maximum of 7 attachments, consistent with overall solution constraints.  
   - The app automatically compresses images if the size exceeds a threshold (e.g., 2 MB).

3. **Offline Storage**:  
   - CPAPP does not store unsent files offline.  
   - If the SAS URL or auth token expires mid-upload, the user must reinitiate the process.

4. **Local Encryption for Sensitive Documents**:  
   - If the CPAPP temporarily caches any KYC files, they shall be stored using secure, device-level encryption to prevent unauthorized access.

#### 3.5 Commission & Payout
1. **View Commission Records** (ref: L1-FRS 3.5):  
   - CP can retrieve commissions via GET /api/v1/commissions, with the CP identity derived from the JWT.  
   - By default, the app displays the current year’s commission history, though date filters are possible.

2. **Commission Status Sync**:  
   - Commission updates occur via polling or manual refresh.  
   - Offline updates are not allowed.

#### 3.6 Offline Access & Sync Strategy
1. **Read-Only Offline Mode** (ref: L1-FRS 3.7):  
   - CP sees a cached subset of leads, quotations, and commissions previously fetched.  
   - No offline creation or update is permitted.

2. **Foreground Polling**:  
   -  By default, automatic data sync occurs every ~180 seconds (3 minutes) when the app is in the foreground, balancing device battery usage and data freshness.  
   - Manual pull-to-refresh triggers additional sync cycles.

3. **Data Retention Policy**:  
   - Cached data remains valid for a maximum of 24 hours offline, after which the user must re-authenticate before accessing sensitive records.  

4. **No Real-Time Push**:  
   - The system relies on poll-based or manual sync only. This supersedes any older references to “automatic push notifications” in higher-level documents for CPAPP use cases.

---

### 4. Component Interfaces
Below are the primary endpoints needed by this component:

| Endpoint                                      | Method | Description                                                                                             |
|-----------------------------------------------|--------|---------------------------------------------------------------------------------------------------------|
| /api/v1/auth/login                            | POST   | CP OTP login (phone, OTP). Returns JWT & CP role.                                                       |
| /api/v1/leads                                 | POST   | Create new lead. Requires JSON with lead details.                                                       |
| /api/v1/leads/{leadId}/status                | PATCH  | Update lead status & remarks.                                                                           |
| /api/v1/quotations                            | POST   | Generate new quotation with minimal inputs (kW, roof type, etc.).                                       |
| /api/v1/quotations/{quoteId}/share           | PATCH  | Mark quotation “Shared” with the customer.                                                              |
| /api/v1/leadDocuments                         | POST   | Request SAS URL for lead doc upload (valid ~5 minutes).                                                 |
| /api/v1/kycDocuments                          | POST   | Request SAS URL for KYC doc upload (valid ~5 minutes).                                                  |
| (SAS URL from BCKND)                          | PUT    | Direct file upload to Azure Blob.                                                                       |
| /api/v1/commissions                           | GET    | Fetch commission records for the CP (from JWT identity).                                                |

All calls must include the “Authorization: Bearer <JWT>” header, except for OTP login requests.

---

### 5. Data Requirements
1. **Local Storage**:  
   - The offline database (e.g., SQLite) holds read-only copies of leads, quotations, and commissions.  
   - No incremental merges: each sync fetches updated records from the server, with the assumption that total data volumes remain manageable at the current scale.  
   - If KYC or other sensitive data is cached, it shall be encrypted at rest using device-level secure storage.

2. **Data Purging**:  
   - Old data is cleared only upon a successful new login, ensuring previously cached information is available offline until connectivity is restored or the maximum 24-hour window is reached.

3. **Captured Fields**:  
   - For lead creation: basic identification (customer name, phone) and any mandatory fields (e.g., remark, follow-up date).  
   - For quotation inputs: minimal system specs (kW, roof type).  
   - For commission retrieval: data is fetched from the server with optional date filters (default to current year).

---

### 6. Validation Rules

#### 6.1 Input & Workflow Validation
1. **Lead Creation**:  
   - Phone number formatting is validated before calling POST /api/v1/leads.  
   - The app displays backend errors (invalid status, missing remark, etc.).

2. **Quotations**:  
   - Must specify system kW, roof type, or other minimal fields.  
   - If the server rejects the quote for missing details, the app prompts the CP to correct them.

#### 6.2 File Upload Constraints
1. **10 MB Max**: CPAPP prevents attempts to upload files > 10 MB.  
2. **Image Downscaling**: If the image is above ~2 MB, the app compresses it before uploading.  
3. **Single Upload Attempt**: If an upload fails, the user must retry manually.  
4. **Maximum Seven Attachments**: CP cannot exceed seven total documents per lead (server-enforced and validated in the app).

#### 6.3 Concurrency & Conflict Handling
1. **Last-Write-Wins** (no user-facing conflict resolution prompts):  
   - The server overwrites prior changes on conflicting updates.  
   - No interactive conflict prompt is shown to the CP.

2. **Multiple Device Sessions**:  
   - No forced logout or concurrency limit. The CP can be logged in across multiple devices.

#### 6.4 Error Handling & Codes
1. The app must handle common HTTP errors gracefully. Examples:  
   - 400 (Bad Request): Display validation error messages from the backend.  
   - 401 (Unauthorized): Prompt for re-login.  
   - 403 (Forbidden): Inform the user they lack the necessary role or permission.  
   -  No dedicated concurrency-related 409 response is expected under a last-write-wins approach; however, the backend may still return 409 for domain-level conflicts (e.g., phone number uniqueness or lead territory mismatches).  
   - 500 (Server Error): Display a generic “Server Unavailable” message and allow retry.

2. Specific known scenarios are mapped to user-friendly prompts (e.g., “OTP expired, please request a new one.” or “File exceeds the 10 MB limit.”).  

---

### 7. Traceability to L1-FRS Requirements
| CPAPP Function                           | L1-FRS Sections     | Notes                                                                                     |
|------------------------------------------|-----------------------------------|-------------------------------------------------------------------------------------------|
| Phone-based OTP Authentication           | L1-FRS 3.1 User Mgmt & Auth       | Permits CP to log in with phone + OTP.                                                    |
| Lead Creation & Status Updates           | L1-FRS 3.2 Lead Management        | Through designated REST endpoints only, online.                                           |
| Quotation Generation & Sharing           | L1-FRS 3.3 Quotation Mgmt         | Minimal config input; final pricing by backend.                                           |
| Document Upload (KYC / Lead Docs)        | L1-FRS 3.4 Document Mgmt          | Single attempt, short-lived ~5 min SAS tokens.                                            |
| Commission Viewing & Payout Tracking     | L1-FRS 3.5 Commission & Payout    | Paginated retrieval of commission records; date-based filters if needed.                  |
| Offline Read-Only & Poll-Based Sync      | L1-FRS 3.7 Offline & Notifications| No offline creation; pull-based refresh only.                                |

---

### 8. Client Clarifications #1–#9
1. No offline creation or updates; only read-only caching while offline.  
2. Minimal input for quotation generation; backend handles pricing logic.  
3.  JWT stored in secure keychain storage, no separate refresh token, 24-hour validity.  
4. Last-write-wins concurrency handling; no conflict prompts.  
5. Single-attempt file upload; user must retry on failure.  
6. Full data refresh (no deltas) on sync; old cache purged on re-login/ or after 24 hours offline.  
7. Local caching for read-only data (SQLite or similar).  
8. Multiple-device concurrent sessions are allowed for CP.  
9. Automatically compress images before upload if exceeding threshold size.

---

### 9. Additional Considerations

1. **Business Metrics**:  
   - Standard KPIs like lead conversion rate, quotation acceptance rate, and commission payout timelines can measure app effectiveness.

2. **Requirement Prioritization**:  
   - All requirements here are essential for baseline operation. Advanced features (e.g., automatic file upload retry) remain future enhancements.

3. **Accessibility & Usability**:  
   - The app includes basic accessibility measures (e.g., screen reader labels, scalable text). Future enhancements may increase compliance with WCAG 2.1 AA standards.

4. **Handling Partial Upload Failures**:  
   - If the SAS URL or token expires mid-transfer, the user must re-login or request a new upload URL. No built-in upload resume capability exists for now.

5. **Security for OTP Sessions**:  
   - The 24-hour token validity (with no auto-refresh) balances convenience and security for ~400–600 concurrent users. After expiry or extended inactivity, re-authentication (new OTP) is required.

6. Performance & Full Data Replacement  
   - The system fetches entire relevant record sets during each sync. Given the moderate project scale (~400–600 concurrent users), total data size is expected to remain manageable. Future releases may explore incremental or delta-based syncing if data volumes grow significantly.

---

### 10. Conclusion
The CPAPP must fulfill each described feature to meet Channel Partner business requirements for lead management, quotation workflows, and commission oversight at the current scale (400–600 concurrent users). This updated specification resolves conflicting details regarding token handling, lead statuses, concurrency, and SAS token lifetimes, ensuring a cohesive and secure functional baseline going forward.

End of Revised Document