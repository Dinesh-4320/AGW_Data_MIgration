# **Milestone 4: Migrating Non-Critical Databases and Data**

---                                        |

### Stakeholder Considerations for Non-Critical Database Migration

| Aspect                                   | Recommendation / Mentor Guidance                                                                                                                                         | Risk / Gap Identified                                                                         |
| ---------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------- |
| **Definition of Non-Critical DBs** | Databases not involved in real-time workflows, lacking SLAs, with low-frequency access or serving internal BI tools and compliance dashboards.                           | In absence of a CMDB, identification must rely on stakeholder inputs and usage monitoring.    |
| **Common Characteristics**         | Not accessed by more than 5 users per week, updated no more than once daily, tagged as archival or internal-only during app review.                                      | Misclassification may delay dependent services or reports post-migration.                     |
| **Database Types**                 | SQL Server, Oracle, MySQL, PostgreSQL, and some embedded/legacy Tibco-MQ based databases. Migration targets must support heterogeneous database engines.                 | Compatibility or licensing challenges may arise if platforms are outdated or vendor-locked.   |
| **Integration Dependencies**       | Many serve batch-mode ETL pipelines; must document these dependencies explicitly before migration to avoid downstream impact.                                            | Unidentified reporting jobs or sync scripts may break silently post-migration.                |
| **Write Frequency**                | Most are read-heavy with batch or end-of-day (EOD) dumps. Suitable for phased trickle migration with minimal sync lag tolerance.                                         | Any deviation from expected write frequency introduces risk of data loss during migration.    |
| **Shared Server Hosting**          | Many legacy databases are co-hosted on shared infrastructure. Use isolation techniques (e.g., VM snapshots, containerization) during migration planning.                 | Shared environments introduce cascading risks if one system’s migration affects others.      |
| **Security & Compliance**          | Evaluate for any PII or sensitive data. Mask or anonymize datasets where necessary, and ensure Azure-native encryption and role-based access is enforced post-migration. | Compliance risks under GDPR or internal audit may surface if classification is incomplete.    |
| **Migration Methodology**          | Use Azure Database Migration Service (DMS) with trickle migration for continuous sync. Cold cutover scheduled during off-hours to minimize disruption. | Improper cutover timing or sync gaps may affect data integrity or application availability.   |
| **File-Based Dependencies**        | Use Azure File Sync for any flat-file dependencies (e.g., logs, CSV exports) during DB migration.                                                                | Incomplete file sync may affect scripts or data pipelines relying on legacy export locations. |
| **Source of Guidance**             | All recommendations are based on mentor-led internal reviews and field-tested strategies. Decisions are informed — not posed as stakeholder questions.                  | Future stakeholder feedback may still be required for validation and iterative planning.      |

---

### Defining “Non-Critical” Databases

Given AGW lacks a formal CMDB (configuration management database), we propose identifying non-critical databases using the following rules :

| Rule # | Description                                                                                  | Justification                              |
| ------ | -------------------------------------------------------------------------------------------- | ------------------------------------------ |
| 1      | Not directly involved in real-time operational workflows                                     | Cannot affect live system uptime           |
| 2      | Serves only internal reporting or compliance dashboards                                      | Low business continuity impact             |
| 3      | Accessed by fewer than 5 users on a weekly basis                                             | Low usage = low criticality                |
| 4      | Data not updated more than once per day                                                      | Eligible for low-sync migration approaches |
| 5      | Hosted on shared servers with minimal interdependencies                                      | Easier to isolate and migrate              |
| 6      | Lacks formal SLAs (Service Level Agreements)                                                 | No contractual uptime guarantees           |
| 7      | Backup and recovery policies are not audited or formally tested                              | Indicates non-critical use                 |
| 8      | Tagged by business units as archival, legacy, or internal-only during the application review | Confirmed by stakeholders                  |

Example Candidate Systems:

| Database Name       | Purpose                            | Type       | Criticality | Migration Feasibility |
| ------------------- | ---------------------------------- | ---------- | ----------- | --------------------- |
| Internal_BI_Reports | Generates monthly performance KPIs | SQL Server | Low         | Easy                  |
| FieldSensor_Archive | Stores EOD sensor dumps            | PostgreSQL | Medium-Low  | With sync delays      |
| RetiredBilling_2020 | Legacy billing backup              | Oracle     | Low         | Cold migration        |
| Dev_Tests_DB        | Dev test environment               | MySQL      | Non-prod    | Low risk              |

---

### Migration Strategy: Trickle Approach + File Sync

**Why Trickle Migration?**

* Trickle (or online/live) migration syncs source and destination databases gradually while source remains active.
* It minimizes downtime and allows multiple verification checkpoints.

#### Phases of Trickle Migration

**1. Inventory and Assessment**

Begin by identifying all non-critical databases that are in scope for migration. For each database, capture metadata including the database type (e.g., SQL Server, Oracle, PostgreSQL), size, number of users, integration points, peak usage times, and business ownership. This step ensures only appropriate candidates are selected and that each has a clearly defined stakeholder responsible for validation. Tools such as Azure Migrate should be leveraged to automate the discovery and assessment process, enabling consistency and speed.

**2. Dependency Mapping**

Once the databases are inventoried, it's crucial to identify all upstream and downstream dependencies. These include scheduled jobs (like ETLs), analytical dashboards, middleware, or internal applications that read from or write to the databases. Tools like the Azure Migrate Dependency Visualization or Service Map can reveal these relationships at the process and port level. This step ensures that any system that might break due to a change in the database endpoint is accounted for ahead of time.

**3. Selection of Migration Tool**

Based on the database engine (e.g., SQL Server, PostgreSQL, Oracle) and the scale of the data, select an appropriate migration tool. For relational databases, the recommended solution is Azure Database Migration Service (DMS) in online mode . This mode supports continuous synchronization , making it ideal for trickle migrations. DMS allows minimal downtime cutovers while preserving data integrity.

**4. Provisioning the Target Environment**

Set up the destination Azure database infrastructure to mirror the source database. This could include:

* Azure SQL Database for modern SQL workloads,
* Azure Database for PostgreSQL (Flexible Server) for Postgres workloads, or
* IaaS-based VMs running SQL Server or Oracle for compatibility-critical scenarios.

Ensure that networking, firewall rules, authentication methods (e.g., managed identities or AAD), and performance tiering are aligned with operational requirements and SLAs.

**5. Initial Full Load (Read-Only Target)**

Kick off the initial full data and schema migration. During this phase, the entire schema and existing data are copied to the Azure target. The source database continues to remain live and operational, while the target remains in a read-only state. This step lays the foundation for the subsequent continuous synchronization of delta changes.

**6. Continuous Synchronization of Changes**

After the initial full load, the continuous syncphase begins. This is where ongoing changes made to the source database are replicated to the Azure target in near real-time.

This is typically achieved using:

* **Change Data Capture (CDC):** Tracks inserts, updates, and deletes in source tables by capturing changes into system tables.
* **Log-Based Replication:** Reads database transaction logs directly to replicate changes efficiently and with minimal overhead.

Azure DMS handles this automatically for supported databases. The goal is to ensure that the Azure target is kept fully synchronized with the live source — without affecting source performance or requiring any downtime during this replication period.

This phase can run for days or weeks, depending on cutover readiness and business constraints.

**7. Cutover and Final Switch**

When all stakeholders have validated the Azure target and the continuous replication is stable, schedule a planned cutover during a predefined low-activity window. During this short downtime:

* Disable all writes to the source database.
* Allow DMS to perform a final delta sync to replicate any last-minute transactions.
* Update application connection strings and switch all integrations to point to the Azure target.

After validation, the Azure-hosted database becomes the system of record.

**8. Monitoring, Validation, and Decommissioning**

Post-cutover, conduct comprehensive validation:

* Run data integrity checks and sampling queries to compare source vs. target.
* Validate application functionality, scheduled jobs, and analytics reports.
* Review audit logs and replication metrics.

Once the new system is stable,  decommission the on-premises database servers, retire unused infrastructure, and update CMDB entries. This also marks the point to archive or destroy redundant backup media to reduce compliance risk and operational overhead.

---

### Syncing Legacy File-Based Databases with Azure File Sync

AGW still operates several file-based databases and engineering tools that store outputs in flat files, such as:

* SQLite
* Access
* CSV-based telemetry logs
* Old SCADA or GIS data dumps

These cannot be migrated using DMS and must be retained in original structure.

| Use Case                             | Solution                         | Benefit                                   |
| ------------------------------------ | -------------------------------- | ----------------------------------------- |
| Flat-file logs used by analytics     | Azure File Sync + DFS            | Cloud storage with local cache            |
| Access DBs still used by field ops   | Azure File Sync                  | Retain folder structure, modern backend   |
| Reporting systems writing .csv files | Azure Files + Azure Data Factory | Cloud-accessible, transformable data flow |

---

## Cloud Tiering Model in Azure File Sync

For AGW, where a significant portion of the data migration involves non-critical database files and long-term archival, optimizing the Azure File Sync tiering model is crucial for cost-efficiency and performance. The following configurations are recommended:

#### **Configuration Settings:**

* **Volume Free Space Policy: Set to 20-30%**: This ensures that sufficient space is consistently available on the local storage for active workloads and file operations. By maintaining a 20–30% buffer, AGW can avoid performance degradation while offloading the least accessed files to the cloud when space is tight.
* **Date Policy: Enable with a threshold of 90+ days**: Files that have not been accessed for a minimum of 90 days will be tiered to Azure Files, helping AGW achieve significant cost savings by offloading infrequently used data. This policy is particularly effective for archival data, ensuring that only essential files remain on local storage.

#### **Expected Benefits:**

* **Cost Efficiency:** The tiering mechanism will ensure that older, infrequently accessed data is moved to Azure Files, resulting in substantial storage savings. This aligns with AGW’s goal of optimizing both on-premises and cloud storage costs.
* **Performance Preservation:** By maintaining a 20–30% free space buffer on the local servers, the performance of critical systems is not impacted, as local storage remains sufficiently available for active workloads.
* **Low Administrative Overhead:** Once configured, the volume free space and date policies require minimal ongoing management. Local storage and cloud tiering will adjust automatically based on usage patterns, minimizing administrative effort while ensuring continuous optimization.

This configuration will allow AGW to achieve a balance between cloud storage efficiency and on-premises performance while ensuring that long-term archival data is managed seamlessly with minimal intervention.

---

### Tools and Services Overview

| Purpose                             | Tool/Service                         | Description                                               |
| ----------------------------------- | ------------------------------------ | --------------------------------------------------------- |
| Database Migration                  | Azure Database Migration Service     | Online (trickle) migration with CDC or log-based sync     |
| File Sync for legacy data           | Azure File Sync                      | Sync on-prem folders to Azure Files, retain access models |
| Data movement/transformation        | Azure Data Factory                   | Schedule file-based ETL flows                             |
| Discovery & dependency map          | Azure Migrate, Dependency Visualizer | Visualize and assess impact                               |
| Performance optimization (Optional) | SQL Insights, Query Store            | Post-migration tuning and analytics                       |
| Security & compliance               | Defender for SQL                     | Vulnerability scanning, encryption enforcement            |

### Example Migration Scenario

#### Database: `FieldSensor_Archive`

* Type: PostgreSQL
* Size: 200 GB
* Use: Stores daily field data dumps
* Access Pattern: Append-only, used weekly by analytics team

**Migration Flow:**

1. Provision Azure Database for PostgreSQL – Flexible Server.
2. Setup Azure DMS with trickle sync mode.
3. Perform full data load over 48 hours (initial sync).
4. Enable Change Tracking to replicate new inserts.
5. After 1 week, test failover to Azure DB during maintenance window.
6. Switch client apps and retire on-prem instance.

---

### When to Use Other Approaches

This section helps stakeholders decide if trickle migration is not suitable and an alternative method should be used  based on specific scenarios . It doesn’t replace trickle for the entire milestone—it provides exceptions.

| Scenario                                                | Suggested Alternative Approach        | Reason                                                               |
| ------------------------------------------------------- | ------------------------------------- | -------------------------------------------------------------------- |
| Small-sized databases (<5 GB)                           | Full Dump + Restore                   | Trickle overhead isn't worth it. A one-time export/import is faster. |
| App tightly coupled with DB (can’t separate)           | Azure Migrate (VM-level rehost)       | Move entire VM (app + DB) together for consistency.                  |
| Schema needs major rework (e.g., multi-tenant refactor) | Replatform to Azure SQL / CosmosDB    | Redesign schema during migration for modernization.                  |
| Custom/Flat-file DBs (e.g., SQLite, Access)             | Azure File Sync or Azure Data Factory | These are non-relational, best handled via file or batch sync.      |

### Use Trickle when:

* DBs are relational (SQL, Oracle, PostgreSQL).
* You want minimal downtime.
* Schema is compatible with Azure target services.
* Data changes can be tracked and synced.

### Consider Other Approaches when:

* DB is very small.
* Coupled tightly with legacy apps.
* Schema overhaul is needed.

---

### Cost and Environmental Impact

* Trickle approach minimizes unplanned outages and business impact.
* Moving to Azure eliminates aging server costs and reduces electricity use.
* Decommissioning legacy servers leads to significant OPEX savings.
* Azure SQL and PostgreSQL managed services offer auto-scaling and reserved instance pricing for savings.
* Microsoft data centers are carbon-neutral, further reducing AGW’s environmental impact.

---

### Summary Table

| Activity                   | Tool/Service Used                 | Value Delivered                             |
| -------------------------- | --------------------------------- | ------------------------------------------- |
| Inventory + Assessment     | Azure Migrate, Stakeholder Review | Classify and shortlist eligible systems     |
| Relational DB Migration    | Azure DMS (Trickle)               | Low-downtime transfer with sync checkpoints |
| File-based DB Sync         | Azure File Sync, Data Factory     | Retains legacy structure in a modern cloud  |
| Target Environment         | Azure SQL, PostgreSQL FS          | Secure, scalable, managed databases         |
| Dependency Mapping         | App Dependency Visualizer         | Reduces migration risk                      |
| Post-Migration Monitoring  | Azure Monitor, SQL Insights       | Tune and validate performance post-cutover  |
| Environmental Optimization | Decommissioning legacy infra      | Cost savings + carbon footprint reduction   |
