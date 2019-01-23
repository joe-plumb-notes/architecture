# AZ-300 notes
My notes in preparation for AZ-300 exam, while studying https://www.udemy.com/70534-azure

## Azure Geos and Regions (re-cap)
- Regions are broken into sevaral data centers - Azure doesn't give you the choice of the exact data center you want to deploy to, it will handle this for us. Each region has a pair for highest speed connections and HA/redundancy. e.g. East US is paired with West US. Lowest latency.

## Virtual Machines in Azure
- Access the compute for as long as you need it.
- 120+ configurations (instant sizes) - can't arbitrarily select the spec of your machine. There are different instance types:
	- _General Purpose_ - a balanced computer. Even distribution between CPU and RAM. Different series available (B, D, DS, A, DC).. 
	- _Compute Optimised_ - high CPU to memory ratio. (F, FS)
	- _Memory Optimised_ - high RAM vs CPU. (E, ES, M, G, GS, D*, DS*). D _is_ general purpose, but is considered memory optimised at the higher levels. Good for DB servers, caching, analytics with a lot of work in memory (Spark..)
	- Storage Optimised - If you've got an application with high I/O on disk, e.g. data warehousing. Where the storage speed is the bottleneck and focus factor. (LS)
	- _GPU series_ - (NV, NC, ND)
	- _HPC_ - massive compute (H) - requires conversation with Microsoft before you can access these. 
- S in all of the above model codes stands for SSD
- https://portal.azure.com/
- Different availability zones have different features. _Which is the go-to AZ for Europe?_
- Stopping the VM stops you incurring charge for it, but not the storage associated.
### Configuring availability options
- Can determine this when provisioning the VM, in the 'Availability Options'. 
- _Availability Set_ assigns VM to a group. Azure will then automatically distribute the VMs across multiple hardware servers. 
	- _Fault domains_ (assign 1-3) are physical unexpected points of failure. So, for example, 2 VMs not in an availability set that by chance end up running on the same physical machine/rack in Azure, if something were to happen to that machine, then both your VMs would go down. Having them across 2 or more fault domains means the VMs will be distributed.
	- _Update domains_ (assign 1-20) enable rollout of Windows and Azure fixes. Azure will automatically patch and apply fixes to your VMs, and these will be applied in 20 segments, one segment at a time. Distributing your machines across different update domains means this will only impact a small subset of those machines.
- _Availability Zones_ are not available worldwide yet. This gives you ability to deploy building that you deploy a VM to within a region. 
- These do not provide load balancing - a load balancer will have to be setup to do this. 
- [SLAs](https://azure.microsoft.com/en-us/support/legal/sla/virtual-machines/v1_8/) are detailed, and by using the above mechanisms you can increase the guarantee of availability of your machines from 99.9% (one instance) to 99.99% (two or more instances across two or more availability zones in the same Azure region). [Downtime stats](https://en.wikipedia.org/wiki/High_availability#Percentage_calculation) are interesting to look at, and should be considered when architecting your application. [NB: these are financial guarantees, not an absolute promise this will never happen]

### Monitoring Azure VMs
- Select for VM > Diagnostics (Left hand side)
- Guest-level monitoring must be turned on in order to access these stats outside of the VM.
- Performance counters enable you to configure what data and the frequency by which this data is sent back to Azure.
- Can access graphs here, pin them to your dashboard for ease of access upon login, and export this data to Excel if required.

### VM Scale Sets (VMSS)
- Allow you to create virtual machines that are scalable, from 1-1000 VMs in a set. 
- Determine instance count i.e. number of VMs, and the instance size. 
- Can set up auto-scaling, with auto-scaling rules. e.g. when CPU in the scale set goes above a threshold, provision another VM (up to the max number of machines). 
- Will also need to be load balanced, with either an *Application Gateway* or a *Load Balancer* service. 
	- Application Gateway supports HTTP/HTTPS/WebSocket traffic, perhaps routing traffic based on the URL contents of the request (e.g. one server serving images, one serving video), and rules defining this distribution.
	- Load balancer is not sophisticated, does not inspect the traffic. You choose the public IP, Domain name label, virtual network, etc.. 

### ARM templates
- Azure Resource Manager (ARM) - used to organise and deploy resources in Azure. (Used to be ASM, don't do this any more.)
- If you look at a resource group, you can see each deployment has a set of details, marking the changes that were made. Each one of these includes a template and parameters files (JSON).  
- These can be downloaded, along with deployment scripts for CLI, Powershell, Ruby, and C#
- Template holds the 'labels', and parameters holds the 'values' (kind of)
	- The values in the parameters are those that were decided through the dialog by the user.
	- "resources" section in the template is where you'll do the most of the hands-on work with the ARM template.
- Good practice to access the ARM template once you have deployed some service or done some work to read through and understand how your GUI actions have been captured and coded.
- Network interface has dependancies (3), VM has a dependacy on the network interface. 
- Also get the schedule you define codified here, e.g. autoShutDownTime
- You can capture the desired state of your environment in an ARM template and deploy as frequently as you wish - if you are deploying an already-existing resource, Azure will check that the configuration that you pushed is the same as what already exists without changing or impacting it. You can update an existing resource with changes using this method too. More details: https://docs.microsoft.com/en-us/azure/architecture/building-blocks/extending-templates/update-resource
- Can use ARM templates to deploy Linux VMs too (of course), e.g. in proeprties of `template.json`
```
"imageReference": {
	"publisher": "RedHat",
	"offer": "RHEL",
	"sku": "7.3",
	"version": "latest"
}
```
## Resource Utilization and Consumption
### Configuring diagnostic settings
- Each of the resources we've been working with have diagnostics options. Different diagnostics settings can be turned on in the diagnostics settings of a resource group or resource, and the level or type of diagnostics that you wish to capture can be configured.
- _Azure Monitor_ is a service to check all diagnostics activities within your account, and can me used to view metrics, analyse logs, and setup alerts and actions.
- Can configure each performance counter to output each metric on a custom time period as required for event logging.
- Crash dumps are available, but quite a specific use case (are you going to trawl through the contents of memory on crash to establish the cause of failure?)
- Sinks enable you to send diagnostic data to other services.
### Creating a baseline for resources
- Can end up with _lots_ of resources in your account. Having a baseline is all about DR - how would you recover from a wrecking ball going through your Azure account? 
- The concept is about storing each of your mission-critical resources in ARM template form in a vcs. You can then use the repo to make and track your changes, meaning you can capture and log changes to your azure environment.
- Two options when capturing the `template` and `parameters` files; can collect as we did above (e.g. from resource group, or a specific resource) - this will capture the deployment parameters at the time of deployment. If changes have been made to the service or group since this point, you can generate the _automation script_ for this service, which will capture future changes since deployment, and give you a script to re-deploy the service in it's current state. NB: there are sometimes services which are not covered in this auto-script generation.
- Templates are then stored in 'Templates' on Azure GUI.
- Can also create powershell scripts to do the above. 
### Alerts, Metrics, and Action Groups
- *Alerts* and aler rules allow you to configure rules for alerting when something happens within a resource group. 
- Something to happen > Action Group
- Pretty simple.
- *Metrics* are really for on the fly analytics of log data, unless to pin somewhere. 
- *Action Groups* are defined sets of actions that you can trigger following an event - can just be sending an e-mail/SMS/Push notification/voice, but can extend to Webhooks, Azure Functions, Automation Runbooks, ITSM, LogicApp. This can then be the resulting action group of any set of errors/alerts.
### Cost management and Log Analytics
- Can look at cost per tagged group, by service, etc, and filter on other dimensions. Good to do when looking for quick answers. Cost management + billing enable you to see costs across subscriptions and organisation level accounts.
- Create budgets
- Azure advisor will look at your account usage and make recommendations to reduce costs depending on your usage of the platform, as well as HA, security, and performance recommendations.
- Cloudyn allows you to monitor and manage cloud spend across AWS, GCP, and Az
- Log Analytics
	- Build a workspace, and then pull in data from resources (VMs, Storage Accounts, Activity Logs, etc)
	- {"question": "diagnostics vs metrics?"}
	- Filtering is good here, to understand what has been going on within your Azure account. 
	- This ties into alert system too - quite like Splunk.

## Azure Storage
### Create and configure Storage Accounts
- Creating storage is very similar to creating a VM. Standard vs Premium storage = HDD vs SSD. Also have to choose Account Kind GeneralPurpose vs Blob. v2 General-purpose seems to be the [recommended](https://docs.microsoft.com/en-us/azure/storage/common/storage-account-overview) type, but need to drill into this (choosing Premium reduces replication options).
- Blob storage can be publically available. 
- Cool storage is cheaper over time but has increased latency to access that data, so need to factor this into your decisioning and architecture. 
- {"question": "what kind of latencies/performance difference should you expect in hot v cold?"}
- {"question": "can you change the storage type once service has been provisoned? how do you move data between one and another type for workloads/analytics/historical reporting/data retrieval?"}
- Storage is generally accessible when you create it as 'all networks' to access, and is accessible from anywhere if you have the keys. However you can only allow access on a selected network, which will turn off public access 
- In the `settings` of the storage account, you can access `firewall and virtual network` properties, where you can change the vnet settings (e.g. add the storage to a virtual network/remove it from access via public internet)
- Can also add DDoS protection, IP firewall, to whitelist access
- Can allow exceptions for trusted Microsoft services to access the account, and read access for storage and metrics.

### Access keys and secure access signatures
- A storage account has two access keys by default. Connection string can be used in an application to connect to your storage in Azure and use the API. Two access keys so if in the case of wanting to change one, you can update the application to use the new access key, and regenerate the old one without suffering downtime/app issues. 
- Don't give access keys away to everyone. Don't want many many applications or partners using these to access the storage - no permissions management, and updating keys is a hassle. To give other trusted parties/applications access, use...
- Shared Access Signatures - allows you to be must more granular with your permissions setting (r/w, file, table, queue, blob etc). Can specify how long the access is valid for, the IP addresses allowed, protocols (HTTP/HTTPS).
- We use the key to sign the connection string., so refreshing the key invalidates the secure signatures that are already generated.

### Storage Explorer
- Open a storage instance in portal, and can access from the blade. Also native app available for download.
- {"question": "blob containers vs file storage vs queues"}
- Different types of blob storage:
	- block blob - files such as image/video that are designed to be read as complete files
	- page blob - akin to virtual HDD where individual pages of the blob can be accessed (good for batch processing perhaps?)
	- append blobs - e.g. log files, IoT sensor data, files you want to write to but don't need dynamic access to other parts of it

### Log Analytics
- Storage interacts with and makes actions available in Log Analytics. 
- `AzureActivity where ResourceGroup == "rg"`
- Every time the keys are accessed via the portal, this is logged. These kinds of logging and any actions will be based off the 'Azure Activity log' - the 'Storage Account logs' are to access other logging outputs (e.g. IIS, events, Syslog, ETW logs, service fabric events) within Log Analytics

### Redundancy
- When provisioning and within the configuration, you can select locally-redundant storage (3 copies of your data within the data center - essentially backed up. Cheapest. 11 9's availability(!). If the datacenter goes down, then won't be able to access your data. Zone redundant storage (Powershell/CLI) would protect against that), geo-redundant storage (data stored in multiple geos. 16 9's. ), and read-access geo-redundant storage (RA-GRS replicates your data to another data center in a secondary region, and also provides you with the option to read from the secondary region. With RA-GRS, you can read from the secondary region regardless of whether Microsoft initiates a failover from the primary to secondary region. ). Can also provision zone redundant storage via CLI/PowerShell
- Pay for the bandwidth of upgrading your storage/landing data and replicating it across the network. 
- More info: https://docs.microsoft.com/en-us/azure/storage/common/storage-redundancy-grs

## Virtual Networks
20-25% of the exam
- Ties all resources together
- Can be part of a resource group. Has default subnet.

### Creating a virtual network
- Address space - can define any IP address range that we want. `10.` are for private use, but can select any private network range.
- Each virtual network must have a subnet, which will be part of the address space. Can have one that spans the entire network, if you know that's what you want to have it configured as.. adding in different subnets enables you to subdivide the network into tiers, e.g. front end, middle ware, back end, with firewalls protecting the tiers.
- [CIDR naming scheme](https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing) 
- DDoS protection included for free (Basic plan) 
- Define service endpoint to allow the virtual network to connect exclusivey to built in azure services, which locks down a secure channel between these and keeps it on the azure backbone - without this, data potentially travelling over public internet.
- There are 5 reserved IP addresses in every subnet - consider this when reserving smaller IP ranges (i.e. 8 would only leave you with 3 available IP addresses)

### Peering
- `Peering` is connecting virtual networks together. Always want Resource Manager model. Use the 'I know my resource ID' if you want to connect to a resource in another subscription. If the IP address ranges clash between virtual networks, you will not be able to connect them. Can enable or disable network access between the peered networks (enable/disable switch). `Connected devices` allow you to see what is connected - a network interface is likely connected to a VM..
- Forwarded traffic checkbox allows traffic to pass from a second network into a third without a direct connection defined, but via an intermediary linked network.
- Gateway transit - if you have a VPN or Virtual Network Gateway installed on this network, do you want to allow the other network to access your gateway?
- Remote Gateways are the opposite - do you want to allow this network to access the gateways on other networks?
- It's one or ther other of the above two. Can only enable gatway transit if you have a gateway, and remote gateway if you don't have a gateway.
- *Peering needs to be set up on both resources* - virtual network >  peering. Check peering status changes from `Initiated` to `Connected`. 

### vnet to vnet connections
- Another way to connect virtual networks together.
- With vnet peering, you pay 0.01USD per GB of data transfer in and outbound. Essentialy all consumption based.
- Virtual network gateway - same tech to connect onto corporate on premises network. Pricing for vnet to vnet is done differently - you pay for bandwidth (Basic SKU has 100Mbps).
- Need to create a vnet subnet specifically for the gateway, and and provision a gateway for each side. As a result, the cost is double. 
- Gateway and virtual networks need to be deployed in the same Location.
- Public IP address is dynamic, so just fill a name in
- Gateway subnet address range automatically allocates 250 IPs, which is a waste if not planning to use them. update to `/28` or `/29` to minimize number of IPs allocated. 

### Public and Private IP addresses
- Create a public IP address by creating one of these services from Azure
- Load balancer supports IPv6 - VMs do not currently.
- Idle timeout (keep-alive settings)
- can map your custom domain into the public IP using your domain name register, or Azure DNS system

## Routing traffic
- Routes or route table - list of IP address ranges, telling Azure how to send traffic that is coming over your network. 