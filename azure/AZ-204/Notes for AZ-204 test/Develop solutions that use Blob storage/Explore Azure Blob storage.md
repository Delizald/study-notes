# Learning objectives
- Identify the different types of storage accounts and the resource hierarchy for blob storage.
- Explain how data is securely stored and protected through redundancy.
- Create a block blob storage account by using the Azure Cloud Shell.

# Explore Azure Blob storage
Blob storage is optimized for storing massive amounts of unstructured data.
Unstructured data is data that does not adhere to a particular data model or definition, such as text or binary data.

Blob storage is designed for:
- Serving images or documents directly to a browser.
- Storing files for distributed access.
- Streaming video and audio.
- Writing to log files.
- Storing data for backup and restore, disaster recovery, and archiving.
- Storing data for analysis by an on-premises or Azure-hosted service.

An Azure Storage account is the top-level container for all of your Azure Blob storage. The storage account provides a unique namespace for your Azure Storage data that is accessible from anywhere in the world over HTTP or HTTPS.

## Types of storage accounts
- Standard: This is the standard general-purpose v2 account and is recommended for most scenarios using Azure Storage.
- Premium: Premium accounts offer higher performance by using solid-state drives. If you create a premium account you can choose between three account types, block blobs, page blobs, or file shares.

The following table describes the types of storage accounts recommended by Microsoft for most scenarios:

- Standard general-purpose v2 (Blob, Queue, and Table storage, Azure Files). Standard storage account type for blobs, file shares, queues, and tables. Recommended for most scenarios
- Premium block blobs (Blob storage). account type for block blobs and append blobs. Recommended for scenarios with high transactions rates.
- Premium page blobs (Page blobs only). storage account type for page blobs only.
- Premium file shares (Azure Files). Supports NFS file shares in Azure Files. account type for file shares only.

## Access tiers for block blob data
- Hot Access: New storage accounts are created in the hot tier by default. optimized for frequent access of objects in the storage account.
- Cool Access. Data is stored for at least 30 days.  optimized for storing large amounts of data that is infrequently accessed.
- Archive:  only for individual block blobs. optimized for data that can tolerate several hours of retrieval latency and will remain in the Archive tier for at least 180 days.

# Discover Azure Blob storage resource types
Blob storage offers three types of resources:

- The storage account: provides a unique namespace in Azure for your data. if your storage account is named mystorageaccount, then the default endpoint for Blob storage is: `http://mystorageaccount.blob.core.windows.net`
- A container in the storage account: organizes a set of blobs, similar to a directory in a file system. The container name must be lowercase. can include an unlimited number of containers, and a container can store an unlimited number of blobs.
- A blob in a container.

Azure Storage supports three types of blobs:
- `Block blobs` store text and binary data, up to about `4.7 TB`. Block blobs are made up of blocks of data that can be managed individually.
- `Append blobs` are made up of blocks like block blobs, but are optimized for append operations. Append blobs are ideal for scenarios such as logging data from virtual machines.
- `Page blobs` store random access files up to `8 TB` in size. Page blobs store virtual hard drive (VHD) files and serve as disks for Azure virtual machines.

## Explore Azure Storage security features
- All data (including metadata) written to Azure Storage is automatically encrypted using Storage Service Encryption (SSE).
- Azure Active Directory (Azure AD) and Role-Based Access Control (RBAC)
  - You can assign RBAC roles scoped to the storage account to security principals and use Azure AD to authorize resource management operations such as key management.
  - Azure AD integration is supported for blob and queue data operations. You can assign RBAC roles scoped to a subscription, resource group, storage account, or an individual container or queue to a security principal or a managed identity for Azure resources.

- Data can be secured in transit between an application and Azure by using Client-Side Encryption, HTTPS, or SMB 3.0.
- Azure Disk Encryption: OS and data disks encryption.
- shared access signature: delegated data access to objects.

## Azure Storage encryption for data at rest.
Data in Azure Storage is encrypted and decrypted using 256-bit AES encryption, one of the strongest block ciphers available, and is FIPS 140-2 compliant.
encryption is enabled for all new and existing storage accounts and cannot be disabled. 
All Azure Storage resources are encrypted, including blobs, disks, files, queues, and tables. All object metadata is also encrypted.
Encryption does not affect Azure Storage performance. 
There is no additional cost for Azure Storage encryption.

## Encryption key management
You can rely on Microsoft-managed keys or if you choose to manage encryption with your own keys, you have two options:
- You can specify a customer-managed key to use for encrypting and decrypting all data in the storage account
- You can specify a customer-provided key on Blob storage operations.

The following table compares key management options for Azure Storage encryption. (Encryption key management.PNG)

# Evaluate Azure Storage redundancy options
Redundancy ensures that your storage account meets its availability and durability targets even in the face of failures.

The factors that help determine which redundancy option you should choose include:
- How your data is replicated in the primary region
- Whether your data is replicated to a second region that is geographically distant to the primary region.
- Whether your application requires read access to the replicated data in the secondary region if the primary region becomes unavailable.

## Redundancy in the primary region
Data in an Azure Storage account is always replicated three times in the primary region.

Two options available:

- `Locally redundant storage (LRS)`: Copies your data synchronously three times within a single physical location in the primary region. LRS is the least expensive replication option, but is not recommended for applications requiring high availability or durability.
- `Zone-redundant storage (ZRS)`: Copies your data synchronously across three Azure availability zones in the primary region. For applications requiring high availability, Microsoft recommends using ZRS in the primary region, and also replicating to a secondary region.

## Redundancy in a secondary region
When you create a storage account, you select the primary region for the account. The paired secondary region is determined based on the primary region, and can't be changed.

Two options available:

- `Geo-redundant storage (GRS)` copies your data synchronously three times within a single physical location in the primary region using LRS. It then copies your data asynchronously to a single physical location in the secondary region. Within the secondary region, your data is copied synchronously three times using LRS.
- `Geo-zone-redundant storage (GZRS)` copies your data synchronously across three Azure availability zones in the primary region using ZRS. It then copies your data asynchronously to a single physical location in the secondary region. Within the secondary region, your data is copied synchronously three times using ZRS.

# Exercise: Create a block blob storage account
https://docs.microsoft.com/en-us/learn/modules/explore-azure-blob-storage/6-create-block-blob-storage-account

## Create account by using Azure Cloud Shell
`az group create --name az204-blob-rg --location <myLocation>`


```
az storage account create --resource-group az204-blob-rg --name <myStorageAcct> --location <myLocation> --kind BlockBlobStorage --sku Premium_LRS
```


## Knowledge check

Which of the following types of blobs are used to store virtual hard drive files? 

Page blobs

Which of the following types of storage accounts is recommended for most scenarios using Azure Storage?

General-purpose v2