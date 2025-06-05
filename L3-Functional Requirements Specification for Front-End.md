## L3-FRS-FE: Functional Requirements Specification (FRS) for FE

---

### 1. Introduction

The front-end (FE) of the SafeLanes Rest Hours solution is an Angular 16 microfrontend that runs in below environment:

1. The Vessel environment will be serving a locally bundled version, integrating with the "Sail App" shell (via Module Federation or a minimal fallback wrapper).

Because maritime vessels can be offline for up to 14 days, the FE must ensure essential functionality remains available locally, and no file uploads or advanced caching while offline. 

This specification confines itself to front-end functionality and does not duplicate back-end requirements. All references to endpoints and data are solely from the FE's perspective, describing how the interface calls or receives data from the (already designed) back-end REST APIs.

---

### 2. Scope

1. Provide an Angular 18.2.0 microfrontend that the vessel crew and office users can access.  
2. Support day-level rest-hour entries, compliance checking, conflict resolution, minimal planning tasks, and read-only dashboards for external auditors.  
3. Integrate role-based authentication via SafeLanes JWT tokens   
4. Offer a user experience that is sufficiently responsive to desktop and tablet/laptop screen sizes, and optionally down to mobile widths for basic functionality.

The front-end does not directly integrate with external third-party services, as all certificate management and auditor-related interactions occur on the back-end.

---

### 3. Functional Requirements

#### 3.1 User Authentication & Role Management

(Implements L1-FRS §§2.1, 2.2)

1. The FE must provide a login flow that:  
   - Accepts JWT tokens from the respective locally available SafeLanes Identity Service.  
   - Requires the user to re-login if the browser is closed (no persistent storage of session keys).  
2. The FE must display role-appropriate functionality and menu items (e.g., Vessel User, Vessel Admin, Vessel Super Admin, Office Admin, etc.).  
3. Endpoints and Methods:  
   - POST /auth/login (when online): FE obtains JWT token.  
   - 

#### 3.2 Offline Operation 

(Implements L1-FRS §§2.2, 2.5, 2.7 – referencing partial compliance-check logic offline for the vessel environment)

1. The FE must load a bundled static version of the Angular app from the local Nest.js server on the vessel   
2. The FE shall:  
   - Continue reading/writing data via local vessel endpoints (e.g., POST /resthours, GET /resthours).  
3. The FE must show an alert if a version mismatch is detected (e.g., older front-end vs. updated back-end) but still allow usage while the vessel is disconnected from the office
4. The FE should not store large data sets or file attachments in the browser and should not do any function when not connected to the backend on vessel's local backend. All data is saved directly to the vessel's local server.  
5. Endpoints and Methods:  
   - All normal REST calls (GET, POST, PUT) to the local Nest.js server (e.g., /resthours, /planning) remain reachable on the ship's LAN.

#### 3.3 Daily Rest Hour Logging

(Implements L1-FRS §§2.3, 2.5)

1. The FE must provide a user interface for entering up to 48 half-hour blocks per day, capturing "work" or "rest" statuses.  
2. When the user edits/saves daily logs, the FE must call the vessel's back-end, e.g.:  
   - POST /resthours/daily  (for a new entry)  
   - PUT /resthours/daily/{id} (to update an existing day's record)  
3. The FE shall retrieve existing logs for a specific date or date range:  
   - GET /resthours/daily?date=YYYY-MM-DD  
4. After saving, the FE should receive violation or warning flags from the back-end if thresholds are exceeded. Final authoritative results still come from the back-end.  
5. The FE must allow editing any daily-resthours-log from past days if the user's role permits.  
6. Endpoints and Methods:  
   - GET /resthours/daily?date=  
   - POST /resthours/daily  
   - PUT /resthours/daily/{recordId}

#### 3.4 Planning and Scheduling

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

#### 3.5 External Auditor (Read-Only) Views

(Implements L1-FRS §§2.1)

1. The FE must provide a restricted read-only interface for users with the External (auditor) role.  
2. The FE must hide or disable all editing capabilities for these external accounts.  
3. Endpoints and Methods:  
   - Typically the same GET endpoints as with other roles, but the UI hides any write or edit actions.

#### 3.6 Screen Inventory & Navigation

(Relates to L1-FRS overall scope)

1. The FE must present approximately 20–25 distinct pages or dialogs spanning:  
   - User login / role-check interface.  
   - Daily rest-hour logs, including add/edit screens.  
   - Planning & scheduling pages for Admins.  
   - Settings / status page for vessel environment.  
   - Optional read-only pages for external auditors (restricted menu).  
2. Users navigate these screens via a main menu or top-level navigation bar. Vessel vs. Office contexts are distinguished by role-based or environment-based logic.

---

### 4. Component Interfaces

1. **Primary Data Exchange**  
     
   - All data calls occur over REST/HTTPS are pointed to the local Nest.js back-end, depending on environment or user role.

   

2. **Authentication & Session State**  
     
   - The FE obtains a JWT from the respective local BE SafeLanes Identity Service (POST /auth/login).  
   - the user must re-login if the browser is closed.  
     

     
3. **Integration with Sail App**  
     
   - By default, the FE integrates with the existing SafeLanes "Sail App" shell via Module Federation or a fallback wrapper (iframe/web component).  
   - A mild runtime check attempts direct Federation if the host is Angular 18.2.0; otherwise, it uses the fallback component.

---

### 5. Data Requirements

1. **Daily Rest-Hour Records**  
     
   - The FE sends and receives a structured payload of 48 half-hour blocks, plus a date, user ID, total rest/work hours, and violation flags.  
   - For each half-hour block, the FE uses enumerated values ("Work," "Rest," "Planned," etc.).

   

2. **Conflict Overwrites**  
     
   - User's inputs are not needed for conflict resolution. Simple last write wins rule is followed.

3. **Planning Data**  
     
   - Each planning entry includes: name/description, start/end times, assigned crew IDs, and optional (but currently disabled offline) attachments up to 5 MB.  
   - When the frontend on a vessel is not connected to a local vessel server, attachments are not supported. 

   

4. **Local Storage**  
     
   - The FE does not store significant data in IndexedDB or localStorage. All rest-hour and planning data is stored in the vessel's local MySQL (accessed via the Nest.js API) or the office MySQL.

   

5. **Errors & Edge Cases**  
     
   - If the LAN connection to the vessel server is lost, the FE notifies the user. The user must retry once reconnected (See §3.2).

---

### 6. Validation Rules

1. **Real-Time Violation Checks**  
     
   - Actual authoritative checks come from the back-end.  
   - The front-end must not block saving if the user has a predicted violation; it only warns.

   

2. **Day-Bound Constraints**  
     
   - Each FE entry form must restrict total "worked" hours to ≤24 for a single calendar day (Implements L1-FRS §§5.1 in spirit). 

   

3. **Conflict Detection**  
     
   - The FE must rely on updatedAt timestamps from the back-end. 

   

4. **Upload Restrictions**  
     
   - The FE must disable file attachments while offline(offline means FE is not connected to local server) , preventing any attempt to store or queue large files on the client side (no partial or re-try logic is implemented).

---

### 7. Summary & Future Considerations

This front-end is designed for moderate scale (~30 vessel users, up to ~100 office users). It prioritizes clarity and offline viability over advanced offline caching or multi-day data entry forms.  
Once the hosting "Sail App" is officially upgraded to Angular 18.2.0, the fallback wrapper approach is removed, simplifying module federation integration.  
If support for advanced offline file attachments or multi-day bulk editing is requested in the future, the code can be extended. For now, the FE meets the documented user flows,, and day-level data entry.

#### 7.1 Accessibility

Given the system's usage scenarios, the FE should adhere to basic accessibility guidelines (e.g., WCAG 2.1 AA) as much as feasible for the current scale. Additional accessibility enhancements can be introduced later without overcomplicating the initial scope.

#### 7.2 Alignment with L1-HLD

At the time of this writing, planning & scheduling features are detailed in L1-FRS §2.4 but are not explicitly covered in the L1-HLD. An update to the L1-HLD is recommended to reflect these functionalities if they remain in scope.  
Additionally, a formal change request is being raised to update L1-FRS regarding offline attachment handling, removing or deferring partial or re-try logic to maintain consistency with this front-end specification.

---

### 8. References

- [L1-FRS Document (High-Level Requirements)](http://../L1-FRS) – for overall system and business requirements.  
- [L1-HLD Document](http://../L1-HLD) – for high-level architecture context (to be updated for planning/scheduling).  
- [L2-LLD-IC Document](http://../L2-LLD-IC) – for inter-component interaction details (endpoints, conflict workflows).  
- [L3-KD-FE Document](http://../L3-KD-FE) – for key decisions specific to the FE.

---

