## L1-TSD: Technology Stack Documentation (TSD) (System-Wide) Document

This document describes the key technologies, frameworks, and tools selected for the Solarium Green Energy solution at the overall (system-wide) level. It reflects our target of approximately 400–600 concurrent users and emphasizes a pragmatic, modular monolith approach in the backend. Component-level specifics (e.g., particular plugins used by the Channel Partner or Customer apps) belong to separate L3-TSD documents.

---

### 1. Overall Context

Solarium Green Energy’s software solution integrates:  
• A backend “modular monolith” deployed on Azure Web App containers (Node.js runtime).  
• Two React Native mobile apps (Channel Partner and Customer).  
• A React + TypeScript Single-Page Application (SPA) for Admin and KAM.  
• Azure Database for PostgreSQL for structured data storage.  
• Azure Blob storage for unstructured files (KYC images, lead documents, PDF quotations).  

All major architectural decisions prioritize reliable operation for ~400–600 concurrent users, straightforward deployment, and a balanced feature set without overengineering.

---

### 2. Programming Languages & Versions

#### 2.1 Node.js (Backend)
• Language: JavaScript/TypeScript running under Node.js  
• Version: Standardize on Node.js 18 LTS (or later stable LTS) for better longevity and ongoing community support.  
• Rationale: Balances modern features with proven stability, suitable for containerized deployments on Azure Web App.  

#### 2.2 JavaScript/TypeScript (Frontend + Backend)
• Both the backend and web frontend are in TypeScript to ensure type safety, reduce runtime errors, and unify code quality practices across teams.  
• The mobile apps use JavaScript/TypeScript in React Native for consistent language usage with shared code possibilities.

---

### 3. Frameworks & Libraries

#### 3.1 Web Frontend: React 18.x
• Chosen for the Admin and KAM SPA (WEBPRT).  
• Integrates with TypeScript for strongly typed UI components.  
• Well-supported ecosystem of libraries for routing, state management, and UI scaffolding.  

#### 3.2 Mobile Apps: React Native 0.70+
• Shared JavaScript/TypeScript code base for CP (Channel Partner) App and Customer App.  
• Streamlines cross-platform development for iOS/Android.  
• Minimizes overhead while allowing ample community support and plugin availability.  

#### 3.3 Database Access & Schema Migrations
• We use Knex (JavaScript query builder) for structured SQL queries and built-in migration tools.  
• Rationale: Offers straightforward, transparent schema versioning/rollback without the heavyweight overhead of a large ORM.  

#### 3.4 Notification Model
• No real-time push server is implemented. Poll-based or on-demand sync logic is used in apps to detect new data or status changes.  
• SMS messages (OTP and critical alerts) complement the poll-based approach for time-sensitive events (e.g., CP or Customer sign-in).  

---

### 4. Database

#### 4.1 Azure Database for PostgreSQL (Flexible Server)
• Primary relational store for leads, quotations, user accounts, commissions, KYC statuses, and more.  
• Provides stable transactions, strong consistency, and scalable options up to the project’s ~600 concurrent user capacity.  
• Supports point-in-time recovery and daily backups, aligning with the documented 7-year data retention policy.

---

### 5. Hosting & Deployment

#### 5.1 Azure Web App (Containerized)
• The backend (Node.js modular monolith) runs as a Docker container on Azure Web App, ensuring a consistent runtime environment.  
• Provides easy scaling options: we can manually scale up/out if concurrency rises sharply beyond 400–600.  

#### 5.2 Deployment Pipeline
• Source Code: Hosted in Git (Azure Repos or GitHub).  
• CI/CD: Automated builds (Node.js + React + React Native) via Azure DevOps or GitHub Actions.  
• On successful build and tests, artifacts are deployed to “staging” for validation; then promoted to “production” with manual approvals.  
• Rollbacks are handled by retaining previous container images and tested migrations.

#### 5.3 Configuration & Secrets
• Sensitive credentials (DB connection strings, MSG91, SendGrid keys) are stored in encrypted environment variables under Azure App Service.  
• Manual rotation is performed as needed. Azure Key Vault integration can be considered for future expansions, but not presently mandated.

---

### 6. DevOps Tools & Process

1. **Version Control**  
   • Git-based workflow with feature branches, pull requests, and code reviews to maintain quality.  

2. **Build & Test**  
   • Automated job steps compile the TypeScript code, run unit/integration tests (e.g., via Jest or Mocha), and generate container images for the backend.  

3. **Release Management**  
   • Staging environment for final QA checks, verifying migrations (Knex) and any new capabilities.  
   • Manual promotion to production ensures oversight for each release cycle.  

4. **Monitoring & Logging**  
   • Azure Application Insights or equivalent for performance metrics and logs.  
   • Container/activity logs are kept for audits; lead timelines track all data changes for compliance.

---

### 7. Third-Party Services

1. **SMS Gateway: MSG91**  
   • Used for phone-based OTP during CP/Customer login.  
   • If MSG91 experiences significant downtime, we currently accept temporary SMS unavailability and inform Admin to notify users.  

2. **Email Service: SendGrid**  
   • Handles Admin/KAM password resets, system notifications, and certain escalation emails.  
   • Similarly, no backup email service is configured—downtime results in paused email capabilities until SendGrid is restored.  

3. **Azure Blob Storage**  
   • All KYC images, site photos (lead documents), and final PDF quotations are stored here.  
   • Each file is encrypted at rest. Access occurs via short-lived SAS tokens or BCKND middle-tier routes.  

---

### Conclusion

The above technology stack provides a stable, maintainable foundation for the Solarium Green Energy solution. By relying on Node.js 18 LTS, React 18, React Native 0.70+, Knex-based migrations, and Azure hosting, we balance modern features with reliable at-scale performance. This L1-TSD ensures alignment across all major technical decisions without overcomplicating future growth or day-to-day operations.  

All further component-specific library choices (e.g., UI kits, charting libraries) and extended configurations (e.g., offline logic or specialized push services) are documented separately in their respective L3-TSD documents.