## L1-KD: Key Technical Decisions

### 1. Concurrency Model
-   
- We will apply a simple “last-write-wins” approach for non-financial lead and quotation data, while enforcing role-restricted updates or manual conflict checks for critical fields (e.g., commission approval, final pricing) to reduce risk of accidental overwrites.  
- An audit trail will log any overwrites in the timeline, explicitly indicating “Overwritten by user X at time Y.”  
- This approach balances minimal conflict-resolution overhead with sufficient safeguards at the ~400–600 concurrent users scale.

### 2. Deployment and Disaster Recovery
-   
- We will initially deploy in a single Azure region, relying on daily backups and geo-redundant blob storage to mitigate data loss. We will consider adding a minimal read replica (“warm standby”) in a secondary region if usage or SLA requirements justify it.  
- In the event we enable a secondary read replica, manual promotion will be required for failover, aligning with the manageable RTO/RPO needs at our current scale.  
- This phased approach balances cost and resilience for the ~400–600 concurrent user load.

### 3. Offline Capabilities for CP App
-   
- At the L1 level, we do not support offline creation or updates for the CP application. Detailed offline policies, if any, will be handled in lower-level (L3) documents. Future enhancements may revisit offline capabilities based on real usage demands.

### 4. “Customer Accepted” Interim Lead Status
-   
- We will retain and consistently implement “Customer Accepted” as a distinct status upon the customer tapping “Accept” on a shared quotation.  
- The CP or an Admin must then finalize the lead to “Won” after verifying details.  
- All related workflow documents (including L1-WF and the Customer App flow) will explicitly include this status for consistency across the funnel.  
- This preserves a clearer funnel stage, improves reporting, and simplifies commission calculation logic.

### 5. Backend Architecture
- We will maintain a single modular monolith codebase with well-defined domain boundaries.  
- This supports our current concurrency range without the operational complexity of microservices.  
- Internal structure will allow future extraction into independent services if load grows significantly.

### 6. Customer Uniqueness by Phone Number
- We will continue enforcing a strict 1:1 mapping between phone number and customer record.  
- Each phone number uniquely identifies a customer for login and KYC tracking.  
- In rare cases of shared or changed numbers, Admins can manually handle exceptions without altering core flows.