
## L3-IB-CUSTAP: Component Implementation Backlog for CUSTAP

This document provides a granular list of implementation backlog items (user stories and technical tasks) for the Customer Mobile App (CUSTAP). Each item references functional requirements from L3-FRS-CUSTAP, design details from L3-LLD-CUSTAP, non-functional requirements from L3-NFRS-CUSTAP, and key decisions from L3-KD-CUSTAP, as well as relevant client clarifications. The items are intended to be directly actionable by developers, with clear acceptance criteria and identified dependencies. Effort and priority indicators are included for planning, but no specific timelines or resource assignments are given.

---

### IB-1: Implement Phone-Based OTP Login

**Description:**  
• Develop the login screen and logic that prompts the user for their phone number and OTP.  
• Integrate with the backend endpoint (POST /api/v1/auth/login) to request and verify OTP.  
• Store the resulting JWT in AsyncStorage.

**References:**  
• L3-FRS-CUSTAP §2.1 (Authentication & User Management)  
• L3-LLD-CUSTAP §2.1 (AuthModule)  
• L3-KD-CUSTAP #5 (Phone-based OTP Sole Mechanism)  
• L3-NFRS-CUSTAP §3 (Security: Token Storage)  

**Acceptance Criteria:**  
1. User can enter phone number and request an OTP.  
2. If OTP matches, the user receives a valid JWT from the backend and is directed to the app’s home screen.  
3. OTP mismatch triggers an error message without crashing the app.  
4. JWT is stored securely in AsyncStorage and used for subsequent requests.  

**Dependencies:**  
• Requires backend OTP endpoints operational.  
• No prior app modules strictly required as long as the Auth flow is the first user interaction.  

**Effort Estimate:** 3 story points  
**Priority:** High

---

### IB-2: Enforce 5 OTP Attempts & 15-Minute Lockout

**Description:**  
• Add logic to count consecutive failed OTP verifications and lock the user out for 15 minutes upon exceeding five failures.

**References:**  
• L3-FRS-CUSTAP §2.1  
• L3-LLD-CUSTAP §2.5.1 (OTP Authentication Flow)  
• L3-NFRS-CUSTAP §3 (Security Requirements)  

**Acceptance Criteria:**  
1. After 5 consecutive invalid OTP attempts, app must inform the user they are locked out for 15 minutes.  
2. Lockout state is enforced until the timer expires (or user reopens the app after 15 minutes).  
3. Lockout is lifted automatically once 15 minutes have passed.

**Dependencies:**  
• IB-1 (basic OTP flow) must be in place.  

**Effort Estimate:** 2 story points  
**Priority:** High

---

### IB-3: Manage Short-Lived JWT & Re-Login Flow

**Description:**  
• Implement token expiration checks (e.g., 15–30 minute validity).  
• Prompt user to re-enter phone + OTP when the token has expired or is invalid.

**References:**  
• L3-FRS-CUSTAP §2.1  
• L3-LLD-CUSTAP §2.1 (AuthModule token logic)  
• L3-KD-CUSTAP #6 (Store Tokens in AsyncStorage)  

**Acceptance Criteria:**  
1. Token-based calls fail gracefully if token is expired or invalid (HTTP 401).  
2. User is prompted to re-login via OTP if the stored JWT is expired.  
3. Expired token must not allow access to any protected endpoints.  

**Dependencies:**  
• IB-1 and IB-2 (fundamental auth flow).  
• Backend token expiration policy alignment.  

**Effort Estimate:** 2 story points  
**Priority:** High

---

### IB-4: Enforce Unique Phone Numbers

**Description:**  
• If a customer attempts to log in with a phone number that is already registered for a Channel Partner or conflicts with existing users, handle the backend-rejected scenario (e.g., display an error).

**References:**  
• L3-FRS-CUSTAP §2.1  
• L3-KD-CUSTAP #6 (Phone Uniqueness)  

**Acceptance Criteria:**  
1. Any attempt to register or log in with a conflicting phone number triggers a user-friendly error message.  
2. The user must contact Admin if they need to change their phone number.  

**Dependencies:**  
• IB-1 (OTP login mechanism).  
• Backend validation must be in place.  

**Effort Estimate:** 1 story point  
**Priority:** Medium

---

### IB-5: Fetch Service Catalog & Display Multi-Service Cart

**Description:**  
• Dynamically retrieve services (IDs, names, prices) via GET /api/v1/services.  
• Show a list of services for the user to add to a “cart” in the app.

**References:**  
• L3-FRS-CUSTAP §2.2 (Multi-Service Cart)  
• L3-LLD-CUSTAP §2.1 (LeadsModule)  

**Acceptance Criteria:**  
1. On app launch or manual refresh, fetch the service catalog from backend.  
2. Display service items with their name and basic info.  
3. User can select multiple services to place into a cart with optional remarks.  

**Dependencies:**  
• IB-3 (token-based calls) to ensure user can pull service data.  

**Effort Estimate:** 2 story points  
**Priority:** High

---

### IB-6: Create New Lead from Cart

**Description:**  
• Implement POST /api/v1/leads to submit selected services, forming a new lead.  
• Capture lead ID from backend response and store locally.

**References:**  
• L3-FRS-CUSTAP §2.2  
• L3-LLD-CUSTAP §2.1 (LeadsModule)  

**Acceptance Criteria:**  
1. When the user confirms cart submission, the app sends a valid POST body to the backend.  
2. Successful API response returns a new lead ID (e.g., LEAD-101).  
3. Offline lead creation attempts are blocked; user is notified they must reconnect.  

**Dependencies:**  
• IB-5 (service catalog + cart UI).  
• IB-3 (valid JWT).  

**Effort Estimate:** 3 story points  
**Priority:** High

---

### IB-7: View Quotation List

**Description:**  
• Fetch and display all quotations associated with a lead via GET /api/v1/quotations?leadId=xxx.  
• Show summary details: price breakdown, items, validity date.

**References:**  
• L3-FRS-CUSTAP §2.3  
• L3-LLD-CUSTAP §2.1 (QuotationModule)  

**Acceptance Criteria:**  
1. Quotation screen lists quotations for a selected lead, if any exist.  
2. Quotations are displayed in the correct status (Shared, Invalid, etc.).  
3. If offline, only previously cached quotations are visible.

**Dependencies:**  
• IB-6 (creating leads).  
• IB-3 or existing token.  

**Effort Estimate:** 2 story points  
**Priority:** High

---

### IB-8: Accept or Reject a Quotation

**Description:**  
• Allow the user to accept (PATCH /quotations/{id}/accept) or reject a “Shared” quotation.  
• Handle errors if the quotation is no longer valid (HTTP 409).

**References:**  
• L3-FRS-CUSTAP §2.3  
• L3-LLD-CUSTAP §2.5.2 (Quotation Acceptance Flow)  
• L3-KD-CUSTAP #4 (Use “Customer Accepted” status)  

**Acceptance Criteria:**  
1. “Accept” changes lead status to “Customer Accepted” in the backend.  
2. “Reject” leaves lead status unchanged, but user sees a confirmation message.  
3. Any attempt to accept an invalid or closed quote triggers an error message.  
4. After acceptance or rejection, local cache updates accordingly.

**Dependencies:**  
• IB-7 (quotation viewing).  
• Leads must exist on the backend.  

**Effort Estimate:** 3 story points  
**Priority:** High

---

### IB-9: KYC Document Upload with Short-Lived SAS

**Description:**  
• Implement file upload flow using short-lived SAS tokens from GET /api/v1/documents/sas.  
• PUT the file (max 10 MB) to Azure Blob storage.

**References:**  
• L3-FRS-CUSTAP §2.4  
• L3-LLD-CUSTAP §2.5.3 (KYCModule & Azure Blob flow)  
• Clarification #1 (Manual Retrigger on expired SAS token)  

**Acceptance Criteria:**  
1. When user selects a file (PDF/JPG/PNG ≤10 MB), the app requests a SAS token from the backend.  
2. Upload to Azure Blob completes within ~5 minutes or fails with an “Upload Failed” if token or network interrupts.  
3. If token expires mid-upload, the user must see a “Upload Failed” message and manually retry (no auto-refresh logic).  
4. After successful upload, backend or the app confirms the file is posted (status = “Pending Review”).

**Dependencies:**  
• IB-3 (JWT for requesting SAS token).  
• IB-6 (lead must exist to associate the KYC doc).  

**Effort Estimate:** 5 story points  
**Priority:** High

---

### IB-10: KYC Document Status & Rejected Handling

**Description:**  
• Display each document’s approval status: Pending Review, Approved, or Rejected (without a specific reason).  
• If Rejected, user can delete and re-upload a new file.

**References:**  
• L3-FRS-CUSTAP §2.4  
• Clarification #2 (Simple “Rejected” Flag Only)  

**Acceptance Criteria:**  
1. Approved documents are locked (cannot overwrite).  
2. Rejected documents are clearly marked “Rejected,” with no reason displayed.  
3. User can tap “Replace” or “Re-upload” for Rejected docs if they wish.  

**Dependencies:**  
• IB-9 (upload logic).  

**Effort Estimate:** 2 story points  
**Priority:** Medium

---

### IB-11: Create & View Basic Support Tickets

**Description:**  
• Allow user to submit a ticket once the lead is in “Executed” status.  
• Fetch and list existing tickets via GET /api/v1/supportTickets?leadId=xxx.

**References:**  
• L3-FRS-CUSTAP §2.5  
• L3-LLD-CUSTAP §2.1 (TicketModule)  

**Acceptance Criteria:**  
1. Users with a lead status = “Executed” can file a short text description.  
2. Single optional file attachment (up to 10 MB).  
3. Ticket statuses: Open or Closed; user sees the current status in the app.  
4. Offline ticket creation is blocked (user sees “No internet connection”).  

**Dependencies:**  
• IB-6 (existing lead).  
• IB-3 (valid JWT).  

**Effort Estimate:** 3 story points  
**Priority:** Medium

---

### IB-12: Offline Read-Only & Poll-Based Refresh

**Description:**  
• Retain minimal cached data (leads, quotations, etc.) so the user can view them offline.  
• Provide manual pull-to-refresh or automatic screen-based refresh for updates (no real “push notifications”).

**References:**  
• L3-FRS-CUSTAP §2.6  
• L3-KD-CUSTAP #2 (Poll-Based, No True Push)  
• L3-LLD-CUSTAP §2.3 (Internal Data Handling)  

**Acceptance Criteria:**  
1. Offline read-only: user sees previously fetched leads, quotations, or docs.  
2. Attempting new lead creation, quote acceptance, or file upload offline shows an error (“Operation requires internet”).  
3. Pull-to-refresh triggers an immediate GET to backend endpoints, updating local cache.  
4. Automatic data refresh occurs when user navigates into a screen if connectivity is available.  

**Dependencies:**  
• IB-5, IB-6, IB-7, IB-8 (basic data flows).  

**Effort Estimate:** 3 story points  
**Priority:** High

---

### IB-13: Integrate Minimal Crash Reporter

**Description:**  
• Integrate a lightweight crash analytics library (e.g., Sentry or Firebase Crashlytics) to capture runtime errors and crashes.

**References:**  
• L3-NFRS-CUSTAP §3 (Security & Crash Logging)  
• Clarification #3 (Minimal Crash Reporter)  

**Acceptance Criteria:**  
1. On crash, relevant diagnostic info is sent to the crash analytics tool.  
2. Works in release builds with minimal performance overhead.  
3. Non-critical sensor or usage data is not uploaded; only crash details.  

**Dependencies:**  
• Basic app scaffolding in place.  

**Effort Estimate:** 2 story points  
**Priority:** Medium

---

### IB-14: Basic On-Device Compression for Large Images (>10 MB)

**Description:**  
• If the user selects an image file exceeding 10 MB, automatically prompt them to compress or resize.  
• Re-encode and reduce file size before uploading to Blob. PDF or other file types remain blocked at >10 MB.

**References:**  
• L3-FRS-CUSTAP §2.4 (KYC limit)  
• L3-NFRS-CUSTAP §2 (Performance & File Uploads)  
• Clarification #4 (Basic On-Device Compression for Images Only)  

**Acceptance Criteria:**  
1. Selecting an image >10 MB prompts a compression flow.  
2. Compressed image must reliably fall under 10 MB, unless the image is extremely large.  
3. If compression still fails to reach <10 MB, the user is instructed to pick a smaller file.  
4. PDFs or other file formats >10 MB are blocked with a clear error message.  

**Dependencies:**  
• IB-9 (upload logic).  
• A suitable React Native module for image compression.  

**Effort Estimate:** 3 story points  
**Priority:** Medium

---

## Additional Notes on Dependencies and Sequencing

• Items IB-1 through IB-4 collectively form the user authentication flow and must be completed before the user can access the rest of the app features (creating leads, uploading documents, etc.).  
• Items IB-5 and IB-6 enable lead creation, which is the core functionality the user interacts with post-login.  
• Items IB-7 and IB-8 for quotations, and IB-9 and IB-10 for KYC handling, can be developed in parallel with proper stubs or mock endpoints.  
• IB-11 (support tickets) depends on leads being creatable and statuses being recognized from the backend.  
• IB-12 (offline read-caching and poll-based refresh) extends and refines existing flows, so it can be layered in once basic data fetches are stable.  
• IB-13 (crash reporting) and IB-14 (image compression) are feature enhancements that can be added at any stage but typically occur after core flows are functional.  

No advanced or over-engineered solutions (e.g., background sync, real-time push) are required at the current ~400–600 concurrent user scale. This backlog focuses on delivering a robust, pragmatic solution aligned with the clarified functional, design, and non-functional requirements.

---
```
