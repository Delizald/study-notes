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

## Azure Cosmos containers
An Azure Cosmos container is the unit of scalability both for provisioned throughput and storage.
The items that you add to the container are automatically grouped into logical partitions then distributed across physical partition
based on the partition key.

When you create a container, you configure throughput in one of the following modes:


- `Dedicated provisioned` throughput mode: The throughput provisioned on a container is exclusively reserved for that container and it is backed by the SLAs.
 
- `Shared provisioned` throughput mode: These containers share the provisioned throughput with the other containers in the same database (excluding containers that have been configured with dedicated provisioned throughput). In other words, the provisioned throughput on the database is shared among all the “shared throughput” containers.

## Azure Cosmos items
The following shows the mapping of API-specific entities to an Azure Cosmos item:

- Cosmos entity: Azure Cosmos item
- SQL API: 	Item
- Cassandra API: Row
- Azure Cosmos DB API for MongoDB: Document
- Gremlin API: Node or edge
- Table API: Item

## Explore consistency levels

With Azure Cosmos DB, developers can choose from five consistency models.
From strongest to more relaxed:

- `strong`
- `bounded staleness`
- `session`
- `consistent prefix`
- `eventual`

The consistency levels are region-agnostic and are guaranteed for all operations regardless.
Read consistency applies to a single read operation scoped within a partition-key range or a logical partition. The read operation can be issued by a remote client or a stored procedure.

(five-consistency-levels.png)

# Choose the right consistency level
The following simple considerations will help you make the right choice in many common scenarios.

## SQL API and Table API
- For many real-world scenarios, session consistency is optimal and it's the recommended option.
- 
- If your application requires strong consistency, it is recommended that you use bounded staleness consistency level.
- 
- If you need stricter consistency guarantees than the ones provided by session consistency and single-digit-millisecond latency for writes, it is recommended that you use bounded staleness consistency level.
- 
- If your application requires eventual consistency, it is recommended that you use consistent prefix consistency level.
- 
- If you need less strict consistency guarantees than the ones provided by session consistency, it is recommended that you use consistent prefix consistency level.
- 
- If you need the highest availability and the lowest latency, then use eventual consistency level.
- 
- If you need even higher data durability without sacrificing performance, you can create a custom consistency level at the application layer.

## Cassandra, MongoDB, and Gremlin APIs
(https://docs.microsoft.com/en-us/azure/cosmos-db/cassandra/apache-cassandra-consistency-mapping)
(https://docs.microsoft.com/en-us/azure/cosmos-db/mongodb/consistency-mapping)

## Consistency guarantees in practice

- When the consistency level is set to `bounded staleness`, Cosmos DB guarantees that the clients always read the value of a previous write, with a lag bounded by the staleness window.
- 
- When the consistency level is set to `strong`, the staleness window is equivalent to zero, and the clients are guaranteed to read the latest committed value of the write operation.
- 
- For the remaining three consistency levels, the staleness window is largely dependent on your workload. For example, if there are no write operations on the database, a read operation with eventual, session, or consistent prefix consistency levels is likely to yield the same results as a read operation with strong consistency level.

If your Azure Cosmos account is configured with a consistency level other than the strong consistency you can find out the probability that your clients may get strong and consistent reads for your workloads by looking at the `Probabilistically Bounded Staleness`(PBS) metric.

`Probabilistic bounded staleness` shows how eventual your eventual consistency is.

# Explore supported APIs

These APIs allow your applications to treat Azure Cosmos DB as if it were various other databases technologies, without the overhead of management, and scaling approaches.

## Core(SQL) API
This API stores data in document format. It offers the best end-to-end experience as we have full control over the interface, service, and the SDK client libraries. Any new feature that is rolled out to Azure Cosmos DB is first available on SQL API accounts.

If you are migrating from other databases such as Oracle, DynamoDB, HBase etc. SQL API is the recommended option.

## API for MongoDB

This API stores data in a document structure, via BSON format. it does not use any native MongoDB related code.
API for MongoDB is compatible with the 4.0, 3.6, and 3.2 MongoDB server versions. Server version 4.0 is recommended as it offers the best performance and full feature support.

## Cassandra API

This API stores data in column-oriented schema. on Cassandra API you don’t need to manage the OS, Java VM, garbage collector, read/write performance, nodes, clusters, etc. he Cassandra API enables you to interact with data using the Cassandra Query Language (CQL).

## Table API

This API allows users to make graph queries and stores data as edges and vertices. Use this API for scenarios involving dynamic data, data with complex relations, data that is too complex to be modeled with relational databases. Gremlin API currently only supports OLTP scenarios.

- `OLTP (On-line Transaction Processing)` is involved in the operation of a particular system. OLTP is characterized by a large number of short on-line transactions (INSERT, UPDATE, DELETE). The main emphasis for OLTP systems is put on very fast query processing, maintaining data integrity in multi-access environments and an effectiveness measured by number of transactions per second. In OLTP database there is detailed and current data, and schema used to store transactional databases is the entity model (usually 3NF). It involves Queries accessing individual record like Update your Email in Company database.
 
- `OLAP (On-line Analytical Processing)` deals with Historical Data or Archival Data. OLAP is characterized by relatively low volume of transactions. Queries are often very complex and involve aggregations. For OLAP systems a response time is an effectiveness measure. OLAP applications are widely used by Data Mining techniques. In OLAP database there is aggregated, historical data, stored in multi-dimensional schemas (usually star schema). Sometime query need to access large amount.

## Discover request units

The cost of all database operations is normalized by Azure Cosmos DB and is expressed by `request units` (or RU).

The cost to do a point read, which is fetching a single item by its ID and partition key value, for a 1KB item is 1RU.

 No matter which API you use to interact with your Azure Cosmos container, costs are always measured by RUs

(requests-units.png)

- The type of Azure Cosmos account you're using determines the way consumed RUs get charged. There are three modes in which you can create an account:

- `Provisioned throughput mode`: In this mode, you provision the number of RUs for your application on a per-second basis in increments of 100 RUs per second. To scale the provisioned throughput for your application, you can increase or decrease the number of RUs at any time in increments or decrements of 100 RUs. You can make your changes either programmatically or by using the Azure portal. You can provision throughput at container and database granularity level.

- `Serverless mode`: In this mode, you don't have to provision any throughput when creating resources in your Azure Cosmos account. At the end of your billing period, you get billed for the amount of request units that has been consumed by your database operations.

- `Autoscale mode`: In this mode, you can automatically and instantly scale the throughput (RU/s) of your database or container based on it's usage. This mode is well suited for mission-critical workloads that have variable or unpredictable traffic patterns, and require SLAs on high performance and scale.

## Exercise: Create Azure Cosmos DB resources by using the Azure portal

(https://docs.microsoft.com/en-us/learn/modules/explore-azure-cosmos-db/8-create-cosmos-db-resources-portal)

# Knowledge check

1. When setting up Azure Cosmos DB there are three account type options. Which of the account type options below is used to specify the number of RUs for an application on a per-second basis?

Provisioned throughput

Which of the following consistency levels below offers the greatest throughput?

Eventual