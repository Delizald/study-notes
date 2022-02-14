# Objectives

- Describe the design considerations for creating a virtual machine to support your apps needs.
- Explain the different availability options for Azure VMs.
- Describe the VM sizing options.
- Create an Azure VM by using the Azure CLI.

# Explore Azure virtual machines

Azure virtual machines can be used in various ways. Some examples are:

- `Development and test` – Azure VMs offer a quick and easy way to create a computer with specific configurations required to code and test an application.
- `Applications in the cloud` – Because demand for your application can fluctuate, it might make economic sense to run it on a VM in Azure. You pay for extra VMs when you need them and shut them down when you don’t.
- `Extended datacenter` – Virtual machines in an Azure virtual network can easily be connected to your organization’s network.

## Design considerations for virtual machine creation

`Availability`: Azure supports a single instance virtual machine Service Level Agreement of 99.9% provided you deploy the VM with premium storage for all disks.
`VM size`: The size of the VM that you use is determined by the workload that you want to run. The size that you choose then determines factors such as processing power, memory, and storage capacity.
`VM limits`: Your subscription has default quota limits in place that could impact the deployment of many VMs for your project. The current limit on a per subscription basis is `20 VMs per region`. Limits can be raised by filing a support ticket requesting an increase.
`VM image`: You can either use your own image, or you can use one of the images in the Azure Marketplace. You can get a list of images in the marketplace by using the az vm image list command. See list popular images for more information on using the command.
`VM disks`: There are two components that make up this area. The type of disks which determines the performance level and the storage account type that contains the disks. Azure provides two types of disks:
  - `Standard disks`: Backed by HDDs, and delivers cost-effective storage while still being performant. Standard disks are ideal for a cost effective dev and test workload.
  - `Premium disks`: Backed by SSD-based, high-performance, low-latency disk. Perfect for VMs running production workload.

   two options for the disk storage:

  `Managed disks`: Managed disks are the newer and recommended disk storage model and they are managed by Azure. You specify the size of the disk, which can be up to 4 terabytes (TB), and Azure creates and manages both the disk and the storage.
  `Unmanaged disks`: With unmanaged disks, you’re responsible for the storage accounts that hold the virtual hard disks (VHDs) that correspond to your VM disks. You pay the storage account rates for the amount of space you use. `A single storage account has a fixed-rate limit of 20,000 input/output (I/O) operations per second`. This means that a storage account is capable of supporting 40 standard VHDs at full utilization.

## Virtual machine extensions

For Windows VMs:

- `Run custom scripts`: The Custom Script Extension helps you configure workloads on the VM by running your script when the VM is provisioned.
- `Deploy and manage configurations`: The PowerShell Desired State Configuration (DSC) Extension helps you set up DSC on a VM to manage configurations and environments.
- `Collect diagnostics data`: The Azure Diagnostics Extension helps you configure the VM to collect diagnostics data that can be used to monitor the health of your application.

For linux VMs: (https://cloud-init.io/)

# Compare virtual machine availability options

## Availability zones
(https://docs.microsoft.com/en-us/azure/availability-zones/az-overview?context=/azure/virtual-machines/context/context)
An Availability Zone is a physically separate zone, within an Azure region. There are three Availability Zones per supported Azure region.

An Availability Zone in an Azure region is a combination of a fault domain and an update domain.

Azure services that support Availability Zones fall into two categories:

- `Zonal services`: Where a resource is pinned to a specific zone (for example, virtual machines, managed disks, Standard IP addresses), or
- `Zone-redundant services`: When the Azure platform replicates automatically across zones (for example, zone-redundant storage, SQL Database).

## Availability sets

(https://docs.microsoft.com/en-us/azure/virtual-machines/availability-set-overview)

is a logical grouping of VMs that allows Azure to understand how your application is built to provide for redundancy and availability. An availability set is composed of two additional groupings that protect against hardware failures and allow updates to safely be applied 
: `fault domains (FDs)` and `update domains (UDs)`.

## Fault domains

A fault domain is a logical group of underlying hardware that share a common power source and network switch, similar to a rack within an on-premises datacenter. 

## Update domains

An update domain is a logical group of underlying hardware that can undergo maintenance or be rebooted at the same time. As you create VMs within an availability set, the Azure platform automatically distributes your VMs across these update domains. Ensuring at least one instance of your application always remains running.


## Virtual machine scale sets

Azure virtual machine scale sets let you create and manage a group of load balanced VMs. The number of VM instances can automatically increase or decrease in response to demand or a defined schedule.

(https://docs.microsoft.com/en-us/azure/virtual-machine-scale-sets/overview?context=/azure/virtual-machines/context/context)

## Load balancer

Combine the Azure Load Balancer with an availability zone or availability set to get the most application resiliency. An Azure load balancer is a Layer-4 (TCP, UDP) load balancer that provides high availability by distributing incoming traffic among healthy VMs.

# Determine appropriate virtual machine size

Workload options are classified as follows on Azure:

General Purpose: Balanced CPU-to-memory ratio. Ideal for testing and development, small to medium databases, and low to medium traffic web servers.
Compute Optimized: High CPU-to-memory ratio. Good for medium traffic web servers, network appliances, batch processes, and application servers.
Memory Optimized: 	High memory-to-CPU ratio. Great for relational database servers, medium to large caches, and in-memory analytics.
Storage Optimized: High disk throughput and IO ideal for Big Data, SQL, NoSQL databases, data warehousing and large transactional databases.
GPU: Specialized virtual machines targeted for heavy graphic rendering and video editing, as well as model training and inferencing (ND) with deep learning. Available with single or multiple GPUs.
High Performance Compute: Our fastest and most powerful CPU virtual machines with optional high-throughput network interfaces (RDMA).


You can resize the VM - as long as your current hardware configuration is allowed in the new size. 
**Be cautious when resizing production VMs - they will be rebooted automatically which can cause a temporary outage and change some configuration settings such as the IP address.**

# Exercise: Create a virtual machine by using the Azure CLI
https://docs.microsoft.com/en-us/learn/modules/provision-virtual-machines-azure/5-azure-virtual-machine-azure-cli

1. Create a resource group
`az group create --name az204-vm-rg --location <myLocation>`

2. Create a VM.

```
az vm create \
    --resource-group az204-vm-rg \
    --name az204vm \
    --image UbuntuLTS \
    --generate-ssh-keys \
    --admin-username azureuser
```

**When creating VMs with the Azure CLI passwords need to be between 12-123 characters, have both uppercase and lowercase characters, a digit, and have a special character (@, !, etc.). Be sure to remember the password.**

## Install a web server
1. By default, only SSH connections are opened when you create a Linux VM in Azure. Use az vm open-port to open TCP port 80 for use with the NGINX web server:

```
az vm open-port --port 80 \
--resource-group az204-vm-rg \
--name az204vm
```

2. Connect to VM usingh SSH

`ssh azureuser@<publicIPAddress>`

3. To see your VM in action, install the NGINX web server. Update your package sources and then install the latest NGINX package.

```
sudo apt-get -y update
sudo apt-get -y install nginx
```

4. Clean up resources
`az group delete --name az204-vm-rg --no-wait`

#Knowledge check
Which of the following Azure virtual machine types is most appropriate for testing and development? 
General Purpose

Which of the below represents a logical grouping of VMs that allows Azure to understand how your application is built to provide for redundancy and availability?

Availability set