## L1-KD: Key Technical Decisions


### Decision 1: Single-Record-Per-Day Schema

- Each crew member’s daily time logs will be stored as one record per day, subdivided into 48 half-hour blocks.  
- Concurrency is handled at the day level, relying on minimal multi-user edits and the existing conflict-resolution (Last-write-wins) approach to prevent data loss.  
- multiple offline edits can occur on the same day for rest-hour compliance data, For all l fields within the same daily record, the last-write-wins approach still applies.  
- This design meets current scaling needs while remaining simpler to implement and maintain than a more granular schema.

### Decision 2: Local Microfrontend for Offline Vessels

- Each vessel retains a locally bundled copy of the microfrontend for use.  
- To mitigate critical version mismatches, we enforce semantic version checks between the bundled UI and the back-end API, blocking usage if versions are incompatible.  
- Vessels can update the bundled application during a stable connection, ensuring essential functionality remains available offline without blocking critical tasks.

### Decision 3: Conflict Resolution Policy

- For all data fields, last-write-wins applies automatically.  
- Overwritten data is logged in a protected audit record, ensuring no silent overwrites of compliance-critical fields.  
- This policy aligns with the rest-hour compliance requirements to prevent loss or accidental overwrite of critical data, while maintaining a simplified last-write-wins model for non-critical information.

### Decision 4: Local Database Management on Vessels

- Each vessel runs a local MySQL database, storing data offline   
- This approach preserves offline-first operations with minimal extra overhead at the current project scale.

