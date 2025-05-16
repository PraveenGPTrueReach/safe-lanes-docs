  
## L1-RAD: Risk Assessment Document (RAD)  Document

This Risk Assessment Document (RAD) provides a high-level view of key risks that could impact the Solarium Green Energy project. It focuses on solution-wide concerns rather than component-level details, aligning with the current scale of approximately 400–600 concurrent users. Each risk is described alongside its likely impact and possible mitigation strategies. Additionally, this document highlights the most complex features and third-party integrations that warrant careful attention.

Note on Naming Consistency: For clarity, we refer to the project as “Solarium Green Energy” throughout all L1 documents, ensuring alignment across solution materials. If the official name differs (e.g., “Solarium Energy”), future documents should be updated consistently.

Risk Rating Methodology: We categorize the Likelihood of each risk as Low, Moderate, or High based on estimated probability (e.g., Low < 10%, Moderate 10–50%, High > 50%). Impact is labeled Low, Medium, or High based on potential repercussions to business value, legal compliance, or project timelines. Our final Priority is determined by combining these two dimensions using a simple probability–impact matrix approach.

---

### 1. Purpose and Scope
The primary objective of this document is to identify, assess, and propose mitigation for major risks affecting the overall solution design, timelines, and operations. It also pinpoints the top five complex features and any third-party integrations requiring domain-specific expertise. These risks and complexities are considered at the solution (L1) level, without duplicating the deeper, component-specific risks that may appear in L2 or L3 documentation.

---

## 2. Key Risk Categories and Mitigations

### 2.1 Compliance and Data Privacy Risks
Data privacy remains pivotal because sensitive KYC documents and personal data must adhere to local regulations. Handling these documents securely and responsibly is essential, especially given the possible Indian PDPB-like environment.

- Risk: Non-compliance with data-privacy regulations could lead to legal issues or forced redesign.  
- Likelihood: Moderate (given the focus on KYC data).  
- Impact: High (potential legal and reputational harm).  
- Mitigation: 
  - Use AES-256 encryption at rest in Azure Blob and Azure Database for PostgreSQL.  
  - Restrict KYC data to a single region (or confirm that any geo-redundant storage remains within approved jurisdictions) with controlled access and 7-year retention.  
  - Maintain comprehensive audit logs of file uploads/approvals.

### 2.2 Business Continuity and Disaster Recovery Risks
This risk concerns system availability if the chosen single Azure region experiences a service disruption. Extended outages hinder sales operations, hamper new lead creation, and generally degrade customer satisfaction.

- Risk: Extended downtime if Azure region fails.  
- Likelihood: Low but potentially higher if extreme outages occur.  
- Impact: High (system unavailability hinders both sales and operations).  
- Mitigation: 
  - Confirm organizational acceptance of the 24-hour RPO/RTO.  
  - Consider optional minimal read replica in a secondary region if usage grows or SLAs tighten; note that business constraints may still favor single-region deployment.  
  - Periodically test restoration procedures.

### 2.3 Third-Party Service Dependency Risks
Multiple external providers (such as SMS and email gateways) are critical for user onboarding and system notifications. Any interruption in these services can disrupt business continuity or hamper user workflows.

- Risk: Single SMS provider downtime blocks new CP/Customer logins, stalling sales funnel operations.  
- Likelihood: Moderate (short-lived gateway outages can occur).  
- Impact: Medium (All OTP and password resets become unusable).  
- Mitigation: 
  - Implement retry/backoff logic if MSG91 fails.  
  - Prompt users to retry after short outages.  
  - Optionally add a fallback SMS provider if downtime becomes chronic.

### 2.4 Concurrency and Data Overwrite Risks
With ~400–600 concurrent sessions, multiple users can unintentionally modify the same record, especially for leads or quotations, risking inconsistent data states.

- Risk: Users overwriting each other’s changes without noticing, causing data confusion.  
- Likelihood: Moderate (esp. with ~400–600 concurrent sessions).  
- Impact: Medium (lost updates or inconsistent states).  
- Mitigation: 
  - Log all overwrites in the timeline (audit).  
  - Enforce stricter role-based controls for commissions, final lead “Won” status, and pricing overrides.  
  - Provide clear user alerts if data has changed since last fetch.

### 2.5 Master Data Completeness and Quotation Accuracy Risks
Master data, such as DISCOM fees and state subsidies, underpins the Quotation Wizard’s accuracy. Any outdated or missing figures can cause incorrect quotes and potential financial disputes.

- Risk: Missing or stale entries in master data can lead to faulty quotations or incorrect subsidies.  
- Likelihood: Moderate (frequent changes in fees/subsidies).  
- Impact: High (financial inaccuracies, possible revenue or legal issues).  
- Mitigation: 
  - Provide Admin UI for managing numeric thresholds and fees (Config-Driven approach).  
  -   
  Block quotation generation when critical fee or subsidy data is missing, unless an Admin override is provided with an appropriate audit log entry.  
  - Periodic reviews/audits of master data records.


### 2.6 Performance and Polling Load Risks
Frequent “pull-based” or manual sync requests can increase server load and consume user device resources if not carefully tuned.
- Risk: Overly aggressive polling intervals for “notifications” degrade performance and inflate hosting costs.  
- Likelihood: Moderate (if default sync intervals are not optimized).  
- Impact: Medium (elevated server load, slow responses, or higher costs for data usage).  
- Mitigation:
  - Implement sensible minimum polling intervals with back-off strategies for repeated failures.  
  - Monitor poll-based traffic and adjust intervals as needed to prevent performance bottlenecks.  
  - Include a plan for load/stress tests at ~400–600 concurrent sessions.

### 2.7 Scheduling and Resource Constraints
Project timelines and resource availability can create unexpected delays or skill gaps, impacting deliverables.
- Risk: Insufficient staffing or missed milestones if specialized roles (e.g., front-end, QA, domain SME) are unavailable.  
- Likelihood: Moderate (resource turnover is common).  
- Impact: Medium (delivery slippages, potential cost overrun).  
- Mitigation:
  - Define clear project schedule with buffer for critical resources.  
  - Identify backup or cross-trained team members for essential tasks.  
  - Escalate major staffing issues early to stakeholders.


---

## 3. Top 5 Highly Complex Features

Below are the five solution-level features that carry higher implementation complexity or algorithmic detail. Each requires precise logic to avoid inaccuracies or performance bottlenecks.

1. **7-Step Quotation Wizard and Subsidy Calculations**  
   - Combines dynamic state/central subsidies, panel capacity, fees, and variable margin.  
   - Complexity arises from tiered logic for capacity-based subsidies and real-time cost generation.

2. **Commission Life Cycle (Pending → Approved → Paid)**  
   - Ties final “Won” lead data to commissions.  
   - Must handle cross-checks with the accepted Quotation Reference and integrate final lumpsum accounting.

3. **“Customer Accepted” Transitional Status Workflow**  
   - Brings an intermediate state between quotation acceptance and “Won.”  
   - Requires careful role-based controls for CP or Admin to confirm and finalize the lead.  
   Note: L1-WF must be updated to reflect this official intermediate status, ensuring alignment with L1-FRS and L1-OVERVIEW.

4. **CSV Bulk-Import with All-or-Nothing Validation**  
   - Validates large sets of leads; a single invalid row aborts the entire import.  
   - Performance and user feedback (detailed error logs) must prevent repeated rework.

5. **Multi-Service Cart Submission for Customers**  
   - Aggregates multiple solar services into one lead/quotation.  
   - Complexity arises in ensuring a single acceptance or rejection covers all services, avoiding partial splits.

---

## 4. 
Third-Party and Platform Integrations Requiring Special Expertise

Within this solution, the following external services  and core platform components warrant focus to mitigate technical and operational risks:

1. **MSG91 SMS Gateway**  
   - Critical for OTP-based logins (CP and Customer).  
   - Outage or rate-limit handling, plus robust retry/backoff strategies, are crucial to avoid login disruptions.

2. **SendGrid for Email Notifications**  
   - Used for Admin/KAM password resets and system alerts.  
   - Requires correct SMTP or API configurations, bounce handling, and spam compliance to ensure consistent delivery.

  


3. **Azure Blob Storage (Platform Service) for KYC and Lead Documents**  
   - Although an Azure offering rather than a separate third-party vendor, specialized configuration is needed for secure uploads, correct SAS token usage, large-file handling (up to 10 MB), and 7-year retention or versioning.  
   - Monitoring costs and verifying local or geo-redundant replication compliance remain necessary.


---

## 5. Overall Risk Prioritization
| Risk/Concern                                   | Likelihood | Impact | Priority (H/M/L) | Owner |
|------------------------------------------------|-----------|--------|------------------|------------------|
| Compliance & KYC Data Privacy                  | Moderate  | High   | High             | Admin Team |
| Single-Region DR (24h RPO/RTO)                 | Low       | High   | Medium           | DevOps |
| Single SMS Provider Downtime                   | Moderate  | Medium | Medium           | DevOps |
| Data Overwrites w/ Last-Write-Wins             | Moderate  | Medium | Medium           | Dev Team |
| Missing/Incorrect Master Data for Fees/Subsidy | Moderate  | High   | High             | Admin Team |
| Polling/Sync Load Issues                     | Moderate  | Medium | Medium           | Dev Team         |
| Scheduling & Resource Constraints             | Moderate  | Medium | Medium           | PM / Management  

This table reflects the identified risk owners, who must actively track and carry out mitigations in their respective domains.

---

## 6. Recommended Next Steps

1. **Formalize Data Compliance**  
   - Ensure minimal PDPB-like rules are met for KYC data.  
   - Retain comprehensive logs of KYC usage and approvals.

2. **Accept/Review Disaster Recovery Policy**  
   - Confirm official acceptance of 24-hour RPO/RTO or introduce a read replica if business demands faster recovery.  
   - Clarify whether enabling GRS remains within the same legal boundaries if data residency mandates a single region.

3. **Implement Thorough Master Data Management**  
   - Provide an Admin UI and mandatory review schedules to keep DISCOM fees/subsidy data updated.  
     
   - Block or warn when critical data is missing, requiring Admin override with an audit trail to avoid under-quoted deals.

4. **Refine Concurrency Handling**  
   - Keep auditing any overwrites in the timeline.  
   - Potentially extend warnings for critical fields if concurrency collisions occur.

5. **Establish Clear Fallback Procedures for OTP**  
   - Carefully log gateway errors from MSG91.  
   - In the event of repeated failures, prompt users to try again or adopt an alternative authentication channel if escalation is requested.

6. Plan and Execute Performance & Load Testing  
   - Schedule regular load/stress tests with ~400–600 concurrent sessions.  
   - Monitor sync/poll intervals to avoid excessive server load.  
   - Validate that the system meets the stated 2–5 second response targets under peak usage conditions.

7. Address Scheduling and Resource Risks  
   - Document key milestones and ensure identified owners for critical tasks.  
   - Define fallback resources or cross-trained team members for specialized roles.  
   - Hold frequent check-ins to spot potential delays early.

These recommendations ensure the solution remains resilient, accurate, and compliant at the scale of ~400–600 concurrent users, supporting future incremental enhancements without overcomplicating the current scope.

---

**End of L1-RAD: Risk Assessment Document (RAD)**