## L3-TSD-BE: Technology Stack Documentation (TSD) for BE Document

### Introduction
This document details the technologies, frameworks, and tools used specifically in the Back-End (BE) component of the SafeLanes Rest Hours solution. It complements the system-wide technology choices from the L1-TSD but focuses exclusively on back-end needs and configurations for both vessel (offline-capable) and office environments. All decisions are aligned with the current project scale (up to ~30 vessel users and ~100 office users) and reflect client clarifications regarding database access, OS support, and process management.

---

### Programming Languages
1. **TypeScript 4.x or Higher**  
   • Used as the primary language for Nest.js application code.  
   • Improves maintainability through static typing and supports modern ES features.

2. **JavaScript (Node.js Runtime)**  
   • Node.js (version 16 or 18 LTS) powers the Nest.js back-end.  
   • Provides single-threaded, event-driven execution that is sufficient for the moderate concurrency levels (~30 users on vessels, ~100 in office).

---

### Frameworks and Libraries

#### Nest.js (v9+)
• Chosen for its modular structure, ease of maintenance, and TypeScript compatibility.  
• Facilitates REST API creation and integration with authentication via decorators and middleware.

#### TypeORM
• Adopted for database access (replacing any “or similar” placeholders from earlier discussions).  
• Simplifies schema definitions, relationships, and migrations for MySQL.  
We recommend TypeORM 0.3.x or the current LTS minor release to ensure stable compatibility with Nest.js v9+ and MySQL 8.x, maintaining a consistent environment across vessel and office deployments.

---

### Databases

#### MySQL 8.x
• Serves as the main relational database engine for both the vessel and office back-end.  
• Chosen for reliability, ACID transactions, and compatibility with offline operation.  
• Hosted locally on each vessel to allow fully autonomous operations when offline; an office instance aggregates fleet data.  

---

### Servers and Hosting

#### OS Support & Recommendations
• Officially supports both Linux and Windows to align with vessel constraints; Linux is recommended for new deployments due to simpler Node.js, PM2, and MySQL hosting.  
• Office server typically runs on a VM (4 CPU cores, 16 GB RAM); vessel server on smaller hardware (2 CPU cores, ~8 GB RAM).

#### Process Management
• PM2 (latest stable LTS release) in fork mode for simplicity (no clustering).  
• Manages start/stop, automatic restarts, and basic logging for the Nest.js process.  
• Nginx (latest stable LTS) used as a reverse proxy to handle SSL/TLS termination and route inbound traffic to the Nest.js application.

---

### DevOps Tools

#### Script-Based Build and Deployment
• No containerization (Docker) for this project.  
• Deployment relies on standard Node.js build scripts and manually copying artifacts to vessel/office servers.

#### Environment and Secret Management
• .env files with OS-level file restrictions, storing database credentials, JWT keys, and other sensitive variables.  
• On vessel servers, .env and other config files remain local. Administrators update these during port calls or via direct secure channels.

#### Logging and Monitoring
• Basic PM2 logs stored locally on each server (vessel and office).  
• No specialized log aggregators; administrators retrieve and review logs manually if needed.  
• A simple “/health” endpoint may be exposed for basic uptime checks.

#### Data Backup and Synchronization  
• We recommend performing regular MySQL database backups (e.g., daily or weekly) in both vessel and office environments using standard export tools (e.g., mysqldump).  
• Vessels should retain local backups to protect against data loss during extended offline periods.  
• When connectivity is available, vessel Nest.js instances can trigger a scripted synchronization of new or updated records to the office database, minimizing any risk of data inconsistency.  
• Office backups are retained separately to maintain a complete view of aggregated fleet data.

---

### Third-Party Services

#### Authentication: SafeLanes Identity Service
• The BE integrates with SafeLanes JWT-based authentication when online and relies on short-lived offline session keys when the vessel has no connectivity.  
• No additional third-party SaaS services are utilized for the core rest-hour functions.

#### Certificates (TLS)
• Clients typically use certificates from an external Certificate Authority (e.g., Let’s Encrypt) or self-signed certificates for vessel environments.  
• No other external API integrations or cloud-based dependencies are required for the back-end.

---

### Final Remarks
The Back-End component’s technology stack is intentionally kept simple yet robust for offline-friendly maritime operations. TypeScript + Nest.js under Node.js, MySQL 8.x, and TypeORM form the core. PM2, Nginx, and standard .env-based configurations fulfill the operational requirements for both office and vessel environments without overengineering. These choices meet the current scale and business needs while allowing future extension if usage grows.