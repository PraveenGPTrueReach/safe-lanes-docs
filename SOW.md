# Statement of Work (SOW) -- Full Replacement
*(This document supersedes all prior SOWs and is the definitive reference.)*

## 1. Project Overview

Solarium Energy will have a unified Lead Management Platform comprising:

- **Channel Partner Mobile App**
- **Customer Self-Service Mobile App**
- **Web Portal for Admin & KAM**
- **Ground-Up Backend Services**

All components will work together to capture, track, quote, document, and resolve sales requests for solar installations.

## 2. Objectives

- **Efficient Lead Handling:** CPs and Customers can create and view leads; Admin/KAM can monitor and manage them.
- **Automated Quotation:** Real-time pricing, PDF generation, sharing, and immutable archives.
- **Comprehensive Audit Trail:** Full history of status changes, documents, tickets, and interactions.
- **Self-Service for Customers:** Registration, request history, document upload, and support tickets.

## 3. Scope of Work

The Project has 4 Components - CPAPP , WEBPRT , CUSTAP AND BCKND

CPAPP : Channel Partner Mobile App 
WEBPRT : Web based interface for Admin and KAM 
CUSTAP : Mobile App for end customers or consumers
BCKND : Backend infrastructure for the solution

### 3.1 Channel Partner Mobile App

1. **Authentication & Profile**
   - SMS-OTP login; app auto-navigates to Home on successful OTP.
   - Profile view/edit (name, email, address); phone locked.
   - KAM contact card with direct call button.

2. **Home Dashboard**
   - Counters for Today's Pending, Overdues, Total Leads.
   - Tap counters to list corresponding leads.

3. **Lead Management**
   - Create new leads (name, phone, address, pin); default follow-up date = T+1.
   - List all leads sorted by follow-up date (oldest first); separate "New Leads" view.
   - Enforce next-follow-up date on every status change; status change blocked without it.
   - New status: **Physical Meeting Assigned**.
   - In-line "Call Customer" button wherever customer phone appears.

4. **Status & Actions Matrix**

| **Status** | **Follow-up Date** | **Quotation #** | **Generate Quote** | **Site Survey** | **Docs View** | **Docs Upload** |
|------------|-------------------|----------------|-------------------|----------------|--------------|----------------|
| **New Lead** | T+1 (default) | No | No | No | No | No |
| **In Discussion** | Yes | No | Yes | No | Yes | Yes |
| **Not Responding** | Yes | No | No | No | Yes | Yes |
| **Physical Meeting Assigned** | Yes | No | Yes | Yes | Yes | Yes |
| **Won** | Yes | Yes | Yes | Yes | Yes | Yes |
| **Pending at Solarium** | Yes | No | Yes | No | Yes | Yes |
| **Under Execution** | Yes | Yes | No | Yes | Yes | Yes |
| **Not Interested** | No | No | No | No | Yes | Yes |
| **Other Territory** | No | No | No | No | Yes | Yes |
| **Executed** | No | Yes | No | No | Yes | Yes |

5. **Lead Details & History**
   - Editable fields, remarks, token number (where required).
   - "Save Changes" button with "View Timeline" link immediately below.
   - Full chronological audit of all status changes.

6. **Quotation Wizard**
   - Add products/services to cart; real-time pricing logic.
   - Generate PDF quote; share via email or device share sheet; archived immutably.

7. **KYC Document Management**
   - Upload/replace/delete . (PDF/JPG/PNG, ≤10 MB).
   -  Documents will be uploaded on document types. 
   - Dropdown selection of document type; status of each document type (Pending/Approved/Rejected).
  -  Documents will be uploaded on document types. 
 




8. **Extras**
   - Three-dot menu on Home: "Earnings" screen.
   - No payment-gateway integration.

### 3.2 Customer Self-Service Mobile App

- **Registration/Login** by phone; links all existing requests by phone.
- **Services:** catalogue Card list grouped by status; "New Purchase / Installation Request" etc. 
- **Services Wizard:**
  1. Select products/services from catalogue → Add to cart -- Confirm address/details → creates lead.
- **Documents:**  7-type of KYC document upload with verification status. 
- **My Records:** List of all the leads created by the Customer. Clickable to view details of the lead. 
- **Help:** Same KAM/CP contact cards and 1800 helpline. Along with this ability to create Lead/service wise support ticket.


### 3.3 Web Portal (Admin & KAM)

- **Login & Profile:** Email/password; "Forgot Password" reset.
- **Dashboards:** Funnel charts of all statuses; date filters; CP-level filters; CSV export.
- **User Management:**
  - Admin: Create/edit/deactivate CPs & KAMs; override phones.
  - KAM: View assigned CPs (read-only).
- **Lead Management:**
  - All-or-nothing CSV bulk upload with row-level error reporting.
  - Lead list/edit; enforce status matrix and mandatory follow-up dates.
  - Activity/history panel on lead details.
- **Quotation Management:** View and generate quotes; immutable archives.
- **Master Data:** CRUD for products, BOM items, discounts, state-DISCOM fees.

### 3.4 Backend Services (Ground-Up)
*

- **Authentication:** JWT-based, role-based access for CP, Customer, KAM, Admin.
- **Lead APIs:** Full CRUD, status-transition enforcement, follow-up scheduling.
- **Quotation Engine:** Line-item cost, discount, tax, subsidy calculations; PDF job queue; object storage.
- **Document & Ticketing:** Type-aware uploads, status tracking; threaded ticket conversations.
- **Catalogue & Cart:** Product listings, cart sessions APIs.
- **Notifications:** SMS (MSG91) and email (SendGrid) for OTPs, approvals, quote shares, ticket replies.
- **Audit & Concurrency:** complete change logs.
- **Storage:** PostgreSQL for data;
- **Security:** HTTPS everywhere; RBAC; encrypted data at rest; regular backups.

### 3.5 Integrations & Testing

- **Third-Party:** MSG91 SMS, SendGrid email.
- **QA:** Automated unit/integration tests; manual UAT with CPs & Customers; performance/load tests for 300--400 concurrent users.

### 3.6 Deployment & Handover

- **Environments:**  Production with CI/CD pipelines.
- **Artifacts:** Source code repos, OpenAPI specs, mobile builds, deployment scripts.
- **Documentation:** User guides (CP, Customer, Admin/KAM), API docs.
- **Support:** 3-month warranty; optional ongoing maintenance.

## 4. Deliverables

- Mobile app binaries & source
- Web Portal source & builds
- Backend services and API documentation
- Test reports & UAT sign-off
- Deployment scripts & infrastructure-as-code
- End-user manuals and training session