## L3-IB-CPAPP: Component Implementation Backlog for CPAPP

<add>**Note on Document Revisions:**  
This revised backlog incorporates various corrections and clarifications based on client validation feedback, aligning the storage of JWT tokens, session management, file upload retries, commission retrieval scope, references to L3-TSD, explicit design patterns, and section numbering consistency. All additions, deletions, or edits are marked with <add>, <delete>, or <edit> tags below.</add>

---

### <edit>1. Introduction</edit>
This document presents a granular, implementation-focused backlog for the Channel Partner App (CPAPP) based on the functional requirements (<edit>L3-FRS-CPAPP</edit>), low-level design (<edit>L3-LLD-CPAPP</edit>), non-functional requirements (<edit>L3-NFRS-CPAPP</edit>), and key decisions (<edit>L3-KD-CPAPP</edit>). It also integrates the latest client clarifications (#1–#5). Each backlog item describes specific development tasks, acceptance criteria, references to the relevant sections in the design documents, dependencies, effort estimates, and priority.

• References:  
  – <edit>L3-FRS-CPAPP for functional specs</edit>  
  – <edit>L3-LLD-CPAPP for detailed design</edit>  
  – <edit>L3-NFRS-CPAPP for performance, security, and reliability constraints</edit>  
  – <edit>L3-KD-CPAPP for component-specific decisions</edit>  
  – <add>L3-TSD for technology constraints and guidelines referenced in these backlog items.</add>  
  – Client Clarifications #1–#5 for revised or additional measures  

The items below are arranged by functional area. They do not include timelines or resource assignments.

<add>**Design Pattern Note:**  
Where polling or concurrency are involved, we apply a straightforward timer-based or observer-like approach (as recommended in L3-LLD). This ensures consistent code structure without overengineering for our ~400–600 concurrency scale.</add>

---

## <edit>2. Implementation Items</edit>

### 2.1 Authentication & Session Management

#### 2.1.1 Implement Phone-based OTP Login
• <edit>Description:</edit>  
  Build the phone-based OTP authentication flow, <edit>storing the JWT in secure device-level storage (e.g., react-native-keychain),</edit> enforcing a maximum of five OTP attempts, a 15-minute lockout upon exceeding attempts, and a 2-minute OTP validity.  
• <edit>References:</edit>  
  – L3-FRS-CPAPP §3.1  
  – L3-LLD-CPAPP §2.1 (Authentication Module)  
  – <delete>L3-KD-CPAPP Decision #3 (Storing tokens in AsyncStorage)</delete>  
  <add>L3-TSD for recommended mobile security practices</add>  
• <edit>Acceptance Criteria:</edit>  
  1. User can request and enter an OTP to authenticate.  
  2. If locked out, user sees an appropriate error until 15 minutes pass.  
  3. JWT is persisted for 24 hours, requiring re-login afterward.  
  4. Login fails gracefully on server or network errors.  
  <add>5. If for practical reasons AsyncStorage is used, the app must clearly document the reduced security, referencing L3-KD-CPAPP Decision #3.</add>  
• Dependencies:  
  – Backend OTP endpoint readiness (/api/v1/auth/login).  
• Effort Estimate: 3 Story Points  
• Priority: P1 (High)

#### 2.1.2 Enforce Hard Block After Token Expiration
• <edit>Description:</edit>  
  Once the 24-hour JWT expires, the app must require the CP to re-authenticate even if offline. <add>There is no auto-refresh mechanism, overriding any prior references to token auto-refresh.</add>  
• <edit>References:</edit>  
  – L3-FRS-CPAPP §3.1 (Token lifetime)  
  – <delete>L3-LLD-CPAPP §6.1 (Error Handling)</delete>  
  <add>L3-NFRS-CPAPP for clarifications on session policy, finalizing no auto-refresh approach</add>  
• <edit>Acceptance Criteria:</edit>  
  1. If token is expired, any attempt to access data triggers a “Session Expired” prompt.  
  2. All read/write operations are blocked until re-login is completed online.  
  3. Expiration is calculated locally based on the token’s issued-at time or server payload.  
  <add>4. Force daily re-login, ignoring any references to extended 30-day inactivity windows.</add>  
• Dependencies:  
  – Item 2.1.1 (token generation and storage).  
• Effort Estimate: 2 Story Points  
• Priority: P1 (High)

#### 2.1.3 Multiple Device Session Handling
• <edit>Description:</edit>  
  Allow concurrent logins on multiple devices under the same CP account, using last-write-wins concurrency for data updates.  
• <edit>References:</edit>  
  – L3-FRS-CPAPP §3.1.2  
  – L3-KD-CPAPP (last-write-wins concurrency)  
• <edit>Acceptance Criteria:</edit>  
  1. No forced logout if a user logs in on a second device.  
  2. Ensure each device uses its own valid JWT for updates.  
  3. Document concurrency approach in the user help screen if needed (optional).  
• Dependencies:  
  – Completed authentication endpoints and concurrency logic on the backend.  
• Effort Estimate: 2 Story Points  
• Priority: P2 (Medium)

---

### 2.2 Lead Management

#### 2.2.1 Online Creation of New Leads
• <edit>Description:</edit>  
  Provide a form and flow to create new leads when online. Sync the newly created lead back to local storage for read-only offline access.  
• <edit>References:</edit>  
  – L3-FRS-CPAPP §3.2.1  
  – L3-LLD-CPAPP §2.2 (Lead Management Module)  
• <edit>Acceptance Criteria:</edit>  
  1. User can open a “New Lead” form, enter required fields (customer name, phone), and submit.  
  2. On success, the new lead is returned from the server and stored locally for offline read.  
  3. If offline, immediate creation is blocked with a clear error message (“No internet connection…”).  
• Dependencies:  
  – Backend endpoint /api/v1/leads.  
• Effort Estimate: 3 Story Points  
• Priority: P1 (High)

<add>#### 2.2.2 Update Lead Status & Finalize “Customer Accepted” to “Won”
• Description:
  Implement status updates for leads, including the step from “Customer Accepted” to “Won.”
• References:
  – L3-FRS-CPAPP §3.2.2, §3.2.4
  – L3-LLD-CPAPP §2.2 (LeadService)
• Acceptance Criteria:
  1. User can set lead statuses (e.g., “In Discussion,” “Customer Accepted,” “Won,” etc.).
  2. Server enforces remarks or follow-up date if mandatory; the app displays any backend errors.
  3. If the lead is currently “Customer Accepted,” the CP can finalize it to “Won.”
• Dependencies:
  – Patch endpoint /api/v1/leads/{leadId}/status.
• Effort Estimate: 2 Story Points
• Priority: P1 (High)
</add>

<edit>#### 2.2.3 Offline Read-Only Lead Viewing</edit>
• <edit>Description:</edit>  
  Cache leads locally to allow viewing and searching offline (customer name, phone). No updates offline.  
• <edit>References:</edit>  
  – L3-FRS-CPAPP §3.2.3  
  – L3-KD-CPAPP Decision #1 (No offline edits, only read mode)</edit>  
• <edit>Acceptance Criteria:</edit>  
  1. Leads previously synced are available in SQLite for offline view.  
  2. If searching offline, user is warned that results may be outdated or incomplete.  
  3. No write operations are allowed offline.  
• Dependencies:  
  – Sync mechanism (see 2.6.1) to populate local DB.  
• Effort Estimate: 3 Story Points  
• Priority: P1 (High)

---

### 2.3 Quotation Management

#### 2.3.1 Generate New Quotation
• <edit>Description:</edit>  
  Provide a form for CP to enter minimal inputs (e.g., system kW, roof type) and call POST /api/v1/quotations to retrieve calculated pricing from the backend.  
• <edit>References:</edit>  
  – L3-FRS-CPAPP §3.3.1  
  – L3-LLD-CPAPP §2.3 (QuotationService)  
  – L3-KD-CPAPP Decision #4 (All pricing logic is backend-driven)  
• <edit>Acceptance Criteria:</edit>  
  1. Quotation creation request includes user inputs (system size, etc.).  
  2. On success, the app displays the returned pricing.  
  3. Errors or missing fields are reported to the user.  
• Dependencies:  
  – /api/v1/quotations endpoint.  
• Effort Estimate: 3 Story Points  
• Priority: P1 (High)

#### 2.3.2 Share Quotation & Handle Accept/Reject
• <edit>Description:</edit>  
  Allow CP to mark a quotation as “Shared” so the customer can accept or reject. If accepted, the lead automatically transitions to “Customer Accepted.”  
• <edit>References:</edit>  
  – L3-FRS-CPAPP §3.3.2  
  – L3-LLD-CPAPP §2.3  
• <edit>Acceptance Criteria:</edit>  
  1. “Share” action triggers PATCH /api/v1/quotations/{quoteId}/share.  
  2. If the customer accepts, the lead status updates to “Customer Accepted.”  
  3. If the customer rejects, the quotation status is set to “Rejected.”  
• Dependencies:  
  – Real-time or poll-based retrieval of updated lead status (see 2.6.1).  
• Effort Estimate: 2 Story Points  
• Priority: P2 (Medium)

#### 2.3.3 Offline Read-Only Quotation Viewing
• <edit>Description:</edit>  
  Cache quotations from the server for offline display. No offline creation or updates.  
• <edit>References:</edit>  
  – L3-FRS-CPAPP §3.3.4  
  – L3-KD-CPAPP Decision #1 (Read-only offline)  
• <edit>Acceptance Criteria:</edit>  
  1. User sees previously synced quotations in local DB.  
  2. If offline, user cannot create new quotations or modify existing ones.  
• Dependencies:  
  – Shared sync mechanism.  
• Effort Estimate: 2 Story Points  
• Priority: P2 (Medium)

---

### 2.4 Document & KYC Management

#### 2.4.1 Implement Strictly Sequential File Upload
• <edit>Description:</edit>  
  For up to 7 attachments (≤10 MB each), retrieve short-lived SAS URLs and upload one file at a time.  
• <edit>References:</edit>  
  – L3-FRS-CPAPP §3.4.1  
  – L3-LLD-CPAPP §2.4 (KycService)  
• <edit>Acceptance Criteria:</edit>  
  1. User selects multiple files; the app requests individual SAS URLs for each in a loop.  
  2. Files are uploaded strictly one-by-one in the order selected.  
  3. If any upload fails or times out, subsequent files are paused, user is prompted to retry.  
  <add>4. Align with 2.7.2 if implementing an automatic retry attempt before manual prompt.</add>  
• Dependencies:  
  – /api/v1/leadDocuments or /api/v1/kycDocuments for SAS URL.  
• Effort Estimate: 3 Story Points  
• Priority: P1 (High)

#### 2.4.2 Automatic Image Compression (Clarification #3)
• <edit>Description:</edit>  
  Use an off-the-shelf React Native image-resizing library to compress images above ~2 MB.  
• <edit>References:</edit>  
  – L3-FRS-CPAPP §3.4.2  
  – Client Clarification #3  
• <edit>Acceptance Criteria:</edit>  
  1. If an image exceeds 2 MB, the library compresses it automatically.  
  2. Compression uses default settings; resulting file must remain ≤10 MB total.  
  3. If compression fails, user is prompted to choose a smaller/lower-quality image.  
• Dependencies:  
  – Need a stable library (e.g., react-native-image-resizer).  
• Effort Estimate: 2 Story Points  
• Priority: P2 (Medium)

#### 2.4.3 Enforce 7 Attachments & 10 MB Max
• <edit>Description:</edit>  
  Restrict the total attachments per lead to 7, blocking files over 10 MB.  
• <edit>References:</edit>  
  – L3-FRS-CPAPP §3.4.2 & 6.2  
  – L3-LLD-CPAPP §2.4  
• <edit>Acceptance Criteria:</edit>  
  1. If user tries to attach >7 total documents, display an error.  
  2. If user selects a file over 10 MB, display a “File size exceeds limit” prompt.  
  3. The app must never attempt an upload exceeding 10 MB.  
• Dependencies:  
  – Checking file metadata pre-upload.  
• Effort Estimate: 1 Story Point  
• Priority: P1 (High)

---

### 2.5 Commission & Payout

#### 2.5.1 Fetch Complete Commission History
• <edit>Description:</edit>  
  <delete>Always fetch all commission records for the CP</delete>  
  <add>By default, fetch the current year’s commission records according to L3-FRS-CPAPP §3.5. Optionally enable retrieval of older data if the user explicitly requests it.</add>  
• <edit>References:</edit>  
  – L3-FRS-CPAPP §3.5  
• <edit>Acceptance Criteria:</edit>  
  1. On refresh, the app requests commission records for <add>the default scope (e.g., current year)</add>.  
  2. Previously cached commissions are replaced by the fresh data set.  
  3. <add>Support user-driven “Load More” or “Load Past Years” if needed, balancing performance concerns.</add>  
• Dependencies:  
  – Commission endpoint readiness with pagination support.  
• Effort Estimate: 2 Story Points  
• Priority: P2 (Medium)

---

### 2.6 Offline Access & Synchronization

#### 2.6.1 Poll-Based Data Sync & Manual Refresh
• <edit>Description:</edit>  
  Implement a foreground polling interval (~3 minutes) for leads, quotations, commissions, plus a pull-to-refresh manual sync.  
• <edit>References:</edit>  
  – L3-FRS-CPAPP §3.6.2  
  – L3-LLD-CPAPP §2.7 (Sync & Network Module)  
  – L3-KD-CPAPP Decision #2 (Poll-based sync)  
  <add>L3-TSD for recommended timer-based approach</add>  
• <edit>Acceptance Criteria:</edit>  
  1. While the app is in the foreground, it automatically syncs data every 3 minutes.  
  2. User can manually trigger a “pull-to-refresh” at any time to fetch updates.  
  3. If offline, sync attempts are skipped or queued until connectivity returns.  
• Dependencies:  
  – All relevant GET endpoints for leads, quotations, commissions.  
• Effort Estimate: 3 Story Points  
• Priority: P1 (High)

#### 2.6.2 Full Data Replacement During Sync
• <edit>Description:</edit>  
  Re-download entire sets of leads, quotations, and commissions from the backend, overwriting local caches. <add>Offline data is valid up to 24 hours, after which re-authentication is required (see 2.1.2).</add>  
• <edit>References:</edit>  
  – L3-FRS-CPAPP (full data refresh references)  
  – L3-KD-CPAPP Decision #5 (Full data refresh over incremental sync)  
• <edit>Acceptance Criteria:</edit>  
  1. Each sync request fetches all relevant data (potentially with paging).  
  2. Local data is replaced with the newly received data set.  
  3. Offline read cache updates only after a successful sync.  
  <add>4. If 24 hours lapse without a successful sync, the user must re-login (no 30-day buffer).</add>  
• Dependencies:  
  – Must coordinate with the same poll-based mechanism (see 2.6.1).  
• Effort Estimate: 2 Story Points  
• Priority: P1 (High)

---

### 2.7 Error Handling & Logging

#### 2.7.1 Graceful Handling of API Errors (400, 401, 403, 500)
• <edit>Description:</edit>  
  Display friendly error messages on common HTTP responses. Prompt the user to retry or re-login as needed.  
• <edit>References:</edit>  
  – L3-FRS-CPAPP §6.4  
  – L3-LLD-CPAPP §6.1 (Error Handling)  
• <edit>Acceptance Criteria:</edit>  
  1. 400 errors show validation feedback from the server to the CP.  
  2. 401 triggers a “Session Expired” or “Unauthorized” prompt with a re-login flow.  
  3. 500 errors show a generic “Server Unavailable” message with a retry option.  
• Dependencies:  
  – All HTTP calls from modules (Lead, Quotation, KYC, Commission).  
• Effort Estimate: 2 Story Points  
• Priority: P2 (Medium)

#### 2.7.2 Single Auto-Retry for File Upload Failures
• <edit>Description:</edit>  
  If an upload fails for transient reasons (e.g., network glitch), attempt one automatic retry before prompting manual retry.  
• <edit>References:</edit>  
  – L3-NFRS-CPAPP §2.3 (File upload retry guidance)  
  <add>– Aligns with L3-FRS-CPAPP after the latest clarifications allowing a single automated retry</add>  
• <edit>Acceptance Criteria:</edit>  
  1. Upon failure, app automatically retries once in the background.  
  2. If the second attempt also fails, the user is prompted with a “Retry” button.  
• Dependencies:  
  – Document & KYC module’s upload logic (Items 2.4.1, 2.4.2).  
• Effort Estimate: 2 Story Points  
• Priority: P2 (Medium)

---

## <edit>3. Dependencies and Sequencing</edit>
Below is a high-level view of dependencies among these items:

• 2.1.1 (OTP Login) must be completed before 2.1.2 (Token Expiry Block).  

• 2.6.1 (Poll-Based Sync) underpins offline read updates for leads (2.2.3), quotations (2.3.3), and commissions (2.5.1).  
• 2.4.1 (Sequential File Upload) and 2.4.2 (Image Compression) can be developed in parallel, but both must integrate in the final KYC upload flow.

Implementation priority is typically highest for core user flows (login, lead creation, offline read, basic sync). Secondary features like extended error handling or optional auto-retry can follow once the fundamentals are stable.

---

## <edit>4. Priority Indicators</edit>
• P1 (High): Core functionalities (authentication, lead creation, read-only offline, sync) that must be delivered first.  
• P2 (Medium): Important features but can be implemented once the P1 items are functional.  

---

## <edit>5. Effort Estimates</edit>
• Story Points range from 1 (very small) to 5 (large complexity).  
• These estimates are subject to revision once detailed implementation begins.

---

## <edit>6. Conclusion</edit>
This <edit>revised</edit> L3-IB-CPAPP: Component Implementation Backlog for CPAPP presents the discrete tasks required to implement the Channel Partner App in alignment with <edit>the updated clarifications.</edit> Each backlog item includes a succinct description, acceptance criteria, references, dependencies, effort, and priority. By addressing these items in sequence—focusing first on fundamental P1 tasks—development teams can deliver a secure, stable, and efficient CPAPP suitable for the ~400–600 concurrent user scale.

<add>**Final Note:**  
All references to session duration, storage mechanism, and upload retry logic are now consistent across L3-FRS-CPAPP, L3-NFRS-CPAPP, L3-KD-CPAPP, and L3-TSD to prevent contradictions and ensure a unified implementation.</add>