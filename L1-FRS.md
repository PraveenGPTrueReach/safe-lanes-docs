## L1-FRS: Functional Requirements Specification (FRS) (High-Level)

### 1. Introduction
This document defines the high-level functional requirements for the Solarium Green Energy software solution. It serves as a reference for understanding the system’s key capabilities, user interactions, data handling constraints, and interfaces with external services. By focusing on overall behaviors and rules, it ensures that all major roles (Admin, KAM, CP, Customer) can effectively manage leads, quotations, commissions, and documents while maintaining data consistency at the current solution scale (~400–600) concurrent users. All references to “push notifications” throughout this specification are clarified as poll-based or manual sync alerts; real-time server-initiated pushes are not implemented at this scale.

### 2. Scope
The solution encompasses four major user personas:  
- Channel Partner (CP) using a mobile app for lead management and quoting.  
- Customer using a mobile app to request services, upload KYC, and accept/reject quotations.  
- Key Account Manager (KAM) using a web portal to oversee CP performance and handle lead reassignments in assigned territories.  
- Admin using a web portal with full authority for system configuration, user management, advanced lead overrides, and commission payouts.

At this level, the document focuses on functional objectives and constraints:  
1. Managing the entire sales funnel (New Lead, In Discussion, Physical Meeting Assigned, Customer Accepted, Won, Pending at Solarium, Under Execution, Executed, and possible terminal states like Not Responding, Not Interested, Other Territory).  
2. Handling quotations, commissions, KYC documents, multi-service cart-based service requests, and user authentications.  
3. Ensuring compliance with clarified policies on offline restrictions, phone number uniqueness, poll-based notifications, and a single-payment commission approach.

### 3. Functional Requirements

#### 3.1 User Management & Authentication
The system must allow four user roles with distinct permissions:  
- Admin and KAM authenticate with email/password via the web portal, session timeout ~30 minutes of inactivity.  
- CP and Customer authenticate via phone-based OTP (~2-minute validity, up to 5 attempts before a 15-minute lockout).  
- The same phone number cannot be used across multiple user records (CP, Customer, Admin, KAM) for the current design, unless overriding it manually via Admin. This enforces phone number uniqueness across roles.

Key behaviors:  
1. The system shall lock CP/Customer OTP requests for 15 minutes after five failed attempts.  
2. Admins can deactivate or reactivate CPs and KAMs, but CP deactivation requires that no active (non-terminal) leads remain assigned.  
3. The web portal shall enforce session logout after 30 minutes of user inactivity.

#### 3.2 Lead Management
This subsection introduces how leads move through various statuses and how they are created or reassigned. By specifying clear status transitions and mandatory fields, the system ensures consistent tracking of prospective solar projects.

Leads represent the primary object for tracking potential solar projects:  
1. The system shall create leads originating from CP, Customer (via a multi-service cart checkout), Admin, or KAM.  
2. Offline creation is disallowed: any new or updated lead must be submitted while online.  
3. Each lead shall have a status that follows a defined matrix, for example:  
   • New Lead → In Discussion → Physical Meeting Assigned → Customer Accepted → Won → Pending at Solarium → Under Execution → Executed  
   plus any relevant terminal states such as Not Responding, Not Interested, or Other Territory.  
4. If the customer approves a shared quotation, the system shall update the lead to a “Customer Accepted” status. A CP or Admin must then verify details and manually set it to “Won” if appropriate.  
5. The system shall record a remark (≥ 10 characters) and, for certain statuses, mandatory fields like next follow-up date (≤ 30 days in future), Quotation Ref, and Token No.  
6. Admin/KAM can override normal status progression, with every change logged in a timeline (audit trail).  
7. The system shall allow Admin or KAM to reassign leads among CPs, provided those CPs are in the relevant territory scope.

#### 3.3 Quotation Management
This subsection covers how quotations are generated and shared, ensuring transparent pricing and final locking of quotes for calculation. It introduces the roles of CP, KAM, and Admin in handling quotes, along with constraints on acceptance and drafting.

Quotations define pricing, subsidies, and configurations for leads:  
1. CPs (mobile) must finalize quotations in one online session; no partial “draft” is retained on the server for CP user flow at this scale.  
2. Admin/KAM (web) may use a partial-save “draft” mode, with completed quotations locked as immutable “Created.”  
3. Quotations can be marked “Shared” to become visible to the customer. If a customer accepts it, the lead can then be moved to “Customer Accepted,” and subsequently to “Won” if appropriate.  
4. The system shall generate a PDF for each finalized quotation, storing it in a secure repository.  
5. Exactly one accepted quotation may be associated with a lead’s final “Won” state for commission calculation.  
6. CSV imports or exports of leads must be all-or-nothing (validation fails if any row is invalid).

#### 3.4 Document & KYC Management
This subsection outlines how user-uploaded documents (KYC and lead-level files) are handled, setting size limits, file types, and states of approval or rejection to ensure a controlled document lifecycle.

Document handling covers both customer-level KYC files and lead-level attachments:  
1. The system shall allow up to seven lead-level documents (PDF/JPG/PNG up to 10 MB each).  
2. KYC document types (Income Proof, Light Bill, etc.) track statuses (Pending Review, Approved, Rejected).  
3. Unlimited re-uploads are permitted if a KYC doc is rejected or still pending; however, the solution owner may introduce additional checks or escalation if multiple repeated rejections occur.  
4. Approved KYC docs cannot be replaced.  
5. The system shall store older file versions but only display the latest version to the user or CP/KAM/Admin.

#### 3.5 Commission & Payout
This subsection focuses on how the system tracks and disburses commissions for Channel Partners after a lead reaches “Executed.” The aim is to provide a straightforward, single-payment approach.

Administrators handle CP commissions once leads are “Executed”:  
1. Commission records must automatically derive from the final chosen quotation’s dealer add-on or margin.  
2. Single-payment approach: commissions follow a simple Pending → Approved → Paid lifecycle.  
3. The system must allow Admin to record UTR details and date upon marking a commission as “Paid.”  
4. CPs track their commissions in a mobile app “Earnings” screen, updated upon each sync or refresh.  
5. For the current scope, partial or milestone-based payouts are not supported, avoiding unnecessary complexity at the ~400–600 concurrent user scale.

#### 3.6 Customer Service Requests (Cart) & Quotation Acceptance
This subsection explains how customers request services via a mobile ‘cart’ and how quotation acceptance or rejection is handled in the app-based workflow.

Customers browse services in an app-based “cart”:  
1. The system shall allow multiple services within a single cart submission, yielding one lead that references all requested services.  
2. Submission of the cart creates a “New Lead” (Origin = Customer).  
3. Customers can view and accept or reject any “Shared” quotation. If a customer chooses “Accept,” the lead transitions to “Customer Accepted.” The CP or Admin must then update it to “Won” after verifying any needed details.  
4. A customer may raise support tickets for the lead once the lead is Executed, consistent with other workflow documents.  
5. A customer may still upload or re-upload relevant KYC documents unless they are approved, in which case re-upload is disallowed.  
6. For the current design, the detailed Quotation Wizard primarily supports single solar-system configurations. If the user selected multiple distinct services, the CP or Admin may need to handle each configuration individually.

#### 3.7 Offline & Notification Requirements
This subsection describes how the system behaves when users are offline and clarifies how they receive new updates. The solution avoids real-time push infrastructures, opting for manual or periodic polling to check for changes.

**Offline**:  
1. CP’s mobile app offers read-only access to previously downloaded data when offline. Attempting new leads, updating status, or uploading documents offline is disallowed.  
2. Customers may also view previously seen data offline, but cannot submit new requests or changes.

**Notifications**:  
1. The solution shall rely on poll-based or manual sync to retrieve new events (e.g., lead updates, commission statuses).  
2. References to “push notifications” indicate in-app alerts triggered upon polling or manual sync; no real-time push channels are active.  
3. SMS is used exclusively for OTP delivery to CP/Customer phone numbers.

### 4. System Interfaces

#### 4.1 Internal Interfaces
This subsection outlines how each major component of the Solarium solution interacts with the others. It ensures clear boundaries for mobile apps, the web portal, and the backend’s role in data validation and persistence.

1. CP Mobile App: Communicates with the system over a REST API, primarily for lead creation, quotations, KYC uploads, and offline read caching.  
2. Customer Mobile App: Provides service catalog exploration, multi-service cart-based lead submission, KYC upload, and support ticket creation (where permitted).  
3. Web Portal (Admin & KAM): Offers comprehensive functionality for user management, lead oversight, quotation generation, and commission approvals.

#### 4.2 External Services
This subsection describes the third-party integrations essential for OTP verification, email workflows, and file storage. It sets expectations on reliability and fallback requirements if these services fail.

1. SMS Gateway (e.g., MSG91) for OTP.  
2. Email Service (e.g., SendGrid) for password resets, admin notifications.  
3. Secure file storage for documents and quotation PDFs, ensuring version retention (implementation details in lower-level design).

### 5. Data Requirements

#### 5.1 Primary Entities
This subsection enumerates the core data objects within the solution, specifying how each supports the overall workflows of leads, quotations, and commissions.

1. User / Role Data: Distinctions for CP vs. Customer vs. Admin vs. KAM, role-based access, unique phone or email.  
2. Leads: Status transitions, assigned CP, references to customers, timeline logs, and ability to contain multiple requested services if created by the customer cart.  
3. Quotations: Config info (panels, inverters, fees), draft vs. created vs. shared states, PDF references, acceptance flags.  
4. Documents & KYC: Up to 7 lead-level docs, multiple KYC file types (Pending, Approved, Rejected), file size ≤ 10 MB.  
5. Commission Records: Mapped to executed leads, single lumpsum payment.  
6. Timeline / Audit Logs: Stores user ID, old/new state, timestamps for every significant update.

#### 5.2 Storage & Retention
This subsection covers how long each data category remains in the system before archiving or deletion, ensuring legal and operational compliance.

1. All final data must be persisted in a relational database with enough capacity for ~400–600 concurrent sessions.  
2. Document files (KYC, lead docs, PDF quotes) are stored outside the DB in a secure and versioned repository.  
3. Historical KYC versions remain archived for compliance, but only the newest version is shown to end users.  
4. The solution shall retain leads, documents, and user records for up to seven years, after which data purging or archival may occur to comply with document retention policies.

### 6. Validation Rules

#### 6.1 Data Input Validations
This subsection ensures that user inputs and file uploads meet mandatory format and business constraints, preventing invalid or malicious data from entering the system.

1. Phone-based OTP: Maximum 5 incorrect attempts, 2-minute validity, 15-minute lock on further attempts if exceeded.  
2. Phone Number Uniqueness: A CP’s or Customer’s phone number must not duplicate an existing user’s phone (across roles) unless corrected by an Admin.  
3. Lead Status Changes:  
   - Next follow-up date cannot exceed 30 days from the current date.  
   - Remarks must have at least 10 characters.  
   - Changing to “Customer Accepted” occurs automatically upon user acceptance if the quotation is “Shared.”  
   - Changing to “Won” requires a valid Quotation Ref and the lead to be in “Customer Accepted” status (unless overridden by Admin/KAM).  
   - Changing to “Under Execution” requires a Token No.  
4. File Upload Checks: PDF/JPG/PNG ≤ 10 MB with correct MIME type; lead docs limited to 7 per lead.  
5. Commission Payment: Single lumpsum approach, no partial disbursements.  
6. CSV Import: All-or-nothing row validation; partial acceptance is disallowed if any row fails checks.

#### 6.2 Business Logic Constraints
1. Only one accepted quotation per lead. Any additional acceptance replaces the previously accepted quotation.  
  
2. CP deactivation requires reassigning or closing all non-terminal leads before final removal.  
3. Unlimited re-uploads for rejected KYC documents, though future policy may impose a limit or escalation flow.  
4. The system relies on poll-based notifications rather than real-time push for major events.  
5. If external SMS/email gateway fails, the system logs an error and may attempt retries or display an error message to the user.  
6. The system applies a simple last-write-wins approach for concurrent data edits; any overwriting is logged in the timeline to mitigate conflicts.

---

*Note: The numbering above reflects consolidation after removing the empty placeholder bullet.*