
## L3-CM-WEBPRT: Configuration Manual (CM) for WEBPRT

This document provides administrators with guidance to configure the WEBPRT (Web Portal) component of the Solarium Green Energy solution after deployment. It focuses on environment variables, parameter definitions, integration points, and security settings specifically tailored to the current scale of ~400–600 concurrent users. 

---

## Table of Contents
1. [Introduction](#1-introduction)  
2. [Component System Settings](#2-component-system-settings)  
3. [Parameter Definitions](#3-parameter-definitions)  
4. [Environment-Specific Configurations](#4-environment-specific-configurations)  
5. [Integration Configurations](#5-integration-configurations)  
6. [Security Configurations](#6-security-configurations)  
7. [Additional Notes](#7-additional-notes)  

---

## 1. Introduction
WEBPRT is a React + TypeScript Single Page Application (SPA) that serves the Admin and Key Account Manager (KAM) roles. It communicates with the backend (BCKND) via REST over HTTPS. This manual outlines how to configure the portal’s runtime options, environment variables, and integrations. The goal is to ensure that administrators can adjust system behavior appropriately for development, staging, and production environments.

Key references and sources of information include:
- [L1-HLD](#) (High-Level Design), which describes the overall system and major design choices.  
- [L1-KD](#) (Key Technical Decisions), which captures decisions about concurrency, session management, lead statuses, etc.  
- [L3-DG-WEBPRT](#) (Deployment Guide), which explains how to build and deploy the WEBPRT SPA.  
- [L2-LLD-IC](#) (Inter-Component Interaction), which provides integration guidance at the cross-cutting level.  

In line with project clarifications, this manual focuses on pragmatic solutions for the ~400–600 concurrency range and avoids overengineering.  

---

## 2. Component System Settings
Administrators can configure various system settings post-deployment, typically through environment variables in Azure Web App (or equivalent hosting).

### 2.1 Session Timeout
• By default, a session inactivity timeout of 30 minutes is enforced for Admin and KAM.  
• Per client clarification #1, we have chosen to expose this as an environment variable so that different environments (development, staging, production) can override it if needed.  

Variable name example:  
• REACT_APP_SESSION_TIMEOUT_MIN — integer representing minutes of inactivity allowed before auto-logout.  

### 2.2 API Endpoint
• WEBPRT must point to the correct BCKND API base URL.  
• Adjust this during deployment so that the portal calls the correct environment backend (DEV, STAGING, PROD).  

Variable name example:  
• REACT_APP_API_BASE_URL=https://api.example.com  

### 2.3 Environment Indicator
• An environment indicator (DEV, STAGING, or PROD) displayed in the portal UI or used for log verbosity can be set.  

Variable name example:  
• REACT_APP_ENVIRONMENT=DEV|STAGING|PROD  

### 2.4 Other Feature Flags or Parameters
• At the current scale, advanced feature-flag toggles have not been adopted as per client clarification #3. New or optional features are activated by code release rather than environment config.  
• The roles and lead statuses remain strictly coded to preserve consistent workflows (clarifications #2 and #4).

---

## 3. Parameter Definitions
Below are the key runtime parameters relevant to configuring the WEBPRT component:

| Parameter                     | Description                                                                  | Acceptable Values                       | Default (Prod)         |
|------------------------------ | ---------------------------------------------------------------------------- | --------------------------------------- | ----------------------- |
| REACT_APP_API_BASE_URL       | Base URL for the BCKND REST APIs.                                            | A valid HTTPS URL                       | https://api.solarium.com |
| REACT_APP_ENVIRONMENT         | String label indicating environment name (used for logs or UI badges).      | DEV, STAGING, PROD                      | PROD                   |
| REACT_APP_SESSION_TIMEOUT_MIN | Session inactivity logout threshold.                                        | Positive integer (minutes)              | 30                     |

• These environment variables should be defined as part of the hosting configuration (e.g., Azure Web App Settings) rather than in source control.  
• For local development, a .env file can be used (but must not be committed if it contains sensitive details).  

No other parameters require post-deployment adjustment for standard usage. The lead status progression is hardcoded, as are role-based differences between Admin and KAM.

---

## 4. Environment-Specific Configurations
Configure environment variables or settings according to the environment life cycle. Below is a typical breakdown:

### 4.1 Development
• REACT_APP_API_BASE_URL = http://localhost:3000 or a local dev server.  
• REACT_APP_ENVIRONMENT = "DEV".  
• REACT_APP_SESSION_TIMEOUT_MIN = 60 (longer timeouts for developer convenience).  
• Verbose debugging console logs are enabled by default in React development mode.

### 4.2 Staging
• REACT_APP_API_BASE_URL = https://staging-api.solarium.com.  
• REACT_APP_ENVIRONMENT = "STAGING".  
• REACT_APP_SESSION_TIMEOUT_MIN = 30 (recommended to simulate production).  
• Staging typically mirrors production settings but may allow for extra debugging or manual validations.

### 4.3 Production
• REACT_APP_API_BASE_URL = https://api.solarium.com.  
• REACT_APP_ENVIRONMENT = "PROD".  
• REACT_APP_SESSION_TIMEOUT_MIN = 30 (or a client-approved value).  
• Logging is minimized to essential errors/warnings only.

---

## 5. Integration Configurations
Although the Web Portal (WEBPRT) primarily interacts with the BCKND, there are minimal direct external integrations. Most third-party services (SMS, email) are handled by the backend. Administrators should confirm the following:

### 5.1 Backend API Credentials
• WEBPRT does not store backend credentials but obtains a JWT token from BCKND during user login.  
• Confirm that BCKND is properly configured for Admin and KAM role-based authentication.  
• No additional keys for SMS (MSG91) or email (SendGrid) reside in WEBPRT, as those are configured server-side in BCKND.

### 5.2 Deployment to Azure or Equivalent Hosting
• If using Azure Web App or Azure Static Web Apps, each environment variable (Section 3) must be set via “Configuration” in the Azure Portal.  
• Always enable HTTPS for the domain or subdomain hosting the SPA.

### 5.3 CDN/Custom Domain
• Optionally, if the portal is served via Azure Storage + CDN, ensure the environment variables are applied during the build phase or using an environment-specific build pipeline step.  
• Map a custom domain (e.g., portal.solarium.com) in Azure App Service or CDN. Configure DNS accordingly.

---

## 6. Security Configurations
Security for the WEBPRT focuses on:
1. Session Timeout  
2. Secure Communication with the Backend  
3. Secure Handling of Environment Variables  

### 6.1 Session Management
• Per client clarifications, session inactivity is enforced with REACT_APP_SESSION_TIMEOUT_MIN.  
• The recommended production value is 30 minutes.  
• Actual enforcement occurs at the front-end logic (auto-logout) and can be complemented by server-side token expiration set in the BCKND.

### 6.2 HTTPS Only
• All traffic from WEBPRT to the BCKND must use TLS 1.2+ (HTTPS).  
• For local dev, self-signed certificates can be used. In production, ensure a valid TLS certificate is installed.

### 6.3 Role-Based Access
• The portal contains logic restrictions for Admin vs. KAM (e.g., only Admin can deactivate CP).  
• These role checks are primarily enforced on the backend. The front-end UI also hides or disables controls as appropriate.  
• The specific role-to-permission mappings are coded and not configurable via environment variables (clarification #4).

### 6.4 Storage of Secrets
• WEBPRT does not store sensitive credentials (like database passwords) in environment variables.  
• The BCKND is responsible for validating user credentials, storing secrets in a more secure vault if necessary.  
• For environment variables in the Web Portal, avoid placing sensitive info in code or public repos.

---

## 7. Additional Notes
1. Lead Status Names: The solution uses fixed status labels (e.g., “New,” “In Discussion,” “Physical Meeting Assigned,” “Customer Accepted,” “Won,” etc.). These status labels are not dynamically configurable; they are hardcoded for consistency (clarification #2).  
2. Feature Toggles: Based on client decisions, no runtime toggles are used for new/experimental functionality. Enhancements become available only through code releases (clarification #3).  
3. Concurrency Model (“Last Write Wins”): There is no configurable concurrency setting. The system logs overwrites in an audit trail, but no custom concurrency policies are adjustable post-deployment.  
4. Maintenance & Patches: For issues requiring config changes, adjust environment variables in Azure and redeploy or refresh. For deeper changes (e.g., new partial-save durations, role expansions), a new code release is required.

---

By following these guidelines, administrators can configure the WEBPRT front end consistently across development, staging, and production environments. The approach ensures the Web Portal remains secure, properly integrates with the BCKND, and respects the solution’s scale of ~400–600 concurrent users without introducing unnecessary complexity.
```
