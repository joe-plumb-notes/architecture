# Pillars of a great Azure Architecture
Notes and additional thoughts from reading the [Azure Architecture Fundamentals](https://docs.microsoft.com/en-us/learn/modules/pillars-of-a-great-azure-architecture/)

## Intro
The role of the architect is to deliver business value through the functional requirements of the application, and ensure the solution is designed in ways that are *scalable, resilient, efficient, and secure*. Solution architecture covers the design, build, and run phases of a technology system. The architecture must _balance_ and _align_ the business requirements with the technical capabilities needed to execute those requirements, by including and evaluation of risk, cost, and capability throughout the system and its components.

And so, our architectures will start with a solid foundation on these four pillars:

### Security
*Data is the most valuable piece of your organisation's technical footprint*. Focus here on securing access via authentication and protecting the application from network vulnerabilities. Data integrity should also be protected using tools like encryption.
Security is not an afterthought, and should be baked into the design from the outset - through design, implementation, deployment, and ops.

### Performance and Scalability
For an architecture to be truly scalable and perform well, it should be able to properly match the resource capacity to demand. This gets to the core of the application design very quickly - 12fa, statelessness, etc. This enables you to provide a great customer experience, while being cost effective too.

### Availability and recoverability
A successful cloud environment is designed in a way that anticipates failure at all levels - aka chaos engineering (?). Part of this is designing a system that can recover from the failure within the time required by customers and stakeholders (SLA).

### Efficiency and Operations
Will want to be cost-effective to operation and develop against. Inefficiencies should be identified to ensure resources are not being wasted - this requires a good monitoring architecture to enable detection of failures and problems before they happen (or before customers notice). Must also have visibility of how the application is using available resources via this monitoring framework. _TODO Need to learn more about application monitoring_.

## Design choices
In an ideal world, we would always build the most secure, high performance, highly available, and efficient environment possible .. unfortunately thats not really possible due to outside constraints and the cost of achieving this (actual money, time to deliver, operational agility). Every organisation and project will have different priorities that will impact design choices made at each pillar. _As you design the architecture, you will need to determine what trade-offs are acceptable and which are not_.

"When building an ~~Azure~~ architecture, there are many considerations to keep in mind. You want your architecture to be secure, scalable, available, and recoverable. To make that possible, you'll have to make decisions based on cost, organisational priorities, and risk."

## Design for Security
### What should I protect?
The data your organisation stores/handles is at the heart of your securable assets. Alongside the data, securing the infrastructure and the identities that are used to access it, are critically important.
Also must consider any regulatory frameworks the organisation is working within, determined by location, type of data stored, or industry that the application operates within.
A security breach can trigger substantial impact to the finances and reputation of an organisation and effect the long term health of the trust customers place in an org.

### Defence in depth
This is all about taking a multi-layered approach to securing the environment - a multi-layered approach has a greater security posture. The layers can be broken down as follows:

- data
- applications
- vm/compute
- networking
- perimeter
- policies and access
- physical security

Each layer focuses on a different area where attacks can happen, and creates this depth of protection, should one layer fail or be bypassed by an attacker. Addressing the overall system security in layers increases the work an attacker must do to gain access to the system and data within. When identifying protections to be put in place, cost will be a concern which is balanced against business requirements and overall risk to the business.

There is no single security system, control, or technology that will fully protect your architecture. This is not just about technology - people and processes are important to factor in too. You must look holistically at security and make it a requirement by default, to ensure your organisation is as secure as possible.

### Common attacks
Each layer has common attacks you will want to protect against:
- *Data layer*: exposed encryption key or using weak encryption can leave the data vulnerable if an attacker gets in.
- *Application layer*: Malicious code injection and execution, like SQL injection and cross-site scripting attacks (XSS).
- *VM/compute layer*: Malware, which involves executing malicious code to compromise a system. Once malware is present, further attacks leading to credential exposure and lateral movement through the environment can occur.
- *networking layer*: Unnecessary open ports to the internet (e.g. leaving SSH or RDP open on VMs), these can allow brute-force attacks against your systems as attackers attempt to gain access.
- *Perimeter layer*: Denial-of-service attacks (DoS) are often seen at this layer. Attempt to overwhelm network resources, forcing them offline or make them incapable of responding to legitimate requests.
- *Policies and Access layer*: This is where authentication occurs for the application. Could be modern auth protocols (OpenID Connect, OAuth), or Kerberos-based auth such as AD. Exposed credentials are a risk here, so it's important to limit the permissions of identities. We also want monitoring in place to look for possible compromised accounts, such as logins coming from unusual places.
- *Physical layer*: Unauthorised access to buildings (door drafting, theft of security badges).

In the same way that overall responsibilities are shared when using cloud, this can be re-framed in the security context depending on the type of service that is selected. Careful evaluation of the services and technologies that are being selected is necessary to ensure you are providing the proper security controls for your architecture.

## Design for performance and scalability
*what.is?*: scaling and performance optimisation are about matching available resources to an application with the demand the app is receiving. Performance optimisation includes scaling resources, identifying and optimising potential bottlenecks, and optimising application code for peak performance.

Compute resources can be scaled up (adding more resources e.g. RAM, CPU to a single instance e.g. VM or PaaS service) or out (adding additional instances). Scaling up does come with a limit, as VMs are limited to the capacity of the host they run on, and hosts have physical limitations. Scaling out adds capacity by increasing the overall total number of instances. You can conceiveably scale out forever, if more machines are added to the architecture. This does require load distribution, e.g. distribution of requests across avaiable servers, service discovery mechanism to identify active servers. _In both cases, resources can be reduced to bring cost optimisation into the picture_.

### Performance optimisation
Network and storage must be assessed to ensure performance is acceptable as both can impact the response time of the application. Selecting the right technologies will ensure you're providing the best experience. It will also include understanding how the applications themselves are performing - errors, poorly performing code, and bottlenecks in dependant systems can all be uncovered by an application performance management tool. Often these issues are hidden from users, but can have an adverse impact on overall application performance.

## Scalability and performance patterns and practices
Some considerations and patterns that can enhance the scalability and performance of your application.
- *Data Partitioning*: Large-scale data and analytics solutions require data to be partitioned for separate management and access. Partitioning strategy must be carefully decided to maximise potential benefits.
- *Caching*: mechanism to store frequently used data or assets for faster retrieval, and can be used at different layers of an application - app servers and a db (to reduce data retreival times), end users and web servers (to reduce time taken to return web pages to users). Has secondary effect of offloading requests from the db or web servers, increasing performance of other requests.
- *Autoscaling*: Process of dynamically allocating resources to match performance requirements. Takes advantage of elasticity of cloud environments, eases management overhead, and prevents requirement of an operator to monitor and make resourcing decisions.
- *Decouple resource-intensive tasks as background jobs*: Many applications have background jobs that run independently of the UI, e.g. batch jobs, intensive processing tasks, longer running processes (e.g. workflows, ETL, etc). These can be executed without requiring user input.
-*Use a messaging layer between services*: Adding a messaging layer creates a buffer for requests between the services so that requests can continue to flow without error if the application can't keep up. The application will work through the message queue in the order the requests were recieved.
-*Implement scale units*: Consider the overall scaling of the system w.r.t each resource when autoscaling services within the architecture, and consider the impact that scaling an activity might have on dependant systems. This will make scaling-out easier, and less prone to negative impact on the application. For example, adding x number of web and worker roles might require y number of additional queues and z number of storage accounts to handle the additional workload generated by the roles. A scale unit could consist of x web and worker roles, y queues, and z storage accounts. Design the application so that it's easily scaled by adding one or more scale units.
-*Performance monitoring*: distributed systems and services running in cloud are inherently complex and made of many moving parts. In any environment, it's important to be able to track the way users make use of a system, trace resource utilization, and generally monitor the health and performance of your system. This can be used as a diagnostic aid to detect and correct issues, and also help spot potential problems and prevent them from occurring.
A thorough approach to performance monitoring will enable you to determine the changes in patterns and practices your architecture will benefit from.

## Design for availability and recoverability
Lots of things can go wrong with an application - hard drives or nodes can fail, deployment issues can wipe out databases, whole data centres can go down.
Designing for availability focuses on maintaining uptime through small-scale incidents and temporary conditions (like partial network outages). You can ensure the application can handle localized failures by integrating HA into each component, and eliminating single points of failure. This also minimises the impact of infrastructure maintenance - HA designs typically aim to eliminate the impace of incidents quickly and _automatically_, and ensure the system can continue to process requests with little or no impact.
Designing for recoverability focuses on recovery from data loss and larger scale disasters. This will often involve active intervention, though automated recovery steps can reduce time needed to recover. These incidents may result in downtime or permanently lost data. DR is as much about careful planning as it is about execution.
*Including HA and recoverability in the design of your architecture protects your business from financial losses resulting from downtime and lost data*. They ensure reputation is not negatively impacted by a loss of customer trust.

### Architecting for availability and recoverability
Ensures you can meet the commitments you make to customers.
For availability, identify the SLA you're committing to. Examine the potential HA capabilities of your application relative to the SLA, and identify where you have coverage and potential gaps. _The goal is to add redundancy to components of the architecture so that you're less likely to experience an outage_. Examples of HA design components include clusters and load balancers.
For recoverability analysis, examine possible data loss and downtime scenarios. This should include exploration of recovery strategies, and the cost-benefit trade off for each. This will give important insight into your organisations priorities and help clarify the role of your application. The results should include the applications _recovery point objective_ (RPO) and _recovery time objective_ (RTO).
- *Recovery Point Objective*: The maximum duration of acceptable data loss. RPO is measured in units of time, not volume: "30 minutes of data", "four hours of data", and so on. RPO is about limiting and recovering from data loss, not data theft.
- *Recovery Time Objective*: The maximum duration of acceptable downtime, where "downtime" needs to be defined by your specification. For example, if the acceptable downtime duration is eight hours in the event of a disaster, then your RTO is eight hours.
With these two metrics defined, you can design backup, restore, replication, and recovery capabilities into the architecture to ensure you can meet these objectives. Every cloud provider offers a suite of services and features which can be used to improve your application's availability and recoverability - use existing services and best practices and avoid creating your own where possible.

## Design for efficiency and operations
Efficiency is focussed on identifying and eliminating waste within the environment. As the cloud is pay-as-you-go, waste comes from provisioning more capacity than demand requires. Waste can show up as a VM that is always 90% idle, payment for a license included in a VM when a license is already owned, retaining data on a storage medium at the wrong optimisation level, or manually repeating the build of a non-production environment.
Gathering data points from components at every layer of the application will alert you when values are outside of acceptable thresholds and track spending over time.

### Efficiency best practices
- Look at cost optimising steps like correctly sizing VMs and de-allocating VMs that aren't in use.
- Where possible, move from IaaS to PaaS services (PaaS typically costs less, and bring reduced operational costs too, as no requirement to patch/maintain VMs as this is handled by service provider)

### Operational best practices
- _Automate as much as possible_. The human element is costly, injecting time and error into operational activities. Automation should be used to build, deploy, and administer resources. This can eliminate the delay in waiting for a human to intervene.
Ensure a thorough monitoring, logging, and instrumentation system is in place throughout your architecture - will give the ability to see what's going on.
Modern architectures should be designed with devops and continuous integration in mind. This gives the ability to automate deployments using infrastructure as code, automate application testing, and build new environments as needed. _Devops is cultural as well as technical_, but can bring many benefits to organisations who embrace it.
