
## L3-KD-CPAPP: Component-Specific Key Decisions for CPAPP

### Overview
This document captures the final, component-specific key decisions for the Channel Partner App (CPAPP). These decisions incorporate clarifications from the overall solution (see L1-HLD constraints and L2-LLD-IC) and address how CPAPP will operate under the ~400–600 concurrent user scale.

---

## Key Decisions

- **Decision #1: Online-Only Data Writes (Reference: L1-HLD Constraints #2)**
  - CPAPP will not allow offline creation or updates of records.  
  - If the CPAPP is offline, users can only view cached data in read-only mode.  
  - Implementation Detail: Display an error or “Online Connection Required” message when attempting to create or edit leads while offline.

- **Decision #2: Poll-Based Sync for Updates (Reference: L1-HLD Constraints #3)**
  - The CPAPP will rely on periodic or user-initiated refresh (pull-to-refresh) rather than implementing real-time push notifications.  
  - Implementation Detail: Poll intervals (e.g., every 60–120 seconds) can be configured to balance performance and data timeliness.

- **Decision #3: JWT Storage in AsyncStorage**
  - The CPAPP will use React Native’s AsyncStorage to store authentication tokens, accepting a modest security risk for convenience.  
  - Implementation Detail: Tokens will remain valid for 24 hours, and the app will automatically log users out upon extended inactivity.

- **Decision #4: Backend-Driven Quotation Calculations (Reference: L1-HLD “Quotation Wizard”)**
  - All price, subsidy, and commission calculations for quotations are handled exclusively by the backend (BCKND).  
  - Implementation Detail: CPAPP will send input parameters (e.g., product selection, quantity) and display the returned pricing details or PDF from the server. No local financial logic is performed on the device.

- **Decision #5: Full Data Refresh over Incremental Sync (Reference: L1-HLD Constraints #7)**
  - CPAPP will fetch entire relevant datasets—using pagination to limit large queries if needed—instead of performing delta-based updates.  
  - Implementation Detail: Each sync request fetches the most recent version of leads, quotations, and other data from the backend; out-of-date local records are replaced.

- **Decision #6: Bare React Native Workflow**
  - The CPAPP will be developed in a bare React Native setup (not Expo-managed).  
  - Implementation Detail: Allows direct integration of native modules (e.g., for file uploads) and aligns with existing container-based CI/CD pipelines.

- **Decision #7: Local Caching via Minimal Database**
  - The CPAPP will store read-only offline data in a lightweight local database (e.g., SQLite, Realm) rather than simple key-value storage.  
  - Implementation Detail: This structured approach supports indexing of leads, quotations, and other data for quick offline access, while still maintaining read-only integrity.

---
```
