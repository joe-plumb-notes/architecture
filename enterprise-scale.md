# Enterprise Scale Landing Zones
Focus on 'Ready' part of Cloud Adoption Framework - getting things set up in Azure, so that you can do what it is you want to do. **Readiness is implemented using Azure Landing Zones**. The landing zone should align to the desired operating model for Cloud - Decentralized Ops, focusing just on workloads, Central Operations where centrail IT controls the production enviornment, Enterprise Operations with centrally managed control, as well as free-form capabilities for innovation, or Distributed Operations, where multiple models are deployed across BUs.

This is important because without an environment that is aligned to the business priorities, handle cloud design principles, and enable them to adopt a cloud operating model, chance of success is dramatically reduced. To alleviate this risk, they need to have the tools to be **agile**, **drive innovation**, and have confidence that their environment is **secure**. This is what Azure Landing Zones are all about. And they're not tied to a particular stream of work (e.g. migration), they are flexible across all priorities, and are not locked in to IaaS or PaaS.

Most likely to interact with Enterprise Scale (for bigger deployments), or Start Small blueprints (which grow through the expansion, in line with CAF methodologies).

> "If you wish to make apple pie from scratch, you must first invent the universe."
> — Carl Sagan

### Qualifier: Is Enterprise Scale for me?
The enterprise-scale architecture is modular by design. It allows you to start with foundational landing zones that support your application portfolios, no matter whether the applications are being migrated or are newly developed and deployed to Azure. The architecture can scale alongside your business requirements regardless of scale point.

## What is Enterprise Scale?
Enterprise scale is an architecture approach that represents a strategic design path and a targeted operational state that follows the Cloud Adoption Framework. The architecture will evolve with Azure and allow customers to adopt new platform features at cloud speed.

Cloud enables you to scale resources based on demand, allowing organisations to transform the way they consume IT, shifting from CAPEX to OPEX. The power of unlimited scale can introduce governance problems, around the number of resources, and management of the estate. Enterprise Scale is the key to organizing resources in your landing zone. **The landing zone is built so that we know where to go, where items are organized, and where we provision resources when we need them**.

## Challenges that inspired ES
- Architectural complexity - lacking understanding of experience on Azure, and struggling to map their requirements into Azure components, concepts, and the security model.
- Operating Compatibility - trying to wrap existing processes around cloud operations (e.g. for server provisioning, waterfall review process of environment from a requirements, security perspective etc). When technical teams want to serve the business and chart their own destiny, a centralized management approach doesn't scale to support this. ES gives them great example of "how to do new".
- Lack of trust -> Desire for control. We need to understand what control central IT desire, and define these so that this can be managed by the platform, reducing burden of management, architectural review, sign-off everything etc. ES uses *Policy driven governance* so that the IT department maintain control, but dev teams can move at their own pace. 

- Subscriptions are used as a unit of scale and control. Helps the customer get on the right path, without having to refactor things, by using the guiding principles around Subs. (More on this later).

### What needs to be built?
Quite a lot! We need to consider, design, and build:
- Platform Ops and Management
- Platform DevOps
- Subscription Organisaiton and Governance
- Application Landing Zones
- Identity and Access Management
- Policy Management
- Security and Compliance
- Network Topology and Connectivity - one of the biggest conversations. (VWAN or Hub/Spoke)
- Shared Services Infrastructure - e.g. centralized logging and monitoring
- BCDR -0 understand customer SLAs and ensure we build a backup plan, and that the policies in place support this.
- Service Management - what does incident management look like?
- Service Catalogue - What do you have in Azure?
And Enterprise Scale provides patterns and templates to deliver this.

## Reference Implementation
Provided as a one click deployment to honour the 5 design principles across the 8 design areas. This can be used for greenfield and brownfield deployments. It is modular and scalable.

https://github.com/Azure/Enterprise-Scale/blob/main/docs/EnterpriseScale-Deploy-reference-implentations.md

- Wingtip - Azure only. Foundational landing zone. The workload is then separate from the platform, so you can scale with new subs as required, _without impacting existing workloads/environment_. Specific requirements can also be met by adding on additional management groups e.g. PCI workload.
- Contoso - vWAN for on prem connectivity
- Adventure Works using Hub and Spoke

## Overview of core components of Enterprise Scale

### Enterprise enrollment and Azure AD Tenants
- It's difficult to represent organisational structure! Especially with different departments/sub-companies/partners etc. Gets complex fast.
- Enterprise enrollment is about enabling organisaitons to build this structure in AAD to accurately reflect the structure.
- Usage and budget can be monitored and set at a department level. Accounts which belong to people can be assigned to organisaitons, recieve alerts, and adminster subscriptions. Many subscriptions can be associated with an account, and workloads land in subscriptions.
- Planning the hierarchy is critical - it is entered in the EA portal. Users or Roles are then assigned to the department. EA admin role manages this. 
- Department admin can create new departments, view usage details for the deptartments they manage. 
- Account owner is associated with subscription
- Set up notification contact email, group mailbox, and have multiple people in the group. Audit the portal to review access. 
- EA account must not move or change. 
- AD tenants ensure, via IAM, that authenticated and authorised users have access to the resources they are auth'd to.
- Each AAD has one or more domain, and only one AAD tenant. 
- Each subscription must have an associated account owner.
- Each account owner will be made a subscription owner for any subscriptions provisioned under that account.
- EA enrollment roles link users with their functional role. These roles are:
    - Enterprise administrator
    - Department administrator
    - Account owner
    - Service administrator
    - Notification contact

### IAM with ESLZs
- Addresses the Azure control plane, data plane, and application requirements. Allows the right people to access the right resources at the right time for the right reasons.
- All starts with AAD. PLP aligned to the operating model, then apply additional layers using Azure, based on regulatory requirements and workload definitions.
-  Shared resources must be managed centrally. 
- PIM flow for azure resources remove chances of elevated access being extended beyond what is necessary. Access reviews to check entitelements periodically are often a regulatory requirement, but another good practice to follow. 
- Integrate AD logs along with central Log Analytics workspace for audit reports and alert notifications. Try to use these to scan for odd behavior and malicious activity. 
- Avoid user and password auth - use managed identities, which provide simple and secure method for service to service auth. Eliminates need to manage creds in the code, and the complexity of manual password rotation.
- MFA and conditional access policies.
- Azure Security Centre JIT access should be used for ephemeral access to IaaS (VMs). Enables network level protection for these resources.
- Automated workflows should be governed by using privileged identities. Use the same tools and policies that you use for users.
- Don't add users directly to resource scopes. 
- If an organization has a scenario where an application that uses integrated Windows authentication must be accessed remotely through Azure AD, consider using Azure AD Application Proxy
- There is a difference between Azure AD, Azure AD DS, and AD DS running on Windows Server. Evaluate your application needs and understand and document the authentication provider that each one will be using. Plan accordingly for all applications.

### Management Group Hierarchy
- Management group structures within an Azure AD tenant support organizational mapping and must be considered thoroughly when an organization plans Azure adoption at scale.
- A management group tree can support up to six levels of depth. This limit does not include the tenant root level or the subscription level.
- Recommendation: Keep the management group hierarchy reasonably flat with no more than three to four levels, ideally. This restriction reduces management overhead and complexity.
- NB: Avoid duplicating your organizational structure into a deeply nested management group hierarchy. Management groups should be used for **policy assignment** and enforement rather than billing purposes. 
- Use resource tags, which can be enforced or appended through Azure Policy, to query and horizontally navigate across the management group hierarchy. Then you can group resources for search needs without having to use a complex management group hierarchy.
- Create a top-level sandbox management group to allow users to immediately experiment with Azure. Users can then experiment with resources that might not yet be allowed in production environments. The sandbox provides isolation from your development, test, and production environments.
- Use a dedicated service principal name (SPN) to execute management group management operations, subscription management operations, and role assignment. Using an SPN reduces the number of users who have elevated rights and follows least-privilege guidelines.
- Assign the User Access Administrator Azure role-based access control (RBAC) role at the root management group scope (/) to grant the SPN just mentioned access at the root level. After the SPN is granted permissions, the User Access Administrator role can be safely removed. In this way, only the SPN is part of the User Access Administrator role.
- Assign contributor permission to the SPN previously mentioned at the root management group scope (/), which allows tenant-level operations. This permission level ensures that the SPN can be used to deploy and manage resources to any subscription within your organization.
- Do not create any subscriptions under the root management group. This hierarchy ensures that subscriptions do not only inherit the small set of Azure policies assigned at the root-level management group, which do not represent a full set necessary for a workload.

### Subscription governance
- Subscriptions serve as boundaries for assigning Azure policies. For example, secure workloads such as payment card industry (PCI) workloads typically require additional policies to achieve compliance. Instead of using a management group to group workloads that require PCI compliance, you can achieve the same isolation with a subscription. This way, you do not have too many management groups with a small number of subscriptions.
- Subscriptions serve as a scale unit so that component workloads can scale within the platform subscription limits. Make sure to consider subscription resource limits during your workload design sessions.
- Subscriptions provide a management boundary for governance and isolation, which creates a clear separation of concerns.
- Planned future automation, which is a manual process, can be conducted to limit an Azure AD tenant to use only Enterprise Agreement enrollment subscriptions. This process prevents creation of Microsoft Developer Network subscriptions at the root management group scope
- Treat subscriptions as a democratized unit of management aligned with business needs and priorities.
- Make subscription owners aware of their roles and responsibilities:
    - Perform an access review in Azure AD Privileged Identity Management quarterly or twice a year to ensure that privileges do not proliferate as users move within the customer organization.
    - Take full ownership of budget spending and resource utilization.
    - Ensure policy compliance and remediate when necessary.
- Avoid a rigid subscription model, and opt instead for a set of flexible criteria to group subscriptions across the organization. This flexibility ensures that as your organization's structure and workload composition changes, you can create new subscription groups instead of using a fixed set of existing subscriptions.

- **One size does not fit all for subscriptions**. What works for one business unit might not work for another. Some apps might coexist within the same landing zone subscription while others might require their own subscription.

### Cost Transparency
- Potential need for chargeback models where shared PaaS resources are concerned, such as Azure App Service Environments and Azure Kubernetes Service (AKS), which might need to be shared to achieve higher density.
- Use a shutdown schedule for nonproduction workloads to optimize costs.
- Use Azure Advisor to check cost optimization recommendations.

- Use Azure Cost Management and Billing for cost aggregation. Make it available to application owners.
- Use Azure resource tags for cost categorization and resource grouping. Using tags allows you to have a chargeback mechanism for workloads that share a subscription or for a given workload that spans multiple subscriptions.

### Network Topology
- Planning for IP Addressing
    - Example: Have an azure region as 10.200.0.0/16, Hub at 10.200.0.0/24, and spokes at 10.200.1.0/24, 10.200.2.0/24, 10.200.3.0/24 etc within that region
    - Prevents pain downstream!
    - Need to be able to resolve names in Azure - Can do this with built in Azure DNS, but going to need more - may need to leverage on prem DNS, or extend it into Azure.. Can have DNS forwarders in Azure, which could be infoblox, windows dns servers.. but we need some resolvers in Azure - if there is XR outage and all DNS is on prem, you've got big issues, and also for Private Link (168.63.129.16), which is only accesibile inside azure vnets - if we have private link in any LZs, we're going to resolve to this
    -Virtual WAN is a microsoft managed Global network transit service. It has Managed Routers (which are not in hub spoke). A Secure vWAN hub also has Azure firewall - XR and GlobalReach enabled. Branch office connectivity through site-to-site VPNs, and end users can be enabled with point to site VPNs. This gives us Any to Any connectivity. vNet connected to different vWAN hubs can communicate too! Can of course lock this down. 
    - MSFT is managing the transit network, which can be appealing. When customers want to manage their own transit network, they will use hub and spoke. 
    - 2000 VMs per vWAN hub limit. This limit is higher in hub and spoke. 
    - Hub and spoke is the traditional approach - a Hub VNet in each region, and XR and VPN getways and firewalls into that hub. Then VNet peering to connect VNets into the hub, and route traffic between VNets either through the firewall with user defined routes, or through VNet peering between spoke VNets.
        - Customer managed transit network! They have to deploy something to do the routing, and manage it themselves.  
    - XR can use connection weight or AS (autonomous system) path prepending for route optimization from Azure to on prem. This is a BGP technique that aims to reduce overall latency by connecting from azure to the closer physical location. For traffic in to azure, we use local preference setting on a customer edge device to define the path into azure. 
    - Main thing here seems to be, for simplicity and new deployments, use vWAN. If you have complex, existing infra, or deeply bespoke requirements, then hub spoke.
    - Over 10gbps, XR Direct, to cross connect 200GBit ports to us. They can carve up the circuits themselves. 
    - Fast Pass will bypass the gateway onto azure
- Azure Firewall and WAF
    - WAF (web application firewall) is, clearly, for web applications. It looks for sql injection, cross site scripting attacks, etc. Used with app gateway which is a layer 7 load balancer.
    - Azure Firewall, for everything else! Allow RDP traffic, lock out internet, etc.
- Private link is the enterprise scale preferred approach to connect to Paas services. PL creates a network card object in your vnet, and maps that to a PaaS service, and then blocks access to that instance of the service from the internet. Means all comms are through RFC 1918 , i.e. private address space. Alleviates the need for UDR or NSG trickery for management IPs in the vnet. 
    - With Vnet injection, e.g. with SQL MI, it's deployed into the vnet , but azure management plane traffic still runs on "public" azure backbone. 
- _Always use private link when you can_.
- WAFs in the LZ democratizes the ability to deploy apps (i.e. its part of the app), and there are scale limits too.
- ASG - app security groups - good for controlling traffic within a vnet. Use NSGs (layer 4 access control - no traffic inspection) to control traffic between VNets (NVAs or Az Firewalls).
- Encryption is going to be required at some level - for XR private peering, can use MACSEC - layer 2 - only for XR direct. This is between the customer edge and the MSFT edge. 
- End to end encryption will need IPSEC. If intera LZ traffic needs encrpytion, then vpn between the vnets is the answer.

- For supported services, Private Link addresses data exfiltration concerns associated with service endpoints. As an alternative, you can use outbound filtering via NVAs to provide steps to mitigate data exfiltration.

- Azure PaaS services that have been injected into a virtual network still perform management plane operations by using public IP addresses. Ensure that this communication is locked down within the virtual network by using UDRs and NSGs.
- Use Private Link, where available, for shared Azure PaaS services. Private Link is generally available for several services and is in public preview for numerous ones.
- Access Azure PaaS services from on-premises via ExpressRoute private peering. Use either virtual network injection for dedicated Azure services or Private Link for available shared Azure services. To access Azure PaaS services from on-premises when virtual network injection or Private Link is not available, use ExpressRoute with Microsoft peering. This method avoids transiting over the public internet.
- Use virtual network service endpoints to secure access to Azure PaaS services from within your virtual network, but only when Private Link is not available and there are no data exfiltration concerns. To address data exfiltration concerns with service endpoints, use NVA filtering or use virtual network service endpoint policies for Azure Storage.
- Do not enable virtual network service endpoints by default on all subnets.
- Do not use virtual network service endpoints when there are data exfiltration concerns, unless you use NVA filtering.
- We do not recommend that you implement forced tunneling to enable communication from Azure to Azure resources.

- Use Azure Firewall to govern:
    - Azure outbound traffic to the internet
    - Non-HTTP/S inbound connections
    - East/west traffic filtering (if your organization requires it)
- Use Firewall Manager with Virtual WAN to deploy and manage Azure firewalls across Virtual WAN hubs or in hub virtual networks. Firewall Manager is now in GA for both Virtual WAN and regular virtual networks.

#### In and out bound connectivity

- Azure Load Balancer (internal and public) provides high availability for app delivery at a regional level.
- Azure Application Gateway allows the secure delivery of HTTP/S apps at a regional level.
- Azure Front Door Service allows the secure delivery of highly available HTTP/S apps across Azure regions.
- Azure Traffic Manager allows the delivery of global apps.
- Perform app delivery within landing zones for both internal-facing and external-facing apps.
- For secure delivery of HTTP/S apps, use Application Gateway version 2 and ensure that WAF protection and policies are enabled.
- Use a partner NVA if you cannot use Application Gateway version 2 for the security of HTTP/S apps.
- Deploy Azure Application Gateway version 2 or partner NVAs used for inbound HTTP/S connections within the landing-zone virtual network and with the apps that they are securing.
- Use a DDoS Standard protection plan for all public IP addresses in a landing zone.
- Use Azure Front Door Service with WAF policies to deliver and help protect global HTTP/S apps that span Azure regions.
- When you are using Azure Front Door Service and Application Gateway to help protect HTTP/S apps, use WAF policies in Azure Front Door Service. Lock down Application Gateway to receive traffic only from Azure Front Door Service.
- Use Traffic Manager to deliver global apps that span protocols other than HTTP/S.


### Monitoring and Logging
- Use a single monitor logs workspace to manage platforms centrally except where RBAC and data sovereignty requirements mandate separate workspaces. Centralized logging is critical to the visibility required by operations management teams. Logging centralization drives reports about change management, service health, configuration, and most other aspects of IT operations. Converging on a centralized workspace model reduces administrative effort and the chances for gaps in observability.
- In the context of the enterprise-scale architecture, centralized logging is primarily concerned with platform operations. This emphasis does not prevent the use of the same workspace for VM-based application logging. With a workspace configured in resource-centric access control mode, granular RBAC is enforced to ensure app teams will have access only to the logs from their resources. In this model, app teams benefit from the use of existing platform infrastructure by reducing their management overhead. For any noncompute resources such as web apps or Azure Cosmos DB databases, application teams can use their own Log Analytics workspaces and configure diagnostics and metrics to be routed here. 
- Export logs to Azure Storage if log retention requirements exceed two years. Use immutable storage with a write-once, read-many policy to make data non-erasable and nonmodifiable for a user-specified interval.
- Use Azure Policy for access control and compliance reporting. Azure Policy provides the ability to enforce organization-wide settings to ensure consistent policy adherence and fast violation detection. For more information, see Understand Azure Policy effects.
- Do not send raw log entries back to on-premises monitoring systems. Instead, adopt a principle that data born in Azure stays in Azure. If on-premises SIEM integration is required, then send critical alerts instead of logs.
- Use a centralized Azure Monitor Log Analytics workspace to collect logs and metrics from IaaS and PaaS app resources and control log access with RBAC.
- Use Azure Monitor metrics for time-sensitive analysis. Metrics in Azure Monitor are stored in a time-series database optimized to analyze time-stamped data. These metrics are well-suited for alerts and detecting issues quickly. They can also tell you how your system is performing. They typically need to be combined with logs to identify the root cause of issues.
- Use Azure Monitor alerts for the generation of operational alerts. Azure Monitor alerts unify alerts for metrics and logs and use features such as action and smart groups for advanced management and remediation purposes.

### BCDR
- Got to be available so that app teams can meet their availability targets, SLOs and SLAs. Need to consider Recovery targets (RPO and RTO) should define using the planning phase of the application.
- HA - handle the loss or degredation of a component or components, generally (but not always) in a geograpghical region.
- DR is the ability to recover from high impact event and recover from data loss.
- Backup - is focused on archival compliance, recovery from data breaches or corruption and long-term retention. Data that is stored in a different region and potentially an offline storage, are characteristics of backup. Drive application of backups through azure policy to ensure compliance.
- Enterprise scale a powerful guardrails and enable and enforce these backups, which will also allow the application owners protect and manage their own data as they see fit.

### DevOps
- A devops culture impacts how people think about work, including their processes and tools. IT is open to testing new approaches and methods. It is not threated by new ideas.
- Each role is recognised as an important contributor to the overall success.
- Finds ways to accelerate business value. It's not just containerization, CI and CD. 
- Find ways to measure the results of your actions. How do you know you're continuously improving?
- ESLZs recommend:
    - pipelines for CI/CD. Any azure changes should be governed for security and tracking. Ensures gates are in place to get approvals and sign off. Can include security, legal, and compliance.
    - Devops methodologies != devops team. This is a mindset around core pillars. 
    - Requires investment in engineering teams, to assist with the transformation to the methodologies and tolls in cloud.
    - Roles should be clearly defined.  
    - Be aware of legacy teams and apps, and dont be afraid of making exceptions (preferably governed).
    - Automate where possible! But dont boil the whole ocean. Start small, and grow. May need to rebuild as you go! But thats ok. 


## Further Notes
- Evolve from enterprise-scale foundation
    - If the business requirements changes over time, such as migration of on-premises applications to Azure that requires hybrid connectivity, you will simply create the connectivity subscription and place it into the Platform Management Group and assign Azure Policy for the VWAN network topology.
- Platform subscriptions
    - Connectivity
        - A dedicated subscription will be used for the centrally managed networking infrastructure that will control end-to-end connectivity for all landing zones within Contoso's Azure platform. Azure resources that will be deployed into this subscription include Azure Virtual WAN and its sub-resources (VHubs, gateways), Azure firewalls, firewall policies and Azure private DNS zones.
        - The "connectivity" subscription has the tag "BusinessUnit" with the value "Platform".
    - Identity
        - The "Identity" subscription will be used to host VMs running Windows Server Active Directory. There will be at least two domain controllers deployed per Azure region for redundancy purposes, and to ensure regions are independent in the case of a regional outage. Windows Server AD replication will ensure all domain controllers are kept in sync.
    - Management
        - A dedicated subscription will be used for centrally managed platform infrastructure to ensure a holistic at-scale management capability across all landing zones in Contoso's Azure platform.
        - The "Management" Subscription has the tag "BusinessUnit" with a value "Platform".
- Recommendations for High availability
    - Application architectures should be built using a combination of Availability Zones across the North Europe and West Europe paired Azure regions. More specifically, applications and their data should be synchronously replicated across Availability Zones within an Azure region (North Europe) for high-availability purposes, and asynchronously replicated across Azure regions (West Europe) for DR protection.
    - Azure services that provide native replication across Availability Zones should be used as a preference, such as Zone-Redundant Storage and Azure SQL DB.
    - Stateless virtual machine workloads should be deployed across multiple instances in Availability Zones behind a Load Balancer standard or Application Gateway (version 2).
    - Stateful virtual machine workloads should take advantage of application-level replication across Availability Zones, such as SQL AlwaysOn.
    - Stateful virtual machine workloads that do not support application-level replication should use Azure Site Recovery Zonal-Replication (preview).
- Recommendations for Disaster Recovery
    - Application architectures should use native application replication technologies such as SQL AlwaysOn, for stateful virtual machines  to replicate data from one Azure region (North Europe) to the paired Azure region (West Europe).
    - Applications should use Azure Site Recovery to replicate stateful virtual machines that do not support application-level replication.
    - Stateless virtual machine workloads can be quickly re-created (or pre-provisioned) in the paired Azure region (West Europe). Alternatively, Azure Site Recovery could also be used.
    - For externally facing applications that must always be available, an active-active or active-passive deployment pattern across the North Europe and West Europe regions should be used, either Azure Front Door or Azure Traffic Manager, to ensure applications are accessible at all times even if one of the Azure regions is not available.
    - Applications should be transformed and modernized where possible to use Azure PaaS services that provide native replication techniques across regions, such as Cosmos DB, Azure SQL DB, and Azure Key Vault.

### Building on from Wingtip
- How to evolve and add support for hybrid connectivity later
    - If the business requirements change over time, such as migration of on-premises applications to Azure that requires hybrid connectivity, the architecture allows you to expand and implement networking without refactoring Azure design with no disruption to what is already in Azure. The Enterprise-scale architecture allows you to create the Connectivity Subscription and place it into the platform Management Group and assign Azure Policies or/and deploy the target networking topology using either Virtual WAN or hub-spoke network topology.
- [Programatically create subscriptions](https://docs.microsoft.com/en-us/azure/cost-management-billing/manage/programmatically-create-subscription)
