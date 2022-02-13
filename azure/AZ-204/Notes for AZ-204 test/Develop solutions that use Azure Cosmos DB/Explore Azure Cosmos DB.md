# Learning objectives

- Identify the key benefits provided by Azure Cosmos DB
- Describe the elements in an Azure Cosmos DB account and how they are organized
- Explain the different consistency levels and choose the correct one for your project
- Explore the APIs supported in Azure Cosmos DB and choose the appropriate API for your solution
- Describe how request units impact costs
- Create Azure Cosmos DB resources by using the Azure portal.

# Identify key benefits of Azure Cosmos DB
Azure Cosmos DB is designed to provide low latency, elastic scalability of throughput, well-defined semantics for data consistency, and high availability.

globally distributed and available in any of the Azure regions. Choosing the required regions depends on the global reach of your application and where your users are located.

Your application doesn't need to be paused or redeployed to add or remove a region.

## Key benefits of global distribution
The multi-master capability enables:

  - Unlimited elastic write and read scalability.
  - 99.999% read and write availability all around the world.
  - Guaranteed reads and writes served in less than 10 milliseconds at the 99th percentil

Your application can perform near real-time reads and writes against all the regions you chose for your database. 

# Explore the resource hierarchy

The Azure Cosmos account is the fundamental unit of global distribution and high availability

Azure Cosmos account contains a unique DNS name and you can manage an account by using the Azure portal or the Azure CLI.

## Elements in an Azure Cosmos account

**An Azure Cosmos container is the fundamental unit of scalability***

Azure Cosmos DB transparently partitions your container using the logical partition key that you specify in order to elastically scale.

you can create a maximum of 50 Azure Cosmos accounts under an Azure subscription (this is a soft limit that can be increased via support request).

(azure-cosmos-account.png) 

## Azure Cosmos databases

You can create one or multiple Azure Cosmos databases under your account. 

A database is analogous to a namespace.

A database is the unit of management for a set of Azure Cosmos containers.

**With Table API accounts, when you create your first table, a default database is automatically created in your Azure Cosmos account.**