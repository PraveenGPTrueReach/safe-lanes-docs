## L1-OVERVIEW: Solution Overview (High-Level)

### 1. Business Overview
This solution provides an end-to-end Lead Management Platform for solar product sales at Solarium Green Energy. It addresses:  
- Capturing leads from multiple channels (Channel Partners, direct Customer requests, and Admin/KAM entries).  
- Managing sales funnels (status transitions, follow-ups, quotations, and commission payouts).  
- Centralizing documentation (lead-level files, KYC documents, immutable quotation archives).  
- Enforcing workflows and security for Admin, KAM, Channel Partners, and Customers in a unified system.

### 2. Key Entities

1. **User Roles**  
   - Channel Partner (CP): Creates/manages leads, uploads KYC on behalf of customers, generates quotes, and earns commissions.  
   - Customer: Initiates service requests, views/accepts or rejects quotations, uploads personal KYC documents.  
   - Key Account Manager (KAM): Oversees CP performance, manages leads in assigned territories, approves or rejects KYC docs.  
   - Admin: Full system authority—manages users (CPs, KAMs), overrides lead statuses, configures system data, and handles commission payouts.

2. **Data Objects**  
   - **Leads**: Central record with customer, status, follow-up dates, and attachments.  
   - **Quotations**: Pricing records with cost breakdown and subsidy calculations, mapped to a single lead.  
   - **KYC Documents**: Customer identity and financial documents.  
   - **Tickets**: Support or service-related issues.  
   - **Products & BOM**: Catalog of panels, inverters, fees, and innate cost parameters for quote generation.

### 3. Flow of Data, Products, Money, and Other Assets

1. **Data Flow**  
   - Leads, documents, and quote data originate in the apps or web portal.  
   - They sync to the backend for centralized storage (PostgreSQL + Azure Blob for files).  
   - Automatic push notifications and status changes flow back to apps.  
   Lead-level documents are limited to a maximum of 7 attachments per lead, consistent with the detailed CP workflow.

2. **Product Flow**  
   - Stored in master data (panels, inverters, BOM, etc.) maintained by Admin.  
   - CP or KAM selects these items via a quotation wizard to generate final system pricing.

3. **Money Flow**  
   - No payment gateway integration.  
   - Commissions (CP earnings) are tracked in the system. Admin marks them “Approved” or “Paid,” sending notifications.  
   When a customer accepts a quotation via the Customer app, the lead now transitions into a “Customer Accepted” status (which has been added to the official status matrix). The CP (or Admin) must then confirm and finalize the lead as “Won,” providing an auditable short step before the lead is marked “Won.”

4. **Other Assets**  
   - Uploaded KYC files (PDF/JPG/PNG ≤10 MB) and generated PDF quotations are stored in Azure Blob Storage.  
   - Although the system currently retains documents with a default retention/archival policy of 7 years, we may revise this period or introduce advanced archival rules later—once load is observed—to manage costs and address future compliance requirements.

### 4. Key Components

1. **Channel Partner Mobile App (CPAPP)**  
   - React Native app supporting offline read caching only; no offline CRUD is supported.  
   - Manages leads, quotes, KYC uploads, and commission tracking.

2. **Customer Mobile App (CUSTAP)**  
   - React Native app for self-registration, lead/service creation, receiving/sharing quotes, uploading personal KYC docs, and raising tickets.

3. **Web Portal (WEBPRT)**  
   - React + TypeScript SPA.  
   - Provides separate role-based interfaces for Admin and KAM.  
   - Supports lead imports, advanced reports, user management, and master data configuration.

4. **Backend (BCKND)**  
   - Single “modular monolith” architecture.  
   - Hosted on Microsoft Azure (Web App / Container approach) with Azure Database for PostgreSQL Flexible Server (General Purpose) sized for ~400 concurrent users and tested for spikes up to ~600.  
   - Exposes REST APIs, handles authentication (JWT), business logic, and integrates with external services (SMS, email).

### 5. High-Level Architecture

- **Cloud Provider**: Microsoft Azure.  
- **Compute & Containers**: Azure Web App (containerized) hosting the backend.  
- **Database**: Azure Database for PostgreSQL Flexible Server for data.  
- **Object Storage**: Azure Blob for documents and PDFs, using secure pre-signed URLs.  
- **Mobile Apps**: React Native, packaged for iOS/Android.  
- **Web Frontend**: React SPA with TypeScript, communicating over HTTPS to REST endpoints.  
- **Integration Adapters**: MSG91 (SMS), SendGrid (email).  
- **Scalability Targets**: Baseline ~400 concurrent users, with short-term spikes up to ~600.  
- **Security**: TLS for all traffic; at-rest encryption (AES-256) for data and object storage.  
- **Backup & DR**: Daily full backups and point-in-time recovery (PITR); currently single-region deployment (with geo-redundant storage as an option) and the possibility of adding a secondary region if future SLAs demand.  
- All Blob access uses short-lived (~5 minute) SAS tokens with optional IP-range filters, ensuring minimal access scope for uploads and downloads.

### 6. Non-Standard Elements

1. **Lead Status Matrix**  
   - Enforces controlled transitions and mandatory fields (e.g., remarks, follow-up date, token number).  
   - A new “Customer Accepted” status is now included in the matrix. Once the customer approves the quotation, the lead enters this transitional status until CP or Admin finalizes it to “Won.”
   - Allows override by Admin/KAM but logs all changes.

2. **Commission Module**  
   - Not a typical payment gateway; rather, an internal tracking and approval system for the CP’s payouts.

3. **Tiered Quotation Wizard**  
   - Automatic subsidy calculation, dynamic product selection, and final PDF locking for immutability.

### 7. High-Level Deployment & Integration Strategy

1. **Deployment**  
   - Containerized backend deployed to Azure Web App.  
   - React SPA served via Azure Web App or CDN.  
   - Two mobile apps built in React Native, distributed via App Store/Play Store.  
   - Environment: initially single-region production deployment with a future option of enabling high availability if business justifies it.  
   However, we will enable Azure geo-redundant storage (GRS) from day one, and we may optionally configure a read replica for PostgreSQL to cover the most basic DR needs.

2. **CI/CD**  
   - Source control in a Git repository (e.g., Azure Repos or GitHub).  
   - Automated build pipeline for each push (backend, web, mobile).  
   - Automated tests (unit + integration).  
   - Continuous delivery to a staging environment; manual promotion to production.

3. **Integration**  
   - **Third-Party APIs**: MSG91 for SMS OTP, SendGrid for email.  
   - **Logging & Monitoring**: Azure Monitor, Application Insights.  
   - **Security**: Role-based access tokens for every API endpoint, with enforced expiration.

4. **Implementation Considerations**  
   - Deploy as a “modular monolith” for current scale, with a flexible single-region approach.  
   - We plan to observe real-world load and only then consider a second region or additional overhead (e.g., GRS, read replicas) if uptime SLAs or usage demand it. (See the note above on minimal GRS/DR configuration for day-one readiness.)  
   - Maintain a 7-year retention policy for documents, with a plan to add TTL or manual archival in future increments if required.  
   - Performance has been tested for surges near 600 concurrent sessions, and typical usage should remain near 400. Further scaling can be handled by adding compute resources or read replicas when usage grows.