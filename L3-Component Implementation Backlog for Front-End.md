## L3-IB-FE: Component Implementation Backlog for FE

This document provides a comprehensive and granular list of implementation tasks for the SafeLanes Rest Hours Front End (FE), derived from the component’s functional (L3-FRS-FE) and design (L3-LLD-FE) specifications, along with relevant non-functional requirements (L3-NFRS-FE) and key decisions (L3-KD-FE). Each backlog item includes acceptance criteria, references, dependencies, effort estimates, and priority indicators. These items are intended to guide the development team in building, testing, and delivering the FE functionality at a scale consistent with the current project requirements—avoiding overengineering while allowing reasonable future extension.

---

### 1\. Authentication & Session Management

#### 1.1 Implement JWT-Based Login for all scenarios

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

---

### 2\. Offline Operation & Local Fallback

#### 2.1 Local Fallback Bundle & Offline Detection

• Description: Serve the Angular application from a local vessel server.  
• Source Requirements:  
– L3-FRS-FE §3.2 (Offline Operation & Local Fallback)  
– L3-LLD-FE §2.2, 2.3 (OfflineStateService, local checks)  
– L3-KD-FE (Decision \#2 local static bundle)

• Dependencies: Integration with local Nest.js hosting; final deployment packaging must include the static bundle.  
• Effort Estimate: 5 story points.  
• Priority: High.

#### 2.2 Version Mismatch Warning

• Description: Provide a user-facing warning if the locally bundled FE version does not match the local Nest.js version, but still allow usage unless a severe incompatibility is detected.  
• Source Requirements:  
– L3-FRS-FE §3.2  
– L3-KD-FE (Decision \#3 re: version checks)  
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

### 3\. Daily Rest Hour Logging

#### 3.1 Daily Log UI with 48 Half-Hour Blocks

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

• Dependencies: OfflineStateService for environment detection.  
• Effort Estimate: 8 story points.  
• Priority: High.

#### 3.2 Predicted Violation Warnings

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

---

### 4\. Planning & Scheduling

#### 4.1 Basic Planner UI

• Description: Develop a minimal daily/weekly planner for tasks using GET/POST/PUT /planning.  
• Source Requirements:  
– L3-FRS-FE §3.5  
– L3-LLD-FE §2.1 (PlanningModule)  
• Technical Requirements:  
– Simple table or day-grid listing tasks, start/end times, assigned crew.

• Acceptance Criteria:  
– Admin or Super Admin can add/edit tasks for the vessel.  
– tasks are stored or retrieved from the vessel’s local server.  
• Dependencies: Offline detection, role-based UI.  
• Effort Estimate: 5 story points.  
• Priority: Medium.

#### 4.2 No Partial Attachment Upload Offline

• Description: Enforce no file attachment logic while offline—prompt user to wait until connectivity is restored. (offline means the FE is not connected to the local server)  
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

---

### 5\. External Auditor (Read-Only) Views

#### 5.1 Restricted Auditor UI

• Description: Provide strict read-only pages for external auditor roles.  
• Source Requirements:  
– L3-FRS-FE §3.7  
– L3-LLD-FE references to role checks  
• Technical Requirements:  
– On login, if role \= “External,” hide editing capabilities, disable rest-hour form inputs, hide conflict resolution.  
• Acceptance Criteria:  
– Auditor sees logs, planning, in read-only form with no editing controls.

• Dependencies: AuthService role checks.  
• Effort Estimate: 2 story points.  
• Priority: Low.

---

### 6\. Navigation & Screen Inventory

#### 6.1 Main Navigation Structure

• Description: Implement a top-level or sidebar menu reflecting the \~20–25 pages/dialogs defined in L3-FRS-FE §3.8.  
• Source Requirements:  
– L3-FRS-FE §3.8  
– L3-LLD-FE §3.1, 3.2 (UI layout)  
• Technical Requirements:  
– Menu items dynamically shown based on role and environment (Office vs. Vessel).  
– Angular routing or lazy-loaded modules for each major feature (RestHours, Conflicts, Planning, etc.).  
• Acceptance Criteria:  
– Vessel environment: shows offline status, daily logs, conflict resolution , planning.  
– Office environment: displays advanced analytics or broader reporting links.  
• Dependencies: All modules must be integrated in the main routing tree.  
• Effort Estimate: 3 story points.  
• Priority: Medium.

#### 6.2 Responsive Behavior

• Description: Ensure screens are usable on desktop and tablets; minimal breakpoints for 768px widths.  
• Source Requirements:  
– L3-FRS-FE §2.4 (mentioning office/vessel usage)  
– L3-NFRS-FE (Responsiveness, Accessibility)  
• Technical Requirements:  
– Use Angular responsive layouts or CSS media queries.  
– Validate that key forms and tables do not break or overflow at typical tablet resolution.  
• Acceptance Criteria:  
– Daily rest-hour grid, and planning UI remain usable on a \~768px wide device.  
– No critical UI elements become inaccessible at narrower widths.  
• Dependencies: None.  
• Effort Estimate: 3 story points.  
• Priority: Low.

---

### Additional Notes

- Use the references to L3-FRS-FE, L3-LLD-FE, L3-NFRS-FE, and L3-KD-FE for detailed requirements, design choices, and constraints.  
- Each backlog item above can be broken down further into development subtasks (e.g., front-end component creation, service integration, unit tests, e2e tests) as the team tracks progress in a project management tool.  
- Priority indicators reflect immediate business need and recommended development sequence for an MVP. Adjust as needed to align with project planning.  
- Effort estimates use simple “story point” placeholders at a modest scale. Actual values may be refined by the development team.

