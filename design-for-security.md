# Designing for Security
Understand key architectural security considerations as you design environments on the cloud.

## Defence in depth
All about delivering security in layers, number of layers is going to vary depending on the application or system, idea being to slow the advance of an attack _aimed at acquiring unauthorised access to information_. Each layer is in place to provide protection, so if one later is breached, a subsequent layer is in place to prevent further exposure. The common principles used to define a security posture are confidentiality, integrity, and availability, known collectively as CIA.
- *Confidentiality* - principle of least privilege. This restricts access to information only to individuals explicitly granted access, including protection of user passwords, remote access certificates, and email content.
- *Integrity* - prevention of unauthorised changes to information at rest or in transit. _How do you ensure the information you have received has come from a reputable source?_ A common approach is for the sender to create a unique fingerprint of the data using a one-way hashing algorithm. The hash is sent to the receiver along with the data. The data's hash is re-calculated and compared to the original by the receiver to ensure the data wasn't lost or modified in transit.
- *Availability* - Ensure services are available to authorised users. DoS attacks are a prevalent cause of loss of availability. Natural disasters also drive system design (what happens when the data centre goes down?) to prevent single points of failure and deploy multiple instances of an application to geo-dispersed locations.

In almost all cases, attackers are after *data* stored in dbs, on a disk inside virtual machines, stored on SaaS applications, or in cloud storage. It's the responsibility of those storing and controlling access to the data to ensure it's properly secure, in line with regulatory requirements etc.
For *applications*, ensure they are free of vulnerabilities, that sensitive application secrets are stored in a secure medium, and that security is a design requirement for all application development. Bake security considerations into the culture of the organisation, encouraging all teams to ensure their applications are secure by default.
Malware, unpatched systems, and improperly secured systems open your environment to attacks. Always ensure that *compute* resources are secure, by securing access to VMs and implementing endpoint protection to keep systems patched and current.
At the *networking* layer, the focus is on limiting network connectivity across all resources and only allow what is required (deny by default). Segment the resources and use network level controls to restrict communication - by doing this you reduce the risk of lateral movement throughout the network. Also ensure connections to on-premises networks are secure (VPN).
Identifying attacks and eliminating their impact at the *perimeter* is important - have an alert system, use DDoS protection, and perimeter firewalls to identify and alert on malicious attacks against the network.
*Policies and access control* is all about ensuring identities are secure, that access is only granted to what is needed, and all changes are logged. Use SSO and multi-factor auth, audit events and changes, and control all access to infrastructure and change control mechanisms.
The *physical security* of the building(s), and access to computing hardware within the data centre is the first line of defence.

The threat landscape is constantly evolving, therefore a security architecture is never complete. Azure Security Centre provides unified security management to understand and respond to security events.

## Identity management
Digital identities are integral in todays business and social interactions on premises and online - more recently, mobile devices have become the primary way people interact with services. People expect to be able to access services anywhere, at any time, which has driven the development of identity protocols that can work at scale across many devices and OS's.

### Single Sign-on (SSO)
The more identities a user has to manage, the greater the risk of a credential-related security incident. More passwords and logins with differing password policies make it harder for people to remember credentials. More identities also costs more to manage - it increases the number of helpdesk tickets requesting password resets and lockouts, and creates a challenge if a user leaves the organisation.
SSO = one ID and password, and access across applications is granted to a single user identity (vastly simplifying the security model).
Azure Active Directory (AD) is a cloud-based identity service, can be sync'd with existing on-premises AD or used stand-alone. Admins and developers can control access to data and apps using centralised rules and policies. You can also use Azure AD to combine multiple data sources into an intelligent security graph, enabling threat analysis and real-time identity protection on all accounts in AD.
Directories are sync'd with _Azure AD Connect_.

### Authentication and access
*Multi-factor authentication* provides additional security by requiring 2 or more elements for full authentication - based on _something you know_ (password, answer to security question), _something you posess_ (mobile app/notification/SMS, togen generating device), or _something you are_ (biometrics).

*Conditional access policies* are about ensuring additional requirements are met before access can be granted to the user. Blocking logins from a suspicious IP address, or denying access from devices without malware protection could limit access from risky sign ins.

### Securing legacy applications
This can be done using _Azure Application Proxy_, for users who currently authenticate to an application using Windows Integrated Authentication from domain-joined machines behind the corporate firewall.

## Infrastructure protection
https://docs.microsoft.com/en-gb/learn/modules/design-for-security-in-azure/4-infrastructure-protection
