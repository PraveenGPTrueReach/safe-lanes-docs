## L1-OVERVIEW: Solution Overview (High-Level)

## 1. Business Overview

SafeLanes' Rest Hours Submodule addresses maritime rest-hour compliance (MLC, STCW, OPA) by capturing crew work/rest logs, detecting violations, logging non-conformities (NCs), and providing multi-level analytics (vessel and office). The solution runs partially offline on vessels and synchronizes with an office server when connectivity is available. It extends the existing SafeLanes "Sail App" with modular front-end integration (Module Federation using Angular 18.2.0 for more robust support) and a standalone Nest.js back-end microservice.

Additionally, certain external or auditor-type users may be granted read-only access for compliance reviews, if configured in the SafeLanes RBAC settings. For clarity, references throughout other documents to "SAIL login" and "SafeLanes login" are synonymous, referring to the same credential and authentication system.

**Note:** If the existing Sail App is not yet on Angular 18.2.0 or higher, the SafeLanes team must upgrade it to ensure module federation compatibility. The Rest Hours microfrontend will remain on Angular 18.2.0.

## 2. Key Entities

- **Vessel**: Hosts local server + local DB (MySQL). Runs the Rest Hours service offline.  
- **Crew**: Have role-based permissions (User, Admin, Super Admin) to enter/approve hours.  
- **Office**: Primary location with aggregated data usage, advanced analytics, and managerial oversight. Roles include Office User, Office Admin, and Office Super Admin.  
- **Non-Conformities & Violations**: System-detected indicators of rest-hour breaches. Handled both at vessel and office levels.  
- **External**: Read-only (auditor) role that can view rest-hour data for oversight, without editing permissions.

## 3. Data Flow

1. **Onboard Data Entry**: Crew record daily rest/work blocks on the vessel (local DB).  
     
2. **Validation & Violation Checks**: The system applies rule-based checks (hard-coded thresholds and codes).  
     
3. **Offline Storage**: Entries remain in the local DB; updates are timestamped (UTC).

4. **Office Aggregation**: Office dashboards aggregate vessel data for fleet-level compliance tracking.  
     
5. **Predicted Violation Calculations**: A minimal rule-based forecast runs within the Nest.js microservice.  
     
6. **Data-in-Transit Encryption**: All data synchronization and REST calls are protected with TLS 1.2 or higher to safeguard sensitive crew information.

There is no direct financial flow in this module. The system primarily handles operational and compliance data.

## 4. Key Components

- **Microfrontend (Angular)**  
  In order to streamline Module Federation, the team will utilize Angular 18.2.0, which offers better integrated support for microfrontends and reduces configuration complexities. On vessels operating offline, the microfrontend can be loaded from a local server

    
- **Nest.js Back-End Service**  
    
  - Provides REST endpoints for recording hours, retrieving schedule/plan data, detecting violations, and performing basic rule-based predictions.  
  - Handles authentication locally with cached JWT tokens for offline scenarios,   
  - Deployed with PM2, reverse-proxied by Nginx.


- **MySQL Databases**  
    
  - Main server DB for consolidated records (office).  
  - Local DB on each vessel for offline operation.


- **Authentication & RBAC**  
  Utilizes SafeLanes JWT-based auth. Existing roles are extended (or new claims created) for Vessel/Office user distinctions and optional External read-only access. 

## 5. High-Level Architecture

- **On-Prem Vessel Servers**: Each vessel runs the Nest.js service on a local server (Linux or Windows) with MySQL. Nginx and PM2 handle application processes.  
- **Office Server**: Another Nest.js instance (the same codebase) and MySQL run on on-prem or hosted hardware (16 GB RAM, 4 cores).  
- **Offline Microfrontend Loading**: During isolated operation, each vessel serves the Rest Hours microfrontend from its local server, ensuring the UI remains accessible. A local copy of the compiled microfrontend is bundled in the vessel deployment package, preventing any dependence on external URLs.  
- **Development Stack**:  
  - Angular microfrontend (version 18.2.0) with Module Federation support.  
  - Nest.js (TypeScript) for back-end logic.  
  - MySQL 8.x or similar for both vessel and office databases.  
  - PM2 for process management, Nginx for reverse proxy.

## 6. Non-Standard Elements

- **Offline Operation**  
  Each vessel instance is fully autonomous while offline, storing data locally.  
    
- **Last-Write-Wins Conflict Resolution**  
  Last-write-wins remains the default for all fields. All replaced records are preserved in an overwrite log to maintain an audit trail. The overwrite log is stored in a protected, read-only format to fortify audit integrity. Authorized personnel can reconstruct overwritten records for compliance reviews if needed.   
    
- **Configurable Regulatory Rules**  
  MLC, STCW, and OPA regulatory logic is externalized (e.g., in versioned JSON or YAML) so that the vessel can still use a cached version in extended offline scenarios.  
    
- **UTC Timestamps**  
  All entries are stored in UTC. The UI offsets these to local ship time to align with the vessel's actual day boundaries. If needed for borderline day-bound compliance checks, the system can also capture a local-time component (or offset) alongside UTC to help reduce the risk of false violation triggers.

- **Manual Deployments**  
  SafeLanes uses a script-based build/test/package approach for vessel and office installations, with no Docker usage.  
    
- **Module Federation**  
  Leveraging Angular 18.2.0's built-in support allows the rest-hour front-end to load dynamically in the existing Sail App at runtime.

## 7. High-Level Deployment & Integration Strategy

- **Deployment Environment**  
    
  - Office side: Single-cloud VM (16 GB RAM, 4 cores).  
  - Vessel side: Local server with 8 GB RAM, 2 cores. Deployed behind local Nginx, with PM2 controlling the Nest.js process.  
  - No container usage remains the standard.


- **Integration with Sail App**  
    
  - The microfrontend references shared libraries through Angular 18.2.0's Module Federation support.  
  - The back-end microservice integrates with existing SafeLanes JWT auth, retrieving crew/role data from the Sail DB or identity service.


- **Future Growth**  
  Architecture is designed for moderate concurrency (~100 concurrent office users, ~30 crew per vessel). Basic horizontal scaling can be introduced if usage grows, but the current design is considered sufficient for the stated scale.