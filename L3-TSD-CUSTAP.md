## L3-TSD-CUSTAP: Technology Stack Documentation (TSD) for CUSTAP Document

### 1. Introduction
This document describes the selected technologies, frameworks, and tools used within the Customer Mobile App (CUSTAP) component of the Solarium Green Energy solution. It references the high-level decisions in L1-TSD (Technology Stack Documentation, System-Wide) as well as relevant details in L1-OVERVIEW and L1-HLD to ensure consistency across the overall solution. The objective is to ensure a secure, consistent, and maintainable implementation at the current scale of approximately 400–600 concurrent users.

---

### 2. Programming Languages & Platforms

#### 2.1 Language: JavaScript/TypeScript
• CUSTAP is developed primarily in TypeScript.  
• We standardize on TypeScript 4.5+ to ensure alignment with modern language features while maintaining backward compatibility and stability.  
• Rationale: Strong typing improves code reliability, reduces runtime errors, and aligns with the broader use of TypeScript in the solution.

#### 2.2 React Native Platform  
• Version: React Native 0.70+ (as decided at the L1 level).  
• We have tested and officially support iOS 12+ and Android 8+ to balance device coverage with performance and security considerations.  
• Rationale: Enables cross-platform development with a single codebase, providing a strong ecosystem of third-party libraries and community support suitable for ~400–600 user concurrency.

---

### 3. Frameworks & Libraries

#### 3.1 React Native Core
• Comprises React 18+ under the hood with native bindings.  
• Offers ready-made UI components, device APIs, and an active ecosystem.

#### 3.2 Navigation & UI
• Typical usage of React Navigation (v6+) for screen-to-screen transitions.  
 • We standardize on React Native Paper for a consistent theme and look across the application, reducing style fragmentation.

#### 3.3 AsyncStorage (Encrypted)
• Used to store JWT tokens and minimal cached data (e.g., leads, quotations). An encryption layer is applied on top of AsyncStorage to guard sensitive information.  
• Encryption Method: We rely on a well-established library (e.g., react-native-encrypted-storage) that uses AES-256.  
• Key Management: The encryption key is stored in each platform’s secure Keychain/Keystore, reducing exposure in scenarios of device compromise. We maintain a simple rotation procedure to replace or invalidate the key if a security incident arises (e.g., distributing a new app version that re-encrypts existing data with the updated key).

#### 3.4 HTTP/Networking
• Libraries such as axios (version ^0.27) or the fetch API for REST calls to the backend.  
• JWT tokens included in Authorization headers for all protected routes.

#### 3.5 Testing
• Jest (version ^28) + React Native Testing Library (version ^11) for unit and component tests.  
• Rationale: This ensures consistent version usage and avoids unexpected compatibility pitfalls.  
• Per clarifications, we defer comprehensive E2E frameworks (e.g., Detox/Appium) until usage growth or complexity justifies it.


#### 3.6 Monitoring & Crash Reporting
• We recommend integrating a crash reporting/analytics solution (e.g., Firebase Crashlytics or Sentry) to capture runtime errors and performance metrics in production.  
• This approach helps diagnose unexpected issues at scale (400–600 concurrent users) without overengineering.


---

### 4. Data & Storage

#### 4.1 Local Data
• Minimal offline read-only caching in AsyncStorage (encrypted).  
• No complex local RDMS or document-based DB is implemented, in line with solution constraints.  
• Data Privacy & Retention: We store only the necessary user data (e.g., cached leads, quotations) and purge or overwrite outdated info periodically. This reduces the risk of sensitive data lingering on devices, aligning with the broader 7-year document retention policy (primarily enforced on the server side).  
• Data includes:  
  - JWT tokens.  
  - Recently fetched leads and quotations.  
  - User preferences (optional).

#### 4.2 Backend Database Reliance
• CUSTAP does not manage its own full database; persistent data resides in Azure PostgreSQL (accessed via the backend).  
• Any additional reference data (e.g., product catalogs, statuses) is fetched from the backend on-demand or cached locally for short durations.

---

### 5. Hosting & Distribution

#### 5.1 Mobile App Release
• iOS: Distributed through Apple App Store (TestFlight for staging/pre-release).  
• Android: Distributed through Google Play Store (internal/alpha tracks for staging).  
• The app itself is not “hosted” like a web service; end users install it on their devices.  
• We officially support iOS 12+ and Android 8+ to ensure a stable user experience while still covering most modern devices.

---

### 6. DevOps Tools & CI/CD Approach

#### 6.1 Version Control & Build
• Source code managed in Git (Azure Repos or GitHub).  
• Pull request workflow with code reviews.  
• Automated builds triggered by commits via Azure DevOps or GitHub Actions.

#### 6.2 Stages
1. Install Dependencies & Lint/Unit Test  
   - Runs yarn or npm install, followed by lint (ESLint/Prettier) and Jest tests.  
2. Build (Android and/or iOS)  
   - Gradle build or Xcode build steps for generating release artifacts.  
3. Distribute  
   - Upload to Play Console (Android) or App Store Connect (iOS).  
   - Use separate tracks for staging vs. production.  
4. Environment Variables  
   - Example: MSG91 keys, backend API URL, encryption secret for AsyncStorage. Injected via CI pipeline at build time.  
   - We store and retrieve these secrets from secured variable stores (e.g., Azure Key Vault, GitHub Actions secrets) and mask them in logs to prevent accidental exposure.

---

### 7. Third-Party Services

#### 7.1 OTP via MSG91
• The Customer App triggers SMS-based OTP for login, but all requests are ultimately handled server-side by the backend.  
• CUSTAP only calls the backend’s OTP endpoints and does not directly integrate with MSG91.

#### 7.2 Azure Blob Upload
• For KYC file uploads, CUSTAP obtains a short-lived SAS token from the backend and then PUTs the document directly to Azure Blob.  
• No direct account credentials or secret keys are stored in the app.

#### 7.3 Email Notifications (SendGrid)
• Not invoked directly from CUSTAP—handled exclusively by the backend for system or password reset emails.


#### 7.4 Monitoring & Crash Reporting
• Optionally integrate Firebase Crashlytics or Sentry for runtime error tracking and performance monitoring.  
• Since the concurrency scale is moderate (400–600), adopting a lightweight crash reporting solution can provide valuable insights without undue complexity.


---

### 8. Summary
The Customer App (CUSTAP) leverages React Native 0.70+, TypeScript 4.5+, and an encrypted AsyncStorage layer for a secure, lightweight mobile experience. All heavier tasks (database operations, server scaling) reside within the backend, while CI/CD pipelines facilitate consistent build and distribution for iOS and Android.

### 9. Licensing Compliance
• We rely on open-source libraries (e.g., React Native, React Navigation, axios). We track their licenses (MIT, BSD, or Apache) in our repository, ensuring we remain compliant with all requirements, including attribution if necessary.  
• We plan periodic reviews to ensure license obligations do not conflict with our deployment strategy.



### 10. Reference Note
• This document aligns with the system-wide technology decisions captured in L1-TSD and references additional contextual details from L1-OVERVIEW and L1-HLD to maintain a unified architectural approach.
