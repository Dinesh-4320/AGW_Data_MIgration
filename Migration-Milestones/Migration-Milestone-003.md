# Milestone 3: Networking and Identity Service Setup

## Data Received from Stakeholders & Identified Risks
- **Data Received:**
  1. Rough bandwidth usage requirements (Service Domain).
  2. Office locations and region-wise breakdown of public facing services, employee facing services, and bandwidth utilization.
  3. Name resolution services information.
  4. Existing enterprise Office 365 subscription information.
- **Identified Risks:**
  - Lack of detailed bandwidth analytics for long-term network performance evaluation.
  - Incomplete specifics on Office 365 configurations and automated system parameters.
  - Potential gaps in security and performance insights that may affect the reliability of future architectural decisions.

## Networking Setup

### Secure Networking Access to Cloud Resources

##### High Bandwidth Utilization Regions or Regions Near Peering Locations
- Use of ExpressRoute is preferred as it offers faster, secure, and reliable connectivity with lower latency; S2S is used as a fail-over mechanism.
- No data inbound costs and significantly lower outbound costs with options for Unlimited Data Plans via ExpressRoute Direct.
- Cost reductions of up to 45% available if office locations qualify for "ExpressRoute Metro" regions.

##### Low Bandwidth Utilization Regions or Remote Regions
- S2S is recommended in these areas for its cost effectiveness.
- The setup remains quick and scalable, with the option to downgrade connections during off-hours to reduce expenses.
- Although costs might be up to 60% higher, the upfront savings compensate for the increased rates.

##### Individual or Automated Systems
- P2S VPN gateways allow remote employees to connect directly to Azure resources without requiring self-hosted solutions, lowering IT overhead.
- Like S2S, these gateways are resizable, and savings can be optimized during off-peak hours.

##### Inter-Networking between VNets in Azure
- Plan VNet pairing in advance to effectively manage costs.
- Alternatively, leverage S2S VPN to reduce expenses by up to 50%.
- A selective peering strategy helps minimize IP range overlaps and associated costs.

### Setup Entra ID (Architectural Overview)
For architectural purposes, the Entra ID setup should include:
- Selection and validation of a unique domain name through DNS (using TXT record verification).
- Configuration of necessary DNS records (e.g., CNAME, MX) to support identity services.
- An evaluation of whether to implement federation with on-premises Active Directory for SSO.
- Enabling external identity management and assigning appropriate administrative roles.
- Setting up multi-factor authentication to ensure enhanced security.
