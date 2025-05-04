## L3-CM-FE: Configuration Manual (CM) for FE

This Configuration Manual provides guidelines for post-deployment configuration of the SafeLanes Rest Hours Front End (FE). It details how administrators can set up runtime parameters, security options, and minimal environment-based considerations. Importantly, it aligns with the single-build-artifact approach described in L3-DG-FE, wherein the same compiled bundle dynamically adapts to vessel or office usage at runtime. The goal is to ensure proper alignment with the vessel and office environments at the moderate scale described in L1-HLD and related documents.

---

### 1. Introduction

The Rest Hours FE is implemented as an Angular 16 microfrontend that can run in both vessel and office environments via runtime checks in a single compiled artifact. It is served either by a local fallback on vessels (via Nginx) or dynamically loaded by the SafeLanes “Sail App” in the office environment.

Because the solution supports partially offline operation, the FE employs runtime detection (e.g., environment variables, window scope checks, or small config files) to switch between vessel and office settings after a single build. This avoids duplicative artifacts and simplifies updates, especially for offline vessels.

---

### 2. Component System Settings

Below are the principal system settings relevant to the FE. In line with the single-artifact design, these settings are primarily handled via runtime detection or a unified environment file (e.g., environment.prod.ts) that reads environment-specific values at runtime.

#### 2.1 Unified Production Build

In production, a single main environment configuration (e.g., environment.prod.ts) is compiled into the artifact. This build includes logic to detect whether it is running on a vessel or in the office. Development and staging still may utilize separate environment.dev.ts or environment.staging.ts files for convenience, but for the final production deployment, one universal artifact is produced.

#### 2.2 Runtime Checks and Fallback

To keep the deployment process simple at the current scale, the project uses a single compiled artifact that checks whether it is on a vessel or in the office at runtime. Critical settings like the base API URL, module-federation remoteEntry URL, and offline session rules are read from environment variables or minimal detection logic, ensuring minimal overhead in offline scenarios while staying aligned with the “single build” approach from L3-DG-FE.

---

### 3. Parameter Definitions

The following are key FE-related parameters that typically appear in the unified environment or runtime configuration. Adapt names as needed in your codebase.

1. **baseApiUrl**  
   - Description: URL pointing to the Nest.js backend.  
   - Acceptable Values:  
     - Vessel usage: http://vessel.local:3000/ (or https:// if TLS is configured).  
     - Office usage: https://office.server/api/  
   - Impact: Ensures the FE makes correct REST calls.

2. **remoteEntryUrl**  
   - Description: The Module Federation endpoint loaded by the Sail App (primarily relevant in the office environment).  
   - Acceptable Values: e.g. "https://office.server/fe/remoteEntry.js"  
   - Impact: If changed, ensure the single artifact can detect and use the correct endpoint at runtime. Typical usage is for the office environment. On vessels, the local fallback is used.

3. **offlineSessionMaxDays**  
   - Description: Maximum number of days for offline session validity. Tied to L1-KD #5.  
   - Acceptable Values: Typically 14. Can be reduced for stricter offline security.  
   - Impact: The FE references this at runtime to display warnings or block usage if the offline session passes the threshold.

4. **enableVersionMismatchWarning**  
   - Description: When true, the FE warns users if a local fallback bundle is outdated relative to the Nest.js version.  
   - Acceptable Values: true or false  
   - Impact: Helps administrators encourage updates to the offline fallback on vessels.

5. **environmentName**  
   - Description: A short string (e.g. "Production") for debugging or display in the UI.  
   - Acceptable Values: Any descriptive string.  
   - Impact: Used mainly for logging or status displays in the FE.
  
Administrators can still add or remove parameters as the project evolves, but the approach remains a single compiled artifact that checks for vessel vs. office usage at runtime. This reduces overhead and matches L3-DG-FE directives.

---

### 4. Applying Configurations at Runtime

When using a single artifact, the FE automatically detects the appropriate runtime context once it loads. For example, a simple method is to check for a “vessel.local” domain or an environment variable specifying “OFFICE” vs. “VESSEL.” The following points illustrate how these configurations might appear or be updated post-deployment:

• If running on a vessel:
  - The FE sets baseApiUrl to http://vessel.local:3000/.
  - The local fallback is served via Nginx from /home/safelanes/resthours_fe/current.
  - The system displays offline session warnings if connectivity is lost.

• If running in the office:
  - The FE sets baseApiUrl to https://office.server/api/.
  - The remoteEntryUrl points to https://office.server/fe/remoteEntry.js.
  - Typically, no dedicated offline fallback is needed since connectivity is stable.

Care must be taken to ensure that the single artifact correctly loads or references any dynamic environment variables at startup, avoiding compile-time splits for vessel vs. office.

#### 4.3 Development & Staging Builds
For non-production testing, separate environment files such as environment.dev.ts or environment.staging.ts may still be used. These allow different API endpoints or debug settings without affecting the final single production artifact. This approach supports developer workflows but remains distinct from production’s unified build process.

---

### 5. Integration Configurations

#### 5.1 Module Federation Setup
The FE must specify its exposed modules and remoteEntry path in Angular’s Module Federation configuration. Since a single artifact is used, it checks at runtime whether remoteEntryUrl is applicable (office) or not (vessel). Administrators need only confirm that the final deployed bundle can dynamically reference the correct endpoint environment.

#### 5.2 Backend Service Connectivity
- baseApiUrl is determined at runtime based on whether the FE is running in the vessel or office environment:  
  - Vessel Example: http://vessel.local:3000/  
  - Office Example: https://office.server/api/  
- Confirm these URLs match any reverse-proxy settings from Nginx or PM2 per L3-DG-FE.

#### 5.3 Data Sync & Conflict Handling
- The FE’s conflict-resolution UI is triggered when the local or office Nest.js detects a record version mismatch.  
- No separate compile-time environment is needed. The Nest.js back-end orchestrates up/down sync, with the FE referencing the single set of runtime environment checks.

---

### 6. Security Configurations

#### 6.1 Offline Sessions
- offlineSessionMaxDays remains a runtime-configurable parameter, default 14 days, ensuring alignment with L1-KD #5. Administrators may adjust it in the final environment configuration or an external variable that the unified artifact can read at startup.
- Matching back-end logic is critical to enforce the same offline session policy on both the vessel and office sides.

#### 6.2 HTTPS & Certificates
- While the FE artifact files are static, secure Nginx proxies should be enabled with TLS certificates (vessel or office).  
- If applicable, ensure the single-artifact approach can handle both HTTP (in local host scenarios) or HTTPS. The baseApiUrl must use “https://” if a valid certificate is deployed.

#### 6.3 Version Mismatch Warnings
- enableVersionMismatchWarning, if set, causes a runtime check that compares the FE’s version to the Nest.js version. Even as a single artifact, the vessel’s local environment can display warnings if it detects an out-of-date fallback.

---

### 7. Post-Deployment Procedures

1. Deploy the single compiled FE artifact in the appropriate server directories for vessel or office.  
2. Confirm Nginx configuration references /resthours_fe/ (or equivalent) if on the vessel, matching L3-DG-FE instructions. 
3. Verify that the runtime detection logic sets:
   - baseApiUrl to vessel or office endpoints  
   - remoteEntryUrl if in office environment  
   - offlineSessionMaxDays consistent with policy  
4. Test final deployment:
   - Vessel side: simulate offline usage, confirm local fallback, including correct baseApiUrl.  
     Office side: load from remoteEntryUrl in the Sail App, ensuring the dynamic checks work as expected.

---

### 8. Additional Notes & References

- For a comprehensive overview of Nginx, PM2, and manual rollback steps, refer to L3-DG-FE: Deployment Guide for FE.  
- For deeper architecture details involving single-artifact logic, offline fallback, and conflict resolution, see L1-HLD and L3-WF-FE. The single build design outlined here prevents confusion and reduces risk of mismatched versions across vessel and office.  
- This manual is designed for the moderate user scale and direct server deployments described in the project. Any expansion should still maintain the single artifact approach to avoid unnecessary complexity at the current scale.

By following these guidelines, administrators ensure that the Rest Hours FE—built once with runtime detection—can be reliably configured for both vessel and office, preserving offline/online scenarios and matching L3-DG-FE’s single artifact strategy.