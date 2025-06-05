L3-KD-BE: Component-Specific Key Decisions for BE
Below are the final key decisions for the Back-End (Nest.js + MySQL) component. Each decision references prior clarifications from L1-HLD, L2-LLD-IC, and the consolidated client clarifications. The focus is on pragmatic solutions aligned with the current solution scale (roughly 30 crew onboard, up to 100 office users,).


Key Decisions

1. Daily Record Structure (48 Blocks) (Ref: L1-HLD "Data Flow & Storage Strategy" + Clarification #4)
• Decision: Retain a single record per crew per day, with 48 half-hour segments stored in a single DB row.
• BE Implementation:

Store the blocks in a single structured field (e.g., JSON or multiple columns) linked to the corresponding date and crew ID.
For merges or edits, the entire day's record is updated;.
Keep the existing validation and rule checks aligned with a day-based record.

2. Separate Per-Vessel Database (Ref: L1-HLD "Architecture Overview" + Clarification #5)
• Decision: Each vessel uses its own MySQL instance.
• BE Implementation:

The Nest.js server on the vessel operates fully offline, storing new or updated rest-hour entries.

3. UTC + Ship-Offset Handling (Ref: L1-HLD "Data Architecture Details" + Clarification #6)
• Decision: Store all rest-hour entries in UTC, and track the vessel's local-time offset at the daily record level.
• BE Implementation:

Persist each day's records with an internal UTC timestamp for official checks and sync.
Include a shipOffset or localTimeOffset field (e.g., ±HH:MM) so the UI and compliance logic can display or calculate the correct local day boundaries.
If the vessel changes time zone mid-voyage, the offset can be updated daily to keep end-of-day calculations consistent.

4. Forward-Only Rule Updates (Ref: L1-HLD "Constraints and Assumptions" + Clarification #7)
• Decision: New or changed rest-hour rules apply strictly to future entries; no retroactive re-check of old data is performed.
• BE Implementation:

The Nest.js back-end tags each rest-hour entry with the rule version used at the time of submission.
When new rule sets come into effect, only logs dated from that point onward use the updated checks.
Old entries remain valid under the previous rule version, reducing complexity and offline overhead.

