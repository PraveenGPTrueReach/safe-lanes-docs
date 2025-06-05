## L3-DG-BE: Deployment Guide (DG) for BE

This document provides step-by-step instructions for deploying the Back-End (BE) component of the SafeLanes Rest Hours solution (Nest.js \+ MySQL). It focuses on practical, script-based deployment in moderate-scale environments: up to \~30 vessel users and \~100 office users. The BE runs equivalently on vessel and office servers, with slight variations in configuration (e.g., database host, environment variables).

---

### 1\. Introduction

The Back-End (BE) is a Nest.js application connected to a MySQL database, deployed on:

- Vessel Servers: Approximately 8 GB RAM, 2 cores, local MySQL instance.  
- Office Server: Approximately 16 GB RAM, 4 cores, separate (office) MySQL instance.

Key points:

- Supports offline operation on vessels, with local DB.  
- No containerization (Docker) is used. Instead, a script-based approach with PM2 and Nginx is employed.  
- All references to other solution components (e.g., Angular front-end, microfrontend host environment) are excluded here. This guide covers only the Nest.js BE deployment.

---

### 2\. Prerequisites

#### 2.1 Hardware Requirements

- For vessel servers:  
  - 8 GB RAM, 2 CPU cores (minimum).  
  - Sufficient disk space for the MySQL database, audit logs, backups, and code.  
- For the office server:  
  - 16 GB RAM, 4 CPU cores (minimum).  
  - Additional storage capacity to aggregate data from all vessels.  
- Backup or snapshot mechanism suitable for daily or weekly backups.

#### 2.2 Software Requirements

- Operating System: Linux (preferred) or Windows Server.  
- Node.js 16+ (test on the same version used in development to avoid compatibility issues).  
- MySQL 8.x installed and running locally (both on vessels and in the office).  
- PM2 (Node process manager) installed globally (npm install \-g pm2).  
- Nginx (or another reverse-proxy) installed on each server.  
- 

#### 2.3 Credentials and Secrets

- Database credentials (DB\_USER, DB\_PASS, DB\_HOST, DB\_PORT).  
- JWT secret key(s) (always for both \- office as well as vessels).  
- TLS certificates or self-signed certificates for each server.

##### Securing Credentials

Given the scale and the requirement for moderate security without overengineering:

- Store credentials as OS-level environment variables with restricted file/OS permissions.  
- Optionally, enhance security by encrypting them in a password-protected archive that an administrator unlocks at startup, if operationally feasible.  
- Regularly rotate secrets, especially if a compromise is suspected.

---

### 3\. Installation Steps

Below is the recommended script-based approach for installing the Nest.js BE on each server (vessel or office). Adjust commands as needed for the specific OS.

#### 3.1 Obtain the Deployment Package

1. Retrieve the latest stable BE build package (e.g., from the SafeLanes internal CI/CD system or physical media if the vessel is offline).  
2. If using a dedicated code repository branch, clone or pull the repository onto the server:  
     
   git clone https://internal-repo/safelanes-be.git   \#\# Or fetch the packaged zip file  
     
3. If the vessel is offline, transfer the package using secure physical media (e.g., encrypted USB drive).

#### 3.2 Install System Dependencies

1. Confirm Node.js (version 16+) is installed:  
     
   node \-v  
     
   npm \-v  
     
2. Confirm MySQL 8.x is installed and running:  
     
   mysql \--version  
     
   service mysql status   \#\# Varies by OS  
     
3. Install PM2 globally:  
     
   npm install \-g pm2

#### 3.3 Install Application Dependencies

1. Enter the BE directory (where package.json is located).  
2. Run:  
     
   npm install  
     
3. Optionally, run a sanity build check:  
     
   npm run build  
     
   This transpiles TypeScript into dist/.

#### 3.4 Database Preparation

1. Create or reference the MySQL database. On the vessel:  
     
   mysql \-u root \-p  
     
   CREATE DATABASE resthours\_vessel;  
     
   On the office server:  
     
   mysql \-u root \-p  
     
   CREATE DATABASE resthours\_office;  
     
2. Create a dedicated MySQL user for the BE:  
     
   CREATE USER 'safelanes\_be'@'localhost' IDENTIFIED BY 'yourStrongPassword';  
     
   GRANT ALL ON resthours\_vessel.\* TO 'safelanes\_be'@'localhost';  
     
   FLUSH PRIVILEGES;  
     
   (Adjust database name and host accordingly.)  
     
3. Apply schema migrations:  
     
   - Option A: Manual SQL scripts (versioned .sql patches).  
   - Option B: TypeORM migration command if available. For example:  
       
     npm run typeorm:migration:run

     
   Ensure you run migrations in the correct order if the vessel is behind on updates.

---

### 4\. Configuration Steps

#### 4.1 Environment Variables

Create or modify an .env file (or set environment variables globally). Typical variables:

- NODE\_ENV=production  
- DB\_HOST=localhost (or internal IP for office)  
- DB\_PORT=3306  
- DB\_USERNAME=safelanes\_be  
- DB\_PASSWORD=yourStrongPassword  
- DB\_DATABASE=resthours\_vessel (or resthours\_office)  
- JWT\_SECRET=yourJWTSecret  
- PORT=3000  
- TLS\_KEY\_PATH=/etc/ssl/private/server.key (if applicable)  
- TLS\_CERT\_PATH=/etc/ssl/certs/server.crt (if applicable)

Store this .env file in a secure location:

- Restrict read-access to the account running the Nest.js process.  
- Periodically rotate DB\_PASS and JWT\_SECRET per security policies.

#### 4.2 TLS Certificates

- For the vessel, generate or install a TLS certificate. A long-lived self-signed cert (1–2 years) is often acceptable if fully offline.  
- For the office, either use a CA-issued cert or self-signed. Renew as scheduled.  
- Configure Nginx to listen on port 443 with SSL, forwarding traffic to the Nest.js port (e.g., 3000).

#### 4.3 PM2 Process Definition

Create a PM2 ecosystem file (e.g., ecosystem.config.js):

module.exports \= {

  apps: \[

    {

      name: "safelanes-be",

      script: "dist/main.js",

      instances: 1, // Single-instance is usually enough at this scale

      autorestart: true,

      watch: false,

      env: {

        NODE\_ENV: "production",

        // Provide essential environment vars or rely on .env file

      },

    },

  \],

};

---

### 5\. Verification Procedures

#### 5.1 Basic Service Check

1. Start the service:  
     
   pm2 start ecosystem.config.js  
     
2. Verify logs:  
     
   pm2 logs safelanes-be  
     
   Ensure no fatal errors appear.  
     
3. Check the Nest.js service port (defaults to 3000 unless overridden):  
     
   curl http://localhost:3000/health  
     
   If it returns a health message, the BE is running.

#### 5.2 Database Connectivity

- Confirm the BE has connected to MySQL by reviewing application logs:  
- Look for “Database connected” or similar.  
- Log into MySQL and check tables:  
    
  USE resthours\_vessel;   
    
  SHOW TABLES;  
    
  Ensure expected tables (resthours, tasks, audit\_log, offline\_sessions, etc.) exist.

#### 5.3 End-to-End Test (Optional)

From a front-end (or a simple REST client):

- Submit a test rest-hour record (POST /resthours).  
- Confirm it is saved without errors and that compliance checks are applied.  
- Verify the new record is visible in the local DB.

---

### 6\. Rollback Procedures

If an update fails or leads to critical errors, use one of the following rollback methods:

#### 6.1 Snapshot & Revert (Recommended)

1. Before deploying the new version:  
   - Zip or archive the existing BE code directory.  
   - Perform a MySQL dump (or snapshot) of the current database:  
       
     mysqldump \-u safelanes\_be \-p resthours\_vessel \> backup\_preupdate.sql

     
2. Deploy the new version and test.  
3. If failure occurs:  
   - Stop PM2:  
       
     pm2 stop safelanes-be  
       
   - Restore the old code directory from the zip.  
   - Restore the DB if needed:  
       
     mysql \-u safelanes\_be \-p resthours\_vessel \< backup\_preupdate.sql  
       
   - Start PM2 with the old version.

#### 6.2 “Blue-Green” Lite (Optional)

If minimal downtime is required (mostly on the office server):

1. Deploy a second instance (e.g., safelanes-be-v2) on a different port.  
2. Test functionality.  
3. Switch Nginx to route production traffic to safelanes-be-v2.  
4. If errors arise, revert Nginx to the original instance’s port.

---

### 7\. Deployment Scripts

Below is a high-level example of a script sequence for automated or semi-automated deployment. Adjust paths and commands as needed for each environment (vessel or office).

\#\!/bin/bash

\#\# 1\. Stop existing process

pm2 stop safelanes-be || true

\#\# 2\. Backup database

mysqldump \-u safelanes\_be \-p'yourStrongPassword' resthours\_vessel \> backup\_$(date \+%Y%m%d\_%H%M).sql

\#\# 3\. Backup old code

TIMESTAMP=$(date \+%Y%m%d\_%H%M)

tar \-czf safelanes-be-old-$TIMESTAMP.tar.gz /opt/safelanes-be

\#\# 4\. Fetch new code or copy from USB

cd /opt/safelanes-be

git pull origin main || echo "Or copy new code from USB here"

\#\# 5\. Install dependencies and build

npm install

npm run build

\#\# 6\. Run migrations

npm run typeorm:migration:run

\#\# 7\. Start with PM2

pm2 start ecosystem.config.js \--only safelanes-be

\#\# 8\. Verification test (healthcheck)

sleep 5

curl \-I http://localhost:3000/health

#### CI/CD Pipeline Considerations

- On the office side, a CI system (e.g., Jenkins, Azure DevOps, GitLab CI) can automate building and packaging.  
- For vessels, packages may need to be physically transferred or downloaded over a limited satellite link.  
- The pipeline should tag each release with an incremental version to track compatibility between front-end and back-end.  
- Optionally, the Nest.js application can reject requests from an incompatible front-end version if strict version matching is required.

---

### 8\. Additional Deployment Notes

#### 8.1 Offline Vessel Updates

- If the vessel is offline for \>14 days, coordinate with onboard admins to apply the new code package via secure USB.  
- Migrations can be run offline. Ensure you keep consistent environment variable files to avoid mismatch or missing secrets.

#### 8.2 TLS Certificate Renewal

- For self-signed certificates on vessels, generate them with up to a 1–2 year validity period.  
- On the office server, if using a publicly issued certificate, renew and distribute the updated chain to vessels if needed.

#### 8.3 Coordinating Microfrontend Versions

- Although not covered here in detail, ensure the vessel’s local microfrontend is updated in tandem with the backend to avoid version conflicts.  
- The BE can log warnings when it detects a mismatched front-end version. For critical changes, block outdated front-ends until an update is installed.

#### 8.4 Data Integrity and Security

- Enforce minimal OS-level user permissions on the environment variable config files. Only the user account running the Nest.js process should read them.  
- Periodically rotate DB and JWT credentials. Update environment variables and reload the PM2 process.

#### 8.5 Backups

- Perform daily or weekly DB backups on vessel and office servers. Include snapshots of the code and environment config for quick recovery.

---

### Final Remarks

By following this deployment guide, organizations can reliably install, update, and maintain the SafeLanes Rest Hours Back-End (Nest.js \+ MySQL) both on vessels (offline-ready) and at the office (aggregated data). These steps balance security, simplicity, and operational practicality for an environment of \~30 vessel users and \~100 office users, while still allowing future enhancements if the project grows in scope.  
