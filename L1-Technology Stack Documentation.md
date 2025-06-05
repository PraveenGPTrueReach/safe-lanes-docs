## L1-TSD: Technology Stack Documentation (TSD) (System-Wide) Document

### Introduction

This document provides a system-wide overview of the primary technologies, frameworks, and tooling that comprise the SafeLanes Rest Hours solution. It highlights the selected programming languages, libraries, servers, databases, and DevOps processes used to meet current operational and compliance requirements while accommodating vessel-offline scenarios. Component-specific details appear in L3-TSD documents.

---

## Programming Languages

### TypeScript

- Used for both front-end (Angular) and back-end (Nest.js) codebases.  
- Standardizes development with static typing, improving reliability and maintainability.  
- Typical version: TypeScript 4.x or higher.

### JavaScript (Runtime)

- Node.js (v16 or v18 LTS) underpins the Nest.js back-end.  
- Enables server-side scripting and a unified codebase across front-end/back-end.

---

## Frameworks and Libraries

### Angular 18.2.0

- Basis for the microfrontend, leveraging Module Federation for integration with the existing SafeLanes “Sail App.”  
- Provides a robust, scalable UI and offline bundle capabilities for vessel deployments.

### Nest.js (v9+)

- Back-end framework for REST APIs, rule checks.  
- Facilitates modular design with decorators, middleware, and easy integration of OAuth/JWT-based security.

### PM2

- Process manager for Node.js on both vessel and office servers.  
- Provides clustering, log management, and graceful start/stop of the Nest.js application.

### Nginx

- Acts as a reverse proxy for HTTP(S) requests, improving performance and security.  
- Handles SSL/TLS offloading and can serve static files (e.g., the offline web bundle on vessels).

---

## Databases

### MySQL 8.x

- Primary relational database for both vessel and office environments.  
- Chosen for its familiarity, offline-readiness, and support for reliable ACID transactions.  
- Stores rest-hour logs, user profiles, conflict-resolution data,.

---

## Servers and Hosting

### Vessel Server

- Typically a small on-premises machine (2 CPU cores, \~8 GB RAM) running Linux or Windows.  
- Hosts Nest.js and MySQL locally for offline operation.  
- Bundles the Angular front-end to remain accessible without internet connectivity.

### Office Server / VM

- Minimum 4 CPU cores, \~16 GB RAM running Linux or Windows.  
- Operates the Nest.js, MySQL, and Nginx stack with sufficient capacity for multi-vessel data aggregation and office dashboards.  
- Manually scaled if concurrency grows beyond \~100 users.

---

## DevOps Tools and Process

### Script-Based Build & Deployment

- No containerization (Docker) used; deployments rely on standard Node.js build and packaging scripts.  
- Angular and Nest.js compilation output is packaged into versioned artifacts.  
- 

### Manual Release Cycles

- Code merges and packaging occur in Git-based repositories (e.g., GitLab).  
- Office deployments involve pulling the latest production branch and updating the local environment via scripts.  
- Vessels periodically receive new builds (including the UI bundle) during port calls or stable connections.

### Offline Handling

- The vessel server automatically serves the local microfrontend bundle   
- Ensures full functional continuity despite limited or intermittent network access.

---

## Third-Party Services

- The solution is predominantly self-contained, leveraging internal SafeLanes identity services for JWT-based authentication.  
- No direct reliance on external APIs or cloud-based SaaS for core rest-hour functions.  
- TLS certificates may be acquired from external Certificate Authorities (e.g., LetsEncrypt), but issuance and rotation remain the client’s choice.

## 7\. Testing and QA

- The project will use standard testing frameworks to maintain code quality and reliability without overengineering for the current scale.  
- For the back-end (Nest.js) code, Jest is the primary framework for unit and integration tests. Critical modules such as rest-hour validation logic receive priority coverage.  
- For the Angular 18.2.0front-end, the default Angular CLI testing setup (Karma \+ Jasmine) is employed for unit tests, verifying components and services.  
- Integration or end-to-end tests focus on key workflows (e.g., offline/online transitions, rest-hour submission, and conflict resolution) to ensure essential functionality remains stable.  
- This balanced testing approach provides moderate coverage (\~70–80% for major modules) while controlling complexity, aligning with practical maritime operational needs.

