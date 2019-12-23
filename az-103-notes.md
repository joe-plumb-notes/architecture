# AZ-103 Notes - Microsoft Certified Azure Administrator 
My notes in preparation for the AZ-103 exam. For this certification, I used the 'Skills Measured' section to identify areas of required study and used that as the basis for reading.

## Manage Azure subscriptions and resources (15-20%)
### Manage Azure subscriptions
	• Assign administrator permissions
		○ This can be done in the portal under RBAC controls. Admin = "Owner" role. https://docs.microsoft.com/en-us/azure/billing/billing-add-change-azure-subscription-administrator
	• Configure cost center quotas and tagging
		○ Resources can be tagged to better understand and filter spend. https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-using-tags. Azure policy can be used to enforce tagging rules and conventions. you can create a policy that automatically applies the needed tags during deployment
	• Configure Azure subscription policies at Azure subscription level
		○ When you create a policy, you define it's scope. This is either a management group or a subscription. https://docs.microsoft.com/en-us/azure/governance/policy/tutorials/create-and-manage. Can define custom policies outside the ootb ms defined ones.
### Analyze resource utilization and consumption
	• Configure diagnostic settings on resources
		○ Can collect logs into [Az Log Analytics](https://docs.microsoft.com/en-us/azure/azure-monitor/platform/diagnostic-logs-stream-log-store). Turn this on under "Diagnostics settings" in the Settings menu.
	• Create baseline for resources
		○ Can create a baseline using ARM templates. 
	• Create and rest alerts
		○ Alerts are created in [Azure Monitor](https://docs.microsoft.com/en-us/azure/azure-monitor/platform/alerts-overview). Alerts can be triggered on Metric values, Log search queries, Activity log events, Health of the underlying Azure platform, Tests for website availability, among others. 
	• [Analyze alerts across subscription](https://docs.microsoft.com/en-us/azure/azure-monitor/platform/alerts-overview#alerts-experience)
	• Analyze metrics across subscription
		○ Metrics can be aggregated across subscriptions in [Azure monitor](https://docs.microsoft.com/en-us/azure/azure-monitor/platform/metrics-supported)
	• Create action groups
		○ An action group is a collection of notification preferences defined by the owner of an Azure subscription. Azure Monitor and Service Health alerts use action groups to notify users that an alert has been triggered. https://docs.microsoft.com/en-us/azure/azure-monitor/platform/action-groups
	• Monitor for unused resources
		○ Azure Cost Management has unused disks and reservations.
	• Monitor and report on spend
		○ This can be done in Azure cost management
	• Utilize Log Search query functions
		○ This can be done in azure monitor (based on ADX). Queries are written in KQL. https://docs.microsoft.com/en-us/azure/azure-monitor/log-query/log-query-overview#what-language-do-log-queries-use
	• View alerts in Log Analytics
		○ Go to log analytics, and click alerts. You can create new alert rules here too. The table reflects the contrition, what caused the alert, etc. https://docs.microsoft.com/en-us/azure/azure-monitor/learn/tutorial-response#view-your-alerts-in-azure-portal
### Manage resource groups
	• Use Azure policies for resource groups
	• Configure resource locks
		○ CanNotDelete means authorized users can still read and modify a resource, but they can't delete the resource.
		○ ReadOnly means authorized users can read a resource, but they can't delete or update the resource. Applying this lock is similar to restricting all authorized users to the permissions granted by the Reader role.
		From <https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-lock-resources> 
		○ Unlike role-based access control, you use management locks to apply a restriction across all users and roles. 
		From <https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-lock-resources#how-locks-are-applied> 
	• Configure resource policies
		○ Set policy definition, then a policy assignment (assign it to take place in a certain scope, e.g. Management group, resource group, subscription, etc), and are assigned by all child resources. 
		For example, at the subscription scope, you can assign a policy that prevents the creation of networking resources. You could exclude a resource group in that subscription that is intended for networking infrastructure. You then grant access to this networking resource group to users that you trust with creating networking resources.
		
		From <https://docs.microsoft.com/en-us/azure/governance/policy/overview#policy-definition> 
		Policy parameters can also be used to simplify the number of policy definitions to be created. Parameters make the policy more generic.
		
	• Identify auditing requirements
	• Implement and set tagging on resource groups
		○ https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-using-tags
	• Move resources across resource groups
		○ Subscriptions must be active, and belong in the same AD tenant. Can check using powershell or Az CLI. 
	• Remove resource groups
		○ https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-delete?tabs=azure-powershell
### Managed role based access control (RBAC)
	• Create a custom role
		use the New-AzRoleDefinition or az role definition create commands to create the custom role. You must have the Microsoft.Authorization/roleDefinitions/write permission on all AssignableScopes, such as Owner or User Access Administrator.
		From <https://docs.microsoft.com/en-us/azure/role-based-access-control/custom-roles#steps-to-create-a-custom-role> 
	• Configure access to Azure resources by assigning roles
	• Configure management access to Azure, troubleshoot RBAC, implement RBAC policies, assign RBAC Roles

## Implement and manage storage (15-20%)
### Create and Configure storage accounts
	• Configure network access to the storage account
		○ You can Configure storage accounts to allow access only from specific subnets. The allowed subnets may belong to a VNet in the same subscription, or those in a different subscription, including subscriptions belonging to a different Azure Active Directory tenant.
		○ Enable a Service endpoint for Azure Storage within the VNet. The service endpoint routes traffic from the VNet through an optimal path to the Azure Storage service. The identities of the subnet and the virtual network are also transmitted with each request. 
		○ https://docs.microsoft.com/en-us/azure/storage/common/storage-network-security#grant-access-from-a-virtual-network
	• Create and Configure storage account
		○ Create is easy. You can manage access control, tags, access keys, configuration (hot/cool storage, replication details (RA_GRS, GRS, LRS))
	• Generate shared access signature
	• Install and use Azure Storage Explorer
	• Manage access keys
	• Monitor activity log by using Log Analytics
		○ https://docs.microsoft.com/en-us/azure/storage/common/storage-monitor-storage-account#Configure-monitoring-for-a-storage-account
		○ Diagnistions, select metrics and retention policy, set status to ON
	• Implement Azure storage replication
		○ Azure storage replication  https://docs.microsoft.com/en-us/azure/storage/common/storage-redundancy#choosing-a-redundancy-option		
### Import and export data to Azure
	• Create export from Azure job - this is to export from Azure, by shipping disks.
		○ Selecting the blobs to export.
		○ Obtaining a shipping location.
		○ Creating the export job.
		○ Shipping your empty drives to Microsoft via a supported carrier service.
		○ Updating the export job with the package information.
		○ Receiving the drives back from Microsoft.
		From <https://docs.microsoft.com/en-gb/previous-versions/azure/storage/common/storage-import-export-creating-an-export-job> 
	• Create import into Azure job
		○ Preparing drives with the Azure Import/Export Tool.
		○ Obtaining the location to which to ship the drive.
		○ Creating the import job.
		○ Shipping the drives to Microsoft via a supported carrier service.
		○ Updating the import job with the shipping details.
		From <https://docs.microsoft.com/en-gb/previous-versions/azure/storage/common/storage-import-export-creating-an-import-job> 
	• Use Azure Data Box
		○ Offline transfer. Maximum starge of 80TB per databox. Order via azure portal, set up using web UI, copy data from the servers to the device, and ship the device back to azure. One time migration, Initial bulk transfer, and periodic uploads.
	• Configure and use Azure blob storage
	• Configure Azure content delivery network (CDN) endpoints
		○ Have a storage account, Create a new CDN profile (container for endpoints, with a pricing tier), then add an endpoint, with origin type, protocol, and ports.
### Configure Azure files
	• Create [Azure file share](https://docs.microsoft.com/en-us/azure/storage/files/storage-how-to-create-file-share#create-a-file-share-through-the-azure-portal) - The Quota's current maximum value is 5 TiB
	• Create Azure File Sync service
		Use Azure File Sync to centralize your organization's file shares in Azure Files, while keeping the flexibility, performance, and compatibility of an on-premises file server.
	• Create Azure sync group
	• Troubleshoot Azure File Sync
### Implement Azure backup
	• Configure and review [backup reports](https://docs.microsoft.com/en-us/azure/backup/backup-azure-Configure-reports)
		○ Create a Recovery Services vault. Under monitoring and reports, backup reports can be selected, to create and view custom reports in Power BI.
	• Perform backup operation
		○ Enable backup on the service, with the existing or new RECOVERY SERVICES VAULT. Default set at GRS. Settings on the VM, and click backup, backup now. Retention policy can be set here - default of 30 days. 
	• Create Recovery Services Vault
	• Create and Configure backup policy
		○ Windows server or machines 3 times a day max, daily or weekly backup schedules. 
		○ Azure VMs once a day
	• Perform a restore operation
		○ Number of ways to do this - create a new vm with restore point, restore disk, replace existing disk on existing vm..
		○ https://docs.microsoft.com/en-us/azure/backup/backup-azure-arm-restore-vms

## Deploy and manage virtual machines (VMs) (15-20%)
### Create and Configure a VM for Windows and Linux
	• Configure high availability
		○ Availability Zones to protect against datacenter level failures - means that if one zone is compromised your replicated apps and data are instantely available ina nother zone.(99.99)
		○ Availability set for slightly lower availability (99.95). Avoid leaving a single instance virtual machine in an availability set by itself. VMs in this configuration do not qualify for a SLA guarantee and face downtime during Azure planned maintenance events, except when a single VM is using Azure premium SSDs. For single VMs using premium SSDs, the Azure SLA applies.
		From <https://docs.microsoft.com/en-us/azure/virtual-machines/windows/manage-availability> 
		
	• Configure monitoring, networking, storage, and virtual machine size
	• Deploy and Configure scale sets
		○ Create a scale set in the same way you would a VM, choose an OS image, RG, Location, password, username, et. Load balancing options e.g. Public IP and Domain name label of the load balancer.
		○ Check the inbound NAT rules on the VM scale set to connect to each VM. Here you will find the destination IP and port. 

### Automate deployment of VMs
	• Modify Azure Resource Manager (ARM) template
	• Configure location of new VMs
	• Configure [VHD template](https://docs.microsoft.com/en-us/azure/marketplace/cloud-partner-portal/virtual-machine/cpp-deploy-json-template)
	• Deploy from template
	• Save a deployment as an ARM template
	• Deploy Windows and Linux VMs
### Manage Azure VM
	• add data discs
	• add network interfaces
	• automate configuration management by using PowerShell Desired State Configuration (DSC) and VM Agent by using custom script extensions
	• manage VM sizes; move VMs from one resource group to another
	• redeploy VMs
### Manage VM backups
	• Configure VM backup
	• define backup policies
	• implement backup policies
	• perform VM restore
	• Azure Site Recovery

## Configure and manage virtual networks (30-35%)
### Create connectivity between virtual networks
	• Create and Configure VNET peering
	• Create and Configure VNET to VNET
	• Verify virtual network connectivity
	• Create virtual network gateway
### Implement and manage virtual networking
	• Configure private and public IP addresses, network routes, network interface, subnets, and virtual network
### Configure name resolution
	• Configure Azure DNS
	• Configure custom DNS settings
	• Configure private and public DNS zones
### Create and Configure a Network Security Group (NSG)
	• Create security rules
	• Associate NSG to a subnet or network interface
	• Identify required ports
	• Evaluate effective security rules
### Implement Azure load balancer
	• Configure internal load balancer, Configure load balancing rules, Configure public load balancer, troubleshoot load balancing
### Monitor and troubleshoot virtual networking
	• monitor on-premises connectivity, use Network resource monitoring, use Network Watcher, troubleshoot external networking, troubleshoot virtual network connectivity
### Integrate on premises network with Azure virtual network
	• create and Configure Azure VPN Gateway, create and Configure site to site VPN, Configure Express Route, verify on premises connectivity, troubleshoot on premises connectivity with Azure

## Manage identities (15-20%)
## Manage Azure Active Directory (AD)
    • Add custom domains
        • Just in the portal. Very easy.
    • Azure AD Join
    • Configure self-service password reset
        • From your existing Azure AD tenant, on the Azure portal under Azure Active Directory select Password reset.
        • From the Properties page, under the option Self Service Password Reset Enabled, choose Selected.
            • From Select group, choose your pilot group created as part of the prerequisites section of this article.
            • Click Save.
        • From the Authentication methods page, make the following choices:
            • Number of methods required to reset: 1
            • Methods available to users:
                ○ Email
                ○ Mobile app code (preview)
            • Click Save.
        From <https://docs.microsoft.com/en-us/azure/active-directory/authentication/quickstart-sspr> 
        
    • Manage multiple directories

    Manage Azure AD objects (users, groups, and devices)
    • create users and groups
    • manage user and group properties
    • manage device settings
    • perform bulk user updates
        • Use powershell, import AzureAD module, Connect, then set group id, get users, and loop through each user. 
    • manage guest accounts

### Implement and manage hybrid identities
    • Install Azure AD Connect, including password hash and pass-through synchronization
    • Use Azure AD Connect to Configure federation with on-premises Active Directory Domain Services (AD DS)
    • Manage Azure AD Connect
    • Manage password sync and password writeback
### Implement multi-factor authentication (MFA)
    • Configure user accounts for MFA, enable MFA by using bulk update, Configure fraud alerts, Configure bypass options, Configure Trusted IPs, Configure verification methods

    From <https://www.microsoft.com/en-us/learning/exam-AZ-103.aspx> 







