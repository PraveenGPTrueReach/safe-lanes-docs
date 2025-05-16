## L3-TSD-WEBPRT: Technology Stack Documentation (TSD) for WEBPRT Document

### 1. Introduction
This document provides a focused overview of the specific technologies, frameworks, and tools used by the Solarium Green Energy © Web Portal (WEBPRT). The WEBPRT is designed to serve Admin and Key Account Manager (KAM) roles within a concurrency range of approximately 400–600 users across the entire system.

Operating as a React + TypeScript Single Page Application (SPA), WEBPRT interacts with the backend (BCKND) only through RESTful APIs. While high-level technology decisions are captured in the L1-TSD, this component-level (L3) TSD outlines the concrete versions, libraries, environments, and hosting specifics relevant to WEBPRT.

---

### 2. Programming Languages & Versions

#### 2.1 TypeScript (Frontend)
• Language Version: TypeScript 4.x (aligned with stable releases).  
• Usage Context: WEBPRT is implemented purely in TypeScript for type safety, consistent coding practices, and reduced runtime errors.

#### 2.2 Node.js (Build Environment)
• Node.js Version: 18.x LTS used on developer machines and in CI builds (matching the broader project’s LTS alignment).  
• Rationale: Ensures compatibility with modern tools and community support.

---

### 3. Frameworks & Libraries

#### 3.1 React & Core SPA Infrastructure
• React 18.x – Core library for building the SPA user interface.  
• React Router 6.8.x – Routing solution for single-page navigation .  
• Redux Toolkit 1.9.x – Primary state management for global data and remote fetching/caching logic.

#### 3.2 UI Component / Styling
• We have adopted Material UI 5.x  as our design and components framework to accelerate development and maintain a consistent UX.  
• Supplementary styling via CSS modules or styled-components to refine layout/branding where needed.

#### 3.3 Testing Frameworks & Strategy
• Jest 29.x + React Testing Library for unit and integration tests.  
• No end-to-end (E2E) automation is set up at present ; acceptance testing is performed manually for critical flows (login, leads, commissions, etc.).  
• We may consider introducing a lightweight E2E solution (e.g., Cypress or Playwright) in the future to reduce manual QA overhead.

#### 3.4 Data Fetch & Caching
• We do not introduce React Query or SWR, to avoid duplication and keep a single, centralized approach .  
• Redux Thunk or Redux Toolkit Query for hitting backend endpoints.  

#### 3.5 Other Supporting Libraries
• Axios or Fetch API for HTTP requests to BCKND.  
• Basic form validation with lightweight solutions (e.g., Yup) as needed—more complex validations remain on the backend.

---

### 4. Database
WEBPRT does not directly connect to a database. All persistence occurs via the backend (BCKND) and its Azure PostgreSQL instance. The portal issues REST calls for CRUD operations on leads, quotations, commissions, and user data stored in PostgreSQL.

---

### 5. Servers & Hosting

#### 5.1 Hosting Approach
• Static File Hosting on Azure Storage + CDN .  
• The compiled SPA (HTML, JavaScript, CSS) is served separately from the backend container, ensuring a clean separation of concerns.

#### 5.2 Environment Configuration
• Multiple build artifacts are produced: one for each environment (development, staging, production) using separate .env files .  
• Each environment build points to the appropriate backend endpoints (e.g., REACT_APP_API_BASE_URL).

#### 5.3 Concurrency Considerations
• The WEBPRT is expected to be used by Admin and KAM roles, contributing to the overall ~400–600 concurrent users across the system.  
• Basic optimizations (code-splitting, minimal re-renders, memo hooks) are employed but no complex real-time or offline features are introduced at this scale.

---

### 6. DevOps Tools

#### 6.1 Version Control & CI/CD
• Git-based workflows (Azure Repos or GitHub).  
• CI Pipeline (Azure DevOps or GitHub Actions) runs build and test steps upon commits.  
• Bundled artifacts are automatically uploaded to the staging environment for manual acceptance testing, then promoted to production if approved.

#### 6.2 Testing & Quality
• Linting (ESLint, Prettier) for consistent code styling in TypeScript.  
• Jest-based unit tests and React Testing Library for component validations.  
• Manual acceptance testing covering key user journeys: lead management, commission approvals, quotations.

---

### 7. Third-Party Services
WEBPRT does not directly integrate with external services (e.g., SMS, email). Such integrations reside in the backend (BCKND). The portal may request:  
• Short-lived SAS tokens from the backend to upload/download files in Azure Blob.  
• No direct usage of advanced telemetry; minimal console logs are used in the front-end. The backend relies on Azure Monitor/Application Insights.

---


### 8. Authentication, Security, and Licensing

#### 8.1 JWT Storage and Session Handling
• We store short-lived JWT tokens in localStorage. This simplifies client-side retrieval but can pose risks if malicious scripts gain access to the browser environment.  
• We do not implement silent token refresh. Upon token expiry or invalidation, the user must re-authenticate.  
• While localStorage is pragmatic for this solution scale (~400–600 concurrent users), we recommend employing secure Content Security Policies (CSP) and strict input sanitization to mitigate potential XSS vulnerabilities.

#### 8.2 Future Enhancements for Security
• If business or compliance needs grow, we may transition to HttpOnly cookies with CSRF protections.  
• For now, repeated manual re-login on session expiration aligns with the solution’s simplicity and concurrency scale.

#### 8.3 Licensing & Compliance
• The primary libraries (React, Material UI, Redux Toolkit) are under permissive open-source licenses (MIT), compatible with most enterprise usage policies.  
• A basic dependency audit should be maintained to track any new libraries or license changes.  
• No specialized compliance frameworks (e.g., GDPR) are mandated beyond the standard 7-year retention policy at the backend level.



### Conclusion
The chosen technology stack for the WEBPRT balances reliability, simplicity, and maintainability:
• React 18 + TypeScript for a robust and highly maintainable front-end.  
• Material UI 5.x for a standardized UI/UX.  
• Redux Toolkit for predictable state management and data fetching.  
• Hosted as static files in Azure Storage + CDN, fully separate from the backend container.  
• Automated unit-testing via Jest and manual acceptance testing for critical flows.

This approach fully satisfies the current scale (400–600 concurrent users at the system level) without overengineering. Future enhancements, such as E2E testing or advanced security measures, can be introduced if usage demands grow or requirements evolve.