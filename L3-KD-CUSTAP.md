## L3-KD-CUSTAP: Component-Specific Key Decisions for CUSTAP

### Introduction
This document outlines the key technical decisions specific to the Customer Mobile App (CUSTAP). It references the higher-level constraints and assumptions from L1-HLD, as well as integration details from L2-LLD-IC. These decisions address the moderate scale of approximately 400–600 concurrent users and reflect the client clarifications.

---

## Key Decisions

1. **Retain Online-Only Creation/Updates**  
   - (Ref: L1-HLD Constraints, Clarification #1)  
   - CUSTAP does not permit offline write operations (e.g., creating service requests or updating data while offline).  
   - Component Implementation Detail:  
     - All create/update calls must be done via secure REST endpoints when the device is online.  
     - Local caching (if any) remains read-only to avoid conflict-resolution overhead.

2. **Use Poll-Based Notifications (No True Push)**  
   - (Ref: L1-HLD Constraints, L2-LLD-IC, Clarification #2)  
   - CUSTAP relies on periodic or user-initiated polling to detect new quotations, status changes, or other updates.  
   - Component Implementation Detail:  
     - The app queries the backend at intervals (or on navigational events) to refresh data.  
     - No integration with FCM/APNs for server-initiated push.

3. **Perform Full Data Refresh (No Incremental Sync)**  
   - (Ref: L1-HLD Constraints, L2-LLD-IC, Clarification #3)  
   - CUSTAP downloads the necessary data set in entirety on each sync.  
   - Component Implementation Detail:  
     - The app may apply filters (e.g., active leads from the last 90 days) if record volumes grow.  
     - No complex “delta” logic or timestamps are implemented.

4. **Seven-Year KYC Document Retention**  
   - (Ref: L1-HLD Constraints, Clarification #4)  
   - Uploaded KYC files remain available for up to seven years with no automated purge.  
   - Component Implementation Detail:  
     - The app allows users to view previously uploaded KYC documents without expiring them.  
     - No custom archival or deletion flows are introduced in CUSTAP.

5. **Phone-Based OTP as the Sole Auth Mechanism**  
   - (Ref: L2-LLD-IC, Clarification #5)  
   - The Customer App continues to use only phone-based OTP for authentication (MSG91 gateway).  
   - Component Implementation Detail:  
     - Default retry/lockout rules apply if SMS is delayed or unreachable.  
     - No additional email/password fallback is implemented.

6. **Store Tokens in React Native AsyncStorage**  
   - (Ref: L2-LLD-IC, Clarification #6)  
   - CUSTAP saves JWTs (and short-lived SAS tokens for file uploads) in AsyncStorage.  
   - Component Implementation Detail:  
     - Access tokens are set to expire more frequently (e.g., 15–30 minutes) to limit risk in case the device is compromised.  
     - On token expiry, the app forces a new login flow.

7. **Continue Using React Native**  
   - (Ref: L1-HLD, Clarification #7)  
   - CUSTAP remains a React Native application for cross-platform efficiency.  
   - Component Implementation Detail:  
     - No additional native iOS/Android code layers introduced; existing RN modules suffice.  
     - Revisit the framework choice only if essential, advanced native features become necessary.

---

By adhering to these decisions, CUSTAP remains aligned with the overall solution architecture and the scale requirements, providing a straightforward, maintainable experience without unnecessary complexity.