#  Migrating Application services to Azure

##  Overview

We're migrating **On-Premise applications** of a water-based company to **Azure Cloud**. The applications can be categorized as **Critical** and **Non-Critical**

---

##  What to Communicate to Stakeholders

| Topic                          | Message                                                                 |
|--------------------            |-------------------------------------------------------------------------|
| Why re-platforming?            | Enables Cloud-native features like Scalablity, Security and Cost control .     |
| What is the scope of migration? | The parts of the application will be moved and will remain on-premises. Modernizing any old applications. |
| What will be the impact on Business? | Maximum downtime expected during and post migration. Business loss incurred when app is unavailable. |
| Any Rules and Regulations needs to complied? | Regulations like GDPR needs to be followed |

---

##  Tools & Services to Use

| Tool/Service             | Purpose                              |
|--------------------------|--------------------------------------|
| Azure Migrate            | Discovery, assessment, migration     |
| Azure Virtual Networks   | Establising Network environment and isolating infrastructure components in subnets |
| Azure Express Route      | For secure connectivity | 
| Azure Virtual Machines   | Compute alternaties for On Prem Servers |
| Azure Kuberentes Service | For Containerized Applications |
| Azure App Service        | Deploying APIs or Apps without Infrastructure Management |
| Azure Active Directory   | For Identity Management              |
| Entra ID                 | Authentication and Access Control    |
| Azure Functions          | For Serverless Computing             |
| Azure Arc (Optional)     | Hybrid/on-prem integration           |

---
## Step-by-Step Migration Plan

This plan is structured to be followed in phases, ensuring minimal downtime, controlled change, and successful validation.

### Phase 1: Discovery & Inventory (Preparation)
- ##### Inventory Analysis: 
    It involves creating a detailed catalog of all applications, databases, and infrastructure components, including their specifications, versions, and usage patterns. Tools like Azure Migrate can automate this process, providing insights into the existing on-premises environment.
- ##### Dependency Mapping: 
    This critical step identifies the web of interconnections between applications and services. It helps understand which components rely on each other.
- ##### Performance Baseline: 
    Establishing On Prem performance metrics is essential for post-migration comparison. This includes CPU usage, memory utilization, storage I/O, and network throughput. These baselines will help right-size Azure resources and validate the success the Azure cloud migration.
- ##### Compliance and Security Review: 
    It is important to identify any industry-specific regulations that the Azure environment must adhere to. Current security protocols, including data encryption, access controls, and network security, should be reviewed.

### Phase 2: Assessment & Planning
This phase focuses on creating a comprehensive roadmap for migration based on the insights gathered during assessment.

- ##### Defining Migration Goals: 
    Clearly articulate what the migration aims to achieve with **Slight Replatforming**.

- ##### Selecting The Appropriate Migration Approach: 
    Azure supports various migration strategies:
    - **Rehost (Lift and Shift)**: Moving applications without major changes.
    - **Refactor**: Making minimal changes to optimize for the cloud.
    - **Rearchitect and Rebuilding** *(Only if Necessary)*: Modifying the application architecture significantly or again creating from scratch.

- ##### Creating A Detailed Migration Roadmap: 
    It should include a timeline for each phase of the migration. Significant things to be taken care of.
    - Cutover windows (off-peak hours)
    - Parallel run/testing phase
    - Rollback trigger conditions

- ##### Plan Target Azure Architecture
    The target architecture should most probably replicate the On Premises Architecture substituted with Azure Native Resources.

- ##### Developing A Risk Mitigation Plan: 
    Identify potential risks in the migration process and develop strategies to mitigate them. This includes plans for data backup, rollback procedures, and strategies for minimizing downtime during migration.

### Phase 3: Migration Execution
This phase involves the actual implementation of the Aazure cloud migration strategy. It's where planning turns into action.

- ##### Setting Up the Azure Environment:
    It involves configuring the Azure infrastructure to mirror and improve upon the on-premises environment. Key steps include:
    - Establishing Azure Virtual Networks that align with the on-premises network topology.
    - Implementing Azure ExpressRoute or VPN for secure connectivity.
    - Setting up Azure Active Directory for identity management.
    - Configuring Azure Security Center and Azure Monitor for security and monitoring.

- ##### Application Migration: 
    The approach here depends on the chosen migration strategy
    - For rehosting, Azure Site Recovery can be used to lift-and-shift VMs.
    - Refactoring involves containerizing applications using Azure Kubernetes Service.
    - Rearchitecting could mean leveraging Azure App Service or Azure Functions for serverless computing.
    - Establishing connectivity data resources.

---
### Structured application migration strategy for Critical Applications.

1. ##### Active-Passive Setup (On-Prem Active, Azure Passive)
    - Ensure app can replicate data/state to cloud (stateless preferred or use DB replication).Identify dependencies and latency sensitivity.
    - Use Azure Site Recovery (ASR) or geo-redundant storage for failover. Manual or semi-automated failover using Traffic Manager or DNS switch. 

2. ##### Active-Active (On-Prem + Azure)
    -  App must support distributed deployment with servers available in both Azure and On Premx. Data layer sync with active replication of On Prem Databases.

3. ##### Passive-Active (Azure Active, On-Prem Passive)
    - Transition Azure to handle primary traffic. Keep on-prem ready for failover.
    - Use Azure Backup, Snapshotting, and Load Balancer failback options.

4. ##### Migrate Fully to Azure (Cloud-Only)
    - Fully transition compute, storage, and identity to Azure. Both Active and Cold standby should be handled in Azure. Decommission on-prem infrastructure with minimal loss.

5. ##### Optimize for Cost, Performance, and Security
    - Employ Auto-scaling feautures during peak hours. Scale Down idle resources based on monitoring feeds. Enforce security ensuring compliance and regulations
---
### Rollback Plan
1. ##### Pre-migration preparation. 
    - This involves taking full system and database backups of the source environment, including all configuration files, environment variables, and secrets. Snapshots of virtual machines or container images should also be created. All the snapshots should be tagged to keep track of versions and rollback implementation. 
    - It is also important to retain full access to DNS management tools during and after the migration.

2. ##### Parallel System Setup
    - The original application environment should be kept operational—preferably in a read-only state—until the Azure deployment is fully validated. Smoke tests covering all critical functionality (like login, read/write operations, and external service integrations) must be performed on the Azure environment. 


3. ##### Rollback Triggers
    - These include unresponsiveness of the application, failures in authentication or key integrations, significant performance degradation, or any form of data inconsistency or corruption. When any of these issues are detected, rollback must be initiated without delay. The first step is to revert DNS records to point back to the original infrastructure.
---
### Benefits of migrating to Azure
- ##### Scalability with Containers & VMs: 
    Easily scale applications using Azure Kubernetes Service (AKS) or VM Scale Sets to handle fluctuating workloads with minimal manual effort.

- ##### High Availability
    Azure ensures high availability through availability zones, load balancers, and global distribution with Service Level Agreements (SLAs) up to 99.99%.

- ##### Use of Arm64 SKUs
    Arm64-based VMs offer improved performance-per-watt, reduce compute costs, and are ideal for containerized apps, web services, and microservices.

- ##### Security: 
    Azure provides built-in threat protection, encryption, and compliance tools like Microsoft Defender and Entra ID for enterprise-grade secure access.

- ##### Cost Savings:
    Azure offers reserved instances, spot VMs, and autoscaling to optimize resource usage and significantly lower infrastructure and operational costs.
