```
## L3-TSD-CPAPP: Technology Stack Documentation (TSD) for CPAPP Document

### 1. Introduction
This document details the technologies, frameworks, and tools selected for the Channel Partner App (CPAPP), ensuring compatibility with the overall Solarium Green Energy solution while meeting the scale of approximately 400–600 concurrent users. It incorporates final client clarifications on local database libraries, state management, React Native versions, and minimum OS requirements.

---

### 2. Programming Languages & Platform

#### 2.1 React Native (Mobile)
• Language: TypeScript (with some JavaScript usage as needed).  
• React Native Version: Targeting the newest stable 0.71.x or 0.72.x, depending on verified library compatibility.  
• Minimum OS Versions: iOS 13+ and Android 8+.  
• Rationale: Provides cross-platform development efficiency and established community support, well-suited to the project’s scale of ~400–600 concurrent users.

---

### 3. Frameworks & Libraries

#### 3.1 Core Framework: React Native
• Provides the fundamental UI building blocks for iOS/Android.  
• Selected for cross-platform alignment with other solution components (e.g., the Customer App).

#### 3.2 State Management: Redux Toolkit + RTK Query
• Manages shared in-app state (leads, quotations, commissions) and handles API caching.  
• Reduces development overhead compared to custom Redux thunks.  
• Ensures a structured approach for data fetching, error handling, and offline caching integration.

#### 3.3 React Navigation (v6+)
• Offers a routing solution for stack and tab navigations.  
• Chosen for a robust navigation structure, widely adopted within the React Native ecosystem.

#### 3.4 HTTP/Networking: Built-in Fetch or Axios
• In practice, RTK Query often wraps Fetch or Axios for network calls.  
• Ensures consistent JSON-based communication with the backend’s REST endpoints.

#### 3.5 Local Database: react-native-sqlite-storage
• Provides an SQL-based approach to store offline data (read-only caching of leads and quotations).  
• Lightweight yet reliable for moderate amounts of data encountered at scale (~400–600 concurrent users).  
• Rationale: Familiar and widely supported plugin for React Native, enabling straightforward queries and indexing.

#### 3.6 Additional Utilities
• ESLint + Prettier: Enforces code style and formatting.  
• Jest + React Native Testing Library: Unit test and component test framework.

---

### 4. Database (Local Storage)

#### 4.1 Purpose
• The CPAPP caches essential lead and quotation data for quick offline reads.  
• No offline writes are allowed, aligning with the system’s constraints to maintain data integrity.

#### 4.2 Technology: SQLite via react-native-sqlite-storage
• Caches leads, quotations, and minimal user info in a structured manner.  
• Supports indexing for faster retrieval.  
• Selected due to simpler integration, broad community support, and minimal overhead for read-only caching.

---

### 5. Hosting & Distribution

#### 5.1 Mobile Stores
• CPAPP is built and released as a standalone React Native app for iOS (App Store) and Android (Google Play).  
• No separate backend hosting is required; the app connects to the existing backend hosted on Azure Web App.

#### 5.2 Versioning & Publishing
• Adopts semantic versioning (major.minor.patch) for tracking releases.  
• Distributed privately to authorized Channel Partners or publicly (depending on business decisions).

---

### 6. DevOps Tools & Process

#### 6.1 Version Control
• Git (Azure Repos or GitHub) for storing CPAPP source code.  
• Feature-branch workflow with pull requests and code reviews to maintain quality.

#### 6.2 CI/CD Pipelines
• Automated builds and tests triggered on code pushes (via Azure DevOps or GitHub Actions).  
• Generates app binaries (iOS .ipa, Android .apk/.aab) for staging.  
• Approved builds are promoted to production release channels in the App Store and Play Store.

#### 6.3 Testing & Quality Checks
• Unit Tests (Jest): Covers services (e.g., AuthService, LeadService) and Redux logic.  
• UI Tests (React Native Testing Library): Validates component rendering and user flows.  
• Linting & Formatting (ESLint, Prettier): Ensures consistent coding style.

---

### 7. Third-Party Services

#### 7.1 SMS OTP: MSG91
• CPAPP initiates phone-based authentication requests.  
• The backend relays OTP via MSG91, and CPAPP verifies it through the backend’s REST API.

#### 7.2 No Direct Push Notifications
• The app uses a poll or manual refresh mechanism to detect new updates, supplemented by SMS for critical alerts.

#### 7.3 Cloud Storage & File Upload
• Uploads performed via signed URLs from the backend, which leverages Azure Blob Storage.  
• CPAPP interacts only with the backend’s endpoints (no direct Azure SDK usage in the app for file management).

---

### 8. Additional Observations

#### 8.1 Scalability
• The chosen mobile technology stack (React Native + Redux Toolkit + SQLite) comfortably supports the current load of 400–600 concurrent users.  
• Potential expansions (e.g., additional offline features or more advanced push notifications) can be layered on if user growth or usage patterns require it.

#### 8.2 Security & OS Compatibility
• Local caching is limited to essential data to mitigate security risks if a device is lost or stolen.  
• By targeting iOS 13+ and Android 8+, the app remains compatible with most Channel Partner devices while leveraging stable React Native features.

---

### Conclusion
The above technology choices ensure CPAPP remains performant, maintainable, and aligned with the larger Solarium Green Energy ecosystem. By relying on a modern React Native stack, Redux Toolkit for state management, SQLite for lightweight offline caching, and standard CI/CD practices, the CPAPP is well-positioned to serve ~400–600 concurrent users effectively and scale modestly as future needs evolve.
```
