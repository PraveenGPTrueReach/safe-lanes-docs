# L3-KD-FE: Component-Specific Key Decisions for FE

## Key Decisions

### Decision #1: Local Static Bundle for Offline

All vessel environments will receive a prepackaged Angular 18.2.0 bundle served entirely from the local server when offline.
Updates to this bundle occur manually during scheduled port calls, ensuring version alignment with the vessel's Nest.js back-end.
No service-worker-based caching will be used to avoid complexity in offline mode.
(Ref. L1-HLD - "Offline Fallback Bundling on Vessels" and L2-LLD-IC - "Generic Interaction Flow (§2.1)")

### Decision #2: Continued Module Federation

The Rest Hours front-end will remain a separate microfrontend exposed via Module Federation, preserving standalone deployment and future extensibility.
Shared dependencies (e.g., Angular libraries) must be version-aligned with the main Sail App once it is upgraded to Angular 18.2.0.
While the fallback wrapper remains, this federation runs in an isolated container; once the host is on Angular 18.2.0, federation seamlessly integrates with the host shell.
(Ref. L1-HLD - "Front-End (Angular Microfrontend)" and L1-OVERVIEW - "Module Federation Using Angular 16")

