# Objectives

- Describe the differences between logical and physical partitions
- Choose the appropriate partition key for your solution
- Create a synthetic partition key

## Explore partitions

Logical partitions are formed based on the value of a partition key that is associated with each item in a container. All the items in a logical partition have the same partition key value.

Example:

a container holds items. Each item has a unique value for the UserID property. If UserID serves as the partition key for the items in the container and there are 1,000 unique UserID values, 1,000 logical partitions are created for the container.

## Logical partitions

A logical partition consists of a set of items that have the same partition key.

A logical partition also defines the scope of database transactions. You can update items within a logical partition by using a transaction with snapshot isolation. When new items are added to a container, new logical partitions are transparently created by the system. You don't have to worry about deleting a logical partition when the underlying data is deleted.

## Physical partitions

A container is scaled by distributing data and throughput across physical partitions. Internally, one or more logical partitions are mapped to a single physical partition.

Unlike logical partitions, physical partitions are an internal implementation of the system and they are entirely managed by Azure Cosmos DB.

The number of physical partitions in your container depends on the following:

- The number of throughput provisioned (each individual physical partition can provide a throughput of up to 10,000 request units per second). The 10,000 RU/s limit for physical partitions implies that logical partitions also have a 10,000 RU/s limit, as each logical partition is only mapped to one physical partition.
- 
- The total data storage (each individual physical partition can store up to 50GB data).

Focus on logical partitions instead of physical partitions.

Throughput provisioned for a container is divided evenly among physical partitions. A partition key design that doesn't distribute requests evenly might result in too many requests directed to a small subset of partitions that become "hot." Hot partitions lead to inefficient use of provisioned throughput, which might result in rate-limiting and higher costs.

# Choose a partition key

A partition key has two components: `partition key path` and the `partition key value`. (`{ "userId" : "Andrew", "worksFor": "Microsoft" }`)

once you select your partition key, it is not possible to change it in-place. If you need to change your partition key, you should move your data to a new container with your new desired partition key.

For all containers, your partition key should:

- `Be a property that has a value which does not change`. If a property is your partition key, you can't update that property's value.

- `Have a high cardinality`. In other words, the property should have a wide range of possible values.
 
- `Spread request unit (RU) consumption and data storage evenly across all logical partitions`. This ensures even RU consumption and storage distribution across your physical partitions.

## Partition keys for read-heavy containers

For large read-heavy containers you might want to choose a partition key that appears frequently as a filter in your queries.

efficient routing (https://docs.microsoft.com/en-us/azure/cosmos-db/sql/how-to-query-container#in-partition-query)

If most of your workload's requests are queries and most of your queries have an equality filter on the same property, this property can be a good partition key choice.

Your container will require more than a few physical partitions when either of the following are true:

- Your container will have over 30,000 RU's provisioned

- Your container will store over 100 GB of data

## Using item ID as the partition key

If your container has a property that has a wide range of possible values, it is likely a great partition key choice (item id for example).

The item ID is a great partition key choice for the following reasons:
- There are a wide range of possible values (one unique item ID per item).
- Because there is a unique item ID per item, the item ID does a great job at evenly balancing RU consumption and data storage.
- You can easily do efficient point reads since you'll always know an item's partition key if you know its item ID.

## Create a synthetic partition key

It's the best practice to have a partition key with many distinct values, such as hundreds or thousands. If such a property doesn’t exist in your data, you can construct a synthetic partition key. 

## Concatenate multiple properties of an item

For example, consider the following example document:

```
{
"deviceId": "abc-123",
"date": 2018
}
```

For the previous document, one option is to set /deviceId or /date as the partition key.
Another option is to concatenate these two values into a synthetic `partitionKey` property that's used as the partition key.

```
{
"deviceId": "abc-123",
"date": 2018,
"partitionKey": "abc-123-2018"
}
```

In real-time scenarios, you can have thousands of items in a database. Instead of adding the synthetic key manually, define client-side logic to concatenate values and insert the synthetic key into the items in your Cosmos containers.

## Use a partition key with a random suffix

Another possible strategy to distribute the workload more evenly is to append a random number at the end of the partition key value. When you distribute items in this way, you can perform parallel write operations across partitions.

Because you randomize the partition key, the write operations on the container on each day are spread evenly across multiple partitions. This method results in better parallelism and overall higher throughput.

Example:

You might choose a random number between 1 and 400 and concatenate it as a suffix. This method results in partition key values like `2018-08-09.1`, `2018-08-09.2`  ,

## Use a partition key with pre-calculated suffixes

Consider the previous example, where a container uses a date as the partition key. Now suppose that each item has a `Vehicle-Identification-Number (VIN)` attribute that we want to access. suppose that you often run queries to find items by the VIN, in addition to date.  Before your application writes the item to the container, it can calculate a hash suffix based on the VIN and append it to the partition key date. The calculation might generate a number between 1 and 400 that is evenly distributed.

With this strategy, the writes are evenly spread across the partition key values, and across the partitions. You can easily read a particular item and date, because you can calculate the partition key value for a specific Vehicle-Identification-Number.

The benefit of this method is that you can avoid creating a single hot partition key.

## Knowledge check

Which of the options below best describes the relationship between logical and physical partitions?

Physical partitions are collections of logical partitions

Which of the below correctly lists the two components of a partition key?

Key path, key value.