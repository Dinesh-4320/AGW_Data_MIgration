# Milestone 2: Migrating Archival Data                                            |

### Stakeholder Considerations for Archival Data Migration

| Aspect                               | Recommendation / Mentor Guidance                                                                                                                            | Risk / Gap Identified                                                                         |
| ------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------- |
| **Current Storage Medium**     | Archival data is stored on SAN/NAS appliances with minimal automation.                                                                                      | Legacy infrastructure with limited scalability or integration with cloud-native services.     |
| **Data Formats**               | Predominantly CSV/XML; transformation to Parquet/JSON is**strongly recommended** for analytics and efficiency.                                        | Retaining legacy formats hinders analytics, increases storage, and reduces performance.       |
| **Retention Period**           | Typically ranges between 5–15 years, based on compliance requirements.                                                                                     | Retention timelines may not be clearly documented or enforced.                                |
| **Data Categories**            | Includes historical sensor readings, maintenance logs, billing records, and compliance documents.                                                           | Lack of classification may lead to over-retention or loss of critical data.                   |
| **Access Frequency**           | Data is accessed very rarely — typically for audits, legal needs, or long-term trend analysis.                                                             | Over-provisioning resources for infrequent access increases operational cost.                 |
| **Total Data Volume**          | Estimated between 10–100 TB, based on similar water utility organizations.                                                                                 | Storage growth without management may increase cost and complicate migration.                 |
| **Retention Policy**           | Appears to be undocumented or loosely followed; creation and enforcement of retention rules is essential.                                                   | Absence of formal policies leads to regulatory non-compliance and operational inefficiencies. |
| **Backup Strategy**            | Not clearly defined; backup and disaster recovery processes must be reviewed and integrated into cloud policies.                                            | Incomplete backups pose a high risk of data loss during or post migration.                    |
| **Access Controls & Security** | Existing controls are likely basic — typically limited to file share permissions; recommend encryption, role-based access, and MFA for sensitive datasets. | Weak security increases risk of unauthorized access, data leakage, and compliance violations. |
| **Transformation Needs**       | Transformation from CSV/XML to Parquet/JSON must be part of the migration pipeline; automated ETL processes should be implemented.                          | Manual or delayed transformation reduces cloud compatibility and analytics readiness.         |
| **Source of Guidance**         | Recommendations are based on mentor discussions and internal reviews — these are guiding inputs, not posed as open-ended questions to stakeholders.        | Lack of stakeholder validation may require future alignment and feedback loops.               |

### Archival Data Migration Plan

#### Chosen Azure Technologies

Based on stakeholder discussions and mentor guidance, we recommend:

| Requirement                       | Chosen Azure Service or Tier                                |
| --------------------------------- | ----------------------------------------------------------- |
| Cost-effective long-term storage  | Azure Blob Storage - Archive Tier                           |
| Medium retention, semi-active use | Azure Blob Storage - Cold Tier                              |
| Secure upload over private link   | Azure ExpressRoute                                          |
| Data lifecycle management         | Azure Blob Lifecycle Management Policies                    |
| Format optimization               | Azure Data Factory (for converting CSV/XML to Parquet/JSON) |

---

### Migration Approach:

For AGW, we recommend a **phased migration** approach.

- Phased migration splits data by category (e.g. sensor logs, billing) or by year.
- Each phase can be tested, transformed, and verified before moving to the next.
- Reduces operational risk, avoids disruption to ongoing operations.
- Easy to monitor storage cost and usage per data group.

Example phase plan:

| Phase | Data Group             | Storage Tier | Estimated Size | Notes                               |
| ----- | ---------------------- | ------------ | -------------- | ----------------------------------- |
| 1     | Compliance Reports     | Archive Tier | 10 TB          | Rarely used, good first candidate   |
| 2     | Historical Sensor Data | Cold Tier    | 30 TB          | Medium retention, used for analysis |
| 3     | Billing Records        | Cold Tier    | 20 TB          | Needed for audits/legal             |
| 4     | Maintenance Logs       | Archive Tier | 15 TB          | Rarely needed, archive suitable     |

---

### Step-by-Step Migration Plan

#### Step 1: Setup Blob Storage with Lifecycle Tiers

- Create a separate Azure Storage Account for archival data.
- Define blob containers based on data type (e.g. /sensor-data/, /compliance/, /billing/).
- Use Archive Tier for rarely accessed data like compliance history or legacy logs.
- Use Cold Tier for moderately accessed datasets such as billing or sensor snapshots.

#### Step 2: Configure Retention and Lifecycle Management Policies

Azure Blob Lifecycle Management is critical to automate transitions and optimize costs.

Example lifecycle policy setup:

- On upload, all data defaults to Cool or Cold tier.
- After 90 days: move to Cold if not modified.
- After 180 days: move to Archive tier.
- After 10 years: delete if not accessed (subject to retention rules).
- Policies can be container- or metadata-based (e.g. move only if tag = "archive").

This helps minimize manual intervention while aligning with compliance goals.

#### Step 3: Data Transformation and Format Optimization

Use Azure Data Factory (ADF) to transform and optimize data before ingestion:

- Convert CSV or XML formats into Parquet for sensor and log data.
  - Parquet supports columnar storage and compression, reducing storage size.
- Apply schema enforcement and tagging for easier querying later via Azure Synapse or Purview.
- Store only transformed (final) version in Blob Storage. Intermediate files processed by ADF are temporary and deleted post-transformation.
- Azure Data Factory is billed per activity run, data volume processed, and data movement. The cost remains low when used only for batch transformation.

#### Step 4: Migrate Data via ExpressRoute (Off-Peak)

As archival data is **not business-critical**, it can be uploaded securely via **ExpressRoute** during off-peak hours without business disruption.

- Use **AzCopy** for high-throughput parallel uploads.
- Schedule transfers during nights or weekends to reduce bandwidth contention.
- Compress data before transfer to reduce time and cost.
- Integrate with ADF pipelines if transformation is needed before ingestion.
- Monitor upload performance and validate integrity using checksum-based verification.

---

### Verification and Logging

- Use MD5 checksum or Azure import logs to confirm file integrity.
- Enable Azure Monitor and diagnostic logs to track access, updates, and billing.

---

### Cost and Environmental Benefits

- Archive Tier is up to 80% cheaper than Hot storage.
- Eliminates maintenance of local SAN/NAS, backup hardware, and cooling infrastructure.
- Microsoft Azure data centers run on renewable energy, reducing AGW’s carbon footprint.
- Pay-as-you-go billing ensures cost control as more data phases are migrated.

---

### Summary Table

| Activity              | Tool/Service Used             | Benefit                                              |
| --------------------- | ----------------------------- | ---------------------------------------------------- |
| Storage Setup         | Azure Blob (Cool + Archive)   | Low-cost, scalable, tiered storage                   |
| Lifecycle Management  | Azure Blob Lifecycle Policies | Auto-tiering based on age and access pattern         |
| Migration Method      | ExpressRoute + AzCopy         | Secure, private, scheduled during off-peak hours     |
| Transformation        | Azure Data Factory            | Optimizes format, improves queryability              |
| Environmental Benefit | Retire old SAN/NAS hardware   | Reduced power use, Microsoft’s green infrastructure |

### References:

1) https://medium.com/@shalinpatel./identify-azure-data-migration-options-6687dd51073d
2) https://medium.com/@syedhaseebahmad97/all-you-need-to-know-about-azure-data-factory-2c35ec1a833f
3) https://medium.com/@11amitvishwas/life-cycle-management-for-azure-blob-d67b2cb0c7a8
