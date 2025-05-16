
## L3-CM-BCKND: Configuration Manual (CM) for BCKND

### 1. Introduction
This Configuration Manual provides administrators with a comprehensive guide to configuring the BCKND (Backend) component of the Solarium Green Energy solution after deployment. The BCKND is a containerized Node.js “modular monolith” responsible for the system’s business logic, data persistence, and integrations (SMS, email, blob storage) at a scale of ~400–600 concurrent users.

This document focuses on:
- Core system settings relevant to BCKND behavior.  
- Parameter definitions and acceptable values.  
- Environment-specific configurations (Development, Staging, Production).  
- Integration settings with external services.  
- Security-related configurations and encryption parameters.  

All configurations are designed to be as simple as possible while allowing room for future adjustments without introducing unnecessary complexity.

---

### 2. Component System Settings
Here are the key system settings and features that administrators can configure post-deployment.

#### 2.1 Application Configuration File or Environment Variables
All settings described in this manual can be defined as environment variables (e.g., in Azure Web App “Configuration”, or a .env file in a non-containerized approach). A typical Node.js container deployment will load these environment variables at startup.

Below is an overview of high-level settings:

1. NODE_ENV  
   - Typical values: “development” or “production”.  
   - Affects logging verbosity and error detail.  
   - Must be set to “production” in Production environments for best performance and minimized debugging outputs.

2. PORT  
   - The container listening port.  
   - Default is 80 or process-assigned for Azure Web App Container.  
   - Usually, you will not need to override this in Azure; ensure the Dockerfile exposes the correct port.

3. LOG_LEVEL  
   - Defines logging verbosity (e.g., “info”, “warn”, “error”, “debug”).  
   - Recommended “info” for Production and “debug” or “info” for Staging/Development.

4. MAX_DOCUMENTS_PER_LEAD  
   - Sets the maximum number of documents allowed per lead.  
   - Default is 7 in line with the business requirement.  
   - If the value is exceeded, the BCKND returns an HTTP 409 (Conflict).

5. MAX_DOCUMENT_SIZE_MB  
   - Restricts file upload size (MB).  
   - Default is 10 MB as per business requirement.  
   - If a file exceeds this size, the upload fails with a 413 (Payload Too Large).

6. FEATURE_CUSTOMER_ACCEPTED  
   - A simple on/off feature flag.  
   - Default: “true” (the system retains “Customer Accepted” as an official status).  
   - If “false”, the lead flow transitions directly from “Quotation Accepted” to “Won,” skipping the “Customer Accepted” stage.  
   - Toggling this impacts the lead lifecycle and relevant API validations.

7. DOCUMENT_RETENTION_POLICY  
   - Represents the intended retention period in years (e.g., “7”).  
   - For reference only; no automated purge is enforced by default.  
   - Administrators use this to plan manual cleanup/archival after the specified timeframe.

---

### 3. Parameter Definitions
Below is a more detailed breakdown of key parameters and recommended values. All parameters are generally stored in environment variables.

| Parameter Name             | Description                                                                                   | Example/Default        | Notes                                                                                                                                          |
|----------------------------|-----------------------------------------------------------------------------------------------|------------------------|------------------------------------------------------------------------------------------------------------------------------------------------|
| NODE_ENV                   | Environment indicator.                                                                        | “production”           | Set to “production” in Production and “development” for local dev. Affects logging and error messages.                                        |
| PORT                       | HTTP port the Node.js app listens on.                                                         | 80                     | In Azure Web App Containers, the port is often automatically injected; verify container configuration.                                        |
| LOG_LEVEL                  | Logging verbosity.                                                                            | “info”                 | Use “debug” or “info” in dev/staging, “info” or “warn” in production to balance performance with sufficient logs.                             |
| DB_CONNECTION_STRING       | PostgreSQL connection string, including host, user, password, and database name.             | postgres://user:pwd@host:5432/db | Must be updated for each environment. Avoid storing in plaintext; recommended to link from Azure Key Vault or a secure pipeline variable.  |
| JWT_SECRET                 | Secret used for signing JWT tokens.                                                          | (Random 32+ chars)     | Keep secret. Rotate periodically. Store securely.                                                                                             |
| MSG91_API_KEY              | API key for SMS integration (OTP, notifications).                                            | (Provided by MSG91)    | Required if phone-based OTP is used for CP/Customer logins.                                                                                   |
| SENDGRID_API_KEY           | API key for email services.                                                                  | (Provided by SendGrid) | Used for system emails (Admin approvals, password resets, etc.).                                                                              |
| AZURE_BLOB_CONNECTION      | Connection string or endpoint for Azure Blob storage.                                        | See Azure Portal       | Required for document/PDF uploads.                                                                                                            |
| AZURE_BLOB_CONTAINER       | Name of the Azure Blob container.                                                            | “bcknd-documents”      | Adjust if using multiple containers or different naming.                                                                                      |
| MAX_DOCUMENTS_PER_LEAD     | Maximum number of documents that can be attached to a single lead.                           | 7                      | Enforced by the backend.                                                                                                                      |
| MAX_DOCUMENT_SIZE_MB       | Maximum size of each file upload in MB.                                                      | 10                     | Enforced by the backend.                                                                                                                      |
| FEATURE_CUSTOMER_ACCEPTED  | Enables or disables the “Customer Accepted” intermediate lead status.                        | “true”                 | If set to “false,” leads move directly to “Won” on acceptance.                                                                                |
| DOCUMENT_RETENTION_POLICY  | Reference for manual document retention in years.                                            | “7”                    | No automated purge; used as a guideline.                                                                                                       |
| MIGRATION_APPROVAL_REQUIRED| Whether schema migrations require manual approval before running in production.             | “true”                 | If “true,” a separate admin step is needed to confirm DB migrations.                                                                          |

---

### 4. Environment-Specific Configurations
BCKND typically runs in three main environments: Development, Staging, and Production. Below are the recommended configurations and key differences.

#### 4.1 Development
- NODE_ENV: “development”.  
- LOG_LEVEL: “debug” or “info”.  
- MIGRATION_APPROVAL_REQUIRED: “false” for quick iteration.  
- Usually connected to a local or test Postgres instance.  
- Lower security demands, but do not store secrets in plaintext if using a shared environment.

#### 4.2 Staging
- NODE_ENV: “production” (for realistic testing).  
- LOG_LEVEL: “info”.  
- MIGRATION_APPROVAL_REQUIRED: “true”, but typically approved automatically by DevOps or a staging admin.  
- Connects to a staging Postgres database, typically hosted in Azure.  
- Azure Blob container is separate from production to avoid mixing logs and documents.  
- SMS/Email keys might be test credentials or real keys with caution.

#### 4.3 Production
- NODE_ENV: “production”.  
- LOG_LEVEL: “warn” or “info”.  
- MIGRATION_APPROVAL_REQUIRED: “true” to ensure any schema changes are deliberately reviewed before running.  
- DB_CONNECTION_STRING: Points to the production Azure Database for PostgreSQL.  
- Secure secrets (e.g., JWT_SECRET, MSG91_API_KEY, SENDGRID_API_KEY) are managed via Azure Key Vault or pipeline variable groups.  
- Periodic manual or scheduled tasks to handle document archival or cleanup once older than 7 years (per organizational policy).  

---

### 5. Integration Configurations
Below are the main integration points that an administrator must configure for BCKND once the container is deployed. These integrations are controlled via environment variables or service connections.

#### 5.1 Database (PostgreSQL)
- A valid DB_CONNECTION_STRING is required for the BCKND to function.  
- Administrators must confirm the database migrations are aligned with the environment’s schema expectations.  
  - In Production, set MIGRATION_APPROVAL_REQUIRED = “true.”  
  - The deployment pipeline should prompt an admin to confirm schema upgrades before running.

#### 5.2 Azure Blob Storage
- Configure AZURE_BLOB_CONNECTION and AZURE_BLOB_CONTAINER.  
- The BCKND service must have permission (RBAC or connection strings) to read, write, and version files.  
- For large file requirements, confirm effective Azure Blob service tier (Hot/Archive). Default is typically “Hot” for immediate access.

#### 5.3 MSG91 (SMS/OTP)
- Set MSG91_API_KEY for CP/Customer phone-based logins and SMS notifications.  
- If the key is missing or invalid, phone-based login flows will fail.  
- Ensure region and compliance with local SMS regulations if applicable.

#### 5.4 SendGrid (Email)
- Set SENDGRID_API_KEY for system emails such as Admin or KAM password resets, and lead/quotation confirmations if used.  
- Check SendGrid plan limits for daily email volumes.  
- Validate the “from” email address domain verification in SendGrid.

#### 5.5 Optional Third-Party Services
- Any advanced integrations (e.g., third-party CRM or advanced analytics) must be configured similarly with environment variables.  
- By default, the BCKND only integrates with the listed providers. Additional connectors or webhooks require code updates and new environment variables.

---

### 6. Security Configurations
Security within the BCKND is configured through environment variables, Azure roles, and secret storage in Key Vault or equivalent. Below are critical considerations:

#### 6.1 Authentication and JWT Secrets
- JWT_SECRET must be long, random, and changed periodically.  
- Set token expiry durations within the code or environment variables if desired (e.g., 24 hours).  
- Keep secrets out of plain text in your repository; use secure storage (e.g., pipeline variables, Key Vault).

#### 6.2 Role-Based Access
- The BCKND enforces role checks on each endpoint (CP, Customer, KAM, Admin).  
- Ensure correct role assignments in the user directory or database.  
- If a user changes roles (e.g., CP becomes Admin), update the DB or identity store accordingly.

#### 6.3 Database Security
- Follow Azure guidance for restricting IP addresses or enabling private endpoints.  
- Enforce SSL connections from the BCKND to PostgreSQL.  
- Use separate credentials for Staging vs. Production to avoid accidental data overlap.

#### 6.4 SSL / TLS
- SSL termination typically occurs at Azure Web App or the container level.  
- Confirm TLS 1.2+ for inbound external traffic.

#### 6.5 Document Retention
- The solution references a 7-year retention policy but does not automatically purge files.  
- The recommended approach is to mark or manually archive older documents on a regular schedule.  
- Administrators can optionally configure Azure Blob Lifecycle Management if they wish to automate moving older files to cheaper storage tiers.

---

### 7. Additional Notes and References

1. DB Migration Approach  
   - Per Clarification #1, the recommended approach is to require a manual review step of migrations in Production (MIGRATION_APPROVAL_REQUIRED = “true”). In Development or Staging, you can auto-apply if desired.  

2. Document Retention  
   - Per Clarification #2, a manual or scheduled process for cleaning up older documents is preferred, rather than an automated Azure Lifecycle policy.  

3. “Customer Accepted” Status Configuration  
   - Per Clarification #3 and the “FEATURE_CUSTOMER_ACCEPTED” setting, you can toggle “true” or “false” to either keep or remove the intermediate lead status. For initial deployments, “true” is recommended, as it supports more detailed funnel reporting.  

4. References  
   - High-Level Design (L1-HLD) for overall architecture patterns and technology stack details.  
   - L3-DG-BCKND: Deployment Guide for instructions on deploying this container, including CI/CD pipeline steps, building/pushing Docker images, and staging to production swaps.  
   - L3-WF-BCKND: Workflow Details for additional context on how the BCKND processes requests, enforces business rules, and integrates with external services.  

---

All configuration items in this manual address the environment scale (400–600 concurrent users) and remain aligned with the solution’s current complexity. Administrators can adjust certain parameters (e.g., feature flags, log levels, document upload limits) without redeploying or re-building the container, provided these are managed as environment variables through Azure or equivalent services.
```
