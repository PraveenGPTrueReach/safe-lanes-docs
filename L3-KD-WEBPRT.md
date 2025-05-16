
## L3-KD-WEBPRT: Component-Specific Key Decisions for WEBPRT

### Introduction
This document captures the final, agreed-upon technical decisions specific to the Web Portal (WEBPRT) component. It draws from:
- The client clarifications (#1–#8).  
- Relevant high-level and cross-cutting decisions described at L1/L2 (e.g., concurrency, authentication, integration patterns).

Below is a concise list of the chosen resolutions, along with their direct relevance and any Web Portal–specific implementation details.

---

### Key Decisions

1. **Single SPA for Admin & KAM (Ref: Clarification #1)**  
   - A single React + TypeScript codebase is used for both Admin and KAM roles, accessed via role-based UI gating.  
   - Implementation Detail:  
     - The user’s role (Admin or KAM) is checked on login. Each route/component checks this role and conditionally renders features (e.g., user management is shown only to Admin).  
     - Build/deployment remains a unified process, reducing overhead at the current scale.

2. **Poll-Based Data Refresh (Ref: Clarification #2, L2-LLD-IC)**  
   - No real-time events (websockets or SSE) are introduced. The portal relies on manual or periodic polling to refresh leads, quotations, etc.  
   - Implementation Detail:  
     - A “Refresh” button or timed interval triggers REST calls to the backend.  
     - This approach aligns with the ~400–600 concurrent user load and the current backend design.

3. **JWT Storage in Browser (Ref: Clarification #3, L2-LLD-IC)**  
   - Short-lived JWT tokens are stored in the browser (local/session storage).  
   - Implementation Detail:  
     - On login, the token is cached in session storage with strict Content Security Policy (CSP). Users must re-authenticate once the token expires or if inactivity timeout is reached.  
     - Client code includes this token in Authorization headers for subsequent API calls.

4. **Last-Write-Wins with Basic Timestamp Check (Ref: Clarification #4, L2-LLD-IC)**  
   - The Web Portal continues using a last-write-wins concurrency model. A UI-level timestamp check warns users if data changed on the server.  
   - Implementation Detail:  
     - When a record (e.g., lead) is loaded, the last updated timestamp is stored locally.  
     - Before saving, the portal checks if the server-side updated timestamp matches. If not, it warns the user that someone else modified the record.

5. **Full Data Refresh, No Incremental Sync (Ref: Clarification #5, L2-LLD-IC)**  
   - The portal downloads the entire relevant data set (with paging/filters) each time rather than pulling deltas.  
   - Implementation Detail:  
     - For leads/quotations, typical usage involves GET /api/v1/... with offset/limit.  
     - The portal simply re-fetches the list on each refresh, keeping the logic simpler and matching the backend’s current design.

6. **Domain Logic Primarily in Backend (Ref: Clarification #6)**  
   - The WEBPRT UI only enforces basic form validation; complex business rules (lead state transitions, commission calculation, etc.) remain server-side.  
   - Implementation Detail:  
     - The portal checks required fields or basic formatting on forms.  
     - Any advanced or role-based rules (e.g., permissible lead status changes) come from the backend, which the portal respects.

7. **Direct SAS Usage for File Operations (Ref: Clarification #7, L2-LLD-IC)**  
   - The portal requests a one-file, short-lived (e.g., 5–30 min) SAS token from BCKND, then uploads or downloads directly to Azure Blob.  
   - Implementation Detail:  
     - A dedicated endpoint, e.g. GET /documents/sas?docId=..., returns a single-object SAS URL with minimal scope.  
     - Admin or KAM roles can view or upload lead documentation based on their permissions; the backend verifies these roles before issuing the SAS.

8. **Single Bundle with Role-Based Feature Hiding (Ref: Clarification #8)**  
   - All Admin/KAM functionality ships in the same SPA bundle; the UI hides features based on the logged-in user’s role.  
   - Implementation Detail:  
     - Endpoints also enforce server-side RBAC, so even if a KAM user attempts Admin-only calls, the backend denies access.  
     - Maintains simpler builds while preventing accidental feature leakage.

---

### Final Remarks
These decisions keep WEBPRT aligned to the scale of ~400–600 concurrent users with a straightforward deployment pipeline. They balance manageability (single codebase, no partial sync) and security (basic token storage plus strictly enforced backend rules). As usage grows, any of these aspects can be revisited or refined, but for now they address the confirmed requirements with minimal overhead.
```
