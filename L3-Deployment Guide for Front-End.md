## L3-DG-FE: Deployment Guide (DG) for FE

This document provides a step-by-step guide for deploying the SafeLanes Rest Hours Front End (FE). It covers required prerequisites, installation procedures, minimal configuration, verification, rollback, and CI/CD pipeline details. The goal is to ensure that both the vessel and office environments can reliably host the same Angular 16 artifact—one that provides a local offline fallback on vessels and a module-federation–based microfrontend in the office.

---

### 1\. Introduction

The Rest Hours FE is an Angular 18.2.0application that:  
• Runs on vessels in offline mode, served by the local Nest.js backend and Nginx.  
• Loads dynamically via Module Federation in the SafeLanes “Sail App” in the office environment.

A single build artifact is created, using environment-based runtime checks to adapt to each scenario. This matches the current moderate project scale (up to \~30 vessel users, \~100 office users) without overengineering.

---

### 2\. Prerequisites

Before deploying, ensure the following:

#### 2.1 Hardware & OS

• Vessel Server: 8 GB RAM, 2+ CPU cores, sufficient disk space to store at least two FE build folders (for rollback).  
• Office Environment: 16 GB RAM, 4 CPU cores for the main server hosting the Sail App and the remoteEntry.

#### 2.2 Software Requirements

• Node.js 16+ (or 18+) and npm on the build machine (not necessarily on the vessel).  
• Angular CLI 18.x on the build machine (optional if using npm scripts alone).  
• Nginx and PM2 on both vessel and office servers (as per L1-HLD).  
• Git, SSH, or similar for transferring build artifacts to the vessel.  
• Browsers supporting ES2017+ (Chrome, Edge, Firefox, Safari) for end users.

#### 2.3 Network & Security

• TLS certificates for any HTTPS access to the vessel or office environment.  
• Vessel can be offline. Deployment typically occurs in port or when connectivity is available.  
• Office environment uses stable network connections and has no special inbound restrictions beyond standard HTTPS.

---

### 3\. Installation Steps

Below is a general outline for building and installing the FE in both vessel and office environments.

#### 3.1 Build the Angular FE

1. Obtain the FE source code from the repository (e.g., via Git).  
     
2. On a build machine (local or CI server), run:  
   • npm ci  
   • npm run lint (optional but recommended)  
   • npm run test (executes unit tests)  
   • npm run build  
     
   The last command typically produces a “dist/fe” folder containing the production-ready files.  
     
3. Confirm the build was successful and no critical errors were reported.

Note: Per Clarification \#1, we use a single build artifact that includes runtime checks for identifying “vessel vs. office” usage.

#### 3.2 Deployment to Vessel Environment

1. Copy the build output (e.g., dist/fe) to the vessel server. Common practice is to use an SSH-based sync tool (scp, rsync) or a script-run approach.  
2. Place the build files in a directory, for instance:  
   /home/safelanes/resthours\_fe/current  
3. Ensure PM2 and Nest.js remain operational. The FE files will be served by Nginx alongside the Nest.js API.

At this point, the vessel has a local offline copy. If the vessel is offline, these steps must be performed manually (e.g., from a USB drive), referencing the same folder structure.

#### 3.3 Deployment to Office Environment

1. Copy or upload the same dist/fe build to the office server that hosts the “Sail App Shell.”  
2. The microfrontend typically exposes a remoteEntry.js file within dist/fe.  
3. Configure the Sail App’s module-federation configuration to reference the FE’s remoteEntry file. Usually this means adjusting the Sail App’s webpack config or environment to point to “[https://office.server/fe/remoteEntry.js”](https://office.server/fe/remoteEntry.js”).  
4. Reload or redeploy the Sail App so it recognizes the newly updated microfrontend. Confirm the remoteEntry is accessible to connected office browsers.

---

### 4\. Configuration Steps

In both environments, minimal runtime configuration ensures the FE knows which backend base URL to call and whether it should function as a local fallback.

#### 4.1 Environment Variables & Files

• The FE can read a base API URL (like VESSEL\_API\_URL or OFFICE\_API\_URL) at runtime.  
• Typically, environment.\*.ts files or a small JavaScript config snippet in index.html can hold these references.  
• For the vessel, point to the local Nest.js server (e.g., [http://vessel.local:3000/](http://vessel.local:3000/)).  
• For the office, point to the central host’s API (e.g., [https://office.server/api/](https://office.server/api/)).

#### 4.2 Nginx Configuration

• Use a single Nginx server block that:  
• Serves the Angular FE files at a path such as /resthours\_fe/.  
• Proxies API requests (e.g., /api/) to Nest.js on port 3000\.  
• Example snippet (vessel side):

server {

  listen 80;

  server\_name vessel.local;

  location /resthours\_fe/ {

    root /home/safelanes/resthours\_fe/current;

    try\_files $uri $uri/ /resthours\_fe/index.html;

  }

  location /api/ {

    proxy\_pass http://127.0.0.1:3000/;

  }

}

• This approach keeps the offline fallback (the copied dist folder) accessible under /resthours\_fe/. The same dist folder also includes remoteEntry.js if used in an office scenario.

#### 4.3 Security & HTTPS

• In production, we recommend enabling HTTPS with a valid certificate.  
• Self-signed or internal CAs may be used on vessels.  
• Update the Nginx config’s listen directives to 443 and supply ssl\_certificate / ssl\_certificate\_key paths.

---

### 5\. Verification Procedures

Use these checks after deployment to confirm the FE is functioning properly.

1. Open a browser on the same network (vessel or office).  
2. Navigate to:  
   - Vessel: [http://vessel.local/resthours\_fe/](http://vessel.local/resthours_fe/) (or https if configured)  
   - Office: The Sail App URL that references the new microfrontend. For example, [https://office.server/sail/\#/resthours](https://office.server/sail/#/resthours)  
3. Log in. Confirm that the FE loads without errors and that you can access daily logs or relevant screens.  
4. If offline on the vessel, confirm that the FE still loads from the local fallback. Verify that the Nginx logs show local file serving (no external requests).  
5. Make a test rest-hour entry or fetch data from the local or office Nest.js. Ensure read/write operations succeed without error.  
6. Check the browser console for any 404 or TypeScript errors that might indicate a path misconfiguration.

---

### 6\. Rollback Procedures

Because vessels may lack consistent connectivity, it is critical to have a local rollback plan if a newly deployed build is unsuccessful.

1. Maintain a “previous-version” folder on the vessel server (e.g., /home/safelanes/resthours\_fe/prev).  
2. If the new build fails, edit Nginx’s configuration (or a symlink) to point back to the “previous-version” folder, then reload Nginx:  
     
   sudo nginx \-s reload  
     
3. Confirm the older version is now served.  
4. (Optional) If connectivity is available, a stable version can be re-downloaded from a central repository or artifact storage in the office. Place it in the rollback folder, then switch Nginx to that version.

---

### 7\. Deployment Scripts & CI/CD Pipeline Details

#### 7.1 Recommended Steps in CI

1. Pull Source: Git clone the FE repository.  
2. Install & Test:  
   - npm ci  
   - npm run lint  
   - npm run test (Jest/Cypress or similar)  
3. Build & Package:  
   - npm run build  
4. Artifact Versioning:  
   - Rename or tag the dist/fe folder with a version (e.g., fe\_1.2.3).  
   - Store the artifact in a local or remote repository (GitLab, Nexus, or a file share).

#### 7.2 Office Automated Deployment

• The CI pipeline can deploy automatically to the office server, where an environment variable or configuration file points the Sail App to the updated remoteEntry.js.  
• The pipeline may:

- SSH into the office server.  
- Copy the newly built dist/fe.  
- Update the module-federation configuration for the Sail App (if needed).  
- Restart or reload the Sail App.

#### 7.3 Vessel Manual Deployment

• Office or IT staff provide the new artifact to the vessel via a secure channel (e.g., scp, portable media).  
• Vessel personnel place it into /home/safelanes/resthours\_fe/current, test locally, and reload Nginx or PM2 if necessary.  
• Keep the previous folder for immediate rollback on the vessel.

#### 7.4 Version Control & Angular Library Mismatches

• Use Angular 18.2.0 in the host (Sail App) to avoid any major version conflicts.  
• If the host cannot be upgraded, preserve a fallback approach that bundles Angular in the FE. This leads to a larger artifact but reduces risk of mismatched library versions.

---

### 8\. Final Remarks

By following these steps, the SafeLanes Rest Hours FE can be deployed seamlessly in both vessel and office environments. A single Angular 18.2.0 artifact, served by Nginx under PM2 with Module Federation. The manual rollback process on vessels and minimal CI/CD pipeline integration ensures an efficient, controlled deployment aligned with the moderate scale of the project.

For deeper configuration details (e.g., advanced environment variables, custom ingress/SSL configuration, or RBAC logic), refer to the related L3-CM-FE (Configuration Manual) or direct references in the L1-HLD. However, the above guidelines should cover the most common deployment scenarios and enable consistent, error-free operations for the FE component.  
