## L3-KD-BE: Component-Specific Key Decisions for BE

Below are the final key decisions for the Back-End (Nest.js + MySQL) component. Each decision references prior clarifications from L1-HLD, L2-LLD-IC, and the consolidated client clarifications. The focus is on pragmatic solutions aligned with the current solution scale (roughly 30 crew onboard, up to 100 office users, offline operation up to ~14 days).

---

### Key Decisions

- **Encryption at Rest (Ref: L1-HLD “Security Architecture” + Clarification #1)**  
  • Decision: Defer encryption deployment. Maintain a roadmap to enable database or full-disk encryption later if regulatory or policy requirements mandate it.  
  • BE Implementation:  
    - Store crew data in plain MySQL tables on vessel and office servers, relying on OS-level access controls.  
    - Document a future rollout plan for BitLocker (Windows) or LUKS (Linux) if encryption becomes mandatory.  
    - Ensure minimal refactoring is needed later by keeping DB schemas and code paths straightforward.

- **Offline Authentication & 14-Day Session Keys (Ref: L1-OVERVIEW “Offline Operation” + Clarification #2)**  
  • Decision: Rely on SafeLanes’ existing auth system and short-lived “offline session keys” for vessel use.  
  • BE Implementation:  
    - Generate and store these keys in the local vessel database with a 14-day TTL.  
    - On expiry or compromise, a local override by the Master or Admin is logged.  
    - Once reconnected, the vessel revalidates all users against the central SafeLanes identity service (JWT-based).

- **Conflict Resolution for Compliance-Critical Data (Ref: L2-LLD-IC “Conflict Resolution” + Clarification #3)**  
  • Decision: Handle conflicts at the day level, requiring Admin or Super Admin approval before overwriting any rest-hour records.  
  • BE Implementation:  
    - If an incoming sync detects overlapping rest-hour blocks for the same day, mark the day as “conflict.”  
    - Provide an API endpoint to let office admins choose whether to KEEP_VESSEL or KEEP_OFFICE for the entire daily record.  
    - Log overwrites in an immutable audit table, preserving pre- and post-merge data for regulatory review.

- **Daily Record Structure (48 Blocks) (Ref: L1-HLD “Data Flow & Storage Strategy” + Clarification #4)**  
  • Decision: Retain a single record per crew per day, with 48 half-hour segments stored in a single DB row.  
  • BE Implementation:  
    - Store the blocks in a single structured field (e.g., JSON or multiple columns) linked to the corresponding date and crew ID.  
    - For merges or edits, the entire day’s record is updated; partial updates to individual half-hour blocks are consolidated server-side.  
    - Keep the existing validation and rule checks aligned with a day-based record.

- **Separate Per-Vessel Database & Daily Sync (Ref: L1-HLD “Architecture Overview” + Clarification #5)**  
  • Decision: Each vessel uses its own MySQL instance, syncing incrementally with the office once a day or on demand.  
  • BE Implementation:  
    - The Nest.js server on the vessel operates fully offline, storing new or updated rest-hour entries.  
    - A daily (or triggered) sync call from the vessel to the office updates the main DB with incremental changes based on updatedAt timestamps.  
    - Conflicts on compliance-critical data (rest-hours) are flagged for manual admin resolution.

- **UTC + Ship-Offset Handling (Ref: L1-HLD “Data Architecture Details” + Clarification #6)**  
  • Decision: Store all rest-hour entries in UTC, and track the vessel’s local-time offset at the daily record level.  
  • BE Implementation:  
    - Persist each day’s records with an internal UTC timestamp for official checks and sync.  
    - Include a shipOffset or localTimeOffset field (e.g., ±HH:MM) so the UI and compliance logic can display or calculate the correct local day boundaries.  
    - If the vessel changes time zone mid-voyage, the offset can be updated daily to keep end-of-day calculations consistent.

- **Forward-Only Rule Updates (Ref: L1-HLD “Constraints and Assumptions” + Clarification #7)**  
  • Decision: New or changed rest-hour rules apply strictly to future entries; no retroactive re-check of old data is performed.  
  • BE Implementation:  
    - The Nest.js back-end tags each rest-hour entry with the rule version used at the time of submission.  
    - When new rule sets come into effect, only logs dated from that point onward use the updated checks.  
    - Old entries remain valid under the previous rule version, reducing complexity and offline overhead.
