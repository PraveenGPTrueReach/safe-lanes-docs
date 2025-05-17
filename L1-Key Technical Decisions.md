## L1-KD: Key Technical Decisions

### Decision 1: Angular 16 Upgrade and Temporary Fallback
- We will deploy the new microfrontend within a temporary wrapper to avoid delaying release while the existing host application remains on an older Angular version.
- A strict timeline (6–12 months) is defined to upgrade the host to Angular 16 and remove the fallback, ensuring eventual unification on a modern Module Federation approach.

### Decision 2: Single-Record-Per-Day Schema
- Each crew member’s daily time logs will be stored as one record per day, subdivided into 48 half-hour blocks.
- Concurrency is handled at the day level, relying on minimal multi-user edits and the existing conflict-resolution approach to prevent data loss.
- If multiple offline edits occur on the same day for rest-hour compliance data, the system will flag these overlapping changes for mandatory Admin or Super Admin review before finalizing the sync. For non-critical fields within the same daily record, the last-write-wins approach still applies.
- This design meets current scaling needs while remaining simpler to implement and maintain than a more granular schema.

### Decision 3: Local Microfrontend Fallback for Offline Vessels
- Each vessel retains a locally bundled copy of the microfrontend for use when remote federation endpoints are unreachable.
- To mitigate critical version mismatches, we enforce semantic version checks between the bundled UI and the back-end API, blocking usage if versions are incompatible.
- Vessels can update the bundled application during a stable connection, ensuring essential functionality remains available offline without blocking critical tasks.

### Decision 4: Conflict Resolution Policy
- For non-critical data fields, last-write-wins applies automatically.
- All compliance-critical data conflicts require an Admin or Super Admin decision (to keep vessel or office data) prior to overwriting.
- This includes scenarios where multiple half-hour edits for the same day conflict. Any such partial-day overlap is escalated for manual review.
- Overwritten data is logged in a protected audit record, ensuring no silent overwrites of compliance-critical fields.
- This policy aligns with the rest-hour compliance requirements to prevent loss or accidental overwrite of critical data, while maintaining a simplified last-write-wins model for non-critical information.

### Decision 5: 14-Day Offline Session Keys
- Users can remain authenticated offline for up to 14 consecutive days with a short-lived session key.
- A local override by the Vessel Master (Super Admin) or Admin is required if the vessel stays offline beyond that period.
- Automatic reminders will prompt for re-auth, and all overrides or renewals are logged for auditing.

### Decision 6: Local Database Management on Vessels
- Each vessel runs a local MySQL database, storing data offline with daily or on-demand backups.
- In the event of unrecoverable corruption, a documented procedure will attempt a differential or point-in-time restore that merges any unsynced local changes, preserving offline-entered rest-hour data and preventing silent overwrites. This approach avoids permanent loss of compliance-critical logs and maintains audit traceability.
- This approach preserves offline-first operations with minimal extra overhead at the current project scale.
