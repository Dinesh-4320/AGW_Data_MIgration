# **Milestone 6: Migrating Critical Databases and Data**

---

### Questions to Ask the Stakeholder

| Question                                                              | Assumption for a typical large water management company               |
| --------------------------------------------------------------------- | --------------------------------------------------------------------- |
| What databases support customer-facing or real-time operational apps? | Billing, telemetry, SCADA, sensor streaming, compliance tracking      |
| Are these databases subject to strict SLAs or compliance mandates?    | Yes — high availability, real-time access, and full data integrity   |
| What is the maximum allowable downtime?                               | Zero-downtime or at most a few seconds during cutover                 |
| Are there specific performance benchmarks to retain post-migration?   | Yes — low-latency, high IOPS, concurrent connection support          |
| Are disaster recovery and failover plans tested regularly?            | Yes — enforced by regulatory audits                                  |
| Can the DBMS type change, or must it remain the same?                 | Must remain the same (rehost-only constraint) for contractual reasons |

---

### Classification of Critical Databases

A critical database in AGW's context is defined as one that:

* Supports customer billing, water quality reporting, real-time telemetry, or field operations
* Has <1 second response time and >99.99% availability requirements
* Is integrated with applications that operate 24x7, including mobile and SCADA systems
* Is subject to legal and audit requirements (e.g., for financial records or regulatory compliance)

Example Critical Systems:

| Database Name         | Purpose                                    | Type       | Criticality | Notes                                   |
| --------------------- | ------------------------------------------ | ---------- | ----------- | --------------------------------------- |
| Billing_Prod          | Real-time metered billing engine           | Oracle     | High        | Must retain transactional logs          |
| SCADA_Streaming       | Ingests sensor feeds from treatment plants | PostgreSQL | High        | High-ingest, high-availability required |
| Compliance_Live_2024  | Legal data for inspections/reporting       | SQL Server | High        | Immutable records, audit logs           |
| MobileApp_LiveBackend | Serves live API for field worker app       | MySQL      | High        | Concurrent API traffic, frequent writes |

---

### Migration Strategy: **Zero-Downtime Rehosting**

We will adopt a zero-downtime rehosting strategy using continuous data replication and controlled cutover , ensuring that the schema, data, and services remain intact. We cannot replatform or change DB engines — so the same database engine must be retained on Azure IaaS (VM) using managed disks and High Availability features.

### Phases of Critical Database Migration

The critical database migration process will be executed in a series of structured phases to ensure minimal risk and zero downtime.

1. **Baseline Assessment**

   The first step involves taking a complete backup of the database, collecting performance benchmarks, mapping all current application connections, and capturing active session logs. This assessment serves as the foundation for designing a like-for-like environment in Azure.
2. **Provisioning Azure VM Infrastructure**

   Using Azure Migrate, we will size and provision an Azure Virtual Machine that mirrors the on-prem configuration—matching the operating system, database engine version, and hardware profile. Managed disks and high availability configurations will be applied as needed.
3. **Replication Setup**

   At this stage, native database replication tools will be configured. For example, Oracle Data Guard, SQL Server Always On, or MySQL replication mechanisms will be deployed to establish a reliable sync channel between the source and target environments.
4. **Initial Data Sync**

   A full database dump or initial replication snapshot will be sent to the Azure-hosted instance. Post-import, consistency checks will be run to verify row counts, schema alignment, and basic query responsiveness.
5. **Enabling Real-Time Synchronization**

   Once the initial sync is complete, continuous replication will be enabled. This may use Change Data Capture (CDC), log shipping, or streaming replication depending on the DBMS. This ensures that changes on the source are reflected instantly on the target.
6. **Cutover Execution During Planned Maintenance**

   A brief planned maintenance window will be scheduled for cutover. During this window, writes on the source database will be paused. The Azure replica will be promoted as the new primary, and DNS or application connection strings will be redirected to point to the Azure-hosted database.
7. **Post-Cutover Validation and Monitoring**

   After cutover, intensive validation will be conducted to verify that all data is intact. Row counts, hash-based checksums, and performance metrics will be compared against pre-migration baselines. Active monitoring will be enforced through Azure Monitor and Log Analytics.
8. **Decommissioning the Legacy Environment**

   Finally, once stability is confirmed, the legacy database will be made read-only for 48 hours (as rollback buffer), after which it will be decommissioned. All access will be revoked, backups will be archived, and the server will be retired from the infrastructure stack.

---

### Tools and Services to Use

| Purpose                      | Tool/Service                               | Description                                                      |
| ---------------------------- | ------------------------------------------ | ---------------------------------------------------------------- |
| VM-level rehosting           | **Azure Migrate**                    | Rehost DB server as-is with OS and DB intact                     |
| Replication setup            | **DB-native tools**                  | Oracle Data Guard, MySQL replication, SQL Server Always On, etc. |
| Backup and sync verification | **Azure Backup, Log Analytics**      | For periodic backups and monitoring                              |
| High availability            | **Availability Sets / Zones**        | Deploy in HA topology with Azure Disk replication                |
| DNS cutover and failover     | **Traffic Manager / Private DNS**    | Control routing at the moment of cutover                         |
| Ongoing sync / hybrid period | **Azure Site Recovery / GoldenGate** | For continuous updates until full switchover                     |
| Monitoring post-migration    | **Azure Monitor, Log Analytics**     | Performance alerts, anomaly detection                            |
| Security                     | **Defender for SQL, NSGs**           | Enable encryption, firewalling, and threat detection             |

---

### Example Migration Scenario

#### Database: `Billing_Prod`

* Type: Oracle 12c
* Size: 800 GB
* Uptime: 24x7, response time SLA of < 200ms
* Contains: Customer account balances, invoice data, meter readings

**Migration Flow:**

1. Provision matching Azure VM (OS: RHEL 7.9, DB: Oracle 12c) with reserved capacity and managed disks.
2. Configure Oracle Data Guard on source and target VMs.
3. Set up primary → secondary sync with real-time log shipping.
4. Let systems sync continuously for a week, ensuring zero data lag.
5. During planned maintenance,  perform cutover : stop writes on source, promote Azure replica.
6. Redirect all apps (via DNS or connection string update) to Azure DB.
7. Keep source read-only for rollback (for 48 hours), then decommission.

---

### Key Considerations and Safeguards

| Concern                       | Mitigation Strategy                                             |
| ----------------------------- | --------------------------------------------------------------- |
| Schema drift during migration | Lock schema changes during final sync window                    |
| In-flight transactions        | Use transactional replication or pause writes during cutover    |
| Security and compliance       | Enforce encryption-at-rest and in-transit, audit access logs    |
| Failover testing              | Test replica promotion and rollback at least once pre-migration |
| Latency impact post-migration | Benchmark and auto-scale VMs as needed                          |

---

### When to Use Other Approaches

| Scenario                                       | Alternative Approach                           | Justification                                           |
| ---------------------------------------------- | ---------------------------------------------- | ------------------------------------------------------- |
| DB engine can be changed and app is modernized | Replatform to Azure SQL / PostgreSQL           | PaaS advantages, but not valid in this rehost-only case |
| App + DB migration needed in lockstep          | Rehost app and DB together using Azure Migrate | Prevents mismatch or version drift                      |
| Custom DB or file-based formats involved       | Azure File Sync, Blob Storage                  | Handle unusual or non-relational data                   |

---

### Cost and Environmental Impact

* Rehosting avoids license re-purchase costs while preserving SLAs.
* Azure Reserved Instances for DB VMs reduce long-term costs (up to 72%).
* High availability zones offer built-in DR with minimal hardware footprint.
* Eliminating legacy on-prem DB servers reduces AGW’s carbon footprint.
* Migration allows implementation of modern observability and threat detection that were not possible on legacy platforms.

---

### Summary Table

| Activity               | Tool/Service Used                 | Benefit                                              |
| ---------------------- | --------------------------------- | ---------------------------------------------------- |
| Infrastructure Setup   | Azure Migrate                     | Identical DB environment in Azure VM                 |
| Real-Time Replication  | Oracle DG / Always On / CDC       | Enables near-zero downtime                           |
| HA + Monitoring        | Availability Zones, Azure Monitor | Ensures SLAs and performance post-migration          |
| Migration Verification | Log Analytics, custom scripts     | Validate data integrity and query consistency        |
| Cutover & Failback     | DNS + Controlled Promotion        | Allows rollback or hybrid operation if needed        |
| Security               | Azure Defender for SQL            | Encryption, threat detection, access control         |
| Sustainability         | Decommission old servers          | Reduce power usage and cooling needs in data centers |

---
