# Milestone 8: The Switch Over

## Objective
To execute the final migration from the legacy infrastructure to the new cloud/on-prem hybrid environment by switching production workloads and services to the new environment while minimizing downtime, ensuring business continuity, and validating full operational readiness.

---

## Goals

- Seamless cutover of critical and non-critical systems to the target environment
- Verification of all service functionalities post switch-over
- Final user acceptance and production sign-off from stakeholders
- Validated fallback plan to revert in case of major failure
- Smooth communication and escalation process

---

## Pre-Switch Checklist

| Item                    | Description                                  | 
|-------------------------|----------------------------------------------|
| CMDB Finalization       | All systems and interdependencies mapped     | 
| Pre-Prod Testing        | All systems tested in staging/UAT            | 
| Knowledge Transfer      | Teams trained on new infra/apps              | 
| Security Validated      | IAM, Firewall, Compliance validated          | 
| Change Request Approval | Final change request reviewed and approved   | 
| Rollback Plan           | Documented and tested                        | 


---


##  Cost Considerations

| Category          | Notes                                        |
|-------------------|----------------------------------------------|
| Storage           | Object-based storage cheaper on cloud        |
| Compute           | EC2/Azure VMs reserved instance pricing      |
| Migration Tooling | One-time data replication setup              |
| Networking        | VPN + ExpressRoute/DirectConnect             |
| Backup/DR         | Cloud-native backups are more cost-effective |

---

## System Validation & Approval

### Readiness Checks (Per System)

| System            | Checklist                          | Test Method                  | Responsible             |
|-------------------|------------------------------------|------------------------------|-------------------------|
| SCADA             | Data flow, latency, alerting       | Simulated signal tests       | Network Ops             |
| CRM               | Login, transaction logs, billing   | Test user login & bill       | Customer Ops            |
| Meter APIs        | Realtime updates, error rates      | Load test & failover         | IoT Lead                |
| Payment Gateway   | Transaction success, rollback      | Dummy payment runs           | Finance IT              |
| Identity Provider | SSO, MFA, RBAC enforcement         | Pen testing IAM              | Cybersecurity           |
| Leak Detection    | Real-time alert flow               | Simulated sensor breach      | Asset Monitoring        |
| Legal Audit DB    | Timestamp accuracy, report logging | Compliance report generation | Regulatory & Compliance |

---

## Metrics to Determine Success

| Area                | Metric              | Threshold          |
|---------------------|---------------------|--------------------|
| Availability        | Uptime post-cutover | ≥ 99.95%           |
| Latency             | API Response Time   | ≤ 200ms            |
| Data Accuracy       | Data sync mismatch  | ≤ 0.01%            |
| Security            | Vulnerability score | Zero critical      |
| Support             | Issue escalation    | ≤ 5 per day        |
| Rollback Activation | Revert usage        | 0 (unless planned) |

---

## Approval Flow

1. **Stage Validation** by individual technical owners (via checklist validation)
2. **Business Stakeholder UAT Sign-off** per domain (CRM, SCADA, Billing)
3. **Security Clearance** from Cybersecurity head
4. **Final Go/No-Go Meeting** with migration leads, architects, management
5. **Formal Switchover Execution Window** (off-peak hours, fully staffed)
6. **Post Switch Monitoring** for 72 hours before final approval

---

## Rollback Plan

- **Infra Snapshot** pre-switchover (VM/Container + DB + Configuration)
- **Switchback DNS TTL set to 30s** for quick reversion
- **Pre-validated Restore Scripts** tested in staging
- **All fallback assets retained for 14 days** post switch

---

## Post-Switchover Testing

- **Automated Smoke Tests** (login, workflow, APIs, billing)
- **Performance Benchmarks** (against baseline)
- **Manual Edge Case Testing** (UAT team)
- **Security Scan** with SIEM integrations
- **CMDB Update** post switch with production IPs, ownership

---

## Communication Plan

-  All-hands migration mail with schedule and points of contact
-  Bridge call open during cutover period with migration lead + L2 team
-  Escalation matrix clearly shared to all domains


---

## Tools & Technologies for UAT and Post-Switchover Validation

| Tool / Service                              | Purpose                                                           | Where It’s Used                                    |
|---------------------------------------------|-------------------------------------------------------------------|----------------------------------------------------|
| **Azure DevTest Labs**                      | Create isolated, scalable UAT environments that mirror production | Simulating final workloads before go-live          |
| **Azure Load Testing**                      | Simulate real-world HTTP traffic load, stress testing             | Check application scalability under load           |
| **Azure Chaos Studio**                      | Fault-injection testing for resilience and failover               | Ensuring system stability under adverse conditions |
| **Azure Application Insights**              | Real-time app-level telemetry: requests, dependencies, failures   | Application behavior validation                    |
| **Azure Monitor**                           | Infrastructure-level metrics (CPU, memory, network)               | Infrastructure readiness verification              |
| **Postman / Swagger**                       | API endpoint testing & response validation                        | Backend service and API integrity                  |
| **K6 / Apache JMeter**                      | Custom load testing for APIs and services                         | Performance benchmarking during UAT                |
| **Azure Test Plans (Azure DevOps)**         | Manual + automated test case execution, bug tracking              | Functional validation, test case mapping           |
| **PowerShell / Bash Scripts**               | Automation of test tasks, backend validations                     | Custom assertions and validation of CLI/backend behaviors |
| **Azure Resource Health**                   | Check resource availability and uptime                            | Verify all services are healthy post-deployment    |
| **Azure Policy & Compliance Manager**       | Ensures all policies (e.g.,encryption) are enforced               | Prevent misconfiguration in final environment      |
| **Azure Site Recovery (ASR) Test Failover** | DR validation without impacting production                        | Disaster-readiness testing                         |
| **Azure Diagnostics Extension**             | Collect system logs, events, performance counters                 | Analyze issues during UAT                          |
| **Power BI Dashboards**                     | Real-time visibility into test KPIs, success rates                | Stakeholder-level UAT status tracking              |
| **Microsoft Defender for Cloud**            | Vulnerability assessments, security baselines                     | Confirm post-migration environment is secure       |
| **Microsoft Purview**                       | Data classification and compliance scanning                       | Ensure migrated data complies with regulations (PII, etc.) |
| **Azure Log Analytics**                     | Collect logs from all services and correlate them                 | Deep root-cause analysis during testing            |
| **Application Map (App Insights)**          | Visual mapping of dependencies between components                 | Identify bottlenecks and unexpected behaviors      |
| **Custom Alert Rules**                      | Notify on performance degradation or failure                      | Rapid reaction during UAT cycles                   |
| **Azure Backup / Snapshot**                 | Pre-switch backup of critical VMs / Data                          | Enables rollback if UAT fails                      |
| **Azure Boards (Azure DevOps)**             | Track bugs, feedback, test progress                               | Central UAT management hub                         |
| **Teams / Outlook Integration**             | Notify stakeholders of sign-off requests                          | Communication + approvals                          |
| **OneNote / SharePoint**                    | Document results, test logs, lessons learned                      | For retrospectives and audit trail                 |


---

## Sign-Off Sheet

| Role               | Person | Sign-off Status |
|--------------------|--------|-----------------|
| Migration Lead     | [Name] | ☐              |
| Solution Architect | [Name] | ☐              |
| Department Head    | [Name] | ☐              |
| Department Head    | [Name] | ☐              |
| Cybersecurity      | [Name] | ☐              |
| Finance            | [Name] | ☐              |
| Regulatory         | [Name] | ☐              |

---

# Test Case: Functional Testing – CRM Login and Customer Billing Generation

---

## Test Case Summary

| Field | Description |
|-------|-------------|
| **Test Case ID** | FT-CRM-001 |
| **Title** | Validate CRM login, customer lookup, and bill generation |
| **Category** | Functional Testing |
| **Priority** | High |
| **Module** | CRM Web App |
| **Environment** | UAT (Azure DevTest Labs – cloned production config) |
| **Preconditions** | - User account `testuser1` exists  
- At least 1 customer exists with valid meter reading data |
| **Tools Used** | - Azure DevTest Labs  
- Postman  
- Azure SQL DB  
- Azure Application Insights  
- Azure Test Plans (Azure DevOps) |
| **Test Data** | Username: testuser1  
Password: [valid pass]  
Customer ID: CUST10023 |
| **Dependencies** | CRM DB, Billing API, Auth System (Entra ID) |

---

## Test Setup

### Provisioning Test Environment

- Clone the production configuration using **Azure DevTest Labs**
- Deploy CRM backend to **Azure App Service (Staging Slot)**
- Connect CRM to **Azure SQL Database** with anonymized test data
- Bind to **Entra ID** and configure test users

### Monitoring & Logging

- Enable **Azure Application Insights**
- Log all HTTP transactions, user interactions, and exceptions

### Connectivity Checks

- Validate connections to:
  - Billing API (via API Management)
  - Payment Gateway
  - Azure SQL using Private Endpoint

---

## Step-by-Step Test Execution

| Step | Action                                | Expected Result                                   |
|------|---------------------------------------|---------------------------------------------------|
| 1    | Launch CRM UAT portal                 | Login screen appears                              |
| 2    | Login with testuser1 via Entra ID SSO | Login successful, dashboard loads                 |
| 3    | Search for Customer ID `CUST10023`    | Customer profile appears                          |
| 4    | Trigger "Generate Bill" action        | Invoice generated and stored in Billing DB        |
| 5    | Download invoice as PDF               | File downloads with correct data                  |
| 6    | Review App Insights logs              | No unhandled exceptions, response time < 500ms    |
| 7    | Run query on Billing DB               | Invoice entry exists with correct amount and date |

---

## Expected Results

- CRM allows login via Entra ID SSO  
- Customer data is accurate and accessible  
- Invoice is generated and saved correctly  
- PDF is downloadable and matches billing info  
- App Insights shows healthy metrics  
- Billing DB entry is correct and timestamped  

---

## Failure Criteria

- Login fails with valid credentials  
- System takes >5 seconds for responses  
- Billing API errors or missing invoice data  
- No record in database  
- Exceptions or HTTP 5xx/4xx errors in telemetry  

---

## Post-Test Actions

- Mark test **Passed/Failed** in **Azure Test Plans**
- Archive all relevant **logs and screenshots**
- **File bugs** in Azure Boards if needed
- Update **CMDB** with final CRM status and test reference

---

