## L3-TSD-FE: Technology Stack Documentation (TSD) for FE Document

This document captures the specific technologies, frameworks, and tools chosen to implement the SafeLanes Rest Hours Front-End (FE). While many system-wide decisions appear in the L1-TSD, this component-level TSD focuses on the front-end microfrontend requirements and constraints, ensuring clarity for developers maintaining and extending the FE.

---

### 1. Introduction

The SafeLanes Rest Hours FE is an Angular-based microfrontend that operates both in office and vessel environments. It integrates seamlessly with the existing Sail App (or temporarily via a fallback wrapper if the host is on older Angular). It interfaces with the Nest.js back-end services for data storage, conflict resolution, and user authentication. This TSD document outlines the key technologies used to build, test, and deploy the FE in a manner aligned with the overall system scale and offline operation requirements.

**Note on PlanningModule:** While not explicitly mentioned in the L1-HLD, this FE can also include an optional “PlanningModule” for scheduling tasks. We strongly recommend updating the L1-HLD documentation to reflect this feature so that the scope is consistent across all levels of project documentation.

---

### 2. Programming Languages

#### TypeScript
- Version: 4.x or higher.  
- Purpose: Main development language for all Angular code, providing static typing and improved maintainability.  
- Notes: A single build artifact is produced for both office and vessel deployments, with minimal environment-based runtime checks.

#### JavaScript (Runtime)
- The front-end ultimately transpiles to JavaScript, which runs in standard web browsers.  
- Node.js (v16 or v18 LTS) is used during build time (Angular CLI, package management).

---

### 3. Frameworks and Libraries

#### Angular 16
- Core front-end framework for building the Rest Hours microfrontend.  
- Uses Module Federation to integrate with the Sail App shell (once that shell is also on Angular 16 or later).  
- Provides powerful built-in features (routing, forms, HTTP, etc.) suitable for moderate-scale maritime solutions.

#### Angular CLI & Tooling
- The Angular CLI (16.x) manages builds, test execution, and local development (`ng serve`).  
- Leverages established best practices (lazy loading, environment-specific configurations).

#### Minimal RxJS State Management
- Uses lightweight “service + BehaviorSubject” patterns.  
- Chosen to avoid overengineering at the project’s current size.  
- Leaves open the possibility of adopting more advanced state libraries (e.g., NgRx) if complexity grows.

#### Supporting Libraries
- Simple UI components (e.g., Angular Material or custom components) may be included, but the project deliberately avoids heavy UI frameworks or large third-party libraries.  
- Basic date/time utilities (e.g., date-fns) may be used to format or parse daily logs.

---

### 4. Databases

- No direct database access occurs in the FE.  
- All offline data storage and synchronization tasks reside in the vessel Nest.js + MySQL environment.  
- The FE submits/receives data exclusively via HTTP REST calls to the relevant Nest.js backend (vessel or office).

---

### 5. Servers and Hosting (Front-End Perspective)

- **In Office Mode:**  
  - The FE is served through the Sail App shell (via Module Federation) or an interim fallback wrapper, typically hosted behind Nginx.  
  - End users access it via standard web browsers in an office network environment.

- **In Vessel Mode (Offline Capable):**  
  - The FE bundle is deployed onto the local Nest.js server.  
  - When the vessel is offline, a local fallback copy is served so crew can continue to log hours and check compliance.  
  - Nginx + PM2 manage the Node.js processes serving the Angular bundle, but these are part of the vessel’s server configuration (see L3-TSD-Backend or L3-DG documents for details).

---

### 6. DevOps Tools and Process

#### Script-Based Build & Deployment
- Angular CLI (16.x) is used for building the production artifacts.  
- No containerization (e.g., Docker) is used for front-end deployment; standard Node-based or system scripts handle packaging into versioned artifacts.

#### Manual Release Cycles
- Developers merge changes into a Git-based repository; the pipeline produces an Angular build artifact (dist folder).  
- Office deployments pull the updated artifacts and place them in the Sail App environment.  
- Vessel deployments update the local fallback bundle manually or via scheduled scripts during stable connectivity windows (port calls, etc.).

#### Testing Tools
- **Unit Testing:** Karma + Jasmine.  
- **End-to-End Testing:** We use Karma + Jasmine for E2E in alignment with the default Angular CLI approach and L1 documentation, to ensure a consistent testing framework across all project levels.  
- Focus is on verifying core services, components, and domain logic without overextending the test scope.  
- Offline or vessel scenarios are validated with carefully designed test scenarios, but remain relatively simple due to environment constraints.

### 6.1 Security Scanning and Technology Lifecycle

- **Dependency Vulnerability Checks:** We use npm audit (or equivalent) to detect and address known security issues in front-end dependencies. Teams are encouraged to run these checks regularly (e.g., weekly) and update packages promptly if high-severity vulnerabilities are found.  
- **Version Updates:** Node.js LTS (16 or 18) and Angular 16.x are monitored for their respective end-of-life dates. An internal schedule or policy ensures we review and upgrade these dependencies at least once per year (or as critical patches arise) to stay on supported versions and minimize exposure to unpatched security flaws.

---

### 7. Third-Party Services

- No direct external SaaS integrations for the front-end.  
- Authentication relies on the SafeLanes Identity Service, but all token handling and validation occur via the Nest.js back-end or local short-lived session keys.  
- The front-end itself does not call out to third-party APIs for core functionality.

---

### 8. Conclusion

The Rest Hours FE leverages Angular 16, TypeScript, and a select set of lightweight libraries to meet moderate maritime needs without adding unnecessary complexity. By focusing on a single artifact build, minimal offline logging, and standard Angular toolchains (Karma + Jasmine), this stack provides an efficient, maintainable path forward for both vessel and office deployments. Regular security scanning and version lifecycle reviews are incorporated to address evolving threats and keep dependencies updated.

Future expansions can build on these choices if broader requirements or higher concurrency emerges, but the current technology selection aligns well with the scale and offline requisites described in higher-level documents. Where new functional modules like “PlanningModule” are introduced at L3 level, corresponding L1 documentation updates are recommended to avoid scope discrepancies.
