
## L3-TSD-BCKND: Technology Stack Documentation (TSD) for BCKND Document

### 1. Introduction
This document describes the specific technologies, frameworks, and tools used by the Backend (“BCKND”) component of the Solarium Green Energy solution. It builds on the system-wide decisions noted in the L1-TSD and adapts them for BCKND’s “modular monolith” approach. The scale target of approximately 400–600 concurrent users informs the choices below, ensuring a pragmatic balance between performance and maintainability.

---

### 2. Programming Languages & Versions

#### 2.1 Node.js / TypeScript
• Runtime: Node.js 18 LTS (containerized)  
• Language: TypeScript (compiled to JavaScript)  
• Rationale:  
  - Aligns with the broader solution’s JavaScript/TypeScript stack.  
  - Node.js 18 LTS offers long-term support, stability, and security updates suitable for container deployments.

---

### 3. Frameworks & Libraries

#### 3.1 Express
• Description: Minimalist Node.js web framework.  
• Version: Express 4.x  
• Rationale:  
  - Flexible, well-known approach for building REST APIs.  
  - Easy integration with third-party middleware.  
  - Fits the modular monolith design without imposing heavy overhead.

#### 3.2 Knex (Strict Version Lock)
• Description: SQL query builder and migration tool for PostgreSQL.  
• Version: 2.4.1 (locked in package.json for predictable behavior)  
• Rationale:  
  - Provides transparent migrations and rollback.  
  - Reduces ORM overhead while allowing direct SQL control.  
  - Strict version locking prevents breaking changes from minor upgrades.

#### 3.3 Winston (Logging)
• Description: Popular logging library that can output in JSON or other formats.  
• Version: 3.x  
• Rationale:  
  - Flexible and pluggable log transports; easy to integrate with Azure or console output.  
  - Emits structured logs that can be picked up by Azure services (Application Insights, Monitor).

#### 3.4 pdfmake (Server-Side PDF Generation)
• Description: JavaScript library to generate PDF documents from JSON-based definitions.  
• Version: 0.2.x or latest stable  
• Rationale:  
  - Straightforward text and layout generation for quotations or reports.  
  - Does not require the overhead of a headless browser (Chromium/Puppeteer) for this scale.

#### 3.5 Jest (Testing)
• Description: All-in-one JavaScript/TypeScript test runner, assertion library, coverage tool.  
• Version: 29.x or latest stable  (kept in sync with TypeScript updates)  
• Rationale:  
  - Strong community adoption and TypeScript support.  
  - Simplifies test configuration for unit and integration tests in a single toolset.

---

### 4. Databases

#### 4.1 Azure Database for PostgreSQL
• Description: Managed PostgreSQL service for structured data (leads, users, quotations, commissions).  
• Hosted Service: Azure Database for PostgreSQL (Flexible Server)  
• Rationale:  
  - Offers point-in-time recovery, daily backups, and scaling that meets ~400–600 concurrency.  
  - Seamless integration with Azure Container-based deployments.  
• Access & Queries:  
  - BCKND uses Knex for schema migrations and query building.  
  - Last-write-wins concurrency approach (no row-level version checks).

---

### 5. Server & Hosting

#### 5.1 Containerized Node.js on Azure Web App
• Base Image: node:18-alpine  
• Rationale:  
  - Alpine image is lightweight with smaller footprint for efficient pulls and deployments.  
  - Sufficient for typical Node.js modules used here.  
• Hosting Environment: Azure Web App (Container)  
  - Can scale vertically (larger instance) or horizontally (multiple container instances).  
  - Integrated with Azure networking, logging, and environment variable management.

---

### 6. DevOps Tools & Processes

#### 6.1 Source Control & CI/CD
• Repository: Typically Git-based (e.g., GitHub).  
• CI/CD:  
  - Automated builds and tests on each push or pull request.  
  - Container image built with Node.js 18 on Alpine, using Express + TypeScript.  
  - Deployment to staging and production via manual approvals.

#### 6.2 GitHub Advanced Security / Dependabot
• Purpose: Automated dependency vulnerability scanning and security checks.  
• Rationale:  
  - Helps ensure that known CVEs or library issues are identified promptly.  
  - Automatically proposes updates for minor or patch releases, aligned with strict version pinning in critical packages.

#### 6.3 Azure Application Insights (Monitoring)
• Purpose: Collect runtime metrics, performance data, and errors for the Node.js backend.  
• Rationale:  
  - Provides a single pane of glass to view request performance, logs, and traces.  
  - Integrates easily with Winston logs and Express middleware.

---

### 7. Third-Party Services

#### 7.1 MSG91 (SMS)
• Use Case: Sending OTP-based authentication codes for Channel Partners and Customers.  
• Integration: BCKND triggers SMS events via MSG91 REST APIs.  
• Monitoring: Basic fallback if MSG91 is down (logging warnings, no automated switch to another SMS provider).

#### 7.2 SendGrid (Email)
• Use Case: Sending password reset links for Admin/KAM, system notifications.  
• Integration: BCKND calls SendGrid REST APIs using stored credentials.  
• Monitoring: BCKND logs any failures to Winston logs / Azure insights. No backup email service is currently used.

#### 7.3 Azure Blob Storage
• Use Case: Storing uploaded documents (KYC, lead attachments) and generated PDFs.  
• Integration: BCKND issues short-lived SAS tokens, verifying user/role prior to file operations.  
• Retention: 7-year default for compliance, consistent with system-wide policy.

---

### 8. Final Notes
• This technology stack is tailored for the ~400–600 user concurrency range, balancing simplicity and future flexibility.  
• The modular monolith approach in BCKND avoids excessive microservice overhead.  
• Strict version locking in key libraries (Knex) helps maintain stable migrations.  
• Logging, monitoring, and CI/CD choices (Winston, Azure App Insights, GitHub Advanced Security) ensure continuous observability and secure operations without undue complexity.

```
