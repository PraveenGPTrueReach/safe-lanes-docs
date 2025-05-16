```markdown
## L3-DG-BCKND: Deployment Guide (DG) for BCKND

### 1. Introduction
This Deployment Guide provides step-by-step instructions for deploying the BCKND (Backend) component of the Solarium Green Energy solution. BCKND is a containerized Node.js “modular monolith” application responsible for business logic, data persistence, and integrations (SMS, email, blob storage) at a scale of ~400–600 concurrent users. The instructions below focus on using Azure services (Azure Web App and Azure Container Registry) and a CI/CD pipeline (e.g., Azure DevOps or GitHub Actions) to reliably build, test, and release new BCKND versions.

### 2. Prerequisites

#### 2.1 Azure Resources
- An active Azure subscription.  
- An existing or newly created Azure Container Registry (ACR) to store BCKND container images.  
- An Azure Web App (configured for container deployment) or Azure App Service Plan sized to handle ~400–600 concurrent users.  
  - For moderate loads, a Standard or Premium plan typically suffices, with the option to scale up or out if usage spikes.  

#### 2.2 CI/CD Pipeline Tools
- Azure DevOps organization or GitHub repository (or comparable) for source control and automated pipelines.  
- Build agents that can run Docker builds and push to ACR.  
- Secure storage for secrets (connection strings, API keys, JWT secrets), such as pipeline variable groups or Azure Key Vault integrations.

#### 2.3 Network & Security
- Ensure the Azure Web App can connect to the required external endpoints (Azure Database for PostgreSQL, MSG91, SendGrid, Azure Blob Storage).  
- If using private endpoints or firewalls, configure the necessary rules for inbound/outbound traffic.  

### 3. Installation Steps

Below is a recommended approach to setting up and installing the BCKND container:

#### 3.1 Create or Configure Azure Container Registry (ACR)
1. In the Azure Portal, create a new resource of type “Container Registry” (if one isn’t already created).  
2. Note the registry name (e.g., “myacr.azurecr.io”) and create a service connection in your CI/CD pipeline that has permission to push/pull images.

#### 3.2 Prepare the CI/CD Pipeline
1. In your pipeline definition (Azure DevOps YAML or GitHub Actions), include steps to:  
   - Check out BCKND source code.  
   - Run build and tests (e.g., npm install && npm run build && npm test).  
   - Build the Docker image for BCKND.  
   - Tag the image consistently (e.g., semantic version “v1.0.0” or commit-hash-based “bcknd-<shortsha>”).  
   - Push the image to the ACR.  

   Example snippet (pseudocode):
   ```yaml
   - task: Docker@2
     displayName: Build and push BCKND image
     inputs:
       containerRegistry: 'MyAcrServiceConnection'
       repository: 'bcknd'
       command: 'buildAndPush'
       tags: |
         $(Build.BuildNumber)
   ```

2. Store sensitive environment variables (DB connection strings, MSG91/SendGrid API keys, JWT secrets) in a protected pipeline variable group or Azure Key Vault so they are not exposed in source code.

#### 3.3 Deploy to Azure Web App (Container)
1. Create the Azure Web App (Linux) with Docker support if it does not already exist.  
2. In the pipeline’s release stage (or as part of the same pipeline after successful build and test), add a deployment step that pulls the newly built image from ACR and updates the Web App.  
3. Configure environment variables in the Web App’s “Configuration” section or via the pipeline to ensure they match your BCKND needs:
   - NODE_ENV=production  
   - DB_CONNECTION_STRING=“YourPostgresConnectionString”  
   - MSG91_API_KEY=“YourSMSKey”  
   - SENDGRID_API_KEY=“YourEmailKey”  
   - JWT_SECRET=“YourSigningSecret”  
   - And any other relevant keys from the project’s .env or pipeline variable group.  

4. For staged deployment:
   - Deploy to a “Staging” slot first.  
   - Verify correctness.  
   - Manually approve and “swap” the Staging slot to Production for a zero-downtime release (blue-green deployment).

### 4. Configuration Steps

#### 4.1 Application Settings & Secrets
- Add or update environment variables either via the Azure Portal (Web App → Configuration → Application Settings) or through your pipeline scripts.  
- For secure rotation:   
  - Update the variable in the pipeline variable group (or Key Vault).  
  - Re-run a deployment to propagate the changes without rebuilding the container image.  

#### 4.2 Connection Strings and Ports
- By default, Azure Web App for Containers uses port 80 (and 443 if HTTPS is configured). Expose the app on port 80 in your Dockerfile (e.g., `EXPOSE 80`).  
- Configure the PostgreSQL connection string in the Web App’s environment variables. A typical format might be:
  ```
  DB_CONNECTION_STRING=postgres://user:password@myazureserver.postgres.database.azure.com:5432/db_name
  ```
- Confirm BCKND references process.env for these values at runtime.

### 5. Verification Procedures

Once the new version of BCKND is deployed, perform these checks:

1. **Container Status**: In the Azure Portal under your Web App, ensure the container status is “Running” with no startup errors.  
2. **Health Endpoint / Logging**:  
   - If you have a health-check endpoint (e.g., “/health”), call it via a browser or Postman to confirm HTTP 200 OK.  
   - Check logs in Azure Monitor, Application Insights, or the Web App “Log streaming” to ensure no critical errors appear.  
3. **Functional Verification**:  
   - Test a few key API endpoints (leads, quotations, documents). Confirm calls return expected data.  
   - Validate user authentication by logging in with test credentials.  
4. **Performance Check**:  
   - If feasible, run a small load test (e.g., ~50–100 concurrent calls) to ensure latencies remain acceptable.  
   - Monitor CPU/memory usage in Azure to confirm no immediate need to scale up or out.

### 6. Rollback Procedures

In case the latest deployment exhibits errors or performance regressions:

1. **Deployment Slots (Recommended)**  
   - If you used a Staging slot, simply swap back to the previous Production slot. The system returns instantly to the previous working version.  
2. **Redeploy the Previous Image Tag**  
   - In your pipeline or ACR, locate the last known good image tag.  
   - Re-run a deployment referencing that tag.  
   - Verify the rollback succeeded (check logs, run quick tests).  
3. **Database Schema Considerations**  
   - If a release introduced a schema migration that breaks backward compatibility, ensure you have a rollback script or confirm the new schema is still backward-compatible.  
   - Always test DB migrations thoroughly in Staging before promoting to Production.

### 7. Deployment Scripts

Below is a summarized example pipeline flow (Azure DevOps or GitHub Actions) showing the relevant steps for building, testing, pushing, and deploying the BCKND container:

1. **Build and Test (CI Stage)**  
   ```yaml
   - script: npm install
   - script: npm run build
   - script: npm test
   ```
2. **Docker Build & Push**  
   ```yaml
   - task: Docker@2
     displayName: Build and Push BCKND
     inputs:
       containerRegistry: 'MyAcrServiceConnection'
       repository: 'bcknd'
       command: 'buildAndPush'
       tags: |
         $(Build.BuildNumber)
   ```
3. **Deploy to Staging Slot (CD Stage)**  
   ```yaml
   - task: AzureWebAppContainer@2
     displayName: Deploy BCKND to Staging
     inputs:
       azureSubscription: 'AzureSubscriptionServiceConnection'
       appName: 'bcknd-webapp'
       slotName: 'staging'
       containers: |
         myacr.azurecr.io/bcknd:$(Build.BuildNumber)
   ```
4. **Manual Approval**  
   - After the Staging slot is verified, a manual approval step proceeds.  
5. **Swap Staging → Production**  
   ```yaml
   - task: Azure App Service manage@0
     displayName: Swap Staging to Production
     inputs:
       azureSubscription: 'AzureSubscriptionServiceConnection'
       WebAppName: 'bcknd-webapp'
       SourceSlot: 'staging'
       TargetSlot: 'production'
   ```

With this workflow, you can maintain a reliable, repeatable deployment pipeline. Version tags in ACR allow you to easily revert to previous images if needed, and environment variables stored externally let you rotate secrets without rebuilding the container.

---

**End of Document**
```
