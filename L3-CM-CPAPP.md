```
## L3-CM-CPAPP: Configuration Manual (CM) for CPAPP

### 1. Introduction
This document provides administrators with a concise guide for configuring the Channel Partner App (CPAPP) after it has been deployed. In line with the solution’s current scale of ~400–600 concurrent users, these configurations aim to balance simplicity and maintainability.

This manual covers:
- Core system settings and environment variables.  
- Parameter definitions for .env files and code signing.  
- Configuration differences among development, staging, and production environments.  
- Third-party and cross-component integration setups (e.g., Google Services, CodePush).  
- Security-related settings relevant to CPAPP.

For details on build and deployment steps, please see the “L3-DG-CPAPP: Deployment Guide for CPAPP.” Higher-level solution decisions and architectural guidelines come from the L1-HLD and L1-KD documents.

---

### 2. System Settings
Administrators should understand which parts of CPAPP are configurable post-deployment and how these configurations align with the overall solution approach.

#### 2.1 Overview of Configurable Items
1. Environment Variables (.env):
   - Base API endpoints, push notification keys, and CodePush deployment keys.  
   - If any environment-based branding or feature toggles exist, they are also managed via .env files.

2. Google Services Files:
   - Android: google-services.json  
   - iOS: GoogleService-Info.plist  
   The chosen approach is to manually swap in the correct file for staging or production, ensuring each build references the appropriately named file.

3. Attachment & File Size Enforcement:
   - Per client clarifications, the backend will store and enforce these limits (currently 10 MB/file, up to 7 attachments/lead).  
   - CPAPP does not keep a hardcoded limit; instead, it queries a configuration endpoint. Admins may adjust limits on the backend without requiring an app update.

4. CodePush / Over-The-Air (OTA) Updates:
   - CPAPP reads separate CodePush keys (staging vs. production) from its environment variables to control which release channel is used for an OTA update. Administrators must ensure correct .env usage to avoid cross-environment confusion.

5. Session & Token Rules:
   - CPAPP implements a fixed 24-hour token refresh window and 30-day inactivity logout. These values are not externally configurable; changes would require another app release.

---

### 3. Parameter Definitions
The table below outlines key parameters used in the CPAPP .env files, focusing on staging and production environments. These parameters must be set correctly post-deployment to ensure the app behaves as expected.

| Parameter Name                  | Description                                                                                      | Example Value                          | Notes                                                           |
|--------------------------------|--------------------------------------------------------------------------------------------------|----------------------------------------|-----------------------------------------------------------------|
| REACT_APP_BASE_URL             | Backend API URL                                                                                  | https://api.staging.solarium.com       | Point to staging vs. production backend API.                    |
| REACT_APP_PUSH_KEY             | Push notification credential (e.g., FCM key)                                                     | STAGING-FCM-KEY                        | Used to send push messages. Must align with environment used.   |
| REACT_APP_ENV                  | App environment label                                                                            | “staging” or “production”              | Used mainly for logging, text labels, or environment checks.     |
| REACT_APP_CODEPUSH_DEPLOYMENT_KEY | Deployment key for CodePush updates                                                           | STAGING-KEY-XXXXXXXXXXXXX             | Distinct keys for staging vs. production to prevent mix-ups.     |

Administrators should store these files (.env.staging, .env.production) in a secure location, typically within the CI/CD pipeline’s secure variables or a secrets manager.

---

### 4. Environment-Specific Configurations
Below are recommendations for managing differences between development, staging, and production configurations in CPAPP.

#### 4.1 Development Environment
- Often uses mock or local backend endpoints, allowing quick iteration without impacting a shared or live server.  
- google-services.json / GoogleService-Info.plist may be placeholders if push notifications are not tested in dev.  
- CodePush might be disabled or pointed to a Dev channel with minimal usage.

#### 4.2 Staging Environment
- Use .env.staging to define:
  - REACT_APP_BASE_URL pointing to the staging backend.  
  - Distinct staging push key and CodePush deployment key.  
- The correct google-services.json and GoogleService-Info.plist for staging should be placed in the native Android/iOS folders. Name them exactly as Google requires (no suffix). Manually swap them (or script the swap) before building the staging APK/IPA.  
- Re-confirm that any staging in-app events or notifications do not overlap with production traffic.

#### 4.3 Production Environment
- Use .env.production to define:
  - REACT_APP_BASE_URL pointing to the production backend.  
  - Production FCM/APNs push key and CodePush key.  
- google-services.json and GoogleService-Info.plist for production must be in place prior to generating the final release build.  
- Strictly verify the .env.production values before performing a live release to avoid misdirecting production users to staging or inadvertently mixing credentials.

---

### 5. Integration Configurations
CPAPP integrates with a few key services. Post-deployment configuration ensures these integrations work correctly.

#### 5.1 Google Services Configuration
• Android requires a valid google-services.json.  
• iOS requires a valid GoogleService-Info.plist.  
Both files should be named precisely as required by the React Native Firebase tooling or any library in use for push notifications.  

Recommended structure:
1. For Android, store multiple versions of google-services.json in a folder (e.g., /android/firebase-config/). When targeting staging, copy the staging google-services.json into /android/app/google-services.json before building; do the same for production.  
2. For iOS, store multiple versions of GoogleService-Info.plist in /ios/firebase-config/. Copy the correct file into /ios/GoogleService-Info.plist prior to building.  

Though manual swapping is straightforward, it can be automated with a simple script in the CI pipeline that checks the environment and copies the correct file before the build step.

#### 5.2 CodePush (OTA) Configuration
• Hardcode distinct deployment keys for staging and production in .env.staging and .env.production. E.g.:
  - REACT_APP_CODEPUSH_DEPLOYMENT_KEY=STAGING-KEY-XXXXX
  - REACT_APP_CODEPUSH_DEPLOYMENT_KEY=PRODUCTION-KEY-XXXXX  
• This ensures that updates are sent to the correct environment channels.  
• For rollbacks, admins may quickly revert from the CodePush dashboard or CLI, choosing the matching environment channel.  
• Partial rollouts or canary-style distribution are optional aspects of CodePush and can be used if needed, but typically remain beyond the immediate scope at the current user scale.

---

### 6. Security Configurations
CPAPP’s security posture is shaped by both the backend (BCKND) and the front-end app. Post-deployment, administrators should check:

1. Token Timeout & Refresh:
   - The 24-hour token refresh and 30-day inactivity logout are not dynamically configured. They are hardcoded in CPAPP. If business policies change, a new CPAPP release is required.

2. Push Notification Certificates:
   - Ensure FCM (Android) and APNs (iOS) certificates/keys used in .env files match the environment. Mismatched certificates can cause staging test push notifications to appear in the production app (or vice versa).

3. Build Integrity:
   - Keep Android keystore files and iOS signing certificates in a secure location. The signing process is part of deployment but must be verified whenever environment or store credentials are updated.

4. Attachments and File Upload Limits:
   - Although the app references the back-end for actual file size constraints (currently 10 MB) and maximum lead attachments (7), confirm that your BCKND config endpoint is set appropriately for each environment. The CPAPP automatically respects whatever is served by the backend.

---

### 7. References to Higher-Level Documents
- L1-HLD: Provides the overall architecture, including security posture, environment structure, and design constraints for ~400–600 concurrent users.  
- L1-KD: Documents critical technical decisions—for example, the single-region deployment approach, “last-write-wins” concurrency model, and separate “staging vs. production” environment strategies.  
- L3-DG-CPAPP: The Deployment Guide for CPAPP, detailing the build and release process, recommended hardware, and more technical instructions on packaging/distributing the mobile app.

Administrators should confirm that all environment variables and external files align with the decisions found in the above references to ensure a coherent, stable configuration for CPAPP.

---

### Final Notes
Properly maintaining configurations for different environments is critical for predictable and secure operation of CPAPP. By adhering to the settings and procedures outlined in this manual, administrators can confidently manage the app post-deployment, ensuring that changes—such as environment-specific push keys or CodePush updates—do not accidentally impact production users. If the solution’s concurrency or business needs evolve, revisit these settings in conjunction with solution-level documents (L1-HLD, L1-KD) to gauge the need for more advanced configuration patterns.
```
