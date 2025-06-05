## L3-CM-BE: Configuration Manual (CM) for BE

This document also references L2-LLD-IC for relevant low-level design and integration constraints. Adhering to both L1-HLD and L2-LLD-IC ensures the Back-End (BE) remains consistent with the broader system architecture and interface requirements.

This document provides administrators with guidelines for configuring the Back-End (BE) component of the SafeLanes Rest Hours solution after deployment. It focuses on post-deployment system settings, environment parameters, security configurations, and integration touchpoints for both vessel- and office-based installations at the moderate project scale (~30 vessel users, ~100 office users). The document references and consolidates relevant information from higher-level architecture and design documents (e.g., L1-HLD, L3-DG-BE, and L2-LLD-IC).

---

## 1. Introduction

The BE component is a Nest.js application, connected to a MySQL database, that supports maritime rest-hour tracking on both vessel and office servers. This Configuration Manual outlines how to fine-tune environment, security, integration, and operational options after installing and running the application. It assumes that the basic deployment steps (e.g., file transfers, Node.js installation, MySQL configuration, PM2 setup, etc.) have already been performed as described in the L3-DG-BE: Deployment Guide.

Key points:

- The solution must operate offline on vessel servers and online in the office environment.  
- Administrators should apply consistent configurations across vessel and office setups, making environment-specific adjustments where necessary (e.g., database host, certificate usage).  
- Designed for moderate-scale usage (up to ~30 vessel users, ~100 office users).  
  Additionally, review L2-LLD-IC for low-level interface configuration details that may affect the BE setup, particularly where the system design or naming conventions require precise alignment.

---

## 2. Component System Settings

This section outlines the main system-level settings for the BE component. Most are set via environment variables or a .env file. Adjust values as appropriate for your environment, ensuring alignment with any organizational requirements for security and performance.

### 2.1 Environment File Location

Place the .env file within the BE project directory, or configure environment variables at the OS level if preferred. Restrict file permissions to the account running the Nest.js process.

### 2.2 Primary Configuration Variables

Below is an example set of environment variables commonly used in the BE application. Adapt or extend as needed:

| Variable | Description | Example Value |
| :---- | :---- | :---- |
| NODE_ENV | Node environment mode | production |
| DB_HOST | Hostname or IP address of the MySQL server | localhost |
| DB_PORT | Port on which MySQL is listening | 3306 |
| DB_USER | MySQL username | safelanes_be |
| DB_PASS | MySQL password | yourStrongPassword |
| DB_NAME | Database name for rest-hour data | resthours_vessel |
| SERVER_PORT | Port on which the Nest.js application listens | 3000 |
| JWT_SECRET | Key(s) used to sign/verify JWTs | yourJWTSecret |
| TLS_KEY_PATH | Path to the SSL private key (if applicable) | /etc/ssl/private/server.key |
| TLS_CERT_PATH | Path to the SSL certificate chain (if applicable) | /etc/ssl/certs/server.crt |

### 2.3 PM2 Process Configuration

Use an ecosystem file (ecosystem.config.js) to define how PM2 manages the BE process (see L3-DG-BE for examples). Common configuration fields:

- script: The main entry point (e.g., dist/main.js).  
- instances: Usually "1" at this scale.  
- autorestart: true (recommended).  
- watch: false (to avoid restarts on file changes).

---

## 3. Parameter Definitions

This section further explains the meaning, constraints, and recommended values of the most critical parameters.

Consult L2-LLD-IC for deeper insights into parameter-level interactions and any additional interface constraints that must be configured consistently across all solution components.

### 3.1 Database Connection

- DB_HOST, DB_PORT, DB_USER, DB_PASS, DB_NAME  
  - Must be consistent with the MySQL server setup.  
  - Vessel servers typically use "localhost" with a local MySQL instance.  
  - Office servers may use an internal IP or hostname if MySQL is hosted separately.

### 3.2 Application Mode

- NODE_ENV  
  - Typically set to "production" on vessel and office servers.  
  - "development" or "test" can be used on lab or QA setups, enabling more verbose logs.

### 3.3 Security Keys

- JWT_SECRET  
  - Used to sign/verify JSON Web Tokens for staff authentication.  
  - Must match the identity service's expectations if running online.  
  - Should be rotated regularly (every 6–12 months) following the organization's security policy.  
- TLS_KEY_PATH, TLS_CERT_PATH (optional)  
  - If using HTTPS through the application itself, specify the private key and certificate.  
  - If employing Nginx for TLS termination, these may not be needed at the application layer.  

---

## 4. Environment-Specific Configurations

Different environments (development, staging, and production) typically share similar configurations, with small variations in endpoints, credentials, and logging levels.

### 4.1 Development

Use an ephemeral local MySQL instance that does not rely on containers (Docker), aligning with the official no-container policy.

- Use a simpler JWT_SECRET for local testing.  
- TLS may be optional or replaced with self-signed certificates.  
- Logging can be verbose (debug mode).

### 4.2 Staging (Optional Environment)

- Mirrors production settings for realistic testing.  
- Use a staging MySQL server with proper user credentials.  
- Use intermediate certificates or the same CA as production but with staging domain names.  

### 4.3 Production

- Full security measures enabled.  
- Properly issued or organization-trusted certificates.  
- Unique DB credentials, locked-down firewall rules.  
- Ensure logs are minimally verbose.

### 4.4 Vessel vs. Office Differences

- Vessel: Typically DB_HOST=localhost, limited or no inbound firewall exceptions.  
- Office: DB_HOST may be a separate server or local. If a separate MySQL host is used, ensure network reliability and secure connectivity.

---

## 5. Integration Configurations

The BE integrates with:

1. MySQL for data storage.  
2. SafeLanes Identity Service (JWT-based auth) always.

Additionally, see L2-LLD-IC for more detailed interface definitions and configuration directives relevant to ensuring compatibility between the BE and other SafeLanes solution components.

### 5.1 Database Integration

- Ensure MySQL is running and accessible on the configured DB_HOST and DB_PORT.  
- For autonomous vessel scenarios, the local MySQL server must start at system boot.  
- On the office side, regular backups to a central backup system are recommended. 

### 5.3 Identity Service 

- Whether it is  the vessel or office, the BE verifies JWT tokens with the local SafeLanes Identity Service.  
- Set JWT_SECRET or store the public key if tokens are signed externally.  

---

## 6. Security Configurations

Security settings center on authentication, encryption at rest, secure transport (TLS), and managing conflict-resolution logs.

Refer to L2-LLD-IC for additional low-level security design details that may influence parameter configurations, particularly where advanced or specialized interfaces are described.

### 6.1 Authentication and Authorization

- Maintain role-based access definitions: Vessel Admin, Vessel User, Office Admin, Office User, External, etc.  
- Remove or disable any default sample credentials before production.

### 6.2 Secret Rotation

- DB_PASS and JWT_SECRET should be rotated every 6–12 months, or earlier if a compromise is suspected.  
- Vessel servers may require a manual or scripted approach:  
  - Place new secrets in the .env file or OS environment variables.  
  - Restart PM2 gracefully to load updated secrets.  
- Office servers typically rotate secrets in coordination with any centralized secret-management tool or standard IT procedure.

### 6.3 Conflict Resolution Log Retention

- Overwrites to compliance-critical data are recorded in a protected audit log.  
- By default, indefinite retention is recommended for regulatory and compliance reasons.  
- Administrators may archive or export older logs if disk space is constrained.

---

## 7. Additional Notes

1. Keep the .env file or environment variables outside source control to avoid exposing sensitive credentials.  
2. Vessel and office configurations should remain consistent; differences primarily exist only in DB_HOST and the presence (or absence) of reliable internet connectivity.  
3. Refer to L3-DG-BE (Deployment Guide) for detailed installation steps and rollback procedures.  
4. Refer to L1-HLD and L2-LLD-IC for an overview of system architecture, interface contracts, and security rationale.  
5. At this scale (~30 vessel users, ~100 office users), a single-instance PM2 process coupled with MySQL is sufficient. Horizontal scaling or complex clustering is typically unnecessary.  
6. If additional compliance regulations mandate data encryption or retention, ensure the relevant adjustments are made in the environment configurations.

By following these guidelines, administrators can configure the Back-End (BE) component to operate securely, reliably, and in accordance with the SafeLanes Rest Hours solution's requirements—both on vessels operating offline and at the office environment.  
