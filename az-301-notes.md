# AZ-301 notes
My notes in prep for the AZ-301 exam. Notes primarily taken while following content from https://app.pluralsight.com/paths/certificate/microsoft-azure-architect-design-az-301

## Designing a Data Protection Strategy
### Data Protection basics
- This is important! Data is the most important resource for any company, and the integrity and privacy of that data needs to be safeguarded. Additionally, data might need to be shared with customers or partners, so you need to ensure you are doing this in a secure and governed way. 
- There are many types of data within an organisation, which can be classified in different ways (sensitivity to company confidentiality, it's content e.g. PII)
- If the data is classified, this can be used to direct it's privacy, secirity, and levels of protection required.
	- Some regulation is mandated by regulations or compliance standards, or may simply be highly desirable to ensure the organisation is safeguarded against unnecessary risk.
	- If encrypted data is leaked, there is no requirement to alert affected parties (?!). Ensuring data is encrypted and managed securely saves potential issues down stream in the event of a security breach. Data loss can result in financial loss, organisaitonal reputation damage, customer dissatisfaction, etc.
- There are [compliance offerings listings](https://www.microsoft.com/en-us/trustcenter/compliance/complianceofferings) available on Azure, which detail statements and list services that are compliant vs different industries regulations.
	- 'Learn more about compliance' gives more info about how to manage compliance in Azure, and perform Risk Assessment using compliance manager. _Cloud Security Diagnostic Manager_ shows what you should be doing to track overall solution compliance against different compliance requirements 
 
### How can data be protected?
- Can put a big wall around it - a _firewall_ in networking terms. Iscolated networks, deep packet inspection, traffic inspection. Focus here is ensuring the perimeter is safe and nothing unexpected can get through.
- Useful but .. what if someone with authorized access makes a copy of the data to USB? 
- _Encryption_ is the other way. So even if the data does get out, it's useless, provided it's the right level of encryption (and unless the method and keys used for encryption escape too). 
- Need to think about what happens when assets are decomissioned - what happens when the disk fails or a server is decomissioned? How do you make sure this data remains secure? 
- Depending on the requirement, you may want to use both. (Likely!)
	- May have multiple layers of encryption!
	- Think _defence in depth_ - protect access in the first place, as well as encryption, and be mindful of the _user experience_ as well - no use being secure if you're out of business!
- Need to ensure your employees and customers _remain productive_. What can you do to enable the business to be as secure as possible without getting in the way?
- Need to also know if there is a breach. How can you alert on this? Assume it is going to happen, and have a plan in place for when it does. Need to be able to detect rouge activity. 

### Where is the data?
- Can be spread across many locations, in different formats, with different levels of data access. Could be on prem, in the cloud, as files, via an API, a stream.. in a DC, in the cloud, on premises.. 
- You must understand all the types of data in your organisations landscape, and your responsibility level and requirements for protecting this data.
- It's the organisation's responsibility to ensure data is stored according to regulations - not the vendor.

### Protecting user data
- Could be any user data (docs, spreadsheets, etc) - encrpytion is the best method of protection here. Data can be encrypted at rest on the primary central storage. 
- Different levels of potential encryption (file, volume (Bitlocker/dencrypt))
- Azure VMs have same volume level encryption.
- Secure enclaves can be created on mobile devices to ensure data is protected even if the entire device is not encrypted. Can also remote wipe without wiping the entire device.
- Potential to lose secure oversight if users move data around, deposit in their personal onedrive etc
- _User Document Protection_ (Azure information protection) ensures data is encrypted at rest and in transit, and control how the data can be used (e.g. can read, but not copy and paste). Can time lock it, and track whether successful or unsuccessful attempts have been made to access data resources. 
- Relies on users classifying data and applying correct policies. Auto-classification (requires AIP P2) can be applied by Azure Information Protection and Cloud App Security based on the content.
- CAS can also detect unusual actions around data, and block it if necessary. 
- Labels = Classification definitions. Can automatically apply or recommend the application of a classification rule, based on your own custom or ootb rules.
- CAS has policies for mass download etc, as well as content detection, and automatically encrypt. 
- The labels live with the data too.

### Enterprise data loss prevention
- Many best practices from on premises world translates to cloud, but the implementation may be different. 
- RBAC and governance is key - full RBAC in azure across subscriptions, resource groups, and individual resources. Gets very very granular, down to the actions available for different resources.
- Can also use privilaged identity management - giving the ability to elevate up for additional resource actions if required. E.g. privaliged actions via MFA to enable access for a time-limited session.
- Apply principle of least privalige - whats the minimum required access required to perform the role?
- Blueprints useful to ensure a new subscription/rg follows pre-thought corporate policy for access control and permissions.

### Network Security
- If the best defence is not to be there, then in security terms, the best defence is to not let people touch the data == network control.
	- vnets are iscolated from other resources by default, so if you want resources to be able to communicate across different vnets, you have to enable it using e.g. network peering. There is also the ability to go out to the internet, or publish things to the internet via public IPs. Policies can prevent this, by denying ability to create public IPs unless on a certain subnet which you might be using as a DMZ.
- Network and Application security groups allow you to define rules to say "traffic of these IP ranges, these protocols, these ports, are allowed to talk to these other IP addresses". App Security Groups can be used to tag some network cards, as web front ends, or middle ware components. Then use rules to say NICs tagged as web front end are allowed to accept traffic on 443 from the internet, or NICs tagged as database can accept traffic on 1433. _You want to control the flow of traffic_
- vnets can also expose service endpoints, which enable the vnet subnet to be specified as part of a firewall rule for certain services (e.g. SQLDB Cosmos DB, Key Vault, Storage Accounts). Rather than saying "these certain public IP addresses can talk to me, and _everything in Azure_, you can create rules to accept traffic from those public IPs, and from this virtual subnet in this vnet _only_.
- Virtual appliances are pre-configured software in a vm that make connetions to one or more subnets, and then use user defined routing to allow you to define the next hop for packet flow, so rather than going the default azure route, you can bounce via that virtual applicance.
- Also app gateway, WAF, windows firewall, DDoS protection to help protect
- ExpressRoute - rather than going via a public IP, can use a private network for traffic.

## Data Encryption
### Types of encryption
- Lots of different types.
	- Symmetric - same key for encrypting and decrypting data. Highly performant but key handling can be difficult. (How do I transport that key?)
	- Asymmetric - set of keys (typically public and private), where one is used to encrypt, the other can use it to decrypt. Computationally more expensive, but solves the key distribution problem. Need a trusted key repo for the public key - certs are commonly used. 
	- One-way hash - hash-generated that cannot be reversed to original value (not useful for storage), but good for guaranteeing "this is the original data"
- Combination can be used, e.g asymmetric to share a symmetric key
### Azure key vault
- Secure storage solution for keys and secrets via hardware security modules (HSMs). 
	- Secrets : data that can be written and read (e.g. passwords)
	- Keys : written to the service but cannot be read, and are instead used with the service (e.g. sign, encrypt operations etc)
	- Certs : really just a key in an envelope, with actions and lifecycle mechanisms that key vault can do for you around those certs.
- Passwords can be extracted in the portal and via API
- Keys (HSM and Soft) - Both are stored in the HSM, for software protected keys, any cryptographic operations are performed in the compute of the VM. For HSM, the computation is done in the HSM itself. 
	- No option to look at it or extract from the portal.
- All protected and authenticated by AAD
- Can elect to BYOK.
- If you;re trying to use your key vault, how does your application authenticate with the KV to get access to the secrets? Almost every compute service can be assigned a _Managed Identity_, which means allows the apps to authenticate and get access to the key vault.
### Azure storage service encryption
- All storage accounts have automatic 256 AES encrytion at rest for both standard and premium services enabled, with keys stored in a Key Vault. Ordinarily MS managed, customer can manage the keys for *Blob and file storage* i.e. not managed disks. 
- Storage Accounts > Encryption > [] Use your own key to apply your own key to suit your requirements if necessary.
- VMs can have additional level of encryption, within the guest OS, e.g. using bitlocker/dencrypt to encrypt the volume. These keys would also be stored in key vault. This is using Azure Disk Encryption. 
- Additional encryption available within applications as well. 
### Azure database services encryption
- SQL db and SQL DW support TDE, using MS mananged or Customer managed key.
- Real time encryption and decryption technology to the database, applies across log files, backups, and the db. _Completely transparent to users of the service_.
- Enabled by default. 
- Additionally, Azure SQL DB supports 'Always Encrypted', which keeps the data encrypted at rest, in transit, and in use in the SQL instance. 
- Cosmos also uses encryption at rest for all data except logs - database, attachments, and backups.
- Most other services support encryption at rest. [Lots more info here, including list of supported data sources.](https://docs.microsoft.com/en-us/azure/security/azure-security-encryption-atrest)
### Encryption in transit
- Any comms over the internet is typically encrypted via SSL/TLS protocols. 
- On prem comms to services in vnets, should utilize site to site (S2S), point to site VPN (P2S) where it's a machine connecting into that vnet, or ideally ExpressRoute, where private connection from on premises to azure services. In additon to private peering with ER (connect IP space to one or more vnets), can also do Microsoft Peering, where other services can be advertised over that ER connection, so you can go via ER connection rather than the public internet.
- ExpressRoute comms are not encrypted by default, as it's a private network, however depending on the workload, you can choose to encrypt as required.
- Storage services can be configured to _always require secure transfer_. Storage account > Confiuration > Secure Transfer Required [y/n]. Must use HTTPS on REST APIs, and with files, SMB 3.0 with encryption must be used with this enabled.
- Databases use SSL/TLS by default, best practice to disable `TrustServerCertificate` to ensure trust chain is checked to prevent MITM attacks
### Encrpytion in use
- Some workloads need data to be encrypted as it is being used (!). HyperV ensures protection between the different VMs that are running our workloads and prevents data being seen by different services. However, sometimes need hardware assurance that the data is in an isolated environment.
- Azure Confidential Compute uses new hardware capabilities (Intel SGX) to ensure data cannot be accessed outside the protected enclave (Trusted Execution Environment, TEE)
- DC series VMs will expose this capaibilities to leverage this. 

## Protecting Data Access
### Requirements
- Number of considerations beyond "can I get network communication going here" in order to access the data. Is the service meeting the performance requirements in order to be useful to support the system and experience?
- Am I enforcing least-privilege/JIT access? The more access you give someone, the more chance there is they will do something you don't want them to.
- Is the data available where it's needed, in a resilient manner? What are my requirements for availability?
### Secure data access
- Who, What and When are data being accessed?
- _Encrypting data at rest and in transit does not bring much benefit if sufficient controls are not in place to restrict who has access rights_.
- May also want to control where the data can be accessed from, and the conditions of access. 
- Normally dealing with some kind of identitity (user or service) - _how do we validate an identity_?
	- Avoid using generic identities and keys where possible. Means there is poor auditing. Unique accounts make it much easier. Would also end up with one entity with the sum permissions on a wide range of services required across different apps, which would mean all apps have more permissions than they individually need. 
	- Use SAS / Azure AD integration for storage accounts. Access keys give complete access to a storage account, whereas permissions using Shared Access Signature are much more granular.
	- AD opens up additonal controls (e.g. conditional access)
	- Privileged identitity management enables lifting of permissions as required. 
### Limiting resource access
- Many services have firewall functionality to limit public IPs allowed to communicate with that service.
- Can only enable access to storage account on a specific vnet/subnet range. This will enable a service endpoint on that vnet for the storage, then give it permission via the firewall config of that service. Can add the service endpoint yourself onto the vnet first. 
- Service endpoints are great to use if you need to control access to resources from azure services.
- Using AAD gives you the conditional access functionality, which allows more requirements for defence.
### Data Scalability
- Understand the performance characteristics and scale of the options available and choose the most optimal solution that meets requirements. Can change the scale down the line, too.
	- e.g. blobs have three tiers available to impact accessibility and cost of data storage, in addition to performance options.
		- Performance, access tier options can be changed. Archive cannot be accessed. Hot and cool affect availability. Hot is about 2x cold storage to store. Archive is 1/5th cool. However hot storage is cheaper to access. 
		- Premium/general pricing for storage performance.
- Resource using the data may have their own limits, e.g. VM has a level of throughput/IO/etc. Need to make sure this meets/exceeds expected performance required.
- For databases, there are different models around storage and compute scalability - DTU model is simple and pre-configured, blended together, and DTUs are purchased to improve performance. vCore model allows you scale compute and storage separately - _NB: this does not extend to memory - you can only choose GB storage you require, the memory scales with the number of vCores_. [Useful blog post](https://medium.com/@raduvunvulea/demystifying-azure-sql-dtus-and-vcore-78d65d4e15c5)
	- Elastic pool shares resources across instances
	- MI has great compatibility with existing on prem services - as these are dedicated VMs, these are only available in DTUs
- Cosmos is priced on request units per second, and the amount of data you store. Very easy to scale, and pay for the performance you need.
### Data availability
- Is the data usable within reasonable level of latency (defined by application)
- Is the data available in the event of an incident?
- *Any data service should always be available in at least two separate regions.* - geo redundant makes sure your data is replicated [to the paired region](https://docs.microsoft.com/en-us/azure/best-practices-availability-paired-regions). Always within the same geo-political boundary (except brazil > south central us)
- Can add additional replicas (up to 4) with Azure SQL DB - set if this is readable, or failover, etc
- How the data can be used is defined in the default consistency
- Redis Cache can be used to improve app performance by storing commonly accessed data across i-memory components. Data throughout the world can use CDN.

## Data Management Strategies
### Key Data attributes
- Data has various attributes, which describe entites (rows) in the data. This is particualrly important for schema-based data stores e.g. relational databases. 
	- Type (int, char, string, binary)
	- length 
	- Required (null allowed?)
	- Primary key / other key (must be unique, do we want to search based off it?)
- Fixed schemas drive data design.

### Managed vs Unmanaged Data Services
- _What bit am I responsible for?_
- Managed (PaaS) vs unmanaged (anything installed in an IaaS VM) drives:
	- operational responsibility (who's patching, monitoring, backups, performance behind the scenes)
	- Security responsibility (firewalls, auditing, ACLs)
	- Compliance and regulation adherance responsibility
- _It may not be a single entity_ - there is likely a shared responsibility model. e.g. I may still want to know who is accessing a service, if it's not my responsibility to manage, through Log Analytics, Azure Monitor etc.
- Best to remove overhead you have to manage by opting for the highest level of managed service you can adopt that satisfies your requirements.

### Data Solution Requirements
- Number of key requirements:
	- Consistency - how can we be assured that the data read includes all data, including the last writes, in a distributed scenario? We can't acknowledge the application through the writing applicaion until it has been written in a majority of nodes. 
		- Need to balance performance vs assurance that the data read is the last written. (CAP Theorum). 
		- Can have partition tolerance, availability, and consitency. As the app owner and data architect, we need to decide whether availability or consitency is most important. 
	- Availability (SLA) - is it accessible (uptime), can I get to the data?
	- Durabilitiy - ensure data is not lost - typically going to see high durability in data services. 
	- Retention - how long can I keep data for, and when must I remove it?
	- Latency - not considered by CAP theorum - can I have copies of the data near to me to reduce this?
	- Cost - more functionality might make it more costly, can I stop/start? scale independantly? Temperature of data? Data sovreignty 

### Azure SQL Database
- Deployed in a region with up to 4 secondaries in the same or other regions which are asynchronously replicated to
- Secondaries can only be used for read access - writes always go to primary.
	- Therefore, for most recent data, read-write access on primary must be requested, else can read from secondary.
- Not a multi-write solution. Can enable automatic failover using failover groups, which will automate continuity of service.
- 4 9s availability
- SQL MI is about maximising compatibility - rather than deploying to VM, you're bringing it into a PaaS model, and retain compatibility
- Elastic pool is a shared resource model that enables higher resource utilisation efficiency, with all the databases within an elastic pool sharing predefined compute resources within the same pool. 

### Data Analysis Services
- Data is only valuable if it can be used to provide answers.
- Commonly, services are based around the storage - decoupling compute from the storage allows you to optimise your spend and use different compute resources without having to move/change where the data is. 
- Azure data lake storage = just store the data
	- Compatible with many different analytics services
	- Presnets via WebHDFS too
	- Land as much data as you want without limit of data stored or ingestion speed.
	- Gen1 used specific storage tied to the analytics, but Gen2 uses regular blob storage, meaning optimal cost and compatibility via blob and HDFS-compliant interfaces
- HDInsight (Hadoop + other bits, can't autoscale) and Databricks (Spark, can autoscale, priced based on nodes deployed and databrick units) for analytics for data processing. 
- Azure Analysis Services and Power BI - modeling and reporting on data. AAS supports wide range, and supports semantic models across disparate data sources. Power BI is the common front end. 

### Data Cache Services
- Goal is to improve performance, using non-durable cache of results.
- Azure Redis Cache is most commonly used. Good to consider as an intermediary layer when system is interacting with a back end, that could be optimized by faster access to that data. 

## [DTUs (Database Transaction Units) Sizing](https://docs.microsoft.com/en-us/azure/sql-database/sql-database-service-tiers-dtu)
- Basic, Standard, and Premium sizing.
	- All suitable for dev and prod
	- all 4 9s availability
	- Basic has 7 days backup retention, Standard + has 35
	- Different CPU options on Standard and Premium, just Low on Basic
	- Premium has significantly higher IOPS (48 vs 2.5 of others) per DTU
	- Premium has better IO latency (2ms, vs 5ms read, 10ms write for others)

## [Designing a Monitoring Strategy for Azure Data](https://app.pluralsight.com/player?course=microsoft-azure-monitoring-strategy-data-platform-designing&author=william-myers&name=a6d203a4-a188-4230-97d0-188a777d14c3&clip=0&mode=live)
### Metrics and alerts for data solutions
- Azure Monitor is what we use here.
	- Different metric namespaces for StorageV2: Account, Blob, File, Queue, Table. 
	- Types of metrics: Transactions and Capacity
		- Transactions: Availiability, Egress, Ingress, Success e2e latency (includes processing time of the request, total round trip time), success server latency, transactions
		- Capacity: Used capacity (for storage account, in bytes), Blob capacity, and more... this is boring

### Alerts and action types
- Alert target (specific resource or all of a type of resource)
- Alert criteria - what causes the alert to trigger? If availability is less / greater than a number
- Specific details
- Action groups - who and how to send the alert to

- Action types 
	- email/sms/push/voice 
	- Azure functions
	- Logic app
	- Webhook (i.e. call out to external webhook not necessarily in Azure)
	- ITSM (e.g. servicenow)
	- Automation Runbook - Azure automation to fire off powershell script for .. anything

## [Designing an API Management Strategy](https://app.pluralsight.com/player?course=microsoft-azure-api-management-strategy-designing&author=reza-salehi&name=dfe0e04d-dc3a-48a2-b8ee-bcc9aefaffac&clip=0&mode=live)
- APIM helps orgs publish existing internal APIs. Acts as a proxy for the back end APIs. Clients call Azure API Management Gateway, which calls the backend APIs in turn.
- So, why API management? 
	- Makes consumption easy
	- API versioning and revisioning
	- Authentication via AAD
	- Usage reporting and monitoring
	- Define quotas for callers, and throttling
	- Provides documentation, mocking (returning something for the client in the event the backend server becomes un-responsive), IP filtering, and response caching.
- API Gateway is the endpoint that accepts the calls from the client.
	- Update clients to point to the gateway.
	- Enables you to transform APIs on the fly without needing to update the client, using policies.
	- The gateway verifies API keys, tokens, certs, and other credentials, acting as another layer of the security landscape.
	- Can cache backend responses if set up
	- API activity is also logged for monitoring and analytics
- Azure Portal is the admin interface where APIM is setup
	- Define or import back-end APIs
	- Logical API groupings into 'products', logical containers for operations within an API. Can have multiple APIs within a product.
	- API access can be managed for users.
- Developer portal is a web panel for developers who intend to consume the APIs.
	- Read documentation, try the APIs via interactive console, create an account and get an API key for use, and access API call analytics on their own usage.
- SLAs - Developer (No SLA), Basic (99.9), Standard (99.9), Premium (99.95)
- Can add legal terms to your API
- [API policies](https://docs.microsoft.com/en-us/azure/api-management/api-management-howto-policies) can be authored as inbound (changes to the request from the client), backend (statements to be applied before the request is forwarded to the backend service), and outbound (statements to be applied to the response) processing. 
### Scaling APIM
- To increase performance of your API management instance.
	- Can change the service tier, or add or remove performance units to a tier (developer tier does not support changing units). A nit is composed of dedicated azure resources and has a certain load-barding capacity as number of API calls per month. Add these under 'scale and pricing'. 
- No AAD on basic, No vnet on basic or standard, multi-region only on premium, 0,2,4,unlimited scale units, 10MB, 50MB, 1GB, 5GB cache per unit, and 500, 1k, 2.5k, 4k max rps/unit
- _When to scale?_ - autoscale (standard and premium) for scale up and down, or use 'capacity indicator' to make informed decisions for manual scaling. 
### APIM and automation
- can write powershell scripts to automate APIM tasks, which can be run manually or by Azure Automation Workflows. cmdlets in Azure Automation can be paired with cmdlets for other services, to automate complex tasks across azure services.

## Azure Policy
[Docs link](https://docs.microsoft.com/en-us/azure/governance/policy/overview)

TODO: [Cosmos DB](https://app.pluralsight.com/player?course=microsoft-azure-data-management-strategy-design&author=john-savill&name=f016bd92-a4b8-4906-9124-6f4d029caa48&clip=6&mode=live)
