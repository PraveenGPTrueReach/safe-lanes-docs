## SafeLanes Rest Hours Submodule \-- Business Requirements & Technical SOW

Below is a **Business-Oriented SOW** that blends **functional requirements** (i.e., what the submodule needs to do from a business perspective) with **critical technical details** (i.e., how it will be designed and integrated). This document can serve as a **business requirements** reference for stakeholders, while still containing enough detail for the development team to proceed confidently.

### 1\. Introduction

SafeLanes aims to enhance its existing maritime ERP ("Sail App") with a dedicated **Rest Hours Submodule**. The goal is to **ensure regulatory compliance** (e.g., MLC, STCW, OPA) by accurately tracking crew work/rest hours, identifying non-conformities (NCs), and providing analytics.

**Key Business Drivers:**

- Consolidate rest-hour data in a respective single system for **both vessel and office** users.  
- Automate detection of **violations** to reduce compliance risk.  
- Facilitate **planning** (fixed/variable tasks) and post-hoc analysis via **dashboards**.

To achieve these objectives, the submodule will be delivered as a **standalone microservice** that seamlessly integrates with the existing Sail App via **Module Federation** (frontend) and **PM2 \+ Nginx** (backend).

### 2\. Scope & Functional Requirements

1. **Core Recording**  
   - **Daily Work/Rest Hours**: Each crew member inputs hour-by-hour rest/work status (including partial hours if needed).  
   - **Validation**: Automated check for missing or overlapping entries.  
   - **Overriding Planned Hours**: If actual conditions differ from planned schedules, updates must be allowed.  
2. **Planning & Scheduling**  
   - **Fixed Tasks**: Recurrent tasks or watch schedules (e.g., 4-on/8-off patterns).  
   - **Variable Tasks**: Ad hoc tasks (port calls, drills) with start/finish times.  
3. **Compliance & Violations**  
   - **Immediate Violation Detection**: Trigger industry-standard codes (e.g., "MLC violation code 1--8").  
   - **NC Logging**: If repeated or serious violations occur, mark them as non-conformities.  
4. **Dashboards & Analytics**  
   - **Vessel-Level**: Summaries of incomplete data, daily or weekly violation rates.  
   - **Office-Level**: Fleet-wide compliance metrics, trending analyses, predicted future violations, rank/department breakdowns.  
   - **Historical Reporting**: Yearly, quarterly, monthly violation patterns, plus drill-down to specific vessels.  
5. **User Roles & Permissions**  
   - **Vessel**: Vessel User (crew), Vessel Admin (senior officer), Vessel Super Admin (Master).  
   - **Office**: Office User (superintendents), Office Admin (managers), Office Super Admin (top-level).  
   - **External**: Possibly read-only for auditors.  
   - Access is enforced via **JWT** tokens and existing SafeLanes RBAC.

### 3\. Business Needs & Outcomes

1. **Regulatory Compliance**: Reduces risk of fines or detentions by automatically detecting rest-hour violations.  
2. **Operational Efficiency**: Streamlined daily data entry and no manual checks on vessel/office side.  
3. **Actionable Insights**: Dashboards help management pinpoint frequent NCs or departments prone to overwork.  
4. **Better Planning**: Align actual recorded hours with watch schedules or unique tasks.

### 4\. High-Level Technical Architecture

#### 4.1 Microservice Delivery

- **Frontend**:  
  - **Angular** microfrontend (remote) with **Module Federation**.  
  - SafeLanes (the "host" application) references the rest-hour remote in webpack.config.js (ModuleFederationPlugin).  
  - The microfrontend reuses shared dependencies (e.g., AuthService, UI libraries) from the host.  
- **Backend**:  
  - **Nest.js** microservice (resthour\_service.js) managing REST endpoints for recording, planning, and compliance checks.  
  - **PM2** used to run the process (e.g., pm2 start resthour\_service.js \--name resthour\_service), with **Nginx** reverse proxy (location /resthour/ { ... }).  
- **Database**:  
  - **MySQL** schema, with new tables for rest-hour logs, violations, tasks, etc.  
  - **Migration Scripts** (SQL or Knex) provided by the vendor, deployed by SafeLanes.  
  - **Soft Delete & "updatedAt"** fields to handle concurrency with  "last-write-wins" strategy .

#### 4.2 Deployment & Environment

- **Online Servers**: 16GB RAM, 4 cores.  
- **Offline Ship Environments**: 8GB RAM, 2+ cores.

**4.3 Connectivity** 

- Offline access on vessel side with local DB.


**flowchart** LR  
**subgraph** subGraph0\["Vessel LAN Network"\]  
       phone1(\["Phone/ Computer"\])  
       localServer1(\["Local Server"\])  
       localDB1(\["Local DB"\])  
 **end**  
**subgraph** subGraph1\["Vessel 1"\]  
       subGraph0  
 **end**  
**subgraph** subGraph2\["Vessel LAN Network"\]  
       phone2(\["Phone / Computer"\])  
       localServer2(\["Local Server"\])  
       localDB2(\["Local DB"\])  
 **end**  
**subgraph** subGraph3\["Vessel 2"\]  
       subGraph2  
 **end**  
**subgraph** subGraph4\["Vessel LAN Network"\]  
       phone3(\["Phone / Computer"\])  
       localServer3(\["Local Server"\])  
       localDB3(\["Local DB"\])  
 **end**  
**subgraph** subGraph5\["Vessel 3"\]  
       subGraph4  
 **end**  
   phone1 **\--\>** localServer1  
   localServer1 **\--\>** localDB1  
   phone2 **\--\>** localServer2  
   localServer2 **\--\>** localDB2  
   phone3 **\--\>** localServer3  
   localServer3 **\--\>** localDB3

 

### 5\. Detailed Business Workflows

1. **Vessel User**  
   - Logs daily rest/work hours (Screen 2c).  
   - Views personal schedule.  
   - Overwrites planned tasks if reality differs.  
2. **Vessel Admin**  
   - Creates/edit watch schedules (3a) or variable tasks (3b).  
   - Oversees subordinate crew's rest-hour entries, acknowledges violations.  
3. **Vessel Super Admin** (Master)  
   - Full control over planning & recording.  
   - Can lock monthly records or sign off.  
   - Escalates repeated NCs to office if needed.  
4. **Office Admin**  
   - Monitors compliance across a group of vessels (Screens O2.0a, O2.0b, O2.0c).  
   - Investigates trends or repeated NCs.  
   - May correct errors in the vessel data (per client policy).  
5. **Office Super Admin**  
   - Fleet-wide view of all vessels.  
   - High-level analytics, overall system config (e.g., user roles, advanced reports).

### 6\. Deliverables & Responsibilities

#### 6.1 Core deliverables

1. **Frontend Microfrontend (Angular)**  
   - Complete codebase for rest-hour remote module.  
   - Module Federation config details: exposed components, routing structure.  
2. **Backend Service** (resthour\_service.js)  
   - Nest.js code with required REST endpoints.  
   - Integration with existing JWT auth.  
   - Clear documentation on environment variables/config.  
3. **Database Migration Scripts**  
   - Schema creation or updates for rest-hour tables, including indexes, relationships, and "soft-delete" fields.  
   - Provided in a format suitable for SafeLanes to apply to each client DB.  
4. **Integration & Menu Guidance**  
   - Guidance on adding new routes, nav items, or menu links to the monolithic Sail App.  
   - Collaboration on final Module Federation plugin config.  
5. **Documentation & Basic Support**  
   - Brief deployment instructions for testing (dev/staging) and production.  
   - Quick reference for SafeLanes admins on how to run or stop resthour\_service.js with PM2.

#### 6.2 The base app already has:

1. **Host Application**  
   - webpack.config.js for module federation.  
2. **Server Setup & Deployment**  
   - Configured **Nginx** location blocks for /resthour/.  
   - Deployed and started the resthour\_service.js using **PM2** across online/offline environments.  
   - Provide all environment variables, e.g., DB credentials, JWT secrets.  
3. **Database Environment**  
   - Supplied base dump (e.g., \<clientId\>\_resthour\_db) to build upon.  
   - Ran the vendor-supplied migration scripts for each environment (production, staging, offline ships, etc.).

### 7\. Key Assumptions & Constraints

1. **JWT Auth**: The existing SafeLanes authentication scheme remains consistent---no rework needed.  
2. **Offline Environments**: The submodule must function on limited connectivity ships, with local DB usage and a scheduled or on-demand sync approach.  
3. **Performance**: The hardware specs (8GB/2+ cores offline, 16GB/4 cores online) are deemed sufficient for the additional Rest Hours microservice.  
4. **No Docker**: Deployment uses PM2 (and not containerization), given the offline environment's limitations.

