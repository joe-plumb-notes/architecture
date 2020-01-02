## Manage identity and access
-  Configure Azure Active Directory for workloads
    - create App registration
        - Creating an app registration is easy - just do this (through the portal)[https://docs.microsoft.com/en-us/azure/active-directory/develop/howto-create-service-principal-portal#create-an-azure-active-directory-application]. Once the app registration has been created, it needs to be assigned permissions on the subscription/resource group/resources in which it is allowed to act. "When an application is given permission to access resources in a tenant (upon registration or consent), a service principal object is created." The (difference between the app registration and the service principal are documented)[https://docs.microsoft.com/en-us/azure/active-directory/develop/app-objects-and-service-principals#application-and-service-principal-relationship].
    - configure App registration permission scopes
        - Again, through the portal, click on "API permissions" on an App Registration where you can see current and add new permission scopes.
    - configure multi-factor authentication settings
        - "(You can access settings related to Azure Multi-Factor Authentication from the Azure portal by browsing to Azure Active Directory > Security > MFA.)[https://docs.microsoft.com/en-us/azure/active-directory/authentication/howto-mfa-mfasettings]"
    - manage Azure AD directory groups
        - "(The resource owner assigns an Azure AD group to the resource, which automatically gives all of the group members access to the resource. Group membership is managed by both the group owner and the resource owner, letting either owner add or remove members from the group.)[https://docs.microsoft.com/en-us/azure/active-directory/fundamentals/active-directory-manage-groups#ways-to-assign-access-rights]" 
        - Creating and adding members to a group (is also documented)[https://docs.microsoft.com/en-us/azure/active-directory/fundamentals/active-directory-groups-create-azure-portal], and straight forward in AAD.
    - manage Azure AD users
        - User management also straight forward, just... (follow the steps)[https://docs.microsoft.com/en-us/azure/active-directory/fundamentals/add-users-azure-active-directory]!
    - install and configure Azure AD Connect
        - Azure AD Connect is the MS tool that enables Hybrid Identity. It does this by enabling *password hash synchronization*, *pass through auth*, *fedetation integration* (active directory federation services), synchronization of users, groups, and other objects between on premises and cloud (inc password hashes), and health monitoring. (Read more here)[https://docs.microsoft.com/en-us/azure/active-directory/hybrid/whatis-azure-ad-connect]. 
        - (2 installation approaches)[https://docs.microsoft.com/en-us/azure/active-directory/hybrid/how-to-connect-install-select-installation], express and custom. Express covers 90% of customer setups, and requires:
            - a single Active Directory forest on-premises.
            - an enterprise administrator account you can use for the installation.
            - less than 100,000 objects in your on-premises Active Directory.
        - Custom install should be used when:
            - You do not have access to an enterprise admin account in Active Directory.
            - You have more than one forest or you plan to synchronize more than one forest in the future.
            - You have domains in your forest not reachable from the Connect server.
            - You plan to use federation or pass-through authentication for user sign-in.
            - You have more than 100,000 objects and need to use a full SQL Server.
            - You plan to use group-based filtering and not only domain or OU-based filtering.            
    - (configure authentication methods)[https://docs.microsoft.com/en-us/azure/security/fundamentals/choose-ad-authn#authentication-methods]
        - Need to choose based on historical AD implementation. Easier for cloud-only orgs, as they are dealing with cloud-only identites.
        - Cloud auth, where the authentication is done in AAD via password hash sync or pass through auth (password validation for AAD by validating with AD on-premises)
        - Federated auth, where the entire authentication process is handed off to a separate trusted authg system (e.g. AD FS). Enables more advanced auth requirements like smart cards, third party MFA. 
    - implement conditional access policies
        - conditional access is about using the signals from the user (e.g. location, device, application, user, real-time risk), to enable access, but get out the way when low/no risk. 
        - Requires AAD P1 license.
        - More guidance (in the docs)[https://docs.microsoft.com/en-us/azure/active-directory/conditional-access/best-practices#how-are-conditional-access-policies-applied], but not a big steer on how to implement it beyond (navigating to AAD, Security > Conditional access, creating a new policy and defining the access rules)[https://docs.microsoft.com/en-us/azure/active-directory/conditional-access/best-practices#whats-required-to-make-a-policy-work]. (This overview)[https://docs.microsoft.com/en-us/azure/active-directory/conditional-access/app-based-mfa] is also helpful.
    - configure Azure AD identity protection
        - Identity Protection is a tool that allows organizations to:
            - Automate the detection and remediation of identity-based risks.
            - Investigate risks using data in the portal.
            - Export risk detection data to third-party utilities for further analysis.
        - This is a broad requirement - but there are three default policies that fall under "identity protection" - the (AAD MFA policy)[https://docs.microsoft.com/en-us/azure/active-directory/identity-protection/concept-identity-protection-policies#azure-mfa-registration-policy], (sign in risk policy)[https://docs.microsoft.com/en-us/azure/active-directory/identity-protection/concept-identity-protection-policies#sign-in-risk-policy], which is where admins can require MFA if the risk score is higher than threshold (every sign-in has a calculated risk score based on user signals), and a (user risk policy)[https://docs.microsoft.com/en-us/azure/active-directory/identity-protection/concept-identity-protection-policies#user-risk-policy], which triggers MFA/password reset if behavior on an identity is deemed abnormal i.e. indicates potentially compromised identity.
- Configure Azure AD Privileged Identity Management
    - monitor privileged access
        - privaliged access can be monitored in the (resource dashboard)[https://docs.microsoft.com/en-us/azure/active-directory/privileged-identity-management/pim-resource-roles-overview-dashboards]. 
    - configure access reviews
        - Need to be a *Privileged Role Admin* to create an access review.
        - In Azure PIM, select Azure AD Roles, then Access Reviews. Under manage, click access reviews. Here you can view and create the reviews, set how frequently the review needs to be performed, the group/role the review is to access, and who is to do the review (assigned users or the users themselves.)
        - https://docs.microsoft.com/en-us/azure/active-directory/privileged-identity-management/pim-how-to-start-security-review
    - https://docs.microsoft.com/en-us/azure/active-directory/privileged-identity-management/pim-how-to-complete-review 

    - activate Privileged Identity Management
- Configure Azure tenant security
    - transfer Azure subscriptions between Azure AD tenants
        - changing the directory a subscription is associated with (is easy)[https://docs.microsoft.com/en-us/azure/active-directory/fundamentals/active-directory-how-subscriptions-associated-directory#associate-a-subscription-to-a-directory]
        - (Be aware of these caveats)[https://docs.microsoft.com/en-us/azure/active-directory/fundamentals/active-directory-how-subscriptions-associated-directory#before-you-begin]
    - manage API access to Azure subscriptions and resources

## Implement platform protection
### Implement network security
- configure virtual network connectivity
- configure Network Security Groups (NSGs)
    - https://docs.microsoft.com/en-us/azure/virtual-network/security-overview
    - https://docs.microsoft.com/en-us/azure/virtual-network/manage-network-security-group
- create and configure Azure firewall
- create and configure Azure Front Door service
    - "Azure Front Door Service is an Application Delivery Network (ADN) as a service, offering various layer 7 load-balancing capabilities for your applications. It provides dynamic site acceleration (DSA) along with global load balancing with near real-time failover. It is a highly available and scalable service, which is fully managed by Azure."
    - (Tutorial for creating an example Azure Front Door service)[https://docs.microsoft.com/en-us/azure/frontdoor/quickstart-create-front-door].
        - Add a frontend host for Front Door
        - Add application backend and backend pools
        - Add a routing rule
- create and configure application security groups
- configure remote access management
    - https://docs.microsoft.com/en-us/azure/security/fundamentals/management
- configure baseline
    - "(Baseline policies are a set of predefined policies that help protect organizations against many common attacks. These common attacks can include password spray, replay, and phishing. Baseline policies are available in all editions of Azure AD.)[https://docs.microsoft.com/en-us/azure/active-directory/conditional-access/concept-baseline-protection]"
- configure resource firewall
### Implement host security
- configure endpoint security within the VM
    - Security Center?  
- configure VM security
    - https://docs.microsoft.com/en-us/azure/virtual-machines/windows/security-recommendations
    - Lots of ways to do this - JIT access, disk encryption, keep them updated (Azure Update Management) and backed up (Azure Backup), increase resiliency using availability sets, and ensure a business continuity and DR strategy is in place using Azure Site Recovery 
- harden VMs in Azure
    - Harden VMs by enabling JIT access. JIT access enables access by opening ports in lower-priority networking rules as defined when setup. This can be for RDP, ssh, WinRM, and powershell remoting. (WinRM is Windows Remote Management). 
    - JIT access can be requested through portal or powershell (probably by cli too) https://blogs.msdn.microsoft.com/mvpawardprogram/2018/01/09/just-in-time-access-azure-vms/
- configure system updates for VMs in Azure
    - "(You can use the Update Management solution to manage updates and patches for your virtual machines)[https://docs.microsoft.com/en-us/azure/automation/automation-tutorial-update-management]"
    - "(On the VM page, under OPERATIONS, select Update management. The Enable Update Management pane opens)[https://docs.microsoft.com/en-us/azure/automation/automation-tutorial-update-management#enable-update-management]."
- configure baselineConfigure container security
- configure network
- configure authentication
- configure container isolation
- configure AKS security
    - This is a deep topic. 
    - https://docs.microsoft.com/en-us/azure/aks/concepts-security
- configure container registry
- implement vulnerability management
    - Security Center comes with (vulnerability scanning out the box)[https://docs.microsoft.com/en-us/azure/security-center/security-center-vulnerability-assessment-recommendations] for VMs, SQL Databases, and ACR images. 
    - Also built in vulnerability scanning and remediation available, in preview, via (Qualys vuln. scanner)[https://docs.microsoft.com/en-us/azure/security-center/built-in-vulnerability-assessment]. Easy to turn on, assess impact, and remediate vulns, in Security Center. Requires standard pricing tier.
    - Can also use (partner vulnerability scanners)[https://docs.microsoft.com/en-us/azure/security-center/partner-vulnerability-assessment], but requires free tier of SC, and for admin to handle the licensing costs, deployment, and configuration.
### Implement Azure Resource management security
- create Azure resource locks
    - Resource locks can be created on resources, resource groups. https://docs.microsoft.com/en-gb/azure/azure-resource-manager/management/lock-resources#portal
- manage resource group security
    - Resource groups can be secured in the same way as any other resources in Azure, RBAC. 
    - https://docs.microsoft.com/en-gb/azure/azure-resource-manager/management/overview#resource-groups
- configure Azure policies
    - (Author a policy in the portal)[https://docs.microsoft.com/en-us/azure/governance/policy/assign-policy-portal], then select the Scope by clicking the ellipsis and selecting either a management group or subscription. Optionally, select a resource group. You can then view reports on the policy and compliance to the policy on the defined scope(s).
- configure custom RBAC roles
    - Custom roles can be created using Azure PowerShell, Azure CLI, or the REST API
    - (Instructions here)[https://docs.microsoft.com/en-us/azure/role-based-access-control/custom-roles#steps-to-create-a-custom-role]
- configure subscription and resource permissions

## Manage Security Operations
### Configure security services
- configure Azure monitor
    - "(Azure Monitor maximizes the availability and performance of your applications by delivering a comprehensive solution for collecting, analyzing, and acting on telemetry from your cloud and on-premises environments)[https://docs.microsoft.com/en-us/windows-server/storage/storage-spaces/configure-azure-monitor]"
        - Metrics are numerical values that describe some aspect of a system at a particular point in time. 
        - Logs contain different kinds of data organized into records with different sets of properties for each type.
        - Can enable this on VMs in Windows Admin Center (WAC), as well as Update Manager. 
- configure diagnostic logging and log retention
    - Log retention can be changed in Azure Monitor workspace settings (Usage and estimated costs > Data volume management > slider for log retention)[https://docs.microsoft.com/en-us/azure/azure-monitor/platform/manage-cost-storage#change-the-data-retention-period]
### Manage security alerts
- create and customize alerts
    - Again, very easy. (In Azure portal, click on Monitor. Click Alerts then click + New alert rule.)[https://docs.microsoft.com/en-us/azure/azure-monitor/platform/alerts-metric]
- configure a playbook for a security event by using Azure Security Center
    - playbooks are configured from Security Center - under *Automation and Orchestration*, click playbooks. Follow the blade instructions to add a playbook, which will create a LogicApp. Use this to create a new blank logic app, with an Azure Security Center trigger (search for this). Then you can write your logic here. (More details and screenshots in the docs)[https://docs.microsoft.com/en-us/azure/security-center/security-center-playbooks#how-to-create-a-security-playbook-from-security-center]
- investigate escalated security incidents
    - Investigation only works for windows servers
    - Using the MS Graph, incidents are triaged, pulling together all relevant entities that formed part of a (percieved security incident)[https://docs.microsoft.com/en-us/azure/security-center/security-center-investigation].
    - (Instructions on how to perform an investigation here)[https://docs.microsoft.com/en-us/azure/security-center/security-center-investigation#how-to-perform-an-investigation] 

## Secure data and applications
### Configure security policies to manage data
- configure data classification
    - Go to the SQL Database in the Portal > Advanced Threat Protection > Data Discovery and Classification card. Click the Classification tab to start classifying data in the database. (More here)[https://docs.microsoft.com/en-us/azure/sql-database/sql-database-data-discovery-and-classification?tabs=azure-t-sql#classify-your-sql-database]
- configure data retention
    - Manage database backup retention policies in the (manage backups section of the database)[https://docs.microsoft.com/en-us/azure/sql-database/sql-database-long-term-backup-retention-configure#using-azure-portal]
- configure data sovereignty
    - https://docs.microsoft.com/en-us/learn/modules/configure-security-policies-to-manage-data/5-configure-data-sovereignty 
### Configure security for data infrastructure
- enable database authentication
- enable database auditing
    - Server level vs database level audit policy
    - On the database or server, under security, click "Auditing". If on the database, you can turn on audit here. Go to the server to turn on server level audit. This will be visibile here too.
- configure Azure SQL Database Advanced Threat Protection
    - SQL > Advanced Data Security > E-mail in the alerts box.
### Configure application security
- configure SSL/TLS certs
    - (Lots of ways to do this)[https://docs.microsoft.com/en-us/azure/app-service/configure-ssl-certificate].
- configure Azure services to protect web apps
- create an application security baseline