## L1-RAD: Risk Assessment Document (RAD) Document

## 1. Introduction

This Risk Assessment Document (RAD) identifies potential risks that could significantly impact the SafeLanes Rest Hours solution at the enterprise/solution level. It outlines the likelihood and impact of each risk and proposes mitigation strategies aligned with the scope defined in the L1 documentation set. This assessment does not cover detailed, component-specific risks, which may be addressed in separate L3-level risk assessments.

---

## 2. Risk Analysis Approach

The following risks are categorized based on primary domains (e.g., Architecture, Security, Development Process, Regulatory Compliance, etc.). Each risk is assigned a likelihood (Low, Medium, High) and impact (Low, Medium, High). Proposed mitigation strategies target pragmatic solutions suitable for the currently stated scale and requirements.

---

## 3. Major Risk Areas

### 3.1 Architectural Risks

1. **Scalability Limits Underestimated**  
     
   - **Description**: The system is designed for ~30 vessel users and ~100 office users. If the scope increases (e.g., new fleets, multiple concurrent vessels), the architecture may face load issues.  
   - **Likelihood**: Low (current client scope is moderate).  
   - **Impact**: Medium (performance or concurrency constraints).  
   - **Mitigation**: Maintain performance monitoring at the office server. Include an option for horizontal scaling on the Nest.js side if concurrency grows unexpectedly.

2. **Integration with SafeLanes Identity Service**  
     
   - **Description**: JWT-based authentication require careful token revocation and re-auth. Errors or stale tokens could lock out legitimate users.  
   - **Likelihood**: Medium.  
   - **Impact**: High (crew unable to log hours, triggering compliance gaps).

---

### 3.2 Security & Compliance Risks

1. **Regulatory Rule Updates**  
     
   - **Description**: MLC, STCW, and OPA thresholds can change, but vessels may be offline for extended periods. Outdated rules could impact compliance if they remain misaligned with current regulations.  
   - **Likelihood**: Medium (regulations or policies evolve).  
   - **Impact**: Medium (inaccurate violation detection, potential maritime non-conformance).  
   - **Mitigation**: Bundle updated rule sets with major code updates or as small signed JSON packages. Provide version warnings to encourage prompt updates post-sync.

2. **Audit Trail Integrity**  
     
   - **Description**: Overwritten compliance-critical records must remain traceable. If the audit log mechanism fails or is tampered with, the system could hide critical changes.  
   - **Likelihood**: Low (design includes strict read-only logs).  
   - **Impact**: High (compliance violations or potential legal liability if logs cannot be reconstructed).  
   - **Mitigation**: Enforce immutable storage for overwritten daily records. Restrict direct database-level access. Perform periodic log integrity checks.  
     **Contingency Plan**: Upon detecting potential tampering, the system initiates an integrity audit. If any discrepancy is confirmed, a formal forensic investigation is launched. Temporary read-only status may be imposed until the logs are fully verified.

---

### 3.3 Development & Deployment Process Risks

1. **Deployment Coordination with Vessel Schedules**  
     
   - **Description**: Vessel-side updates may be delayed if ships are at sea, preventing timely patching or bug fixes. This can exacerbate version mismatches.  
   - **Likelihood**: Medium (common in maritime environments).  
   - **Impact**: Medium (stale UI or back-end versions lead to confusion, potential offline block).  
   - **Mitigation**: Plan updates during known port arrivals. Provide a fallback grace period for local UI usage with version mismatch warnings.

2. **Overengineering Beyond Stated Scale**  
     
   - **Description**: Attempting advanced solutions (e.g., partial merges or chunked sync) may add excessive complexity without immediate benefit.  
   - **Likelihood**: Medium (developer preference for "elegant" solutions).  
   - **Impact**: Medium (increased code complexity, harder QA, more bugs).  
   - **Mitigation**: Follow stated scope strictly.

---

## 4. Top 5 Complex Features

Below are the logical sub-features or implementations identified as having the most intricate logic or heavy reliance on specialized algorithmic handling:

1. **Daily Conflict Resolution Mechanism**  
     
   - Requires comparing updatedAt timestamps. Involves read-only audit logging of overwritten data for full traceability.

2. **Predictive Violation Logic (24–168h Forecast)**  
     
   - Interprets planned tasks plus current rest-hour records to generate warnings. Must parse half-hour blocks, check overlapping intervals against MLC/STCW thresholds, and present to users in near-real time.

3. **Planning vs. Actual Hours Merge**  
     
   - Vessel Admins define "planned" (grey) blocks. Crew then confirms or overrides them in the daily log, changing color codes and triggering real-time compliance checks. This code path demands robust logic to differentiate planned vs. actual hours in half-hour granularity.

---

## 5. Third-Party Integrations Requiring Specific Expertise

1. **SafeLanes Identity Service**  
     
   - JWT-based authentication. Requires in-depth knowledge of SafeLanes RBAC structure, token refresh logic, and override auditing to avoid security gaps.

2. **Angular 18.2.0 Module Federation**  
     
   - Modern microfrontend architecture that may require specialized Angular build configuration and debugging even if the host remains on an older framework version.

3. **MySQL (Offline + Potential Encryption)**  
     
   - Although MySQL is a standard RDBMS, maritime operational constraints (e.g., full-disk encryption, TDE, daily backups) can require advanced DBA or sysadmin skills to manage frequent merges and offline operation securely.

---

## 6. Conclusion

Overall, the largest threats to project success revolve around maintaining accurate data in offline conditions, ensuring robust security and compliance, and managing the complexities of a microfrontend architecture. By executing the identified mitigations—particularly the offline conflict resolution strategy (last-write-wins), staged updates—the project can meet maritime rest-hour requirements reliably without overengineering for unstated scales. Continuous monitoring of incremental sync, version compatibility, and audit log integrity will remain essential for ongoing risk management.  
