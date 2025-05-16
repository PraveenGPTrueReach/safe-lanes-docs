## L3-FRS-CUSTAP: Functional Requirements Specification (FRS) for CUSTAP

This document defines the specific functionalities and behaviors required for the Customer Mobile App (CUSTAP), ensuring alignment with the overall system requirements (see L1-FRS). All references to “Implements L1-FRS-x” map the features below to their corresponding high-level system requirements. This specification focuses solely on CUSTAP’s scope, leaving out functionality exclusive to other components (CPAPP, WEBPRT, or BCKND).

---

## Table of Contents
1. Introduction  
2. Functional Requirements  
   2.1 Authentication & User Management (Implements L1-FRS-3.1)  
   2.2 Multi-Service Cart & Lead Creation (Implements L1-FRS-3.2, L1-FRS-3.6)  
   2.3 Quotation Viewing & Acceptance (Implements L1-FRS-3.3, L1-FRS-3.6)  
   2.4 KYC Document Upload (Implements L1-FRS-3.4)  
   2.5 Support Tickets (Implements L1-FRS-3.6)  
   2.6 Offline Behavior & Poll-Based Notifications (Implements L1-FRS-3.7)  
3. Component Interfaces  
4. Data Requirements  
5. Validation Rules  
6. Endpoints & Methods  

---

## 1. Introduction
CUSTAP is a React Native mobile application used by customers to:  
• Browse available solar services (dynamically fetched)  
• Create new leads by submitting a “multi-service cart”  
• View, accept, or reject quotations shared by Channel Partners (CPs) or Admin/KAM  
• Upload KYC documents as needed  
• Raise basic support tickets once a project is “Executed”  

The functional requirements below are based on the overall L1-FRS and are tailored to meet the system’s current scale (approximately 400–600 concurrent users). All primary interactions with the backend (BCKND) occur via RESTful APIs over HTTPS.

---

## 2. Functional Requirements

### 2.1 Authentication & User Management  
(Implements L1-FRS-3.1)

1. CUSTAP shall use phone-based OTP as the sole login mechanism.  
2. The app shall allow up to five (5) incorrect OTP attempts before a 15-minute lockout.  
3. When a valid OTP is entered, the user receives a short-lived JSON Web Token (JWT), stored in the device’s secure storage (AsyncStorage).  
4. If the user’s token expires, the app shall prompt them for re-login via OTP.  
5. Customer phone numbers are unique system-wide (i.e., cannot also be a CP phone). If a user wishes to change their phone number, they must contact Admin (no self-service phone update).

### 2.2 Multi-Service Cart & Lead Creation  
(Implements L1-FRS-3.2, L1-FRS-3.6)

1. The app shall dynamically fetch the service catalog (service IDs, names, prices, basic info) from the backend on app launch or manual refresh.  
2. The user can add multiple services to a “cart,” providing minimal information (service ID + optional remarks).  
3. Upon cart submission, CUSTAP shall send a POST request to create a new lead (status = “New Lead”) on the backend.  
4. The user may not create or update leads offline; any new cart submission requires an active online session.  
5. On successful creation, the backend returns a lead identifier (e.g., LEAD-101), which the app stores locally for reference.

### 2.3 Quotation Viewing & Acceptance  
(Implements L1-FRS-3.3, L1-FRS-3.6)

1. CUSTAP shall list all quotations associated with the user’s leads, fetched from the backend via a GET request.  
2. The app shall display the quotation’s summary details (price breakdown, items, valid until date).  
3. A “Shared” quotation can be:  
   - Accepted → triggers the lead to transition to “Customer Accepted” in the backend.  
   - Rejected → leads remain in the same status, and the CP or Admin can subsequently revise or create a new quote.  
4. CUSTAP enforces single acceptance only. The customer must accept or reject the entire quotation rather than partial line items.  
5. If the user tries to accept a quotation that is invalid or closed, the backend shall respond with an error message (HTTP 409), which the app displays.

### 2.4 KYC Document Upload  
(Implements L1-FRS-3.4)

1. For any lead, the customer can optionally upload KYC documents (ID proof, address proof, etc.) via a “Documents” section in the app.  
2. Uploading a file requires a valid JWT. The app shall request a short-lived SAS URL from the backend and then PUT the file (e.g., PDF/JPG/PNG up to 10 MB) to Azure Blob.  
3. Once a document is approved by Admin/KAM, the customer can no longer overwrite that file.  
4. The app shall show each KYC file’s status (Pending Review, Approved, Rejected) if returned by the backend.  
5. Up to seven (7) lead-level documents can be uploaded (including KYC if relevant).

### 2.5 Support Tickets  
(Implements L1-FRS-3.6)

1. Once a lead is marked “Executed” by the backend, the user can raise a basic ticket for post-installation support.  
2. A support ticket includes a short text description of the issue (e.g., “panel not functioning,” “inverter error”).  
3. The system tracks only two ticket statuses: Open or Closed (no complex escalation paths).  
4. The user can attach a single optional file (photo/screenshot up to 10 MB) to illustrate the issue.  
5. The app fetches the list of tickets per lead from the backend (GET), and the user sees the ticket status.  
6. Tickets are closed either by Admin/KAM or automatically after resolution; the customer cannot close their own ticket.

### 2.6 Offline Behavior & Poll-Based Notifications  
(Implements L1-FRS-3.7)

1. CUSTAP shall allow offline read-only access to previously fetched leads, quotations, or documents.  
2. New leads, updates, or file uploads are disallowed without an internet connection.  
3. The app does not implement true push notifications; instead, it relies on:  
   - Manual pull-to-refresh by the user, or  
   - Automatic data refresh when the user navigates to a screen.  
4. If connectivity is lost mid-request, the app shall show a soft warning and allow a manual retry once the connection returns.

---

## 3. Component Interfaces

• CUSTAP communicates exclusively with the backend (BCKND) via REST APIs over HTTPS.  
• No direct interface exists to external SMS/email providers; all OTP and email operations are handled by the backend.  
• Document uploads rely on short-lived SAS tokens generated by the backend; the actual file PUT is directed to Azure Blob Storage.  

---

## 4. Data Requirements

1. The app must locally store:  
   - Basic user profile info (phone, token)  
   - Cached lead list and quotations for offline viewing  
   - Minimal service catalog data for display (fetched on app launch or refresh)  
2. Sensitive data (JWT) is stored in React Native AsyncStorage with short expiry (~15–30 minutes).  
3. Large files (KYC, photos) are not stored permanently on the device; only ephemeral copies are used for upload.  
4. No in-depth lead or quotation data transformation is performed within the app; the backend is the single source of truth.

---

## 5. Validation Rules

1. Phone-based OTP fields must match the format specified by the backend (6-digit numeric).  
2. Cart submission requires at least one service ID; otherwise, the server shall reject the request.  
3. Quotation acceptance is validated on the backend. If the quote is not “Shared,” an error (HTTP 409) is returned.  
4. KYC uploads are limited to 10 MB and must be of type PDF/JPG/PNG; the app should pre-check file size and MIME type before upload.  
5. For offline usage: any attempt at lead creation, status update, or KYC upload shall result in an error message prompting the user to reconnect.  

---

## 6. Endpoints & Methods

Below is a non-exhaustive list of key API calls (all endpoint paths are illustrative; actual paths may differ slightly in implementation):

1. Authentication  
   - POST /api/v1/auth/login  
     - Body (Customer OTP flow): { phone: "9999999999", otp: "123456" }  
     - Response: { success: true, data: { token: "<JWT>", role: "Customer" } }

2. Service Catalog  
   - GET /api/v1/services  
     - Retrieves available service items (IDs, names, descriptions, base prices).

3. Cart/Lead Creation  
   - POST /api/v1/leads  
     - Body: { origin: "Customer", services: [ {serviceId, remarks?} ], ... }  
     - Response: { success: true, data: { leadId: "LEAD-101" } }

4. Quotation Viewing & Acceptance  
   - GET /api/v1/quotations?leadId=LEAD-101  
     - Returns an array of quotations for a given lead.  
   - PATCH /api/v1/quotations/{quoteId}/accept  
     - Allows the customer to accept a shared quotation.  
     - Body: { accept: true }

5. KYC Document Handling  
   - GET /api/v1/documents/sas?docType=KYC&leadId=LEAD-101  
     - Returns a short-lived upload URL.  
   - PUT <sasURL>  
     - Uploads the actual file (PDF/JPG/PNG up to 10 MB).  

6. Support Tickets  
   - POST /api/v1/supportTickets  
     - Body: { leadId: "LEAD-101", description: "Issue detail" }  
   - GET /api/v1/supportTickets?leadId=LEAD-101  
     - Returns all tickets for a lead, along with status (Open/Closed).

7. Offline & Notifications  
   - No dedicated endpoint for real-time push; the app uses GET calls to refresh data on user actions.  

---

By fulfilling these requirements, the Customer App (CUSTAP) will adhere to the constraints, clarifications, and overall design goals laid out in higher-level documents (L1-FRS, L1-HLD). This specification ensures all relevant user flows—service discovery, lead creation, quotation acceptance, KYC uploads, and basic support ticketing—are handled within the moderate scale of 400–600 concurrent users.