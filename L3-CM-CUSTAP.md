## L3-CM-CUSTAP: Configuration Manual (CM) for CUSTAP

This revised version includes clarifications to ensure that the “Customer Accepted” lead status is retained (in alignment with L1-HLD) and references to push notifications are understood within the context of poll/manual sync. Additionally, a reference to L2-LLD-IC has been added to Section 7.

All other content remains the same.

This document guides administrators in configuring the CUSTAP (Customer Mobile App) post-deployment. It addresses system settings, environment-specific parameters, integrations, and security considerations at a scale of approximately 400–600 concurrent users. All configurations described here align with the L1-HLD (High-Level Design) stipulation that the system rely on polling/manual sync rather than real-time push notifications, and they reflect the client clarifications to support configurable KYC file limits and OTP lockout policies.

---

### Table of Contents
1. [Overview](#1-overview)  
2. [Component System Settings](#2-component-system-settings)  
3. [Parameter Definitions](#3-parameter-definitions)  
4. [Environment-Specific Configurations](#4-environment-specific-configurations)  
5. [Integration Configurations](#5-integration-configurations)  
6. [Security Configurations](#6-security-configurations)  
7. [References & Alignment](#7-references--alignment)

---

### 1. Overview

This document guides administrators in configuring the CUSTAP (Customer Mobile App) post-deployment. It addresses system settings, environment-specific parameters, integrations, and security considerations at a scale of approximately 400–600 concurrent users. All configurations described here align with the L1-HLD (High-Level Design) stipulation that the system rely on polling/manual sync rather than real-time push notifications, and they reflect the client clarifications to support configurable KYC file limits and OTP lockout policies.

CUSTAP is a React Native mobile application enabling customers to register via OTP, create solar service requests, view and accept/reject quotations, upload personal KYC documents, and raise support tickets for completed (executed) leads. Unlike the Channel Partner App, CUSTAP focuses on the Customer persona and has the following key configuration areas:

• KYC document upload constraints (file size, formats).  
• OTP lockout policy (attempts, lock duration).  
• Environment-based API endpoints and build variants.  
• Over-the-air (OTA) hotfix capability via CodePush (optional, but enabled by default).  
• Poll-based synchronization intervals (manual or periodic) instead of push notifications.  

Additionally, in alignment with the final approach in L1-HLD, “Customer Accepted” is maintained as a distinct lead status when a quotation is accepted. This precedes the “Won” status finalized by CP or Admin, ensuring consistent funnel metrics across the solution.

Administrators can adjust these settings post-deployment by updating environment variables, using an admin interface (if implemented), or altering the CI/CD pipeline configuration.

---

### 2. Component System Settings

Below are high-level system settings for CUSTAP that can be configured after initial app deployment.

#### 2.1 Poll-Based Sync Configuration
• CUSTAP relies on manual or periodic polling for updates (no real-time push notifications).  
• The polling interval can be specified in the app’s config (e.g., a “SYNC_INTERVAL” in minutes).  
• Administrators can configure this interval as an environment variable if needed, or leave it defaulted to manual sync only.  
If any older documentation references “push notifications,” interpret these events as triggered upon polling or manual refresh, not real-time server-initiated pushes.

#### 2.2 CodePush (OTA) Updates
• By default, CodePush is enabled to allow urgent JavaScript-level patches without requiring a full binary release.  
• App binaries include the CodePush client library; staging and production keys can be specified in environment variables or in a secure store.  
• If desired, administrators can choose not to use this feature by omitting CodePush keys, though the compiled app still ships with the library.

#### 2.3 KYC Document Handling
• CUSTAP’s default limit allows KYC files (PDF/JPG/PNG) up to 10 MB.  
• To facilitate future changes without rebuilding the app, these limits can be externalized in environment variables (e.g., “KYC_MAX_SIZE_MB” and “KYC_ALLOWED_FORMATS”).  
• Administrators can thus raise or lower the limit, or add additional file formats, with caution to ensure user experience and compliance.

#### 2.4 OTP Lockout Policies
• Customers authenticate via phone + OTP. After five invalid attempts, the user is locked out for 15 minutes.  
• These thresholds (attempts, lock duration) may be exposed as environment variables (e.g., “OTP_MAX_ATTEMPTS” and “OTP_LOCKOUT_MINUTES”) for post-deployment tuning.  
• Any modifications should align with user experience concerns and security requirements.

---

### 3. Parameter Definitions

Key parameters for CUSTAP are listed here with typical default values and acceptable ranges. These parameters are typically stored in .env files, a secure configuration manager, or a mobile app build pipeline.

| Parameter                 | Default / Example        | Description                                                                                                                                                           | Acceptable Values                  |
|---------------------------|--------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------|------------------------------------|
| API_URL                   | https://staging-api...  | Points the app to the correct backend environment.                                                                                                                    | Valid HTTPS endpoint               |
| SYNC_INTERVAL             | 0                        | In minutes. 0 means manual only; any positive integer triggers periodic sync.                                                                                         | Integer ≥ 0                        |
| KYC_MAX_SIZE_MB           | 10                       | Maximum file size (in MB) for KYC documents.                                                                                                                          | 1–50 MB (typical range)            |
| KYC_ALLOWED_FORMATS       | pdf,jpg,png              | Comma-separated list of allowed file extensions for KYC uploads.                                                                                                      | Subset of [pdf, jpg, png, ...]     |
| OTP_MAX_ATTEMPTS          | 5                        | Number of incorrect OTP attempts allowed before lockout.                                                                                                              | 3–10 (typical range)               |
| OTP_LOCKOUT_MINUTES       | 15                       | Lockout duration once max attempts are reached.                                                                                                                       | 5–60 minutes (typical range)       |
| CODEPUSH_DEPLOYMENT_KEY   | (staging or prod key)    | Key for CodePush environment. If left empty, OTA updates won’t be dispatched, though the library remains compiled.                                                    | Valid CodePush key or empty        |

Note: For production releases, ensure sensitive values like CodePush keys or endpoints marked for internal use only are stored securely.

---

### 4. Environment-Specific Configurations

The following outlines typical differences and best practices for Development, Staging, and Production environments. In each environment, you can override the parameters in Section 3 to control behavior.

#### 4.1 Development
• Typically uses local or sandbox API endpoints (API_URL pointing to localhost or a dev URL).  
• Lower OTP lockout times to facilitate testing without inconvenience (e.g., OTP_LOCKOUT_MINUTES=1).  
• KYC_MAX_SIZE_MB can be kept small to speed up test uploads.  
• CodePush can be disabled or pointed to a “Development” deployment key for quick iteration.

#### 4.2 Staging
• Points to a staging backend (API_URL for staging).  
• Standard OTP lockout policy (5 attempts → 15 min) to mimic real user flow.  
• Matches the production KYC file size and formats to catch any edge cases before public release.  
• Uses a “Staging” CodePush key for distributing pre-release updates to the QA team.

#### 4.3 Production
• API_URL references the live production system.  
• Enforces the official OTP lockout policy for real users.  
• KYC doc limits are set to an approved threshold (default 10 MB or based on business decisions).  
• “Production” CodePush key is configured, enabling emergency over-the-air JS patches.

---

### 5. Integration Configurations

Although CUSTAP primarily integrates with the general backend (BCKND), below are the integration points relevant to post-deployment configuration:

#### 5.1 Backend (BCKND) API Endpoints
• The main variable is “API_URL.”  
• Verify that the URL corresponds to the correct environment (staging or production) and includes valid SSL certificates.  
• If the backend uses sub-routes for KYC or file uploads, these endpoints remain hidden behind the main base URL, so separate configuration is typically not needed for the app.

#### 5.2 Over-the-Air (OTA) Updates (CodePush)
• To enable push-based JS updates for staging and production, provide valid “CODEPUSH_DEPLOYMENT_KEY” (and optionally “CODEPUSH_STAGING_KEY” if using separate keys) in .env files or in the app’s config system.  
• Keep these keys in a secure pipeline or environment variable store to prevent unauthorized updates.

---

### 6. Security Configurations

#### 6.1 Authentication & Authorization
• CUSTAP uses an OTP-based login flow (phone number and SMS OTP).  
• The backend issues JWT tokens upon successful OTP validation; no special config is required in CUSTAP beyond ensuring “API_URL” is correct.  
• If you change the OTP parameters in the backend, update “OTP_MAX_ATTEMPTS” and “OTP_LOCKOUT_MINUTES” in the app’s environment to keep UI messages consistent with backend policies.

#### 6.2 Data Protection
• All data in transit is encrypted via HTTPS. Ensure the API_URL is “https://” with a valid SSL certificate.  
• KYC files are uploaded to the backend, which then stores them on Azure Blob. The app itself only handles ephemeral file data. No local encryption configuration is typically required.  
• Confirm your staging and production environment variables do not leak any sensitive info in logs or public repositories.

#### 6.3 Poll-Based Sync and No Push Notification Keys
• By design, the system does not require FCM or APNS credentials.  
• Periodic or manual sync fetches updated lead statuses, quotations, and KYC statuses.  
• Reducing poll frequency can lower server load at higher concurrency, while more frequent polling offers near-real-time updates.

---

### 7. References & Alignment

This Configuration Manual for CUSTAP aligns with:  
- L1-HLD statements regarding reliance on polling/manual sync instead of push notifications.  
- Client clarifications that KYC size/format and OTP lockout thresholds should be configurable post-deployment.  
- L3-DG-CUSTAP guidelines for environment variables, signing assets, and optional CodePush usage.  
- L2-LLD-IC for additional low-level design and integration considerations that apply to configuring CUSTAP.

Maintain these configurations within the recommended scale (~400–600 concurrent users). If usage grows significantly, consider revisiting certain parameter defaults (e.g., poll interval, file size limits, concurrency strategies) while retaining the same fundamental environment-based approach.

---

By adjusting the environment variables and parameters outlined here, administrators can fine-tune the CUSTAP mobile app’s functionality and constraints. This ensures a secure, user-friendly experience, consistent with the solution’s overall design and the client’s clarified requirements.