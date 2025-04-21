# Milestone 3: Networking and Identity Service Setup

## Netoworking setup

### Data needed from the stakeholder as part of phase 1

1. Rough bandwidth usage requirements (Service Domain).

2. Office locations and region wise breakdown of public facing services, employee facing services and bandwidth utilization.

3. Name resolution services information. 

4. Existing enterprise office 365 subscription information.

### Secure networking access to cloud resources

##### High bandwidth utlization regions or regions near 'peering locations'

- Use of Expressroute is prefered as it is faster, secure, reliable and have lower latency than typical connections over the internet with addition of S2S as a fail-over connection mechanism.

- No data inbound costs and outbound costs are much lower over ExpressRoutes, and have options for Unlimited Data Plans with ExpressRoute direct.

- Setup can be lowered even further if the office spaces fall under "ExpressRoute Metro" regions with much as 45% discount.

##### Low bandwidth utlization regions or remote regions

- Use of S2S is prefered in such regions as they cheaper than ExpressRoute availble to the region.

- Setup is quick and scalable and connections can be downgraded to reduce expenses during off-hours

- While the networking price can be as high as 60% more but the saved upfront monthy costs make up for it.  

##### Indivisual or automated systems

- Use of P2S VPN gateways can be employed such setups where remote employees can connect directly to the Azure resources without the use of self hosted solution to reduce IT overhead and simplicity.

- P2S VPN gateways, similar to S2S gateways are resizable and expenses can be saved during off-hours. 

##### Inter-Networking between VNets in azure

- A plan must be made on what VNets will be paired in advance to efficiently manage costs.

- Alternatively, S2S VPN can be used to reduce the expenses by 50%

- By not peering all the VNets, IP ranges and Expenses can be reduced.  

##### Setup Entra ID

#### Domain Creation

1. **Choose and Add Domain**  
   
   Select a unique domain name for your organization and add it to Entra ID via the Azure portal. Verify domain ownership by adding a TXT record to your DNS.

2. **Configure DNS**  
   
   Set up DNS records (CNAME, MX) to enable identity services like Azure AD Join and SSO. For Office 365, these settings are automated.

3. **Federation (Optional)**  
   
   Set up federation with on-premises Active Directory for SSO, using ADFS or another identity federation solution.

4. **Enable External Identities**  
   
   Allow collaboration with external users by configuring B2B or B2C identities.

5. **Assign Admin Roles**  
   
   Assign appropriate roles (e.g., Global Admin, User Admin) to control access and manage Entra ID.

6. **Enable MFA**  
   
   Activate multi-factor authentication (MFA) for added security on admin and sensitive accounts.

7. **Final Review**  

8. Test domain setup and identity services to ensure everything is working as expected.
