## L3-NFRS-CPAPP: Non-Functional Requirements Specification (NFRS) for CPAPP


This document uses key words (MUST, SHOULD, MAY) as per RFC 2119 to indicate requirement levels. Mandatory requirements are labeled as “MUST,” recommended requirements as “SHOULD,” and optional features as “MAY.” All changes from the previous version are marked with , 
- JWT tokens auto-refresh approximately every 24 hours for active sessions, with a hard logout after 30 days of inactivity. If the device remains offline beyond the refresh window, the CPAPP MUST prompt re-authentication when back online. Local data may be retained in read-only mode but cannot be updated until valid re-login occurs.  
- Multiple concurrent device logins per CP are allowed; last-write-wins applies if conflicts occur between devices.

### 2.3 Data Protection
This sub-subsection covers local data storage, file-upload security, and minimal device safeguards.

- All offline data (read-only leads, quotations, commissions) MUST be stored securely, relying on built-in OS encryption (Android Keystore/iOS Keychain). Additional in-app encryption layers are not mandated at this scale.  
- File uploads use short-lived (~5-minute) SAS URLs.   
A single automated retry SHOULD be implemented for transient upload failures to reduce user frustration, followed by a manual retry prompt if the failure persists.  
-   
The CPAPP SHOULD perform a lightweight check for rooted/jailbroken devices. If such a device is detected, the user MAY receive a cautionary warning. Mandatory blocking is not enforced at the current ~400–600 user scale, but this warning approach reduces security risk with minimal overhead.


### 2.4 Device and Remote Security
At present, remote wipe or full device management is not implemented due to cost-benefit considerations at this scale. If business or compliance needs increase, an optional “remote invalidation” approach MAY be introduced to instantly revoke tokens or clear caches upon the next app connection.


### 2.5 Logging and Audit
- All lead edits, status changes, or document uploads are logged on the server for seven years, per organizational policy.  
- The CPAPP itself does not retain local audit logs; server-side logging fulfills compliance and traceability needs.

---

## 3. Usability Requirements

### 3.1 Overview
The CPAPP is intended for field agents (Channel Partners) performing lead capture, quotation sharing, and commission checks. Usability requirements ensure consistent, straightforward operation under varying network conditions and device capabilities.

### 3.2 UI Consistency and Accessibility
- Provide a clear “Offline Mode” banner when connectivity is lost; disable all creation or update actions.  
- Use minimal color contrast, scalable fonts, and basic label tags for improved usability. Full WCAG compliance is not required.  
- Maintain consistent navigational elements (Home, Leads, Quotations, etc.) across screens for quick user familiarity.

### 3.3 Offline Read-Only Behaviors
- The CPAPP clearly distinguishes between online and offline states.  
- Users can view and filter previously cached leads, quotations, and commissions offline, but cannot create or update data until reconnected.  
- A short note or popup SHOULD clarify when network actions are blocked due to offline status.

---

## 4. Reliability Requirements

### 4.1 Overview
Reliability ensures the CPAPP can function consistently even if local connectivity fluctuates. While the app defers heavy data operations to the backend, it MUST handle transient errors gracefully and maintain reliable offline read-only data.

### 4.2 Availability and Uptime
This sub-subsection clarifies uptime goals and the user-facing experience during outages or maintenance.

- The CPAPP SHOULD maintain ~99.5% uptime from an app perspective, excluding user-specific hardware or network failures.  
- Any server-side outages affecting login or data operations MUST surface clear error messages (e.g., “Server Unavailable. Please try again later.”).

### 4.3 Offline Data Retention
This sub-subsection details how long offline data remains accessible and when it is purged.

-   
Cached data SHOULD remain intact during short offline windows. If the CP remains inactive for more than 30 days or manually logs out, local caches MAY then be purged to minimize stale data.  
- Expired caches minimize stale or outdated data. Upon new login, the CPAPP retrieves fresh records from the server.

### 4.4 Failure Recovery
This sub-subsection describes how the CPAPP handles partial or failed syncs and uploads.

- If a sync or file upload fails due to network issues or SAS token expiry, the CP receives a retry prompt. A simple automated retry once for file uploads SHOULD be triggered to handle transient failures before prompting user intervention.  
- The CPAPP gracefully handles partial downloads or timeouts by discarding incomplete data and retrying during the next sync cycle.

---

## 5. Compliance Requirements

### 5.1 Overview
The CPAPP MUST align with the broader compliance policies of Solarium Green Energy, local data privacy norms, and the internal 7-year retention policy. Its cross-border usage assumptions remain minimal, reflecting a single-region deployment in Azure.

### 5.2 Data Protection and Retention
- All personal or KYC data is encrypted at rest via platform-level encryption (Azure Blob for server, Keychain/Keystore on device).  
-   
The CPAPP enforces a 30-day inactivity window, after which any local cache is invalidated. This approach aligns with the updated token policy and ensures minimal compliance risk while permitting short-term offline functionality.

### 5.3 Geographic Scope
- The official usage is limited to a single Azure region. If Channel Partners operate in other regions, the same single-region hosting and data residency policy applies, with no additional cross-border measures at this time.

---

## 6. Environmental Requirements

### 6.1 Overview
The CPAPP operates on consumer mobile devices, leveraging React Native technology. These requirements specify recommended device capabilities to ensure stable performance and reflect the solution’s limited offline feature set.

### 6.2 Hardware and OS Constraints
- Recommended minimum: Android 8.0+ or iOS 13+ with at least 2 GB RAM.  
- Devices SHOULD have sufficient free storage (>200 MB) for local caching of leads/quotations and for short-term file upload staging.

### 6.3 Software Dependencies
- React Native framework (bare workflow) with popular JS libraries for image compression and local caching (SQLite or similar).  
- No specialized microservices are needed. The CPAPP depends on the main backend (BCKND) for business logic, data, and authentication.

### 6.4 Network and Connectivity
- The CPAPP requires a moderately stable internet connection for data creation/update and file uploads.  
- Offline usage (read-only) is intended for short periods, with an expectation that CPs reconnect frequently to refresh data.

---

## 7. Dependencies and Requirements

### 7.1 Upstream and Cross-Cutting Influences
- L1-NFRS: Defines overall system performance/security standards (e.g., 2–3s typical response time, ~600 peak concurrency).  
- L1-FRS & L1-HLD: Specify the app’s functional scope (lead management, quotations, no offline writes).  
- L2-LLD and Other L2 Documents: Provide cross-component architectural details, guiding the design for CPAPP’s integration with backend services.  
- L3-KD-CPAPP & L3-FRS-CPAPP: Detail key decisions and functional requirements for the CPAPP (offline caching policy, short-lived SAS uploads, no advanced push notifications).

### 7.2 Third-Party Integrations
- MSG91 for phone-based OTP.  
- Azure Blob Storage for file uploads via short-lived SAS tokens.  
- PostgreSQL (via BCKND) for persistent lead, quotation, and commission data.

### 7.3 Clarifications Incorporated
- Clarification #1 (Offline cache capping): Purge older data after  30 days inactivity or logout to prevent excessive local storage growth.  
- Clarification #2 (Minimum OS versions): Require Android 8.0+/iOS 13+ and 2 GB RAM.  
- Clarification #3 (Local data encryption): Rely on default device-level encryption (Keychain/Keystore).  
-   
Clarification #4 (File upload retry): A basic single auto-retry is recommended for file uploads; if that fails, the user must retry manually.  
- Clarification #6 (Extended offline periods): Data wiped after  30 days of inactivity or explicit logout, prompting re-login.  
- Clarification #9 (Fixed polling interval): 3-minute interval in foreground, none in background.  
- Others (e.g., #7, #10, #14) confirm minimal device security checks, basic accessibility, no offline writes, and no immediate plan to add store-and-forward.


### 7.4 Business Risk and Cost-Benefit Alignment
Non-functional requirements in this document (performance, security, reliability) address the risk of lead loss, unauthorized data exposure, and compliance infractions. Failing to meet them could damage corporate reputation, disrupt sales operations, or incur penalties. By prioritizing pragmatic, lightweight solutions at the 400–600 concurrency scale, the CPAPP balances cost and complexity in a manner suitable for current business goals.


---


## 8. Maintainability and Lifecycle
To avoid technical debt and ensure consistent updates:
- The CPAPP SHOULD adopt versioning for each major/minor release, with clear documentation of new features or breaking changes.  
- Basic coding standards and code reviews are RECOMMENDED to sustain clarity and quality as the application evolves.  
- A straightforward release management flow with staging and production builds SHOULD be maintained, ensuring minimal downtime for updates.


By meeting these non-functional requirements, the CPAPP ensures a stable, secure, and performant experience for Channel Partners, aligned with the broader Solarium Green Energy solution’s constraints and the current project scale of ~400–600 concurrent users.