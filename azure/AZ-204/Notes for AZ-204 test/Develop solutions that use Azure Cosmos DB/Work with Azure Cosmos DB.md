After completing this module, you'll be able to:

- Identify classes and methods used to create resources
- Create resources by using the Azure Cosmos DB .NET v3 SDK
- Write stored procedures, triggers, and user-defined functions by using JavaScript

# Explore Microsoft .NET SDK v3 for Azure Cosmos DB
Because Azure Cosmos DB supports multiple API models, version 3 of the .NET SDK uses the generic terms "container" and "item".

A container can be a collection, graph, or table. An item can be a document, edge/vertex, or row, and is the content inside a container.

## CosmosClient
`CosmosClient` is thread-safe. Its recommended to maintain a single instance of CosmosClient per lifetime of the application  
`CosmosClient client = new CosmosClient(endpoint, key);`

## Database examples
## Create a database
The `CosmosClient.CreateDatabaseIfNotExistsAsync` checks if a database exists, and if it doesn't, creates it.
```
// An object containing relevant information about the response
DatabaseResponse databaseResponse = await client.CreateDatabaseIfNotExistsAsync(databaseId, 10000);
```
## Read a database by ID
`DatabaseResponse readResponse = await database.ReadAsync();`
## Delete a database
`await database.DeleteAsync();`

## Container examples
The `Database.CreateContainerIfNotExistsAsync` method checks if a container exists, and if it doesn't, it creates it.
```
// Set throughput to the minimum value of 400 RU/s
ContainerResponse simpleContainer = await database.CreateContainerIfNotExistsAsync(
    id: containerId,
    partitionKeyPath: partitionKey,
    throughput: 400);
```
## Get a container by ID
```
Container container = database.GetContainer(containerId);
ContainerProperties containerProperties = await container.ReadContainerAsync();
```
## Delete a container
`await database.GetContainer(containerId).DeleteContainerAsync();`

## Item examples
## Create an item
Use the `Container.CreateItemAsync` method to create an item.
`ItemResponse<SalesOrder> response = await container.CreateItemAsync(salesOrder, new PartitionKey(salesOrder.AccountNumber));`
## Read an item
```
string id = "[id]";
string accountNumber = "[partition-key]";
ItemResponse<SalesOrder> response = await container.ReadItemAsync(id, new PartitionKey(accountNumber));
```
## Query an item
```
QueryDefinition query = new QueryDefinition(
    "select * from sales s where s.AccountNumber = @AccountInput ")
    .WithParameter("@AccountInput", "Account1");

FeedIterator<SalesOrder> resultSet = container.GetItemQueryIterator<SalesOrder>(
    query,
    requestOptions: new QueryRequestOptions()
    {
        PartitionKey = new PartitionKey("Account1"),
        MaxItemCount = 1
    });
```

More examples: https://docs.microsoft.com/en-us/azure/cosmos-db/sql-api-dotnet-v3sdk-samples

# Exercise: Create resources by using the Microsoft .NET SDK v3
https://docs.microsoft.com/en-us/learn/modules/work-with-cosmos-db/3-exercise-work-cosmos-db-dotnet

# Create stored procedures

More info: https://docs.microsoft.com/en-us/azure/cosmos-db/sql/how-to-use-stored-procedures-triggers-udfs

## Writing stored procedures
JS:
```
var helloWorldStoredProc = {
    id: "helloWorld",
    serverScript: function () {
        var context = getContext();
        var response = context.getResponse();

        response.setBody("Hello, World");
    }
}
```
## Create an item using stored procedure
JS:
```
function createSampleDocument(documentToCreate) {
    var context = getContext();
    var collection = context.getCollection();
    var accepted = collection.createDocument(
        collection.getSelfLink(),
        documentToCreate,
        function (error, documentCreated) {                 
            context.getResponse().setBody(documentCreated.id)
        }
    );
    if (!accepted) return;
}
```
## Arrays as input parameters for stored procedures
input parameters are always sent as a string to the stored procedure. Even if you pass an array of strings as an input.
JS:
```
function sample(arr) {
    if (typeof arr === "string") arr = JSON.parse(arr);

    arr.forEach(function(a) {
        // do something here
        console.log(a);
    });
}
```
## Bounded execution
All Azure Cosmos DB operations must complete within a limited amount of time. Stored procedures have a limited amount of time to run on the server.
## Transactions within stored procedures
You can implement transactions on items within a container by using a stored procedure. JavaScript functions can implement a continuation-based model to batch or resume execution. 

# Create triggers and user-defined functions
Azure Cosmos DB supports pre-triggers and post-triggers.

Pre-triggers are executed before modifying a database item 

post-triggers are executed after modifying a database item.

## Pre-triggers
JS:
```
function validateToDoItemTimestamp() {
    var context = getContext();
    var request = context.getRequest();

    // item to be created in the current operation
    var itemToCreate = request.getBody();

    // validate properties
    if (!("timestamp" in itemToCreate)) {
        var ts = new Date();
        itemToCreate["timestamp"] = ts.getTime();
    }

    // update the item that will be created
    request.setBody(itemToCreate);
}
```
## Post-triggers
```
function updateMetadata() {
var context = getContext();
var container = context.getCollection();
var response = context.getResponse();

// item that was created
var createdItem = response.getBody();

// query for metadata document
var filterQuery = 'SELECT * FROM root r WHERE r.id = "_metadata"';
var accept = container.queryDocuments(container.getSelfLink(), filterQuery,
    updateMetadataCallback);
if(!accept) throw "Unable to update metadata, abort";

function updateMetadataCallback(err, items, responseOptions) {
    if(err) throw new Error("Error" + err.message);
        if(items.length != 1) throw 'Unable to find metadata document';

        var metadataItem = items[0];

        // update metadata
        metadataItem.createdItems += 1;
        metadataItem.createdNames += " " + createdItem.id;
        var accept = container.replaceDocument(metadataItem._self,
            metadataItem, function(err, itemReplaced) {
                    if(err) throw "Unable to update metadata, abort";
            });
        if(!accept) throw "Unable to update metadata, abort";
        return;
    }
}
```
# Knowledge check
When defining a stored procedure in the Azure portal input parameters are always sent as what type to the stored procedure? String

Which of the following would one use to validate properties of an item being created? Pre-trigger