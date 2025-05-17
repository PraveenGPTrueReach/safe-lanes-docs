## L1-OVERVIEW: Solution Overview (High-Level)

### 1. Business Overview
SafeLanes’ Rest Hours Submodule addresses maritime rest-hour compliance (MLC, STCW, OPA) by capturing crew work/rest logs, detecting violations, logging non-conformities (NCs), and providing multi-level analytics (vessel and office). The solution runs partially offline on vessels and synchronizes with an office server when connectivity is available. It extends the existing SafeLanes “Sail App” with modular front-end integration (Module Federation using Angular 16 for more robust support) and a standalone Nest.js back-end microservice.

Additionally, certain external or auditor-type users may be granted read-only access for compliance reviews, if configured in the SafeLanes RBAC settings. For clarity, references throughout other documents to “SAIL login” and “SafeLanes login” are synonymous, referring to the same credential and authentication system.

Note: If the existing Sail App is not yet on Angular 16 or higher, the SafeLanes team must upgrade it to ensure module federation compatibility. The Rest Hours microfrontend will remain on Angular 16. 
Alternatively, if the host environment cannot be upgraded immediately to Angular 16, a minimal fallback approach (e.g., using a Web Component wrapper or an iframe-based isolation) may be implemented to accommodate older Angular versions. This approach is only recommended as a temporary measure to reduce the immediate upgrade risk; it may limit certain microfrontend features and should ideally be replaced by a full host upgrade as soon as feasible.

### 2. Key Entities
- **Vessel**: Hosts local server + local DB (MySQL). Runs the Rest Hours service offline.  
- **Crew**: Have role-based permissions (User, Admin, Super Admin) to enter/approve hours.  
- **Office**: Primary location with aggregated data usage, advanced analytics, and managerial oversight. Roles include Office User, Office Admin, and Office Super Admin.  
- **Non-Conformities & Violations**: System-detected indicators of rest-hour breaches. Handled both at vessel and office levels.  
- **External**: Read-only (auditor) role that can view rest-hour data for oversight, without editing permissions.

### 3. Data Flow

1. **Onboard Data Entry**: Crew record daily rest/work blocks on the vessel (local DB).  
2. **Validation & Violation Checks**: The system applies rule-based checks (hard-coded thresholds and codes).  
3. **Offline Storage**: Entries remain in the local DB; updates are timestamped (UTC).  
4. **Daily/On-Demand Sync**: Data syncs to the office server. Last-write-wins is the primary conflict resolution mechanism for non-critical data; however, any rest-hour compliance data merges require an Admin review to avoid silent overwrites. Overwritten data is logged, and administrators may optionally review these logs after the merge. This approach preserves traceability and audit compliance by ensuring that overwritten data remains accessible for review. Additionally, the system can optionally notify users or admins when a conflict overwrite occurs, providing a minimal conflict summary interface if enabled.

   To maintain compliance, if both vessel and office edit rest-hour (or similar regulatory) data offline, the system will always prompt an Admin before overwriting, ensuring no silent overwrites occur for such fields.

   To further mitigate clock drift on vessels that remain offline for extended periods, the system can store local-time offsets or monotonic counters in parallel with UTC timestamps, although a consistent manual or NTP-based clock sync is strongly recommended. For the stated scale, this approach is typically sufficient to maintain accurate ordering of updates.

   To ensure reliable last-write-wins merges for non-critical data, each vessel server should maintain as accurate a system clock as possible—preferably through NTP or manual synchronization—so that UTC-based timestamps do not drift.

5. **Office Aggregation**: Office dashboards aggregate vessel data for fleet-level compliance tracking.  
6. **Predicted Violation Calculations**: A minimal rule-based forecast runs within the Nest.js microservice.  
7. **Data-in-Transit Encryption**: All data synchronization and REST calls are protected with TLS 1.2 or higher to safeguard sensitive crew information.

There is no direct financial flow in this module. The system primarily handles operational and compliance data.

### 4. Key Components

- **Microfrontend (Angular)**  
  In order to streamline Module Federation, the team will utilize Angular 16, which offers better integrated support for microfrontends and reduces configuration complexities. On vessels operating offline, the microfrontend can be loaded from a local server if remote fetching is unavailable.  
  Additionally, if the remoteEntry endpoint is unreachable, the system falls back to a bundled local copy (e.g., via environment-based or service-worker rewrite), ensuring the UI remains accessible without an external URL.

- **Nest.js Back-End Service**  
  - Provides REST endpoints for recording hours, retrieving schedule/plan data, detecting violations, and performing basic rule-based predictions.  
  - Handles authentication locally with cached JWT tokens for offline scenarios, re-validating when connectivity is restored.  
  - If tokens expire while the vessel is still offline, the local server can enforce a re-auth procedure under Master or Admin supervision. Instead of locally signed tokens, short-lived offline session keys are stored in the vessel’s local database, rotated frequently to reduce potential replay. These offline session keys are valid for a maximum of 14 days, after which re-authentication with Master or Admin override becomes mandatory, mitigating the risk of indefinite usage. Once connectivity is restored, re-auth with the central identity service is mandatory.  
  - Deployed with PM2, reverse-proxied by Nginx.

- **MySQL Databases**  
  - Main server DB for consolidated records (office).  
  - Local DB on each vessel for offline operation and daily/on-demand sync.

- **Authentication & RBAC**  
  Utilizes SafeLanes JWT-based auth. Existing roles are extended (or new claims created) for Vessel/Office user distinctions and optional External read-only access. Where needed, additional claims may be added in the identity service to enforce fine-grained rest-hours permissions. In fully offline mode on a vessel, pre-existing tokens allow continued access; if tokens approach expiry, onboard procedures and the local server’s re-auth logic handle credential renewal to maintain security. Instead of locally signed tokens, short-lived offline session keys are recognized only within the vessel environment and must be re-validated once connectivity to the central SafeLanes service is restored. These offline session keys are valid for a maximum of 14 days, after which re-authentication with Master or Admin override becomes mandatory, mitigating the risk of indefinite usage. Once connectivity is restored, re-auth with the central identity service is mandatory.  

  During fully offline periods, if a user’s JWT has expired, the local server issues a short-lived session key mirroring the user’s role and identity. This key must be revalidated with the central identity service upon reconnection, ensuring consistent RBAC enforcement.

### 5. High-Level Architecture
- **On-Prem Vessel Servers**: Each vessel runs the Nest.js service on a local server (Linux or Windows) with MySQL. Nginx and PM2 handle application processes.  
- **Office Server**: Another Nest.js instance (the same codebase) and MySQL run on on-prem or hosted hardware (16 GB RAM, 4 cores).  
- **Data Synchronization**: Timestamp-based merges for non-critical data. A brief overwrite log is kept for optional post-merge conflict review when both sides edit the same record offline. For any compliance or rest-hour data conflicts, the system prompts Admin review to prevent silent overwrite. Conflicting changes beyond the last-write-wins approach remain logged, preserving an audit trail for manual review if desired.  
- **Offline Microfrontend Loading**: During isolated operation, each vessel serves the Rest Hours microfrontend from its local server, ensuring the UI remains accessible when remote Federation endpoints are unreachable. A local copy of the compiled microfrontend is bundled in the vessel deployment package, preventing any dependence on external URLs.  
- **Development Stack**:  
  - Angular microfrontend (version 16) with Module Federation support.  
  - Nest.js (TypeScript) for back-end logic.  
  - MySQL 8.x or similar for both vessel and office databases.  
  - PM2 for process management, Nginx for reverse proxy.

### 6. Non-Standard Elements

- **Offline Operation**  
  Each vessel instance is fully autonomous while offline, storing data locally.

- **Last-Write-Wins Conflict Resolution**  
  While last-write-wins remains the default for non-critical fields, admin intervention is mandatory before overwriting any rest-hour compliance data. All replaced records are preserved in an overwrite log to maintain an audit trail. The overwrite log is stored in a protected, read-only format to fortify audit integrity. Authorized personnel can reconstruct overwritten records for compliance reviews if needed. For compliance-related data, the system automatically flags conflicts and requires the Admin to confirm which version to keep, eliminating silent overwrites.

- **Configurable Regulatory Rules**  
  MLC, STCW, and OPA regulatory logic is externalized (e.g., in versioned JSON or YAML) so that the vessel can still use a cached version in extended offline scenarios.

- **UTC Timestamps**  
  All entries are stored in UTC. The UI offsets these to local ship time to align with the vessel’s actual day boundaries. If needed for borderline day-bound compliance checks, the system can also capture a local-time component (or offset) alongside UTC to help reduce the risk of false violation triggers.

- **Encryption at Rest**  
  The solution can optionally employ MySQL’s native TDE or whole-disk encryption for additional security.  
  Given the presence of personally identifiable crew data, encryption at rest is strongly recommended to comply with applicable regulations (e.g., GDPR, flag-state requirements). In some jurisdictions, this may be mandatory. Ultimately, implementing at-rest encryption is subject to client-side policies and infrastructure.

- **Manual Deployments**  
  SafeLanes uses a script-based build/test/package approach for vessel and office installations, with no Docker usage.

- **Module Federation**  
  Leveraging Angular 16’s built-in support allows the rest-hour front-end to load dynamically in the existing Sail App at runtime.

### 7. High-Level Deployment & Integration Strategy
- **Deployment Environment**  
  - Office side: Single-cloud VM (16 GB RAM, 4 cores).  
  - Vessel side: Local server with 8 GB RAM, 2 cores. Deployed behind local Nginx, with PM2 controlling the Nest.js process.  
  - No container usage remains the standard.

- **Integration with Sail App**  
  - The microfrontend references shared libraries through Angular 16’s Module Federation support.  
  - The back-end microservice integrates with existing SafeLanes JWT auth, retrieving crew/role data from the Sail DB or identity service.

- **Data Sync & Collaboration**  
  Instead of a full-table diff for daily sync, the system aims to use updatedAt timestamps for more incremental synchronization, reducing bandwidth usage on limited vessel connections. The system uses updatedAt fields to determine the newest changes, logging overwritten data for post-merge review. All overwritten data is kept in an overwrite log, maintaining an audit trail and preserving previous states for compliance. For routine data, if conflict-flagging is not invoked, last-write-wins applies automatically. For any rest-hour compliance records, the system forces an admin review step, ensuring no silent overwrite occurs.

- **Backup & Recovery**  
  Daily snapshots of each vessel’s local DB and office DB are recommended. SafeLanes will store these backups offsite (where feasible) or on secondary media.

- **Future Growth**  
  Architecture is designed for moderate concurrency (<~100 concurrent office users, ~30 crew per vessel). Basic horizontal scaling can be introduced if usage grows, but the current design is considered sufficient for the stated scale.

- **Encrypted Communication**  
  All data transfers between vessel and office servers (including sync operations) must occur over a secure channel (TLS 1.2 or higher).