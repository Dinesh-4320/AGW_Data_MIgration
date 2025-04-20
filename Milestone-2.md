# Milestone 2: Migrating Archival Data

### Things to ask the stakeholder:

| Question                                                        | Assumption for typical large water management company                                                                      |
| --------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------- |
| How is the archival data stored on-premise?                     | SAN/NAS appliances with minimal automation and Legacy data often exists in CSV, XML                                        |
| How long does the archival data needs to be retained?           | 5 to 15 years                                                                                                              |
| What categories or types of archival data exist?                | Historical sensor readings , maintenance logs ,  billing records ,  compliance reports etc..                               |
| How frequently is archival data accessed?                       | Very rarely — mostly for audits ,  legal inquiries , or  historical analysis . Possibly once or twice a year per dataset. |
| What is the total estimated size of archival data?              | Could range from 10–100 TB                                                                                                 |
| Does AGW already have a retention policy?                       |                                                                                                                             |
| How is backup currently handled for archival data?              |                                                                                                                             |
| Is there a need to transform or reformat data during migration? | Yes — legacy formats like CSV/XML may need transformation to Parquet/JSON for analytics                                    |
| What security and access controls are currently in place?       | Likely basic file share permissions with minimal  encryption, MFA.                                                         |

### Archival Data Migration Plan

#### Chosen Azure Technologies

Based on the above, we recommend:

| Requirement                       | Chosen Azure Service or Tier                                |
| --------------------------------- | ----------------------------------------------------------- |
| Cost-effective long-term storage  | Azure Blob Storage - Archive Tier                           |
| Medium retention, semi-active use | Azure Blob Storage - Cold Tier                              |
| Bulk upload of large datasets     | Azure Data Box (100 TB Physical Device)                     |
| Data lifecycle management         | Azure Blob Lifecycle Management Policies                    |
| Format optimization               | Azure Data Factory (for converting CSV/XML to Parquet/JSON) |

---

### Migration Approach: Big Bang vs Phased

For AGW, we recommend a **phased migration** approach, not a big bang approach.

#### Why not Big Bang?

- Big bang means moving all data in one go, often during a fixed downtime window.
- Considering AGW's archival data is large (up to 100 TB), scattered, and not actively used.
- A big bang needs massive network or logistics coordination and has higher risk.
- If one issue happens (e.g. data format error, failed batch), entire migration delays.

#### Why Phased is Better?

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

This helps minimize manual intervention while aligning with compliance goals. While creating and running the lifecycle management policies themselves are free, one will be charged for the operations involved in moving blobs between different storage tiers.

#### Step 3: Data Transformation and Format Optimization

Use Azure Data Factory (ADF) to transform and optimize data before ingestion:

- Convert CSV or XML formats into Parquet for sensor and log data.
  - Parquet supports columnar storage and compression, reducing storage size.
- Apply schema enforcement and tagging for easier querying later via Azure Synapse or Purview.
- Store only transformed (final) version in Blob Storage. Intermediate files processed by ADF are temporary and deleted post-transformation.
- Azure Data Factory is billed per activity run, data volume processed, and data movement. The cost remains low when used only for batch transformation.

ADF is not a storage service—it only prepares and moves data. The final output (Parquet) is stored in Azure Blob.

---

### Step 4: Migrate Data

Depending on data size, different migration methods are recommended.

**A. For Large Datasets (>10 TB):**

- Use Azure Data Box (e.g. 100 TB capacity, ~80 TB usable).
- Workflow:
  1. Order Data Box via Azure portal.
  2. Receive encrypted device at AGW datacenter.
  3. Copy bulk data from SAN/NAS using standard tools (e.g. Robocopy, rsync).
  4. Optional: run transformation jobs (CSV to Parquet) before or after copy.
  5. Ship device back to Microsoft.
  6. Microsoft ingests data securely into specified Azure Blob containers.
- The transfer is encrypted end-to-end and tracked via portal.

**B. For Small Datasets or Pilot Runs (<5 TB):**

- Use internet-based upload methods:
  - **AzCopy**: CLI tool for large file batch uploads.
  - **Azure Storage Explorer**: GUI for manual upload/management.
- Pre-process and compress data to reduce transfer time.
- Recommended for early validation, metadata tagging tests, or automation script development.

**C. Verification and Logging:**

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
| Migration Method      | Azure Data Box, AzCopy        | Secure and fast transfer (offline or online)         |
| Transformation        | Azure Data Factory            | Optimizes format, improves queryability              |
| Environmental Benefit | Retire old SAN/NAS hardware   | Reduced power use, Microsoft’s green infrastructure |

### References:

1) https://medium.com/@shalinpatel./identify-azure-data-migration-options-6687dd51073d
2) https://medium.com/@syedhaseebahmad97/all-you-need-to-know-about-azure-data-factory-2c35ec1a833f
3) https://medium.com/@11amitvishwas/life-cycle-management-for-azure-blob-d67b2cb0c7a8
