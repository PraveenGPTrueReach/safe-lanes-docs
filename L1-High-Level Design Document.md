## L1-HLD: High-Level Design (HLD) Document

This document provides a concise, high-level overview of the SafeLanes Rest Hours solution’s architecture. It describes the major system components, their interactions, data flows, technology choices, security strategies, and deployment considerations. Detailed low-level design specifications and granular implementation details are documented separately in L3-LLD (component-level) documents.

---

### 1\. Architecture Overview

This section summarizes the overall structure of the system, highlighting how vessel-based subsystems and the office environment achieve rest-hour logging, validation, and reporting. The solution operates offline on vessels,. 

**flowchart** LR  
   A\["Vessel Devices \- Crew/Admin UI"\] **\--** Local LAN **\--\>** B\["Vessel Server \- Nest.js \+ MySQL"\]  
   B **\--** Offline Fallback **\--\>** A

#### Key Highlights

- Vessel environment includes an on-prem server running the Nest.js backend and MySQL.  
- Office environment has a similar Nest.js instance and MySQL database, aggregating data across the fleet.  
- Angular microfrontends (vessel/office) connect to their local or remote Nest.js backends.  
- 

---

### 2\. System Components Descriptions

The solution comprises multiple logical layers and components, each addressing a distinct area of functionality. These components collectively support maritime rest-hour compliance and analytics.

flowchart TB

    subgraph Front-End

    A1\[Angular 16 Microfrontend\<br\>(Office)\] 

    A2\[Angular 16 Microfrontend\<br\>(Vessel \- Offline Fallback)\]

    end

    subgraph Back-End

    B1\[Nest.js \- Vessel\]

    B2\[Nest.js \- Office\]

    end

    subgraph Databases

    C1\[MySQL \- Vessel\]

    C2\[MySQL \- Office\]

    end

    A1 \-- REST/HTTPS \--\> B2

    A2 \-- REST/HTTPS \--\> B1

    B1 \--\> C1

    B2 \--\> C2

    B1 \-- Secure Sync \--\> B2

#### Front-End (Angular Microfrontend)

- Deployed as a Module-Federation-based microfrontend for the existing SafeLanes “Sail App.”  
- On vessels, an offline bundle is served locally.  
- Office-side users see comprehensive analytics for multiple vessels, while vessel-side front-end focuses on daily logs and departmental oversight.

#### Back-End (Nest.js)

- Two main deployments: one on each vessel, another at the office.  
- Provides REST APIs for data submission, validation, and retrieval.  
- Implements business logic for rule checks (MLC/STCW/OPA),.

#### Databases (MySQL)

- Vessel database stores offline data.  
-   
- 

---

### 3\. Inter-Component Communication

High-level API communication ensures consistent data flow between front-end UIs, back-end services, and databases. 

**sequenceDiagram**  
 participant FE as Angular Front\-End  
 participant VesselBE as Nest.js (Vessel)

 Note over FE**:** 1\) Crew enters rest hours  
 FE **\-\>\>** VesselBE**:** POST/PUT /resthours/daily  
 VesselBE **\-\>\>** VesselBE**:** Validate, store in MySQL  
 VesselBE **\--\>\>** FE**:** OK \+ Violation flags  
 Note over VesselBE**:** 2\) Sync triggered  
 VesselBE **\-\>\>** VesselBE**:** Merge updates locally

#### Communication Protocols & Data Formats

- REST/HTTP over TLS (HTTPS).  
- JSON payloads for rest-hour data and metadata.

---

### 4\. Data Architecture Details

Below is a simplified depiction of how data flows between sources (crew, office admins), processes (Nest.js logic), and storage (databases). Each vessel maintains offline autonomy, merging with the office DB.

**flowchart** LR  
**subgraph** Vessel\["Vessel"\]  
       B\["Daily Rest-Hour Records"\]  
       A\["Crew Inputs"\]  
       C{"Local DB MySQL"}  
 **end**  
   A **\--\>** B  
   B **\--\>** C  
   C **\--\>** B

#### Data Flow & Storage Strategy

- One record per crew member per day, subdivided into 48 half-hour blocks.  
- UpdatedAt timestamps support incremental sync.  
- Conflict resolution logs overwritten records in an immutable audit table.  
- Rule sets are versioned; forward-only checking applies to new entries.

---

### 5\. Integration Points

Integration points exist for authentication, external read-only auditor access, and the existing SafeLanes infrastructure:

1. SafeLanes Identity Service (JWT)  
   - Vessel back-end validates tokens.  
2. SAIL App Host (Angular Shell)  
   - Loads the rest-hour microfrontend modules, controlling role-based UI exposure.  
3. Auditor/External Access (Optional)  
   - View-only roles, restricted to read data for compliance reviews.

---

### 6\. Technology Choices

#### Front-End

- Angular 18.2.0with Module Federation for microfrontend integration.  
- 

#### Back-End

- Nest.js (TypeScript) for robust REST services, modular design, and straightforward rule-engine integration.

#### Database

- MySQL 8.x on both vessel and office sides, balancing reliability and ease of offline usage.  
- 

#### CI/CD & Deployment

- Script-based build, test, and package approach (no Docker).  
- PM2 process manager and Nginx reverse proxy on both vessel and office servers.

---

### 7\. Security Architecture Overview

Security focuses on role-based authentication, data encryption, and offline session handling..

**flowchart** LR  
   A\["User w/ JWT"\] **\--** Login Request **\--\>** B\["Vessel Nest.js"\]  
   B **\--** Check Token  **\--\>** C\["Local Auth Logic"\]  
   B **\--** Authorized Access **\--\>** D\["MySQL \+ FDE"\]

#### Authentication & Authorization

- JWT-based login.  
- Predefined roles limit access (Vessel User, Admin, Office User, etc.).  
- 

#### Compliance Considerations

- .  
- Overwritten data is preserved in an immutable audit log for potential external review.

---

### 8\. Deployment Architecture

The software is deployed on a dedicated server for each vessel (offline-ready) and a central office server. Daily snapshot backups are recommended in both environments. Vessel servers use PM2 \+ Nginx to host the Nest.js application and serve the Angular microfrontend fallback.

**flowchart** TB  
**subgraph** subGraph0\["Vessel Deployment"\]  
       V1\["Local Server\<br\>Linux/Windows"\]  
       V2\["NGINX \+ PM2"\]  
       V3\["Nest.js"\]  
       V4\["MySQL Encrypted Disk"\]  
 **end**  
   V2 **\--\>** V3  
   V3 **\--\>** V4  
   V2 **\--** Local Fallback **\--\>** V1

#### CI/CD Processes

- Source code is built, tested, and packaged into installers or deployment bundles.  
- Office environment updates can be applied more frequently.  
- Vessel updates are typically installed during scheduled port calls or stable connectivity periods, including the microfrontend bundle and DB schema scripts.

---

### 9\. Constraints and Assumptions

Finally, the overall solution operates under specific constraints and assumptions that guide its design:

1. **Forward-Only Rule Updates**: New rest-hour rules apply going forward rather than re-validating historical logs.  
2. **Moderate Concurrency**: Up to 30 crew per vessel, \~100 office concurrent users.  
3. **No Overengineering**: The architecture remains streamlined for the current project scale, with flexible but not overly complex features to support future expansion.

---

