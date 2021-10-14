## Introduction to Azure fundamentals

  

**Cloud Computing** it's the delivery of computing services over the internet (cloud). This includes servers, storage, databases, networking, SW, analytics.

  

**Azure Portal** is a web based console where you can manage Azure resources: Build, manage, and monitor everything from simple web apps to complex cloud deployments.

**Azure Marketplace** connects users with MS partners and vendors to purchase apps and services.

**Azure most common services**:
-   Compute:
	- Azure Virtual Machines
	- Windows or Linux virtual machines (VMs) hosted in Azure.
	- Azure Virtual Machine Scale Sets
	- Scaling for Windows or Linux VMs hosted in Azure.
	- Azure Kubernetes Service
	- Cluster management for VMs that run containerized services.
	- Azure Service Fabric
	- Distributed systems platform that runs in Azure or on-premises.
	- Azure Batch
	- Managed service for parallel and high-performance computing applications.
	- Azure Container Instances
	- Containerized apps run on Azure without provisioning servers or VMs.
	- Azure Functions
	- An event-driven, serverless compute service.
	
-   Networking:
	- Azure Virtual Network
	Connects VMs to incoming virtual private network 	
	(VPN) connections.
	Azure Load Balancer
	Balances inbound and outbound connections to applications or service endpoints.
	Azure Application Gateway

Optimizes app server farm delivery while increasing application security.

Azure VPN Gateway

Accesses Azure Virtual Networks through high-performance VPN gateways.

Azure DNS

Provides ultra-fast DNS responses and ultra-high domain availability.

Azure Content Delivery Network

Delivers high-bandwidth content to customers globally.

Azure DDoS Protection

Protects Azure-hosted applications from distributed denial of service (DDOS) attacks.

Azure Traffic Manager

Distributes network traffic across Azure regions worldwide.

Azure ExpressRoute

Connects to Azure over high-bandwidth dedicated secure connections.

Azure Network Watcher

Monitors and diagnoses network issues by using scenario-based analysis.

Azure Firewall

Implements high-security, high-availability firewall with unlimited scalability.

Azure Virtual WAN

Creates a unified wide area network (WAN) that connects local and remote sites.
-   Storage
-   Mobile
-   Databases
-   Web
-   Internet of Things (IoT)
-   Big data
-   AI
-   DevOps

**The Azure free account includes:**

-   Free access to popular Azure products for 12 months.
-   A credit to spend for the first 30 days.
-   Access to more than 25 products that are always free.

**The Azure free student account offer includes:**

-   Free access to certain Azure services for 12 months.
-   A credit to use in the first 12 months.
-   Free access to certain software developer tools.

**Types of cloud models**
**Public cloud**

Services are offered over the public internet and available to anyone who wants to purchase them. Cloud resources, such as servers and storage, are owned and operated by a third-party cloud service provider, and delivered over the internet.

**Private cloud**

A private cloud consists of computing resources used exclusively by users from one business or organization. A private cloud can be physically located at your organization's on-site (on-premises) datacenter, or it can be hosted by a third-party service provider.

**Hybrid cloud**

A hybrid cloud is a computing environment that combines a public cloud and a private cloud by allowing data and applications to be shared between them.

**Cloud computing advantages**:

**High availability**:  Goal is to provide no downtime to users.

 **Scalability**:
	- Vertical: Adding RAM or CPU to VM.
	- Horizonal: Adding more instances and resources.
	
**Elasticity**:  Configure apps to use autoscaling (dynamic resources).

**Agility**: Deploy quickly as requirements change.

**Geo distribution**: Deploy apps and data to datacentes around the globe.

**Disaster recovery**: Ensuring data is safe in event of disaster (backup, replication, geo-distrib).

**Capital vs Operating expenses**

-   **Capital Expenditure (CapEx)**  is the up-front spending of money on physical infrastructure, and then deducting that up-front expense over time. The up-front cost from CapEx has a value that reduces over time.
-   **Operational Expenditure (OpEx)**  is spending money on services or products now, and being billed for them now. You can deduct this expense in the same year you spend it. There is no up-front cost, as you pay for a service or product as you use it.

**Cloud service models**

IaaS: It aims to give you complete control over the hardware that runs your application. Instead of buying hardware, with IaaS, you rent it.

PaaS: Same as IaaS but gives you the following: Agility (no need to config servers), pay for use, no tech skills required to deploy, cloud benefits. productivity (focus on app development).

SaaS: software that's centrally hosted and managed for you and your users or customers


![Illustration showing the cloud responsibility model.](https://docs.microsoft.com/en-us/learn/azure-fundamentals/fundamental-azure-concepts/media/shared-responsibility-76efbc1e.png)

**Serverless computing**

Enables developers to build applications faster by eliminating the need for them to manage infrastructure.


# Azure architectural components

![Screenshot of the hierarchy for objects in Azure.](https://docs.microsoft.com/en-us/learn/azure-fundamentals/azure-architecture-fundamentals/media/hierarchy-372fef74.png)

**Resources**: Instances of services you create (VM, storage or SQL)
**Resource groups**: Logical containers which recources are grouped, managed and deployed.
**Subscriptions**: Groups users accounts and resources created by those accounts. 
**Management groups**: These groups help you manage access, policy, and compliance for multiple subscriptions. **All subscriptions in a management group automatically inherit the conditions applied to the management group.**

**Azure regions**: A regions is a geo area on the planet with at least one datacenter. Some services or VM features are only available in certain regions, such as specific VM sizes or storage types. There are also some global Azure services that don't require you to select a particular region, such as Azure Active Directory, Azure Traffic Manager, and Azure DNS.

**Special Azure regions**:
These are used for compliance or legal purposes:
-   **US DoD Central, US Gov Virginia, US Gov Iowa and more:**  These regions are physical and logical network-isolated instances of Azure for U.S. government agencies and partners. These datacenters are operated by screened U.S. personnel and include additional compliance certifications.
-   **China East, China North, and more:**  These regions are available through a unique partnership between Microsoft and 21Vianet, whereby Microsoft doesn't directly maintain the datacenters.

**Availability zone**: Availability zones are physically separate datacenters within an Azure region. An availability zone is set up to be an _isolation boundary_. If one zone goes down, the other continues working.

[**Regions that support availability zones**](https://docs.microsoft.com/en-us/azure/availability-zones/az-region) 

Azure services that support availability zones fall into three categories:

-   **Zonal services**: You pin the resource to a specific zone (for example, VMs, managed disks, IP addresses).
-   **Zone-redundant services**: The platform replicates automatically across zones (for example, zone-redundant storage, SQL Database).
-   **Non-regional services**: Services are always available from Azure geographies and are resilient to zone-wide outages as well as region-wide outages.

**Azure region pairs**: Azure region paired with another within the same geography at least 300 miles away.

advantages of region pairs: quick restoration, minimize downtime, data resides within the same geography **(except Brazil south)** for tax- and law-enforcement jurisdiction purposes.

 **Azure Resource Manager** is the deployment and management service for Azure. All capabilities that are available in the Azure portal are also available through PowerShell, the Azure CLI, REST APIs, and client SDKs. Functionality initially released through APIs will be represented in the portal within 180 days of initial release.

Benefits:

 - Use declarative templates istead of scripts.  A resource manager template is a JSON file which defines what you want to deploy.
 - Manage resources for solution as group instead of individually.
 - Consistent state.
 - Define dependencies between resources (correcto order deployment).
 - Apply Access control. Role-based access control (RBAC) permission
 - Clarify billing.

# Azure subscriptions and management groups