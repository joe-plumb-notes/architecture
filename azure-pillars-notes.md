# Pillars of a great Azure Architecture
Notes and additional thoughts from reading the (Azure Architecture Fundamentals)[https://docs.microsoft.com/en-us/learn/modules/pillars-of-a-great-azure-architecture/]

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
