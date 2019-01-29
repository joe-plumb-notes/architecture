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
- *Guest-level monitoring must be turned on in order to access these stats outside of the VM.*
- Performance counters enable you to configure what data and the frequency by which this data is sent back to Azure.
- Can access graphs here, pin them to your dashboard for ease of access upon login, and export this data to Excel if required.

### VM Scale Sets (VMSS)
- Allow you to create virtual machines that are scalable, from 1-1000 VMs in a set. 
- Determine instance count i.e. number of VMs, and the instance size. 
- *Can set up auto-scaling*, with auto-scaling rules. e.g. when CPU in the scale set goes above a threshold, provision another VM (up to the max number of machines). 
- Will also need to be load balanced, with either an *Application Gateway* or a *Load Balancer* service. 
	- Application Gateway supports HTTP/HTTPS/WebSocket traffic, perhaps routing traffic based on the URL contents of the request (e.g. one server serving images, one serving video), and rules defining this distribution.
	- Load balancer is not sophisticated, does not inspect the traffic. You choose the public IP, Domain name label, virtual network, etc.. 

### ARM templates
- Azure Resource Manager (ARM) - used to organise and deploy resources in Azure. (Used to be ASM, don't do this any more.)
- If you look at a resource group, you can see each deployment has a set of details, marking the changes that were made. Each one of these includes a template and parameters files (JSON).  
- These can be downloaded, along with deployment scripts for CLI, Powershell, Ruby, and C#
- _Template_ holds the 'labels', and _parameters_ holds the 'values' (kind of)
	- The values in the parameters are those that were decided through the dialog by the user.
	- "resources" section in the template is where you'll do the most of the hands-on work with the ARM template.
- Good practice to access the ARM template once you have deployed some service or done some work to read through and understand how your GUI actions have been captured and coded.
- Network interface has dependancies (3), VM has a dependacy on the network interface. 
- Also get the schedule you define codified here, e.g. autoShutDownTime
- You can capture the desired state of your environment in an ARM template and deploy as frequently as you wish - if you are deploying an already-existing resource, Azure will check that the configuration that you pushed is the same as what already exists without changing or impacting it. You can update an existing resource with changes using this method too. More details: https://docs.microsoft.com/en-us/azure/architecture/building-blocks/extending-templates/update-resource
- Can use ARM templates to deploy Linux VMs too (of course), e.g. in properties of `template.json`
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
- *Alerts* and alert rules allow you to configure rules for alerting when something happens within a resource group. 
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
- {"question": "blob containers vs file storage vs queues vs data lake storage"}
- Different types of blob storage:
	- block blob - files such as image/video that are designed to be read as complete files
	- page blob - akin to virtual HDD where individual pages of the blob can be accessed (good for batch processing perhaps?)
	- append blobs - e.g. log files, IoT sensor data, files you want to write to but don't need dynamic access to other parts of it

### Log Analytics
- Storage interacts with and makes actions available in Log Analytics. 
- `AzureActivity where ResourceGroup == "rg"`
- Every time the keys are accessed via the portal, *this is logged*. These kinds of logging and any actions will be based off the 'Azure Activity log' - the 'Storage Account logs' are to access other logging outputs (e.g. IIS, events, Syslog, ETW logs, service fabric events) within Log Analytics

### Redundancy
- When provisioning and within the configuration, you can select locally-redundant storage (3 copies of your data within the data center - essentially backed up. Cheapest. 11 9's availability(!). If the datacenter goes down, then won't be able to access your data. Zone redundant storage (Powershell/CLI) would protect against that), geo-redundant storage (data stored in multiple geos. 16 9's. ), and read-access geo-redundant storage (RA-GRS replicates your data to another data center in a secondary region, and also *provides you with the option to read from the secondary region*. With RA-GRS, you can read from the secondary region regardless of whether Microsoft initiates a failover from the primary to secondary region. ). Can also provision zone redundant storage via CLI/PowerShell
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

### Routing traffic
- Routes or route table - list of IP address ranges, telling Azure how to send traffic that is coming over your network. 
- Assign to resource group (ofc)
- Can configure rules here to direct traffic from address profiles to hop addresses (virtual appliances, vnets, vnet gateways, or the internet)
- Also need to associate the rule with a subnet.

## Azure Active Directory
Offload account management to Azure AD, and integrate into your apps using the SDK.
- Free tier, 10 apps per user.
- Basic features: reporting, self service password reset, company branding, SLAs. 
- Premium features: self service password reset sync to on premises, MFA, conditional access (e.g. using an unknown device or from another geo, trigger MFA)
- Very simple to create - name for the service, and a domain name. 
- Users, Groups, and Roles. Can then assign access to enterprise apps via these dimensions
- Custom domain required if you don't want to use the `<aad-resource-name>.onmicrosoft.com` url. Head to Custom domain names in the settings. Prove you own it by adding a txt record into the domain registrar
- P2 account gets you to all the advanced features.
- _Identity Protection_ on your account is a series of ML algorithms that will analyse users logins and assess for vulnerabilities. Uses AI to flag suspicious login attempts, and improves your security by bringing this to your attention. Can also set policies to require MFA/password reset under certain risky conditions. 
- Users, risk events, and vulnerabilities are all investigated in this tool. 
- Configuration is where you set the params for what is considered a risky user or risky signin. Can include or exclude teams from the policies, this is a corporate decision. _Conditions_ is where the ML kicks in - set a risk level. Users can be given risk categories too - e.g. users that have not used their account in 12 months. Can force them to change password. 
- Best practices article available on azure docs

### Self service password reset
- Requires premium
- Can turn on for all or select groups of users
- Authentication methods defines the number of auth methods required to reset (MFA), and the methods that are available e.g. SMS code, mobile app code/notificiation, security questions
- Then require users to provide this data - set a time for users to have to confirm (e.g. once every 180 days)

### Conditional Access
- One of the options under security 
- Set conditions for MFA for user groups and roles

### Access Reviews
- Need p2 account or EMS (Enterprise Mobility and Security)
- Access group runs group by group or application by application, and review all members who can access, and provide approval or denial for users.
- Reviewers can be group owners, selected users, or members (i.e. self approval)

### Hybrid Identity Management
- On premises identity provider that integrates with Azure AD
- Sync enables you to extend already existing password and user model on premises with AAD. Means that your registered users on premises can use their auth for cloud services, and changes are propoaged from either end automatically.
- Federation - letting another system handle the user ids and authentication, and define a trust pattern between AAD and another authentication source.
- Seamless single sign on enables people who are signed into a domain's keys to be sync'd, so if we trust that (e.g. windows signon), then they don't have to enter user and password for AAD.
- There's also _pass-through_ auth - I didn't quite understand this
- [Comprehensive write up on considerations and method to design and adopt hybrid identity strategy](https://docs.microsoft.com/en-us/azure/active-directory/hybrid/plan-hybrid-identity-design-considerations-overview)

## Migrating Servers to Azure
### Azure Site Recovery
- _Recovery Services vaults_ is the service to look at here - this is a DR (Disaster Recovery) tool. If you've a set of VMs running on prem, in Azure, or elsewhere, you can use Site Recovery to have a replicated version of your app, multiple VMs, and your data ready to go, in case a disaster happens. 
- If your app were to go down on prem, you can initiate a failover which would deploy your app into a region in Azure, boot it up, and be ready. _This is a manual fail over_, but if it's looking like you're going to breach SLA or giving up hope of a quick fix, this could provide a solution. 
- This can also be used for migration - replicating on-premises VMs and then triggering the failover would deploy the app to Azure, which you could then use as your primary instance of the application moving forwards. 
- For DR scenarios you want the vault to exist in a different data center (obviously!), else if that DC with your application in it goes down, then you lose the vault as well.

### Preparing an ASR site
- This can be used for backups, storing VMs in the protected items area. SQL in Azure VM, Azure Storage, and Azure VMs currently supported (among others)
- Replication is another destination for migration. Can do this from On Premises - need to configure the environment and register it with the Recovery Services vault. 
- Have to download softare and install in environment, supports Hyper-V and VMWare vSphere Hypervisor, as well as non-virtualised machines. There is a tool for deployment planning to understand the cost of the migration. 
- VMWare also has vCenter converter, a tool which allows you to export VMs that run in VMware into MS VM formats. 

## Serverless computing
- VMs obviously leave you responsible for keeping environment secure, that patches are in place for your software, etc 
- Web Apps and Function apps PaaS services (all app services, plus a type of function app) are available, removing chunk of responsibility from you/your org/team. 
- No effective difference between web app, API and a mobile app. Web apps here allow you to upload your code and have it run in Azure. 
- K8s - can have a managed cluster or manage it yourself. Container instances are the quickest way to get containers up and running. 

### Functions
- Functions count as serverless if you configure them this way. Also LogicApps and service fabric. 
	- Deploying Function app, you get to choose the hosting plan. *If you choose consumption, this is true serverless*. App Service hosting is PaaS. 
	- You can sign your own custom domain and have a function app respond to this
	- Pricing is very cheap - 400,000 GB-s execution time and 1 million executions for free - so great for pieces of code that are not run frequently as you end up staying on the free tier.
	- Windows or Linux - a Linux Function app allows you to deploy a container as your function.
- *Functions need a storage account, for the code, logs, etc*.
- Portal interface to author, and lots of predefined templates (HTTP trigger, Timers, Azure Queue Storage Trigger, Blob Storage Trigger (whenever a new file gets added to storage, log it in a database for example), IoT Hub)
- Can also integrate a function with other services, e.g. Excel table, CosmosDB, OneDrive, Blob, Table Storage, 

### Logic Apps
- More like workflows that can time pieces of code together to take a number of functions, chain them together using logic app. They also integrate with many other services and external services.
- Don't require a URL, just needs a unique name within the RG
- Common triggers - events that cause this logic app to run, including time, service bus queue messages, twitter, dropbox, etc (quite cool!)
- There are also boilerplate logic apps.
- Overall, pretty simple GUI driven development enabling non-technical users to string together logic and automate annoying bits of their job.

## Event Grid
- Event driven serverless application that lists for any one event, and passes that on to a number of listneres or handlers. Glue between events that happen, and applications that can handle those kinds of events. 
- [Overview](https://docs.microsoft.com/en-us/azure/event-grid/overview)
- You can achieve similar things with Funtions and Logic apps .. but you might want to connect blob to a queue with no logic or function app in between. This is where Event grid comes in.
- _Resource providers under Subscriptions > Subscription ID  allows you to view what resources you are able to provision._
- _Is event grid Kafka?_
- Different event types that can be subscribed to based on the resource you select.

### Service Bus
- MaaS (Messaging as a Service). Most popular is the Service Bus queue. Can relay message back behind the firewall on prem via this. 
- Designing apps based on messaging allows you to decouple your app layers. Introduces complexity but increases flexibility.
- Basic, standard, and premium messaging plans. Can create queues and topics, define the amount of time a message has to live, enable dead-lettering, partitioning, duplicate detection. You get a public URL which can be used to pass messages to the bus from apps etc.

## Load balancing
- Two kinds of load balancer _Load Balancer_ service which is a level 4 service, and _Application Gateway_ which is a level 7 service. More info on levels here: https://blogs.msdn.microsoft.com/premier_developer/2018/04/25/azure-load-balancing-solutions-a-guide-to-help-you-choose-the-correct-option/
- Load balancing is the concept where you have traffic coming from a.n.other source (internet, another application in your Azure stack, etc). The traffic is trying to reach one server, but instead of having one server do all the work, you've decided to distribute the workload across multiple servers doing the same job. Load balancer distributes the traffic, to ensure one server doesn't get overwhelmed with all the traffic.
	- Level 4 load balancer makes determintations based on 5 factors: source IP, port, destination IP, port, and protocol. Based on these factors, it can determine which back-end pool to send traffic to, using an algorithm to distribute. Not a lot of intelligence applied here, can be round robin (other ways of handling too)
	- Level 7 load balancing does so at the applicaion/presentation/session layer. This additional context information can be used to make more complex and informed load balancing decisions, e.g. unique client sessions can be identified based on the session cookie and used to deliver all a clients requests to the same server. Server persistance using cookies can be based on the servers cookie, or active cookie injection where a load balancer cookie is inserted into the connection. [source](https://freeloadbalancer.com/load-balancing-layer-4-and-layer-7/)
- Can also detect if something goes wrong with one of the servers, and drop it out of the rotation/worker pool. 
- Choices when provisioning in Azure:
	- _Type_: Internal or Public - Public means that it will also create and have a public IP assigned, so it is accessible from public web. Can choose to use an existing public IP too. Ok with a dynamic IP address if you're going to use the domain name to reference it.
		- Can also choose IPv6 - load balancer is one of the few devices that supports this at the moment. So, if you have an application that needs to work over IPv6, you can use a load balancer as a proxy to other underlying services, and the load balancer will address translate from the v6 address to the private IP of the machines/services behind in your subscription.
- When would you use a Standard load balancer?
	- Basic supports up to 100 instances behind the scenes. Standard supports 1000.
	- Basic supports VMs in a single availability set or scale set. Standard load balancer can support any VM in a single vnet, including a blend of VMs, availability sets, scale sets
	- Basic does not support HTTPS
	- Standard has additional stay-alive options (they stay alive on instance probe down _and_ all probes down)
	- Standard supports availability zones, Azure Monitor (Basic only supports Azure Log Analytics)
	- Standard are secure by default, requires IP whitelisting (Basic are open by default, and network security groups are optional)
	- Basic is free, Standard is charged by number of rules, data processes (in and outbound associated with resource)

### Load balancer settings
- _Backend pools_ - where traffic gets sent. When adding, you have to choose what you want to add to the pool. On the basic plan, you can only choose one "associated to" type. 
	- The availability set you want to add to the backend pool must exist within the same region as your load balancer. 
	- Then add VMs into the availability set, and their associated NICs (Network Interface Card)
- _Health probes_ determine health of servers in the backend pool. E.g. if a server is not responding within a certain time frame, it will kick that from the VM pool. Can set up to ping a port on a VM via TCP, or ping a port and path via HTTP. Set interval and unhealthy threshold. Can set a `health.htm` or `probe.htm` so not having to pull home page (potentially heavy) every time. Of course, a HTML page could continue working while the home page is down, so there is some strategy here.
- _Load Balancing Rules_ - define the front end IP address, he port you are accepting traffic on and the port you are sending traffic from. Can do Network Address Translation (NAT), where you send on a different port than you recieve. 
	- You select if you support session persistence too (Client IP, Client IP and Protocol), and idle timeout for this. If you dont set, each piece of incoming traffic will go to a random VM in the pool. 

- _Configuring front end IP_ - there are going to be occasions where you have more that one application/endpoint running on your server farm, so you need to have separate IP addresses to call. To do this:
	- add a new IP to the load balancer
	- add a new load balancing rule, probably with a new health probe to check this service too. 

## Setting up an Application Gateway
- A more sophisticated and intelligent device than a load balancer. 
- More sophisticated pricing - Standard, Standard V2 (just like load balancing, provides load balancing features) WAF, WAF V2 (Web Application Firewall, more sophisticated again)
- Manual vs Autoscaling , so your gateways can grow based on demand. 
- Application gateway also uses availability zones: way of distributing multiple devices (application gateways in this case), and you can select the zones. _NB: note every region has availability zone support_. If HA is your goal, you will have to deploy multiple, and select which zones they go into.
- Then, need to choose which vnet your app gateway gets deployed onto. Choose the address space and subnet range. 
- Same distinction between public and internal load balancing.

### Configuration
- Now have a default backend pool - can target IP addresses or FQDN (fully qualified domain names) as your backend pool - meaning servers that you are directing to can be hosted elsewhere. Can also point to VMs, or App Services (would normally have their own load balancer built in, but if you had multiple instances of app services, you could route traffic using the app gateway).
- Don't select the availability set, just the individual VMs. 
- App Gateway needs to have a listener open, and a rule associated that defines where the traffic is routed. `Basic` is for all traffic, or `Path Based` will allow you to create rules that passes traffic for different endpoints to different backend pools.

## MFA and Role Based Access Control
2 types of MFA servers, downloading the server software and running this in your environment, or cloud based MFA.
### Turning on MFA
- Available as an add-on using AAD, $1.40/mo/user, _unless_ you're a premium AAD user.
- Users > Multi-Factor Authentication (opens in a new tab)
- Need to be in your AD domain to enable MFA - can't be gmail, for example.
- Users need to register for MFA @ https://aka.ms/MFASetup
### Conditonal Access
- Within security settings on the AAD service, you can find the baseline policy - require MFA for high level roles (e.g. Admins). Can set to enable in the future (so people have chance to set it up), or deploy rule immediately. 
- Can create new rules to define when MFA will kick in, e.g. if logging in from a location that is not trusted.
- Sign in risk is an ML algorithm that identifies how typical the sign in is. 
- Named Locations can be added as a trusted location (IP range)
### Fraud alerts
- MFA (under security) settings. Enabling fraud alerts allow users to report if they recieve a 2-step verification address and block that account, i.e. that users login had been compromised. 
- Blocked accounts go under block/unblocked user list, which takes 90 days to go unblocked. 
### MFA one-time bypass
- e.g. if a users phone is out of battery, so they cannot verify the 3rd factor.
- In the same settings area, you can set up one-time bypass with a time limitto enable someone to log in. 
### Verifying your Identity
- In service settings, you have verification options - phone call, text message to phone, notification to mobile app, verifiation code from mobile app or hardware token. 
- Can also enable trusted IPs where MFA is skipped for requests coming from users on (for example) your own network. 
### Role Based Access Control (RBAC)
- Authentication as a service is the remit of what we've just looked at. But.. once that user is authenticted, what can they do? What do they have access to? What is their authorization level?
- We can do this in resource groups - all resources and resource groups have Access Controls (IAM). Here, can control who can access the resources, make modifications, add new resources (to a RG) or new users who could then access.
- Handy 'check access' thing to put in a users creds and see hat they can see. If you want to change that users access, go to 'add a role assignment'. Decide what role you want to give - top 3 are 'owner' (keys to the castle! grant access to other users), 'contributor' (full permissions, create, delete and edit resources, but cannot give other users that permission), and 'reader' (read only, can see, can't make changes).
- Lots of other pre-defined contributor and reader roles for different services, so you can give very fine graind permissions to individuals. 
- Can also author 'deny' rules to definitely make sure a user/group/role cannot accidently through some other way inherit access.
- _Custom RBAC roles_ are created at the subscription permissions level, using powershell. Take an existing role, save it as a JSON file, and then edit this file to allow/deny as you require. 

## App services
- An 'in-between' service for running applications. VMs you control the environment completely, Logic apps and function apps you just upload your codeand Azure is entirely responsible for the execution.
- App Services is the PaaS service to create code, package it up, and deploy this to az.
- Create a web app - must have a name, unique across all of azure. 
- Can choose Linux or Windows web app, either can run containers or uploaded code. Windows apps can push logs into application insights. Linux, you can choose the runtime stack (.net, PHP, Node, Python, Ruby). You choose the app service plan, like a hosting plan. There is a free tier, choose where it goes, and then service plan (Dev/Prod/Isolated)
- Prod includes load balancing, auto scaling, backups, more compute, disk space, etc.
- Scale up and scale down can be done to manually scale your app (service plan).
- Can host many web apps on the same service plan. 10 free apps, 100 on Shared plan, and unlimited apps from basic onwards
- Can deploy via FTP server, or head to deployment center when you can connect github, azure repos, bit bucket, local git, onedrive, dropbox, or external (mercurial etc) _NB: This changes based on the OS you choose. The above options are for windows app services. Linux container option only gives you Azure DevOps deployment option_.
- Deployment slots enable staging/testing/prod slots, that you can wrap permissions around to determine who/what can deploy to them
### Containers
- No free plan for linux dev environments
- Container settings can be changed e.g. point at a new container registry/repo
- Azure k8s is obviously for fully fledged microservices apps. But can run single containers here if you like. 
### Webjobs as background jobs
- _What happens if you have part of your web app that needs to run by someone hitting a URL on you app?_ Traditionally would have installed it as a windows service (windows), set it up as a cron job (linux), or some other way to have the code run. Introducing _webjobs_ (in the settings of an app), an easy way to run scripts as background processes, and runs in the same context as your web app so you have a way of uploading code into azure, and setting that to run however you wish. 
- you upload the file into the app (bit odd..), then select the trigger (schedule?) with a CRON expression, or as a continuous service, and save the job. Lots of filetypes are supported. _NB: Webapps can time out after 20 mins without activity, which would stop your webjob. So need to either disable timeout or ensure you have enough traffic hitting your site/app to ensure this code would run_.

## k8s
- Nothing exceptional or different about provisioning a cluster for k8s, as one would expect. You can attach monitoring, but I'm guessing this is for the k8s cluster not the containers.
- `az aks get-credentials --resource-group $rg --name $clustername` gives cloud shell the context for the cluster.
- `kubectl get nodes` to view nodes in cluster. Deploy app using `yaml` file, which defines where the containers that make up the application come from. No code in the yaml.
- `kubectl apply -f $app.yaml` will push the deployment of the service to the cluster. 
- `kubectl get service` shows you the status of the deployment once running. If one of the instance of a service requires a public IP, can take a moment to deploy this and have it provisioned. 
- Don't get a lot of insights into the cluster from the service page. Have to launch k8s dashboard, which runs on the azure cli.
### k8s dashboard
- Need to install az cli. `az aks install-cli`
- `az login` to login
- `az aks get-credentials` command to provide context to az for your k8s cluster
- `az aks browse` will fire up the k8s dashboard.

### docker
- Docker compose file looks like the k8s yaml, I'm guessing docker compose is an orchestration engine like k8s? it is! just a way to run multi-container apps locally. {"question": "docker compose vs k8s? does anyone use docker compose for local development?"}
- `docker-compose up` to run your app

## Implementing Secure Data Solutions
### Intro to data security
- Clearly lots of benefits to using the cloud - pay for what you use, easy to scale to larger data solutions without having to purchase hardware and undergo effort for scaling, easy to replicate data around the globe, access can be granted to any app if authorized, and lots of features like tuning and optimization done for you.
- Despite this - security is still very much a factor. Don't forget it!
- There are risks of unauthorized data access, data loss, or undetectable data modification.
- Lots of security solutions:
	- Firewalls that require whitelist IP
	- Virtual network service endpoints
	- Services can be deployed on non-public networks so anything that's accessible to a virtual network can remain not-public.
	- Encryption at rest and in transit
	- Store and use access keys and certificates securely (KeyStore)
	- Create special identities for your applications and virtual machines so they are running in a very restricted profile

### Data Storage Encryption
- Azure offers different database options `["SQL Server in a VM", "Azure SQL Database", "SQL Data Warehouse", "Azure Database for MySQL", "Azure Database for PostgreSQL", "Cosmos DB"]`, with different ways of encrypting stored data. 
- _Transparent Data Encryption_ (TDE) means the data is stored to the physical disk in an encrypted state, but when you read and write data using supported databases `[Azure SQL Database", "SQL Data Warehouse"]` you are not dealing with encryption (it's transparent!) Azure manages the keys on our behalf. 
- The `master` database is not encrypted (you get this whe  you provision the db service, along with user and temp dbs). This is where information around the encryption is stored.
- TDE is on by default, and can be turned off, but you should know what you're doing and why you want to switch this off.
- BYOK (Bring your own Key) is supported by TDE. When you create your database Microsoft will generate a pulic/private key pair for you and manage the encryption. To use your own key and CA, generate it and store it in a KeyVault, and grant vault access to the SQL DB using its unique AAD identity. The server will then use it's AAD identity to authenticate with AAD for access to the Key Vault, then send get, wrap key, and unwrap key requests to the asymmetric key in Key Vault for db encryption key protection. [More](https://docs.microsoft.com/en-us/azure/sql-database/transparent-data-encryption-byok-azure-sql)
- TDE supports:
	- Geo-restore (replicate a db from one geo to another) and geo-replication
	- Self-service point-in-time restores, copying a database, and restoring a database.
- TDE does _not_ support:
	- Database export (the data is not encrypted, stored in unencrypted BACPAC files, if you do export), it's like a `zip` format with extra metadata.'
- Azure Storate Encryption (SSE, Storage Service Encryption) is enbaled by default for new accounts, cannot be disabled.
	- Anything will be encrypted when stored on physical storage, so if you were storing the BACPAC files, these would be encrypted (not the files, but the storage account would be).
	- Support BYOK for unmanaged disk. Managed disk is encrypted by Microsoft and they manage the keys.

### Azure Key Vault
- Center of a lot of the security solutions, as this is where secrets and certs are to be stored. 
- Can protect user id's and passwords that an application might need to access external services, access keys for APIs, certificates themselves, any other encryption keys. 
- Follows FIPS (Federal Information Processing Standard 140-2 Level 2)
- Key Vault is used to centeralize your secrets. So instead of having a web config or application config file which contains all passwords and API secret keys etc, you can store these in an place where very very few people can access. 
- Key Vault can monitor access and use of secrets, produce log files to analyse and understand when secrets were accessed, which is a more secure way of operating. 
- Key Vault should be part of your solutions when developing large-scale solutions within Azure, to simplify management - don't need connection string everywhere in your application code with the database password, just point the application to Key Vault and manage it centrally via _Key Vault access Request_. App goes through AAD for authentication, via managed service identities. The application authenticating with AD gives it access to the secrets in the key vault. 

### Storing ARM template secrets
- When your ARM template needs to use passwords and secrets. 
- ARM templates = JSON (declarative), i.e. plaintext
- How do you do it?
	- Store the password in the key vault as a secret 
	- set type to 'securestring' in template:
	```
	...
	},
	"adminPassword: {"type": "securestring"
	  }, 
	...
	```
	- within parameters file, set the password to a reference to your az key vault. All secrets have a unique identifier.
	- the account that runs this code is able to access the secrets.

### Storing application secrets
- Same issue, don't want to embed secrets in your code. 
- Keyvault can store Keys and Secrets
- Secrets are whatever you want them to be (text, 25KB maximum)
- If secret is so big, can consider encrypting it before you pass into the keyvault. 
- Key vault keeps versions of each secret, and each version can be accessed by a URL: `https://{keyvault-name}.vault.azure.net/{object-type}/{object-name}/{object-version}`
- When uploading to key vault, set the name and value, can also set expiry and not before dates, whether or not it is enabled, then it will store alongside the date it was created and last updated. 
- MS does not stop you reading a secret after it has expired or before it's ready to be released (expiry date and not before date are for info for your app.)
	- store the key separate from the app, so app doesn't have to be re-deployed, can just modify keyvault to give applications access as the keys change. 
	- applications can be limited in what secrets they can access in the vault, and can list the secrets it has been given access to (can help with development).

### Storing Certificate secrets
- Certificates aka x509 certificates. 
- Key vault can interact directly with some CAs - if you set it up to work with a CA, it can create keys itself, and renew those keys automatically.
- You can also have someone manage this, being the person in the loop between the key vault and the CA (i.e. a not supported one).

## Develop for the cloud
### Message-based integration architecture
- Lots of message-based solutions in Azure
- _SendGrid_ is an e-mail delivery service - a rest API that your applications can use to send or recieve emails.
	- Scalable, so can use it for sending lots of messages.
	- Third party.
	- Can process incoming mail, too. So programs can monitor an e-mail address, connect to SMTP to read the emails and do your processing based on the contents of those.
	- _Requires a SendGrid account_
- _Event Grid_ allows you to connect lots of event sources in azure to other services. e.g. new files in blob account or new message on service bus queue can trigger any kind of program (logic apps, azure functions, web hooks, etc)
- _Azure Relay Service_ allows messages to be passed from internet, through your firewall to your internal network. 
	- Enable Azure Relay, which gives you an endpoint. You then install an application inside your network on a server, which establishes the communication between the inside of your network and az. 
	- Legacy way is using  (WCF) services.
	- Good thing about Azure Relay is you dont have to make changes to your firewall settings.
	- Does outbound connections too, so messages can pass from azure to your internal network over a channel that was initiated as an outbound communication.
	- Two types of connections: 
		- _Hybrid connections_ are based on BizTalk services, using standard web sockets and multi-platform support
		- _Windows Communication Framework (WCF) relays_ are the legacy offering, only supporting HTTP comms with limited platform support. Microsoft .NET .. maybe proprietary.
- _Notification Hub_ send messages to mobile applications. Very flexible as you can push messages to all devices, or make rules based on users OS/version/user/location/personalized messages. Enables back end services to communicate with apps (and therefore users).
- _Event Hub_ - things produce events. Rather than having things call your applications and services directly, you can have Event Hub accept and store those events, saving you having to worry about scaling and the load, then recievers can take those events and act on them. 
	- Messages can flow in order
	- Can deal with millions of messages
	- Messages can be partitioned
- _Service Bus_ support two different delivery guarantees (at least once, at most once (so different than storage queues))
- _Microsoft Graph API_ for interacting with Office 365. Can send/recieve e-mails via Outlook. 
Messaging is used to develop highly available and responsive apps. It allows you to disconnect the load that the front end application is receiving from the load that the rest of the app needs to deal with, which means with a thin front end you can scale relatively inexpensively. 

### Develop for Autoscaling
- Application resources can grow to meet demand, and shrink when the demand is no longer there - really leveraging the flexibility provided by cloud. 
- There are recommended patterns:
	- On and off - e.g. a job that fires every four hours, so that server should be turned off when it's not running.
	- Adding resources - typically when an organisation or application is in a state of growth, only concerned with being able to add new servers. CPU many no be the measure. 
	- Unpredictable autoscaling - monitoring CPU or memory usage or some other metric that tells you your servers are under heavy load and adding more resources. Azure allows you to expose a metric via Application Insights, and then based your scaling from what your application is reporting. 
	- Predictable autoscaling - my application is typically used by these people, so via schedule (clock), can scale resources accordingly.