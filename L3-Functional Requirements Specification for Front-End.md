
## L3-FRS-FE: Functional Requirements Specification (FRS) for FE

---

## 1. Introduction

The front-end (FE) of the SafeLanes Rest Hours solution is an Angular 16 microfrontend that runs in two primary environments:  
1. The Vessel environment, often offline, serving a locally bundled version.  
2. The Office environment, integrating with the “Sail App” shell (via Module Federation or a minimal fallback wrapper).

Because maritime vessels can be offline for up to 14 days, the FE must ensure essential functionality remains available locally, with a secure approach to offline session keys in memory and no file uploads or advanced caching while offline. Once connectivity is restored, the FE synchronizes data with the Office system via the Nest.js back-end APIs.

This specification confines itself to front-end functionality and does not duplicate back-end requirements. All references to endpoints and data are solely from the FE’s perspective, describing how the interface calls or receives data from the (already designed) back-end REST APIs.

---
## 2. Scope

1. Provide an Angular 16 microfrontend that the vessel crew and office users can access.  
2. Support day-level rest-hour entries, compliance checking, conflict resolution, minimal planning tasks, and read-only dashboards for external auditors.  
3. Integrate role-based authentication via SafeLanes JWT tokens (when online) and short-lived offline session keys (when offline).  
4. Offer a user experience that is sufficiently responsive to desktop and tablet/laptop screen sizes, and optionally down to mobile widths for basic functionality.

The front-end does not directly integrate with external third-party services, as all certificate management and auditor-related interactions occur on the back-end.

---
## 3. Functional Requirements

### 3.1 User Authentication & Role Management
(Implements L1-FRS §§2.1, 2.2)

1. The FE must provide a login flow that:  
   - Accepts JWT tokens from the SafeLanes Identity Service when online.  
   - Accepts short-lived offline session keys (valid up to 14 days) from the vessel’s local server when offline.  
   - Requires the user to re-login if the browser is closed (no persistent storage of session keys).  
2. The FE must display role-appropriate functionality and menu items (e.g., Vessel User, Vessel Admin, Vessel Super Admin, Office Admin, etc.).  
3. The FE must hide or disable controls for editing older logs (beyond 14 days) if the user is not an Admin role.  
4. For re-validating tokens once connectivity is back, the FE shall auto-initiate or prompt re-authentication, calling the identity endpoint on the back-end as needed.  
5. Endpoints and Methods:  
   - POST /auth/login (when online): FE obtains JWT token.  
   - GET /auth/offline/session (when offline): FE requests an offline session key from the local vessel back-end, if permissible. This endpoint is newly introduced. Future updates to L2-LLD-IC will formalize its usage for offline operation.

### 3.2 Offline Operation & Local Fallback
(Implements L1-FRS §§2.2, 2.5, 2.7 – referencing partial compliance-check logic offline for the vessel environment)

1. The FE must load a bundled static version of the Angular app from the local Nest.js server on the vessel when internet connectivity is unavailable.  
2. The FE shall detect offline states and:  
   - Continue reading/writing data via local vessel endpoints (e.g., POST /resthours, GET /resthours).  
   - Display an “Offline” indicator to the user and suppress features that strictly require office connectivity (e.g., real-time office analytics).  
3. The FE must show an alert if a version mismatch is detected (e.g., older front-end vs. updated back-end) but still allow usage while offline.  
4. The FE should not store large data sets or file attachments in the browser for offline usage. All data is saved directly to the vessel’s local server.  
5. Endpoints and Methods:  
   - All normal REST calls (GET, POST, PUT) to the local Nest.js server (e.g., /resthours, /planning) remain reachable on the ship’s LAN.

### 3.3 Daily Rest Hour Logging
(Implements L1-FRS §§2.3, 2.5)

1. The FE must provide a user interface for entering up to 48 half-hour blocks per day, capturing “work” or “rest” statuses.  
2. When the user edits/saves daily logs, the FE must call the vessel’s back-end, e.g.:  
   - POST /resthours/daily  (for a new entry)  
   - PUT /resthours/daily/{id} (to update an existing day’s record)  
3. The FE shall retrieve existing logs for a specific date or date range:  
   - GET /resthours/daily?date=YYYY-MM-DD  
4. After saving, the FE should receive violation or warning flags from the back-end if thresholds are exceeded. For immediate feedback, a simplified client-side rule checker is used to display predicted violations in real time. Final authoritative results still come from the back-end.  
5. The FE must allow editing recent past days (≤14 days) if the user’s role permits. For older data (>14 days), the FE must inform the user that an Admin override is needed (the override process is handled by the back-end).  
6. Endpoints and Methods:  
   - GET /resthours/daily?date=  
   - POST /resthours/daily  
   - PUT /resthours/daily/{recordId}

### 3.4 Conflict Resolution UI
(Implements L1-FRS §§2.6, 2.7)

1. The FE must display a concise interface for Admins or Super Admins to review conflicts detected by the back-end, showing only a day-level “which side to keep” choice (vessel vs. office).  
2. The FE must optionally allow an Admin to view the half-hour details in a popup but ultimately enforce a single day-level resolution.  
3. The FE must call dedicated conflict endpoints to fetch pending overlaps and submit resolutions:  
   - GET /conflicts (list all conflicts relevant to the user’s vessel or domain).  
   - POST /conflicts/resolve (with a body specifying conflictId and “KEEP_VESSEL” or “KEEP_OFFICE”).  
4. When the Admin chooses which version to keep, the FE must display a confirmation message indicating that an immutable audit log is recorded.  
5. Endpoints and Methods:  
   - GET /conflicts  
   - POST /conflicts/resolve

### 3.5 Planning and Scheduling
(Implements L1-FRS §§2.4, 2.7)

1. The FE must allow Vessel Admin or Super Admin to create or update scheduled tasks for selected crew. A minimal daily or weekly planner is sufficient.  
2. The FE must call the vessel end-point to save tasks (offline environment) and also retrieve them for display:  
   - GET /planning (retrieve tasks for a date range)  
   - POST /planning (create new tasks)  
   - PUT /planning/{taskId} (update tasks)  
3. Attachments are excluded from offline uploads in the current scope. The FE does not implement partial or re-try upload logic to keep complexity low for this project scale. If a user attempts to upload a file while offline, the system will prompt them to wait until connectivity is restored.    
A formal change request is in progress to remove or revise partial or re-try logic from the L1-FRS, ensuring alignment with this front-end design that disables offline attachments for now.
4. Endpoints and Methods:  
   - GET /planning?startDate=…&endDate=…  
   - POST /planning  
   - PUT /planning/{taskId}

### 3.6 Basic Reporting & Export
(Implements L1-FRS §§2.9)

1. The FE must provide a simple interface to export daily or monthly logs to CSV or PDF. Users can trigger an export for a single vessel or for the user’s own data.  
2. The FE must call the appropriate back-end route to generate or retrieve the file; if needed, the file is then downloaded or displayed.  
3. Endpoints and Methods (example):  
   - GET /reports/dailyCSV?date=YYYY-MM-DD  
   - GET /reports/monthlyCSV?month=YYYY-MM  
   - GET /reports/dailyPDF?date=YYYY-MM-DD  

### 3.7 External Auditor (Read-Only) Views
(Implements L1-FRS §§2.1)

1. The FE must provide a restricted read-only interface for users with the External (auditor) role.  
2. The FE must hide or disable all editing capabilities for these external accounts.  
3. Endpoints and Methods:  
   - Typically the same GET endpoints as with other roles, but the UI hides any write or edit actions.

### 3.8 Screen Inventory & Navigation
(Relates to L1-FRS overall scope)

1. The FE must present approximately 20–25 distinct pages or dialogs spanning:  
   - User login / role-check interface.  
   - Daily rest-hour logs, including add/edit screens.  
   - Conflict resolution list & detail popups (Admin-only).  
   - Planning & scheduling pages for Admins.  
   - Basic reporting and analytics dashboards (Office side).  
   - Settings / local fallback status page for vessel environment.  
   - Optional read-only pages for external auditors (restricted menu).  
2. Users navigate these screens via a main menu or top-level navigation bar. Vessel vs. Office contexts are distinguished by role-based or environment-based logic.

---
## 4. Component Interfaces

1. **Primary Data Exchange**  
   - All data calls occur over REST/HTTPS to either the local vessel Nest.js back-end or the office Nest.js back-end, depending on environment or user role.

2. **Authentication & Session State**  
   - The FE obtains a JWT from the SafeLanes Identity Service if online (POST /auth/login).  
   - When offline, the FE requests a short-lived session key (GET /auth/offline/session) from the local vessel server. This key is kept in memory only; the user must re-login if the browser is closed.

3. **Conflict Resolution**  
   - The FE interacts with the Office back-end for final conflict resolution steps. The vessel environment alone displays known conflicts in read-only mode until connectivity is restored.

4. **Integration with Sail App**  
   - By default, the FE integrates with the existing SafeLanes “Sail App” shell via Module Federation or a fallback wrapper (iframe/web component).  
   - A mild runtime check attempts direct Federation if the host is Angular 16; otherwise, it uses the fallback component.

---
## 5. Data Requirements

1. **Daily Rest-Hour Records**  
   - The FE sends and receives a structured payload of 48 half-hour blocks, plus a date, user ID, total rest/work hours, and violation flags.  
   - For each half-hour block, the FE uses enumerated values (“Work,” “Rest,” “Planned,” etc.).

2. **Conflict Overwrites**  
   - The FE only displays read-only prior values if the user chooses to see them in a conflict details popup.  
   - The FE never modifies the immutable audit log; it merely shows past and proposed data.

3. **Planning Data**  
   - Each planning entry includes: name/description, start/end times, assigned crew IDs, and optional (but currently disabled offline) attachments up to 5 MB.  
   - For this release, partial or re-try logic to handle offline attachments has been removed in favor of a simpler approach: offline attachments are not supported. Users must be online to upload files.

4. **Local Storage**  
   - The FE does not store significant data in IndexedDB or localStorage. All rest-hour and planning data is stored in the vessel’s local MySQL (accessed via the Nest.js API) or the office MySQL.

5. **Errors & Edge Cases**  
   - If the LAN connection to the vessel server is lost, the FE notifies the user. The user must retry once reconnected (See §3.2).

---
## 6. Validation Rules

1. **Real-Time Violation Checks**  
   - The FE implements a lightweight rule set for immediate feedback (Implements L1-FRS 2.5). Actual authoritative checks come from the back-end.  
   - The front-end must not block saving if the user has a predicted violation; it only warns.

2. **Day-Bound Constraints**  
   - Each FE entry form must restrict total “worked” hours to ≤24 for a single calendar day (Implements L1-FRS §§5.1 in spirit). Overlapping blocks or excess hours trigger warnings.

3. **Conflict Detection**  
   - The FE must rely on updatedAt timestamps from the back-end. If a conflict is returned (e.g., 409 or “CONFLICT” in the response), the FE must route Admin users to the conflict resolution UI. The FE itself does not attempt partial merges.

4. **Older Log Edits (>14 Days)**  
   - The FE must visually warn standard users that an Admin override is required. If the user is not an Admin, the edit form is disabled.

5. **Upload Restrictions**  
   - The FE must disable file attachments while offline, preventing any attempt to store or queue large files on the client side (no partial or re-try logic is implemented).

6. **Session Key Validity**  
   - FE must track the offline session key’s expiration in memory; if it expires, the user is prompted to request an Admin override or sign in again (Implements L1-FRS 2.2).

---
## 7. Summary & Future Considerations

This front-end is designed for moderate scale (~30 vessel users, up to ~100 office users). It prioritizes clarity and offline viability over advanced offline caching or multi-day data entry forms.  
Once the hosting “Sail App” is officially upgraded to Angular 16, the fallback wrapper approach is removed, simplifying module federation integration.  
If support for advanced offline file attachments or multi-day bulk editing is requested in the future, the code can be extended. For now, the FE meets the documented user flows, conflict resolution approach, and day-level data entry.

### 7.1 Accessibility
Given the system’s usage scenarios, the FE should adhere to basic accessibility guidelines (e.g., WCAG 2.1 AA) as much as feasible for the current scale. Additional accessibility enhancements can be introduced later without overcomplicating the initial scope.

### 7.2 Alignment with L1-HLD
At the time of this writing, planning & scheduling features are detailed in L1-FRS §2.4 but are not explicitly covered in the L1-HLD. An update to the L1-HLD is recommended to reflect these functionalities if they remain in scope.  
Additionally, a formal change request is being raised to update L1-FRS regarding offline attachment handling, removing or deferring partial or re-try logic to maintain consistency with this front-end specification.

---
## 8. References

- [L1-FRS Document (High-Level Requirements)](../L1-FRS) – for overall system and business requirements.  
- [L1-HLD Document](../L1-HLD) – for high-level architecture context (to be updated for planning/scheduling).  
- [L2-LLD-IC Document](../L2-LLD-IC) – for inter-component interaction details (endpoints, conflict workflows).  
- [L3-KD-FE Document](../L3-KD-FE) – for key decisions specific to the FE.  

--------------------------------------------------------------------------------

