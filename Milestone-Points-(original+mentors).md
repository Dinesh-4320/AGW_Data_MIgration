### **General misc. pointers shared common to all milestones:**

- Include wherever possible:
  
  - Accomplishments
  
  - Tools to be used
  
  - Process Chart
  
  - Ballpark cost saving (if possible), how we can save more costs

- Try to keep the milestones on a single page

- Think about different tiers and SKUs for storage, compute, and networking.

- Explain about (Cost or Otherwise) benefits of using cloud

- Keep it simple, don't complicate

### **Milestone 1:** Identification and Classification of Resources and their criticality.

**Original Point**

- Think of moving archival-grade data to Azure with Azure Data Box to Azure Blob storage.

**Mentors' advice**

- Discover with business stakeholders

- Set up interview calls to discuss migration queries

- Discuss resources, ownerships, dependencies, requirements, relations, and impact

        (AKA Migration DDA)

- Work on CMDB (check out available applications for this)

**Expected Work**:

- Create a model CMDB (maybe even try to actually make a CMDB demo)

### **Milestone 2:** Migrate Archival Data

**Original Point**

- Think of moving archival-grade data to Azure with Azure Data Box to Azure Blob storage.

- Big Bang Approach

**Mentors' advice**

- Retention time data and if not already exists, create a retention policy

- Data age (is the data months old, years old, and how is it handled)

- Data generation frequency (daily, yearly, monthly, etc.)

- Consider data transformation

- Find alternative methods to Data Box. Check if Data Box is really the best option for migration of archival data

**Expected Work**:

- Explore possible data produced

- Check online what is generally the data age, retention policies

- How data can be transformed to help analysis in the future and/or help in efficient storage

- Examine if Data Box is the best solution; if not, find a more suitable solution for archival data migration

### **Milestone 3:** Networking and Entra ID setup/sync

**Original Point**

- Set up Entra ID tenant and synchronize with the local Active Directory

**Mentors' advice**

- Set up connectivity and networking on cloud before migration

**Expected Work**:

- Create a networking and connectivity plan, VPN/ExpressRoute

- Entra ID setup plan

- Network security

- Public accessibility plan

- Explore hybrid DNS

### **Milestone 4:** Migrate Non-Critical Databases and Data

**Original Point**

- Move non-critical application database & data

- Trickle Approach

**Mentors' advice**

- Get a list of all such databases

- Classify what kinds and what use cases actually fit "non-critical"

- Migration steps and strategy elaboration

- Tool to be used for the task

**Expected Work**:

- Make either a sample list of non-critical data or, if possible, a set of "non-critical" database/data identification rules

- Elaborate migration strategy, how trickle approach can be implemented. Check if better approaches exist for this.

- What tools can be used here. (One good tool is Azure Database Migration Service)

### **Milestone 5:** Migrate Non-Critical Applications

**Original Point**

- With slight re-platforming (authentication and storage access), move non-critical applications

- Think of Azure Migrate Service

- V2C Rehosting (Lift & Shift)

**Mentors' advice**

- Define re-platforming in this context. What to re-platform

- Coordinate with stakeholders

- List out activities involved in re-platforming

**Expected Work**:

- Check how Azure Migrate Service can be employed here to achieve the migration efforts, check if better methods exist. Elaborate on the process.

- Explore what components can be re-platformed. For more context: Authentication will change to Entra ID and storage method might change to File Shares, Azure Blob Storage, and Azure SQL Database (PaaS).

- Elaborate on what all activities can be a part of re-platforming efforts. An elaborate checklist or a general re-platforming workflow would be a good way.

- Consider re-architecting and re-factoring efforts to cloud-native or cloud-compatible systems to better leverage features provided by Azure.

- Consider moving to ARM64 SKUs for better price-to-performance ratio. Elaborate on the drawbacks with conditions when they apply.

- Consider moving to containerized applications for better scaling.

### **Milestone 6:** Migrate Critical Databases and Data

**Original Point**

- Critical database and data migration

- Zero-downtime approach (replicate and change over)

- Change over after the next step (Critical Application Migration)

**Mentors' advice**

- Explain in detail in technical terms how you will achieve this

- You cannot re-platform databases, only rehost. DB should be (typically) intact

- There should be no loss of rows and columns at least during the migration

- Define the change from what kind of database to what kind of database

**Expected Work**:

- Elaborate on zero-downtime database migration approach and explore better approaches which might be more efficient.

- Alternatives to simply rehosting—explore database options, for example Azure SQL as discussed above. Keep in mind the serverless Azure SQL might not be the best option for critical applications.

- Explore if there are better ways of critical data storage.

### **Milestone 7:** Migrate Critical Applications

**Original Point**

- Critical application migration

- Slight re-platforming to support cloud-based storage, database, and authentication/authorization

- Start with Active-Passive (On-Prem : Cloud), then move on to Active-Active, followed by Passive-Active, and then complete migration with only critical application being in the cloud (provided it is not critical to be on-prem)

**Mentors' advice**

- Similar to points defined in Milestone 5

**Expected Work**:

- Similar points from Milestone 5 also apply here

- Check for the best SKUs suitable for critical applications except use of ARM64 arch CPUs

### **Milestone 8:** Full Switch Over

**Mentors' advice**

- Define UAT (User Acceptance Testing)

- When is UAT considered fit for release.

- How will the change over take place

**Expected Work**:

- Define UAT environment, monitoring, and metrics

- Elaborate on how and when UAT can be considered good enough to be considered for General Availability

- Explore and elaborate on how the switch over can be performed
