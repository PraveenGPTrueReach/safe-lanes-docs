## L3-IB-FE: Component Implementation Backlog for FE

This document provides a comprehensive and granular list of implementation tasks for the SafeLanes Rest Hours Front End (FE), derived from the component’s functional (L3-FRS-FE) and design (L3-LLD-FE) specifications, along with relevant non-functional requirements (L3-NFRS-FE) and key decisions (L3-KD-FE). Each backlog item includes acceptance criteria, references, dependencies, effort estimates, and priority indicators. These items are intended to guide the development team in building, testing, and delivering the FE functionality at a scale consistent with the current project requirements—avoiding overengineering while allowing reasonable future extension.

---

## 1. Authentication & Session Management

### 1.1 Implement JWT-Based Login for Online Mode
• Description: Create a login flow that requests JWT tokens from the SafeLanes Identity Service, storing the token in memory (no local storage).  
• Source Requirements:  
  – L3-FRS-FE §3.1 (User Authentication)  
  – L3-LLD-FE §2.1 (CoreModule, AuthService)  
• Technical Requirements:  
  – Use Angular services and HTTP interceptors to attach the JWT on subsequent calls.  
  – Validate role-based UI toggles (e.g., Admin, Crew, Auditor) in the Angular app.  
• Acceptance Criteria:  
  – Successful authentication returns a valid JWT and populates an in-memory token service.  
  – If credentials are invalid, user receives an error message without storing any partial token.  
  – Role checks properly hide or disable unauthorized UI elements.  
• Dependencies: None (authentication is an initial entry point).  
• Effort Estimate: 3 story points.  
• Priority: High.

### 1.2 Implement Offline Session Key Flow
• Description: Enable users to receive a short-lived offline session key from the vessel’s local server if the vessel is offline, valid for up to 14 days.  
• Source Requirements:  
  – L3-FRS-FE §3.1  
  – L3-KD-FE (Decision #2 regarding local static bundle)  
• Technical Requirements:  
  – Retrieve session key via GET /auth/offline/session.  
  – Store key in memory, require re-login if the browser is closed.  
  – Display offline indicator and manage session key expiry.  
• Acceptance Criteria:  
  – Vessel user can log in offline when the main Identity Service is unreachable.  
  – Session key expires after 14 days or if the user intentionally logs out.  
  – FE prompts user to re-authenticate or seek an Admin override if key is expired.  
• Dependencies: Must integrate with the local Nest.js server’s offline endpoint.  
• Effort Estimate: 5 story points.  
• Priority: High.

---

## 2. Offline Operation & Local Fallback

### 2.1 Local Fallback Bundle & Offline Detection
• Description: Serve the Angular application from a local vessel server when internet connectivity is unavailable.  
• Source Requirements:  
  – L3-FRS-FE §3.2 (Offline Operation & Local Fallback)  
  – L3-LLD-FE §2.2, 2.3 (OfflineStateService, local checks)  
  – L3-KD-FE (Decision #2 local static bundle)  
• Technical Requirements:  
  – Implement an OfflineStateService to detect connectivity to either the office or local Nest.js.  
  – Use a local static build that loads instantly on the vessel if the remote module federation endpoint is unreachable.  
• Acceptance Criteria:  
  – When offline, the FE automatically switches to local endpoints (e.g., POST /resthours on the vessel).  
  – “Offline” indicator is visible in the header or status area.  
  – If the local Nest.js is also unreachable, show an error and do not attempt advanced caching or queues.  
• Dependencies: Integration with local Nest.js hosting; final deployment packaging must include the static bundle.  
• Effort Estimate: 5 story points.  
• Priority: High.

### 2.2 Version Mismatch Warning
• Description: Provide a user-facing warning if the locally bundled FE version does not match the local Nest.js version, but still allow usage unless a severe incompatibility is detected.  
• Source Requirements:  
  – L3-FRS-FE §3.2  
  – L3-KD-FE (Decision #3 re: version checks)  
• Technical Requirements:  
  – Compare a simple semantic version string or build timestamp from both FE and API.  
  – Display a non-blocking alert if minor/patch mismatch; block usage if a major mismatch is detected.  
• Acceptance Criteria:  
  – A mismatch triggers a warning banner or modal; user can dismiss or proceed if mismatch is non-critical.  
  – Major mismatches result in a blocking screen, instructing the user to contact admins.  
• Dependencies: The local Nest.js must provide version info via an endpoint (e.g., /health or /version).  
• Effort Estimate: 3 story points.  
• Priority: Medium.

---

## 3. Daily Rest Hour Logging

### 3.1 Daily Log UI with 48 Half-Hour Blocks
• Description: Build a UI that allows editing of day-level logs in half-hour increments (up to 48 blocks per day).  
• Source Requirements:  
  – L3-FRS-FE §3.3  
  – L3-LLD-FE §2.1 (RestHoursModule), 3.1 (UI flows)  
• Technical Requirements:  
  – Angular form with 48 discrete inputs or an efficient half-hour grid.  
  – Real-time local checks for total hours ≤ 24/day.  
• Acceptance Criteria:  
  – Users can select a date, see existing data, and edit each of the 48 blocks as “Work” or “Rest.”  
  – Submitting calls POST /resthours/daily or PUT /resthours/daily/{id} with correct payload.  
  – If the user is offline, the local server endpoint (/resthours) is called instead of the office.  
• Dependencies: OfflineStateService for environment detection.  
• Effort Estimate: 8 story points.  
• Priority: High.

### 3.2 Predicted Violation Warnings
• Description: Perform lightweight client-side checks for likely violations (e.g., more than 14 hours of work, short rest periods) before saving.  
• Source Requirements:  
  – L3-FRS-FE §3.3 (real-time warnings)  
  – L3-LLD-FE §2.1, 2.2 (client-side rule checker)  
• Technical Requirements:  
  – Simple Angular service to scan half-hour blocks and detect potential rule breaks.  
  – Display non-blocking warnings in the UI; do not prevent submission.  
• Acceptance Criteria:  
  – FE instantly flags any daily total rest below regulatory thresholds.  
  – Submission can still proceed, but user sees a “Projected Violation” notice.  
• Dependencies: None beyond the daily log UI.  
• Effort Estimate: 3 story points.  
• Priority: Medium.

### 3.3 Older Log Edit Restriction (>14 Days)
• Description: Restrict editing logs older than 14 days unless the user has an Admin or Super Admin role.  
• Source Requirements:  
  – L3-FRS-FE §3.3  
  – L3-LLD-FE §5.1 (Day-bound constraints)  
• Technical Requirements:  
  – When retrieving logs older than 14 days, the FE must disable fields for non-Admin roles.  
• Acceptance Criteria:  
  – Attempting to edit an older day triggers a clear “Requires Admin Override” message if user is not Admin.  
• Dependencies: AuthService (must decode roles), date logic from the daily logs component.  
• Effort Estimate: 2 story points.  
• Priority: Medium.

---

## 4. Conflict Resolution UI

### 4.1 Conflict List & Detail Popups
• Description: Present a list of conflicts from GET /conflicts. Allow Admin to open a day-level detail screen.  
• Source Requirements:  
  – L3-FRS-FE §3.4  
  – L3-LLD-FE §2.1 (ConflictResolutionModule)  
• Technical Requirements:  
  – Display vessel vs. office data, highlighting differences.  
  – Provide a “Keep Vessel” or “Keep Office” choice.  
• Acceptance Criteria:  
  – Conflicts relevant to the current vessel or domain are listed.  
  – Clicking on a conflict shows half-hour details in a modal (if needed).  
• Dependencies: Must be online to fetch conflicts from the office back-end.  
• Effort Estimate: 5 story points.  
• Priority: High.

### 4.2 Post Conflict Resolution
• Description: Submit the admin’s choice to POST /conflicts/resolve, capturing “KEEP_VESSEL” or “KEEP_OFFICE.”  
• Source Requirements:  
  – L3-FRS-FE §3.4  
  – L3-KD-FE (Decision #4 re: no silent overwrites)  
• Technical Requirements:  
  – Show confirmation that an immutable audit log entry is created on the back-end.  
  – If offline, conflicts remain read-only; resolution is deferred until connectivity is restored.  
• Acceptance Criteria:  
  – Admin sees a success message after confirming a choice.  
  – Any subsequent re-check ensures the conflict is removed from the list.  
• Dependencies: Must confirm final design with Nest.js conflict resolution endpoints.  
• Effort Estimate: 2 story points.  
• Priority: High.

---

## 5. Planning & Scheduling

### 5.1 Basic Planner UI
• Description: Develop a minimal daily/weekly planner for tasks using GET/POST/PUT /planning.  
• Source Requirements:  
  – L3-FRS-FE §3.5  
  – L3-LLD-FE §2.1 (PlanningModule)  
• Technical Requirements:  
  – Simple table or day-grid listing tasks, start/end times, assigned crew.  
  – No offline file attachments; user sees a prompt if attempts are made offline.  
• Acceptance Criteria:  
  – Admin or Super Admin can add/edit tasks for the vessel.  
  – If offline, tasks are stored or retrieved from the vessel’s local server.  
• Dependencies: Offline detection, role-based UI.  
• Effort Estimate: 5 story points.  
• Priority: Medium.

### 5.2 No Partial Attachment Upload Offline
• Description: Enforce no file attachment logic while offline—prompt user to wait until connectivity is restored.  
• Source Requirements:  
  – L3-FRS-FE §3.5 (Attachment approach)  
  – L3-NFRS-FE (removal of partial upload logic)  
• Technical Requirements:  
  – Disable or hide the file upload field if OfflineStateService indicates offline.  
  – On attempt, show a “Attachments not supported offline” message.  
• Acceptance Criteria:  
  – File upload button or field is read-only or hidden offline.  
  – The user is clearly directed to wait until online to attach files.  
• Dependencies: Shared UI components for file uploads.  
• Effort Estimate: 2 story points.  
• Priority: Low.

---

## 6. Basic Reporting & Export

### 6.1 Export to CSV/PDF
• Description: Provide an interface to download daily or monthly logs in CSV or PDF.  
• Source Requirements:  
  – L3-FRS-FE §3.6  
  – L3-LLD-FE §4 (ReportingModule)  
• Technical Requirements:  
  – Link or button triggers GET /reports/dailyCSV, /reports/monthlyCSV, or dailyPDF.  
  – If offline, do not display the office-based advanced analytics or external endpoints.  
• Acceptance Criteria:  
  – Daily or monthly logs are downloadable as CSV or PDF; user can open or save them locally.  
  – Export request gracefully handles offline mode by disabling or warning.  
• Dependencies: The back-end must implement these endpoints.  
• Effort Estimate: 3 story points.  
• Priority: Medium.

---

## 7. External Auditor (Read-Only) Views

### 7.1 Restricted Auditor UI
• Description: Provide strict read-only pages for external auditor roles.  
• Source Requirements:  
  – L3-FRS-FE §3.7  
  – L3-LLD-FE references to role checks  
• Technical Requirements:  
  – On login, if role = “External,” hide editing capabilities, disable rest-hour form inputs, hide conflict resolution.  
• Acceptance Criteria:  
  – Auditor sees logs, planning, and conflicts in read-only form with no editing controls.  
  – Attempted edits show “No permission” errors.  
• Dependencies: AuthService role checks.  
• Effort Estimate: 2 story points.  
• Priority: Low.

---

## 8. Navigation & Screen Inventory

### 8.1 Main Navigation Structure
• Description: Implement a top-level or sidebar menu reflecting the ~20–25 pages/dialogs defined in L3-FRS-FE §3.8.  
• Source Requirements:  
  – L3-FRS-FE §3.8  
  – L3-LLD-FE §3.1, 3.2 (UI layout)  
• Technical Requirements:  
  – Menu items dynamically shown based on role and environment (Office vs. Vessel).  
  – Angular routing or lazy-loaded modules for each major feature (RestHours, Conflicts, Planning, etc.).  
• Acceptance Criteria:  
  – Vessel environment: shows offline status, daily logs, conflict resolution (if admin), planning.  
  – Office environment: displays advanced analytics or broader reporting links.  
• Dependencies: All modules must be integrated in the main routing tree.  
• Effort Estimate: 3 story points.  
• Priority: Medium.

### 8.2 Responsive Behavior
• Description: Ensure screens are usable on desktop and tablets; minimal breakpoints for 768px widths.  
• Source Requirements:  
  – L3-FRS-FE §2.4 (mentioning office/vessel usage)  
  – L3-NFRS-FE (Responsiveness, Accessibility)  
• Technical Requirements:  
  – Use Angular responsive layouts or CSS media queries.  
  – Validate that key forms and tables do not break or overflow at typical tablet resolution.  
• Acceptance Criteria:  
  – Daily rest-hour grid, conflict resolution screens, and planning UI remain usable on a ~768px wide device.  
  – No critical UI elements become inaccessible at narrower widths.  
• Dependencies: None.  
• Effort Estimate: 3 story points.  
• Priority: Low.

---

## Additional Notes

- Use the references to L3-FRS-FE, L3-LLD-FE, L3-NFRS-FE, and L3-KD-FE for detailed requirements, design choices, and constraints.  
- Each backlog item above can be broken down further into development subtasks (e.g., front-end component creation, service integration, unit tests, e2e tests) as the team tracks progress in a project management tool.  
- Priority indicators reflect immediate business need and recommended development sequence for an MVP. Adjust as needed to align with project planning.  
- Effort estimates use simple “story point” placeholders at a modest scale. Actual values may be refined by the development team.