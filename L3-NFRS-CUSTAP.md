## L3-NFRS-CUSTAP: Non-Functional Requirements Specification (NFRS) for CUSTAP

### 1. Introduction  
This document defines the non-functional requirements for the Customer Mobile App (CUSTAP). It outlines performance, security, usability, reliability, compliance, and environmental constraints specifically relevant to the CUSTAP React Native client. These requirements align with the overall project scope of supporting ~400 concurrent users and short spikes up to ~600, without overengineering for higher scales. All guidelines herein reflect the latest clarifications, ensuring a pragmatic, maintainable approach for current needs.

---

### 2. Performance Requirements  
This section establishes key performance guidelines for CUSTAP, balancing user experience with practicality under current scale.

1. **App Startup & Screen Load Times**  
   - The CUSTAP interface should load its initial screen within 2–3 seconds on typical devices (e.g., mid-range Android or iOS) under standard 4G or Wi-Fi conditions.  
   - Subsequent navigation (e.g., switching tabs, viewing a lead’s quotations) should generally respond within 2 seconds, subject to network quality.  
   - Offline read views (cached data) should appear instantly (<1 second) if previously fetched.

2. **Data Fetch & Concurrency**  
   - On each manual sync (pull-to-refresh or screen navigation), the app should retrieve and render new data for leads, quotations, or documents within 2–3 seconds for 95% of requests.  
   - Normal usage is ~400 concurrent users across the system; short bursts up to ~600 will primarily impact the backend, but CUSTAP must handle server responses gracefully (e.g., show a retry prompt on 5xx errors).  
   - During peak concurrency (~600 users), response times may extend up to 5 seconds for a small percentage of requests, aligning with L1 allowances.  
   - While concurrency is the primary measure at this scale, the app should also handle short bursts of requests without significant degradation, ensuring a reasonable user experience.  
   - Performance validation will be done on representative mid-range devices and typical network conditions (4G, Wi-Fi) to confirm compliance with these targets.

3. **Network & File Uploads**  
   - KYC documents (up to 10 MB) may take longer on slow connections; the app should provide a progress indicator without imposing strict client-side timeouts.  
   - The user is free to cancel or retry an upload if the connection is too slow or disrupted.  
   - Note: SAS tokens used for uploads may expire (e.g., ~5 minutes). If a large file upload exceeds the token’s validity, the app should request a refreshed token or gracefully retry.

  
4. Poll-Based Notifications  
   -   
   No automatic background polling is enforced: users must manually refresh (or navigate) to retrieve updated data, which reduces battery/data usage.  
   - Any references to “push notifications” in older documents are interpreted as poll-based or user-initiated sync. No real-time server-initiated push is implemented at this scale.  
   - Any future increase in polling frequency remains optional and should weigh server overhead vs. user experience benefits.

---

### 3. Security Requirements  
Security in CUSTAP focuses on safeguarding user credentials, personal data, and KYC documents within a mobile environment.

1. **Token Storage**  
   - JSON Web Tokens (JWTs) are stored in React Native AsyncStorage, relying on device-level PIN/biometric security.  
   - While this is acceptable for the current scale, using OS-level secure storage (e.g., Android Keystore or iOS Keychain via a library such as react-native-keychain) is strongly recommended for enhanced protection.

2. **Authentication & Lockout**  
   - Phone-based OTP is the only login method for customers. After 5 incorrect OTP attempts, the system locks further OTP requests for 15 minutes.  
   - Session tokens auto-refresh daily, and a hard logout occurs after 30 consecutive days of inactivity, aligning with L1-NFRS.

3. **Data-in-Transit & Access Control**  
   - All API calls use HTTPS (TLS 1.2+).  
   - KYC files are uploaded via short-lived SAS tokens, ensuring minimal scope for unauthorized blob access.

4. **Crash and Diagnostic Logging**  
   - CUSTAP does not integrate a real-time crash analytics library by default.  
   No advanced usage telemetry is transmitted.  
   - However, adopting a lightweight crash reporting framework (e.g., Firebase Crashlytics) is recommended to detect and address widespread errors quickly.

5. **Privacy Disclaimers**  
   - A minimal in-app privacy notice or link to external policies must be provided before storing personal data (KYC).  
   - The user’s agreement to terms is captured once upon initial registration or post-install acceptance.

---

### 4. Usability Requirements  
Usability requirements ensure the CUSTAP interface is straightforward, consistent, and accessible within a basic scope.

1. **User Interface Consistency**  
   - Standard UI patterns (bottom tabs, icons, color schemes) must remain consistent across all screens.  
   - Error messages should be clear and user-friendly, especially when connectivity fails or uploads exceed file size limits.

2. **Basic Accessibility**  
   - Provide adequate color contrast, readable fonts, and sufficiently large tap targets.  
   - No formal WCAG certification required; partial checks (e.g., text scaling, contrast) help accommodate a broad user base.

3. **Language & Version Updates**  
   - English-only for launch; no immediate multi-language support. Revisit if broader user demand emerges.  
   - A basic “minimum supported version” check ensures users on older app builds are prompted to upgrade if critical fixes or security patches are released.

---

### 5. Reliability Requirements  
Reliability requirements address the app’s availability for offline usage, error handling, and resilience to network disruptions.

1. **Offline Read-Only**  
   - Cached data remains viewable without an internet connection, letting users check leads, quotations, or documents retrieved previously.  
   - Any attempt to create or update leads, upload KYC, or accept quotations offline must gracefully show an error prompting reconnection.

2. **Error Handling & Retries**  
   - Network failures or server unavailability should display a clear retry option or an informative offline message.  
   - KYC uploads include a progress indicator and can be canceled or retried upon failure.

3. **Client Stability**  
   - Non-critical app errors (e.g., minor JSON parse failures) should fail quietly or give a brief error prompt, without crashing the entire app.  
   - The user’s session info (JWT) should persist through minor app restarts, provided it is still within the validity period.

4. Availability Target  
   - CUSTAP depends on backend uptime for online operations but aims for ~99.5% availability in alignment with L1-NFRS.  
   - If the backend is down, offline read-only mode remains partially available for previously cached data, lowering the impact on usability.

---

### 6. Compliance Requirements  
CUSTAP must comply with relevant organizational data policies and basic privacy regulations at the current scale.

1. **Data Retention & Encryption**  
   - All personal leads and KYC documents remain in Azure-backed services for up to seven years, as mandated in L1-NFRS.  
   - In-app data (local cache) is minimal and not stored beyond a user’s session or explicit user actions.

2. **Legal & Privacy Notices**  
   - CUSTAP must present a minimal privacy policy link. Users implicitly consent to data processing (KYC, lead management) upon continuing use.

3. **Version Control & T&C**  
   - For major updates that alter data handling, the app may enforce a new T&C acceptance or force an app update to ensure compliance.

---

### 7. Environmental Requirements  
These requirements define the hardware, software, and network conditions in which CUSTAP must operate effectively.

1. **Hardware & OS**  
   - Recommended minimum device specs: Android 6.0+ or iOS 10+. Users below these OS levels may experience degraded performance.  
   - Typical mid-range or better smartphones are assumed for a smooth React Native experience.

2. **Framework & Libraries**  
   - Developed in React Native, tested primarily on the latest stable version.  
   - Integration with backend REST endpoints (hosted on Azure) relies on standard HTTP libraries found in React Native.

3. **Testing Scope**  
   - A basic test matrix covering representative Android (6.0 to latest) and iOS (10 to latest) devices shall be maintained to ensure performance and stability across commonly used hardware.  
   - Load and performance checks should be included in QA workflows to validate startup times, navigation delays, and file upload flows.

---

### 8. Dependencies and Requirements  
Below is a summary of primary dependencies from L1 and L2 levels, as well as clarifications specific to CUSTAP:

1. **Relevant L1 Documents**  
   - L1-NFRS: Overall Non-Functional scope (e.g., concurrency up to ~600, 2–3 second normal response times, up to 5 seconds at peak).  
   - L1-HLD: High-Level Design clarifying a modular monolith backend, poll-based “push,” and single-region Azure hosting.  
   - L1-KD: Key Technical Decisions, including last-write-wins concurrency, minimal DR approach, and modular monolith architecture.

2. **Relevant L2 Documents**  
   - (If applicable) L2-LLD-IC: Inter-Component Interaction emphasis that no real-time push is implemented—CUSTAP uses manual or user-initiated refresh calls to BCKND.  
   - L2-LLD-INTG3P: Outlines reliance on SMS gateway (MSG91) for OTP and email provider (SendGrid) for password resets or user notifications (if triggered from the server).

3. **Component-Level Requirements (from L3-FRS-CUSTAP & L3-KD-CUSTAP)**  
   - Retain online-only creation/updates; offline usage is read-only to avoid conflict resolution.  
   - Store JWT tokens in React Native AsyncStorage with daily auto-refresh.  
   - File uploads up to 10 MB with progress indicators (no forced client-side timeouts).  
   - Refer to design clarifications (#1–#10) regarding token storage location, manual polling intervals, minimal crash logging, single-language support, etc.

4. **Client Clarifications (#1–#10)**  
   - #1: Plain AsyncStorage for JWT.  
   - #2: Manual refresh for “push” updates (poll-based).  
   - #3: Indefinite upload times with a spinner.  
   - #4: Basic “minimum supported version” checks.  
   - #5: Minimal accessibility checks only.  
   - #6: No real-time crash analytics—manual or minimal logs.  
   - #7: Provide minimal privacy disclaimer.  
   - #8: Broader OS coverage (Android 6+, iOS 10+).  
   - #9: English-only initially.  
   - #10: Rely on standard device performance disclaimers.

---

End of L3-NFRS-CUSTAP