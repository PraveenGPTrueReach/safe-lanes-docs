## L3-DG-WEBPRT: Deployment Guide (DG) for WEBPRT

### 1. Overview
This document provides step-by-step instructions for deploying the WEBPRT (Web Portal) component of the Solarium Green Energy solution. WEBPRT is a React + TypeScript Single Page Application (SPA), serving Admin and Key Account Manager (KAM) roles. It is designed for approximately 400–600 concurrent users and integrates with the backend (BCKND) via REST over HTTPS.

In this guide, you will find:
- Prerequisites for deployment
- Detailed installation steps (build & release)
- Configuration instructions for environment variables
- Verification procedures to ensure the portal is functioning correctly
- Rollback strategies in case of deployment issues
- An overview of deployment automation scripts and CI/CD pipeline details

This deployment guide concentrates solely on the WEBPRT front-end component. Integration details for other components (e.g., CPAPP, CUSTAP, BCKND) are not covered here.

---

### 2. Prerequisites
Before deploying WEBPRT, confirm that the following prerequisites are in place:

1. Source Code Management  
   • A Git repository (e.g., GitHub or Azure Repos) containing the WEBPRT codebase.  
   • Branching model or workflow (e.g., GitFlow) for stable releases.

2. Node & Yarn/NPM Environment  
   • Node.js (LTS version; e.g., 16.x or 18.x).  
   • Yarn or npm installed locally (for local builds).  
   • A build server or CI environment (e.g., Azure DevOps, GitHub Actions) configured with Node.js for automated builds.

3. Azure Subscription or Equivalent Hosting Environment  
   • For production, an Azure Web App, Azure Static Web Apps, or Azure Storage + CDN to host the built SPA.  
   • Optional: A custom domain with a managed SSL certificate (as clarified with the client) for professional branding.

4. Network & Security Setup  
   • TLS 1.2+ enforced.  
   • A custom domain name is strongly recommended for production, with DNS mapped to your Azure Web App or Static Web App.  
   • Ensure any required firewall rules or IP restrictions are prepared if using advanced network security.

5. Access to Backend APIs  
   • The BCKND endpoints must be deployed and reachable over HTTPS at a known URL (e.g., https://api.solarium.com).  
   • Confirm that Admin/KAM roles are configured on the backend for login and role-based interactions.

---

### 3. Installation Steps
Below are the steps to set up and deploy the WEBPRT build artifacts.

#### 3.1 Local Build & Test (Optional, for Validation)
1. Clone the Git repository locally.  
2. Navigate to the project root:  
   » cd webprt  
3. Install dependencies:  
   » yarn install  
   or  
   » npm install  
4. Build and run local tests:  
   » yarn build && yarn test  
   or  
   » npm run build && npm test  
5. Confirm there are no critical errors or test failures.  
6. (Optional) Start a local dev server for quick verification:  
   » yarn start  

#### 3.2 Production Build
1. Ensure Node.js is installed on the build agent.  
2. Set environment variables in a secure manner (see Section 4).  
3. Execute the production build command:  
   » yarn build --mode production  
   or  
   » npm run build  
4. After a successful build, verify that the build folder (often named “build” or “dist”) is generated.

#### 3.3 Deployment to Azure
1. Decide on your hosting approach:

   • Azure Web App (Container or standard Web App)  
   • Azure Static Web Apps  
   • Azure Storage + CDN  

2. If using an Azure Web App:  
   • Zip or package the “build” folder contents.  
   • Deploy/Upload these static files to the Azure Web App’s wwwroot (or configured) directory, either via:  
     - Azure DevOps pipeline tasks  
     - GitHub Actions using azure/webapps-deploy actions  
     - Manual upload via Azure Portal or CLI  

3. If using Azure Static Web Apps:  
   • Configure the staticwebapp.config.json if needed (e.g., for route rewrites).  
   • Use the Azure Static Web Apps build and deploy task/action referencing your app’s build folder.  

4. If using Azure Storage + CDN:  
   • Create an Azure Storage account and enable static website hosting.  
   • Upload the compiled “build” folder contents to the $web container.  
   • Optionally enable Azure CDN for better performance.  

5. Optional: Configure a Custom Domain  
   • Purchase or manage a domain (e.g., solarium-portal.com).  
   • In your Azure Web App or Static Web App, add a custom domain, verifying DNS ownership.  
   • Enable HTTPS with a managed certificate.  

---

### 4. Configuration Steps
WEBPRT relies on a handful of environment variables for runtime behavior. The approach chosen (per client clarifications) is:

• For local development: Use an .env file, which must not be committed to repository if it contains sensitive information.  
• For production/staging: Use Azure Web App Configuration or Azure Static Web App environment variables, rather than storing them in source control.

#### 4.1 Common Environment Variables
Below are typical environment variables (names are examples; adapt to your codebase):

1. REACT_APP_API_BASE_URL  
   • URL of the BCKND API, e.g., https://api.solarium.com.  
2. REACT_APP_ENVIRONMENT  
   • Tag to indicate environment (DEV, STAGING, PROD).  
3. REACT_APP_SOME_OTHER_FLAGS  
   • Misc. feature toggles or build-time flags.

#### 4.2 Local .env Example (Never commit real secrets)
Example DEV .env:
```
REACT_APP_API_BASE_URL=https://localhost:3000
REACT_APP_ENVIRONMENT=DEV
```

#### 4.3 Production Environment Variables
1. In Azure Portal → Web App → Configuration (or the equivalent for Static Web Apps), add:
   - Key: REACT_APP_API_BASE_URL  
   - Value: https://api.solarium.com  
2. Repeat for any other needed variables.  
3. Redeploy or restart the Web App so these environment variables become available at build time.

#### 4.4 Security Considerations
• Do not store sensitive or secret keys in source control.  
• For advanced secrets management (e.g., if you require an API key for third-party usage), integrate Azure Key Vault or secure variable groups in your CI/CD pipeline.  
• Use short-lived tokens or ephemeral secrets at runtime where possible.

---

### 5. Verification Procedures
After deployment, validate the WEBPRT portal is functioning correctly:

1. Basic Accessibility  
   • Navigate to the deployed URL (either the *.azurewebsites.net domain or your custom domain).  
   • Confirm the landing/login page loads without errors.

2. Login & Role Functionality  
   • Test Admin login with email/password. Verify the dashboard displays correctly.  
   • Switch to KAM login. Confirm KAM sees territory-constrained leads.  
   • Ensure session timeout (30-min inactivity) is enforced.

3. API Integration  
   • Verify that leads, quotations, commissions, and other data load from the BCKND endpoints.  
   • Check that role-based screens (e.g., Master Data for Admin) appear or hide properly.

4. Environment Variables  
   • Validate that the portal references the correct API base URL.  
   • Confirm any feature flags behave as intended.

5. End-to-End Tests (Optional)  
   • If you have automated UI tests (e.g., Cypress), run them to validate user journeys (login, create lead, generate quote, etc.).

---

### 6. Rollback Procedures
Should critical errors occur after deploying a new version of WEBPRT, use one of these rollback strategies:

1. **Revert via CI/CD Artifacts**  
   • Re-deploy a previously known good build from your CI/CD artifact repository.  
   • E.g., pick the last successful build version in Azure DevOps or GitHub Actions, then deploy it to production.  

2. **Manual Rollback**  
   • If you manually maintain build artifacts, upload the prior stable build folder to the Azure Web App or static hosting location.  
   • Confirm that domain routing and SSL configurations remain correct.

3. **DNS Fallback** (If using distinct staging vs. production hostnames)  
   • Temporarily point the production custom domain back to the older, stable environment, if you maintain separate staging/production slots.  
   • This approach can be quick but requires validated DNS changes or slot swapping (Azure Web App slot swap).

After rollback, thoroughly investigate any production failures prior to redeployment, focusing on logs, error messages, or build configurations that caused the issue.

---

### 7. Deployment Scripts
Below is an outline of scripts typically used for WEBPRT deployment. Adapt commands to your environment.

- “build”:  
  » yarn build  
  (Generates optimized production artifacts under “build/” directory)

- “deploy:azureWebApp” (example script using Azure CLI):
  ```
  "scripts": {
    "build": "yarn build",
    "deploy:azureWebApp": "az webapp deploy --resource-group <ResourceGroup> --name <AppName> --src-path ./build"
  }
  ```
- “deploy:static” (if using Azure Storage & CDN):
  ```
  az storage blob upload-batch \
    --destination "\$web" \
    --source "./build" \
    --account-name <StorageAccountName>
  ```

These can be embedded in your CI/CD pipeline as tasks or run manually. Ensure environment variables are properly injected or set at build time.

---

### 8. CI/CD Pipeline Details
This section summarizes a typical CI/CD flow for WEBPRT using Azure DevOps or GitHub Actions.

1. **Trigger**  
   • On push or pull request to the main (or release) branch, the pipeline starts.

2. **Build Stage**  
   • Install dependencies: yarn install  
   • Lint, test, and build: yarn lint && yarn test && yarn build  
   • Produce build artifacts.

3. **Staging Deployment**  
   • Deploy the generated build artifacts to a staging slot or environment.  
   • Verify functionality (smoke tests, QA checks).

4. **Manual Approval**  
   • A reviewer checks staging. If all good, manually approve the pipeline to deploy to production.

5. **Production Deployment**  
   • Deploy the same build artifacts to the production slot or environment.  
   • Swap slots or set domain routing if needed.

6. **Post-Deployment Tests**  
   • Automated or manual checks confirm successful rollout.  
   • If issues arise, quickly rollback as described in Section 6.

Use pipeline variables to store secrets (e.g., REACT_APP_API_BASE_URL). For advanced scenarios, link Azure DevOps with Azure Key Vault or GitHub Actions Secrets for secure variable management.

---

### 9. Additional Notes

- **Performance & Scaling**  
  The SPA nature of WEBPRT generally scales well for ~400–600 concurrent users via standard Azure Web App or static hosting. For spikes, simply scale up the Azure App Service Plan or enable auto-scaling with metrics-based triggers.  

- **Analytics & Logging**  
  Client-side logging is minimal (console logs in dev). Server-side (BCKND) uses Azure Monitor or Application Insights. Ensure the front-end environment variable for “production” mode is set so dev logs (verbose console logs) are disabled.

- **Future Enhancements**  
  If you anticipate drastically increased load or advanced multi-region scenarios, you can integrate a CDN, add multi-region deployments, or use Azure Front Door. For the current 400–600 user scale, these remain optional.

---

By following these guidelines, you can reliably build, configure, and deploy the WEBPRT component. This structured approach ensures smooth releases, standardized environment variable handling, and straightforward rollback when necessary—all while supporting the project’s concurrency requirements and security considerations.