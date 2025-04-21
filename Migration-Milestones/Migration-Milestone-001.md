# Milestone 1: Identification and Classification of Resources and their Criticality

## Step 1: Define the Scope

**Objective:** Identify all resources, classify them by type and criticality, and understand their dependencies.

### Stakeholder Discovery and Interviews

A series of calls will be scheduled and conducted with business and technical stakeholders to gather information about existing applications, servers, dependencies, and ownership structures. This process is crucial in building a foundational understanding of the on-premise infrastructure before initiating migration.
all the questions to be asked to the stakeholders, technical teams present are included in a document named "migration_interview_questions.md"

**Key activities included:**
- Discover with business stakeholders
- Set up interview calls to discuss migration queries
- Discuss resources, ownerships, dependencies, requirements, relations, and impact (also known as Migration DDA - Discovery, Dependency, Assessment)

**Key Questions:**
- What are the resources? (e.g., applications, databases, IoT devices)
- Who owns or manages these resources?
- What is the criticality of each resource to business operations?
- Are there any existing dependencies?

## Step 2: Data Collection

### Conduct Stakeholder Interviews
- Schedule meetings with business stakeholders to gather information.
- Prepare a list of questions related to resource usage, ownership, dependencies, and criticality.

**Example questions:**
- "Which resources are indispensable for daily operations?"
- "What dependencies exist between these resources?"

### Audit Existing Systems
- Use CMDB (Configuration Management Database) tools or create a model CMDB.
- Leverage discovery tools like Azure Migrate or ServiceNow for initial inventory collection.

## Step 3: Classification Framework

**Define Criticality Levels:**
- **High:** Resources essential to operations with significant business impact if disrupted.
- **Medium:** Important resources with moderate impact if disrupted.
- **Low:** Non-essential resources.

## Step 4: Proposed Deliverables

### CMDB Model

Create a sample or prototype CMDB.  
Include columns for resource name, type, criticality, owner, dependencies, and other metadata.

**Example Columns:**
- Resource Name
- Type
- Owner Department
- Criticality
- Dependencies
- Description
- Environment
- Location
- Status
- Used in Daily Operations

_This model is present in `AGW_CMDB_model.xlsx`._

### Resource Map

Create a visual representation of resources and their interdependencies.

**Key Dependency Types:**

| Type of Dependency        | Meaning                                                  |
|---------------------------|----------------------------------------------------------|
| Control & Data Feedback   | Bidirectional real-time control and response             |
| Data Input                | Resource passively receives data from another            |
| Authentication Backbone  | Requires access control and user identity verification   |
| Financial Input           | Relies on cost/salary/purchase information to function   |
| Communication             | Hardware/software that facilitates data transmission     |
| Analytical                | Processes input from other systems for insights          |
| Aggregation               | Gathers data from multiple sources                       |
| Visualization             | Requires structured data from other systems to display   |
| Regulatory Evidence       | Collects logs and data to prove compliance               |

_The formatted dependency table is present in `Resource_Dependency_Map.xlsx`._

### Tools to Use

- Azure Resource Manager (ARM) for identifying Azure resources.
- CMDB Tools like ServiceNow or OpenDCIM.
