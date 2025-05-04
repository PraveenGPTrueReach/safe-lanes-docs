## L1-SI: High-Level Inventory of All Screens Across Platforms

We have added “Primary API Dependencies” for each screen, at a high-level, to clarify which backend services or endpoints they rely upon. Note that user management and role assignment remain out of scope for this module and are handled by the SAIL Admin Panel; see “User Creation & RBAC” references in the main SAIL documentation.

---

## 1. Screen Catalog

### 1.1 Vessel Web (Offline-Capable)

These screens are accessed by vessel users, vessel admins, and vessel super admins through a local server fallback when offline. They share a single Angular codebase with the office portal but include minimal detail about offline data handling. Note that “Master” is synonymous with “Vessel Super Admin.” Also note that External (read-only) coverage is configured only for vessel access by default, but it can optionally be extended to the office environment if configured in the SAIL Admin Panel.

#### V1.0a – Vessel Dashboard
- Primary User Roles/Access Levels: Vessel Super Admin, Vessel Admin (with department-limited view), Vessel User (read-only for own data), and optionally External (read-only) if configured. 
- Key Features/Purpose: Provides an at-a-glance view of the vessel’s rest-hour status (violations, NCs, watchlist). Offers filters (e.g., date range, OPA inclusion).  
- Critical Data Elements: Aggregated violation/NC counts, predicted violations, watchlist items, timeline charts.  
- Integration Points: Basic vessel data for the dashboard. Implementation details on synchronization are omitted here.  
- Primary API Dependencies:  
  - GET /api/vessel/dashboard  
  - GET /api/vessel/violations  

##### V1.1a – Violation/NC Detail (Drill-Down)
- Primary User Roles/Access Levels: Same as V1.0a (Vessel Super Admin, Vessel Admin, Vessel User read-only for own data, optionally External if configured).  
- Critical Data Elements: Detailed list of each violation or NC (dates, ranks, references to regulatory codes).  
- Integration Points: Returns to V1.0a for summary view.  
- Primary API Dependencies:  
  - GET /api/vessel/violations?detailLevel=full  
  (Optional query param &type=NC may be used to filter non-conformities.)

#### V2.0a – Vessel Recording Summary
- Platform: Vessel Offline-Capable Web  
- Primary User Roles/Access Levels: Vessel Admin (for department oversight), Vessel Super Admin (for all crew), and possibly Vessel User if configured to see a broader summary.  
- Key Features/Purpose: Provides a summarized overview of recording completeness or daily logs for the vessel (or department). It supplements V2.0b and V2.0c by showing how many days are recorded, flagged with violations, or incomplete.  
- Critical Data Elements: Aggregated status of daily logs per user or department, highlight of any missing submissions.  
- Integration Points: Links to V2.0b or V2.0c for more detailed edits; not all deployments may require a separate V2.0a, but it is referenced in L1-WF.  
- Primary API Dependencies:  
  - GET /api/vesselRecording/summary  
  - GET /api/vesselRecording/status  

#### V2.0b – Vessel Overview (Recording Status)
- Platform: Vessel Offline-Capable Web  
- Primary User Roles/Access Levels: Vessel Admin, Vessel Super Admin, and Vessel User (for personal data).  
- Key Features/Purpose: Shows a list of all crew under given departments, their recording status, violation/NC counts, and predicted issues.  
- Critical Data Elements: Overview of daily logs, crew roster, violation counts, predicted NC data.  
- Integration Points: Links to V2.0c for editing daily logs; triggers sub-screen V2.1b for specific violation details if needed.  
- Primary API Dependencies:  
  - GET /api/vesselRecording/overview  
  - GET /api/vesselRecording/predictedViolations  

##### V2.1b – Violations/NC Detail (Sub-Screen)
- Primary User Roles/Access Levels: Vessel Admin, Vessel Super Admin, Vessel User (read-only for their own data).  
- Critical Data Elements: Day-by-day flagged items, codes, timestamps, relevant commentary.  
- Integration Points: Returns to V2.0b for overview; may link to V2.0c if editing is allowed.  
- Primary API Dependencies:  
  - GET /api/vessel/violations?detailLevel=crew  
  (Optionally &type=NC for non-conformities.)

#### V2.0c – Daily Recording Form
- Platform: Vessel Offline-Capable Web  
- Primary User Roles/Access Levels: Vessel User (own data), Vessel Admin, Vessel Super Admin  
- Key Features/Purpose: Time-based rest/work logging, indicating potential immediate violations or planned tasks overlay.  
- Critical Data Elements: Daily rest/work segments, violation indicators, optional comments.  
- Integration Points: Redirects to conflict resolution if offline edits collide with office updates (high-level only).  
- Primary API Dependencies:  
  - POST /api/vesselRecording/saveDaily  
  - GET /api/vesselRecording/dailyView  

#### V3.0a – Planning (Fixed Tasks)
- Platform: Vessel Offline-Capable Web  
- Primary User Roles/Access Levels: Vessel Admin, Vessel Super Admin  
- Key Features/Purpose: Allows creation/editing of routine planned tasks or schedules (e.g., watch, daywork), marked as “planned” for confirmation.  
- Critical Data Elements: Planned tasks with basic scheduling, typical environment (“at sea” or “in port”).  
- Primary API Dependencies:  
  - GET /api/planning/fixedTasks  
  - POST /api/planning/fixedTasks/save  

#### V3.0b – Planning (Variable Tasks)
- Platform: Vessel Offline-Capable Web  
- Primary User Roles/Access Levels: Vessel Admin, Vessel Super Admin  
- Key Features/Purpose: Logs ad-hoc tasks with start/end times, assigned crew, attachments if needed.  
- Critical Data Elements: Additional tasks beyond fixed schedules, with basic oversight for assigned personnel.  
- Integration Points: Limited to display in the daily recording form for final confirmation, as per L1-WF.  
- Primary API Dependencies:  
  - GET /api/planning/variableTasks  
  - POST /api/planning/variableTasks/upsert  

---

### 1.2 Office Web Portal

Office-side users typically remain online, accessing data from multiple vessels. They can view cross-fleet dashboards, run analytics, and possibly edit or approve vessel entries (if permitted). By default, External (read-only) coverage is limited to vessel data, but an organization may optionally extend it to the office environment if configured in the SAIL Admin Panel.

#### O1.0a – Office Dashboard
- Platform: Office Web Portal  
- Primary User Roles/Access Levels: Office User, Office Admin, Office Super Admin  
- Key Features/Purpose: Gives multi-vessel summary: watchlist, significant NCs, aggregated metrics (violations, predicted NCs), plus filter by fleet or vessel group.  
- Critical Data Elements: High-level fleet metrics, watchlist items, and performance insights.  
- Integration Points: Drills into O1.x sub-screens for deeper analytics, consistent with role-based permissions.  
- Primary API Dependencies:  
  - GET /api/officeDashboard/summary  
  - GET /api/officeDashboard/watchlist  

##### O1.1a / O1.2a / O1.3a / O1.4a / O1.5a / O1.6a – Detailed Metrics/Analytics
- Primary User Roles/Access Levels: Office User (read-only), Office Admin, Office Super Admin (edit or approve if permitted).  
- Critical Data Elements: Specific lists or charts of violations, NCs, overdue responses, etc.  
- Integration Points: Filter by date, rank, vessel group; returns to O1.0a.  
- Primary API Dependencies:  
  - GET /api/officeDashboard/metricsDetails  
  - GET /api/officeDashboard/analytics  

#### O2.0a – Office Recording Summary (By Vessel)
- Platform: Office Web Portal  
- Primary User Roles: Office User, Office Admin, Office Super Admin  
- Key Features/Purpose: Allows the user to see each vessel's rest-hour recording status (e.g., incomplete logs, violation summaries) in a consolidated list.  
- Shows aggregated vessel-level data, including violations, potential NCs, and recording progress.  
- Primary API Dependencies:  
  - GET /api/officeRecording/summary  

#### O2.0b – Crew-Level Overview
- Platform: Office Web Portal  
- Primary User Roles: Office User, Office Admin, Office Super Admin  
- Key Features/Purpose: Lists each crew member’s rest-hour status, incomplete days, violations/NC. Provides “View” or “Edit” links to daily logs.  
- Focuses on high-level crew status, with options to open daily logs or see violation details.  
- Integration Points: Navigates to O2.0c (daily form), sub-screen O2.1a for violation details.  
- Primary API Dependencies:  
  - GET /api/officeRecording/crewOverview  

##### O2.1a – Violation/NC Detail (Sub-Screen)
- Primary User Roles/Access Levels: Office User (read-only by default), Office Admin, Office Super Admin (can edit or acknowledge).  
- Critical Data Elements: Similar to vessel side, itemized NCs or violations.  
- Integration Points: Returns to O2.0b for summary; can link to O2.0c if editing is permitted.  
- Primary API Dependencies:  
  - GET /api/officeDashboard/violations?detailLevel=crew  
  (With optional &type=NC for non-conformities—unifying the resource naming.)

#### O2.0c – Daily Recording Form (Office)
- Platform: Office Web Portal  
- Primary User Roles: Office Admin, Office Super Admin (Office User might have limited or read-only access, as configured)  
- Key Features/Purpose: Allows reviewing or correcting vessel crew rest-hour entries from the office side, typically for historical or compliance corrections.  
- Critical Data Elements: Time-based rest/work logs; office user can add remarks or address flagged violations if permitted.  
- Integration Points: Opens the conflict resolution modal for compliance-critical data conflicts, as needed.  
- Primary API Dependencies:  
  - GET /api/officeRecording/dailyForm  
  - POST /api/officeRecording/saveDaily  

#### O3.0a – Planning (Fixed Tasks, Office)
- Platform: Office Web Portal  
- Primary User Roles: Office User, Office Admin, Office Super Admin  
- Key Features/Purpose: Displays or reviews each vessel’s fixed tasks at an office level. By default, all office roles (including Office Super Admin) have read-only access for these planning tasks, in alignment with L1-WF. Any editing rights from the office side require special configuration or policy override.  
- Critical Data Elements: Planned tasks from multiple vessels, presented for oversight rather than editing.  
- Note: Currently, no separate office-side task approval workflow is provided. If future organizational policy requires it, an additional approval feature could be introduced without significant rework.  
- Primary API Dependencies:  
  - GET /api/planning/fixedTasks?mode=office  

#### O3.0b – Planning (Variable Tasks, Office)
- Platform: Office Web Portal  
- Primary User Roles: Office User, Office Admin, Office Super Admin  
- Key Features/Purpose: View ad-hoc tasks across fleets. By default, read-only for all office roles. Editing is out of scope unless customized in the SAIL Admin Panel.  
- Integration Points: Similar to O3.0a, but only for reviewing variable tasks office-wide.  
- Note: As with O3.0a, there is no built-in office approval mechanism for variable tasks. Organizations can optionally configure overrides if needed.  
- Primary API Dependencies:  
  - GET /api/planning/variableTasks?mode=office  

### Conflict Resolution Modal (General)
- Platform: Appears on both Vessel and Office  
- Primary User Roles: Typically Vessel Admin (only if RBAC specifically permits; disabled by default), Vessel Super Admin, Office Admin, or Office Super Admin  
- Key Features/Purpose: Displayed when the system detects an attempted overwrite of compliance-critical data. Authorized users choose which version to keep.  
- Critical Data Elements: Summarizes the differing data, prompting an override choice.  
- Primary API Dependencies:  
  - POST /api/conflict/resolve (both vessel and office usage)  

### 1.3 Mobile Access (Responsive Web)
The same screens described above are rendered via responsive design for mobile devices (phones, tablets). Each Vessel or Office user can access identical content and workflows through a mobile browser. No distinct endpoints are defined for mobile usage, and no separate native mobile app is produced. However, each screen’s layout adapts to smaller form factors while preserving functionality.

#### 1.3.1 Mobile-Specific Notes per Screen
- V1.0a – Vessel Dashboard: Responsive layout ensures watchlist and violation/NC metrics are scrollable on phone screens.  
- V2.0c – Daily Recording Form: Tap-friendly half-hour blocks for work/rest selection.  
- O1.0a – Office Dashboard: All metrics and charts remain view-only with pinch-zoom gestures.  
- Any conflict resolution modal is likewise accessible via responsive design.  
If offline usage is required on a mobile device at the vessel, the local fallback approach remains the same—the user’s mobile device connects to the vessel’s local server over Wi-Fi, subject to security policy.

---

## 2. Screen Inventory & Flow Matrix

The table below summarizes navigation flows among these key screens. Login, logout, and user profile screens are still handled by the main SAIL platform. No separate compliance reporting screen is provided at this time. “V2.0a” is now included per L1-WF references.

| Screen Name               | Platform                                  | Accessible From                                   | Next Possible Screens                                                            | Dependencies                                                                          |
|---------------------------|-------------------------------------------|---------------------------------------------------|----------------------------------------------------------------------------------|--------------------------------------------------------------------------------------|
| V2.0a                    | Vessel (Offline-Capable)                  | SAIL App Menu or side nav                        | V2.0b, V2.0c (for deeper edits)                                                 | Recording summary data for departments or entire vessel                              |
| V1.0a                     | Vessel (Offline-Capable)                  | SAIL App Main Menu                                | V1.1a, V2.0b                                                                     | Aggregated metrics (watchlist, violations, etc.)                                     |
| V1.1a (Drill-Down)       | Vessel (Offline)                          | V1.0a (click metrics)                             | Back to V1.0a                                                                    | Detailed list of violations/NC, user roles: Admin or Master, user read-only           |
| V2.0b                     | Vessel (Offline)                          | V2.0a or V1.0a side nav                           | V2.0c (Edit logs), V2.1b (Detailed violation)                                    | Crew list, daily or weekly overview; triggers sub-screen for detail                   |
| V2.1b                     | Vessel (Offline)                          | V2.0b (violation click)                           | Return to V2.0b                                                                  | Day-by-day or itemized flagged issues, read-only for standard users                  |
| V2.0c                     | Vessel (Offline)                          | V2.0b (Edit icon)                                 | Return to V2.0b, conflict resolution modal as needed                             | Time-based daily logs, potential violation warnings                                  |
| V3.0a                     | Vessel (Offline)                          | Main nav “Plan” toggle (Fixed)                    | Return to main menu                                                              | Creation/editing of routine tasks                                                    |
| V3.0b                     | Vessel (Offline)                          | Main nav “Plan” toggle (Variable)                 | Return to main menu                                                              | Additional tasks with flexible scheduling                                            |
| O1.0a                     | Office Web Portal                         | SAIL App Main Menu (Office)                       | O1.x sub-screens, O2.0a                                                         | Multi-vessel watchlist, analytics                                                   |
| O1.x                      | Office (portal)                           | O1.0a (click metric)                              | Return to O1.0a                                                                  | Expanded analytics; includes sub-screens O1.1a - O1.6a for details                   |
| O2.0a                     | Office (portal)                           | O1.0a or side nav                                 | O2.0b (vessel details)                                                           | Summaries of vessel rest-hour records                                               |
| O2.0b                     | Office (portal)                           | O2.0a (View/Edit)                                 | O2.0c, O2.1a (detailed violation/NC)                                             | Crew-level rest-hour overview for a vessel                                          |
| O2.1a                     | Office (portal)                           | O2.0b (violation/NC)                              | Return to O2.0b                                                                  | Displays flagged items, possible editing if admin/super admin                       |
| O2.0c                     | Office (portal)                           | O2.0b (Edit)                                      | Return to O2.0b, conflict resolution if needed                                   | Daily form with office corrections                                                  |
| O3.0a                     | Office (portal)                           | Main nav “Plan” (fixed)                           | Back to O1.0a/O2.0a                                                              | View-only oversight of vessel tasks (Office can’t edit by default)       |
| O3.0b                     | Office (portal)                           | Main nav “Plan” (variable)                        | Back to O1.0a/O2.0a                                                              | View-only oversight of ad-hoc tasks (Office can’t edit by default)       |
| Conflict Resolution Modal | Vessel & Office Web                       | Auto-invoked within V2.0c or O2.0c                | Dialog closes back to previous screen                                            | Allows authorized override of conflicting data changes (Vessel Admin only if RBAC allows) |

For new user creation or role changes (e.g., promoting Vessel User to Vessel Admin), see the SAIL Admin Panel. This solution intentionally defers all user management and role assignment tasks to the existing SafeLanes platform tools.

At this scale, no separate compliance reporting screen is provided. Any future expansions to generate official or aggregated compliance reports may be introduced in additional modules or screens.

This concludes the revised L1-SI document with adjustments to Vessel User access on the dashboard and clarifications around office task approval (or lack thereof), ensuring consistency for actual user workflows at the stated scale.