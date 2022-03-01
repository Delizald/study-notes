- Choose the appropriate queue mechanism for your solution.
- Explain how the messaging entities that form the core capabilities of Service Bus operate.
- Send and receive message from a Service Bus queue by using .NET.
- Identify the key components of Azure Queue Storage
- Create queues and manage messages in Azure Queue Storage by using .NET.

Service Bus queues are part of a broader Azure messaging infrastructure that supports queuing, publish/subscribe, and more advanced integration patterns. They're designed to integrate applications or application components that may span multiple communication protocols, data contracts, trust domains, or network environments.

Storage queues are part of the Azure Storage infrastructure. They allow you to store large numbers of messages. You access messages from anywhere in the world via authenticated calls using HTTP or HTTPS. A queue message can be up to 64 KB in size. A queue may contain millions of messages, up to the total capacity limit of a storage account. Queues are commonly used to create a backlog of work to process asynchronously.

# Choose a message queue solution

As a solution architect/developer, you should consider using Service Bus queues when:

- Your solution needs to receive messages without having to poll the queue. With Service Bus, you can achieve it by using a long-polling receive operation using the TCP-based protocols that Service Bus supports.
- Your solution requires the queue to provide a guaranteed first-in-first-out (FIFO) ordered delivery.
- Your solution needs to support automatic duplicate detection.
- You want your application to process messages as parallel long-running streams (messages are associated with a stream using the session ID property on the message). In this model, each node in the consuming application competes for streams, as opposed to messages. When a stream is given to a consuming node, the node can examine the state of the application stream state using transactions.
- Your solution requires transactional behavior and atomicity when sending or receiving multiple messages from a queue.
- Your application handles messages that can exceed 64 KB but won't likely approach the 256-KB limit.

As a solution architect/developer, you should consider using Storage queues when:

- Your application must store over 80 gigabytes of messages in a queue.
- Your application wants to track progress for processing a message in the queue. It's useful if the worker processing a message crashes. Another worker can then use that information to continue from where the prior worker left off.
- You require server side logs of all of the transactions executed against your queues.

# Explore Azure Service Bus
Microsoft Azure Service Bus is a fully managed enterprise integration message broker. Service Bus can decouple applications and services. Data is transferred between different applications and services using messages. A message is a container decorated with metadata, and contains data

Some common messaging scenarios are:
`Messaging`. Transfer business data, such as sales or purchase orders, journals, or inventory movements.
`Decouple applications`. Improve reliability and scalability of applications and services. Client and service don't have to be online at the same time.
`Topics and subscriptions`. Enable 1:n relationships between publishers and subscribers.
`Message sessions`. Implement workflows that require message ordering or message deferral.

## Service Bus tiers

Service Bus offers a standard and premium tier.

Some high-level differences are highlighted in the following table.

SERVICE BUS TIERS:

Premium	
- High throughput
- Predictable performance
- Fixed pricing
- Ability to scale workload up and down
- Message size up to 1 MB. Support for message payloads up to 100 MB currently exists in preview.

Standard:

- Variable throughput
- Variable latency
- Pay as you go variable pricing
- Message size up to 256 KB

## Advanced features
- `Message sessions`	To create a first-in, first-out (FIFO) guarantee in Service Bus, use sessions. Message sessions enable exclusive, ordered handling of unbounded sequences of related messages.
- `Autoforwarding`	The autoforwarding feature chains a queue or subscription to another queue or topic that is in the same namespace.
- `Dead-letter queue`	Service Bus supports a dead-letter queue (DLQ). A DLQ holds messages that can't be delivered to any receiver. Service Bus lets you remove messages from the DLQ and inspect them.
- `Scheduled delivery`	You can submit messages to a queue or topic for delayed processing. You can schedule a job to become available for processing by a system at a certain time.
- `Message deferral`	A queue or subscription client can defer retrieval of a message until a later time. The message remains in the queue or subscription, but it's set aside.
- `Batching`	Client-side batching enables a queue or topic client to delay sending a message for a certain period of time.
- `Transactions`	A transaction groups two or more operations together into an execution scope. Service Bus supports grouping operations against a single messaging entity within the scope of a single transaction. A message entity can be a queue, topic, or subscription.
- `Filtering and actions`	Subscribers can define which messages they want to receive from a topic. These messages are specified in the form of one or more named subscription rules.
- `Autodelete` on idle	Autodelete on idle enables you to specify an idle interval after which a queue is automatically deleted. The minimum duration is 5 minutes.
- `Duplicate` detection	An error could cause the client to have a doubt about the outcome of a send operation. Duplicate detection enables the sender to resend the same message, or for the queue or topic to discard any duplicate copies.
- `Security protocols`	Service Bus supports security protocols such as Shared Access Signatures (SAS), Role Based Access Control (RBAC) and Managed identities for Azure resources.
- `Geo-disaster recovery`	When Azure regions or datacenters experience downtime, Geo-disaster recovery enables data processing to continue operating in a different region or datacenter.
- `Security`	Service Bus supports standard AMQP 1.0 and HTTP/REST protocols.


## Compliance with standards and protocols
The primary wire protocol for Service Bus is Advanced Messaging Queueing Protocol (AMQP) 1.0, an open ISO/IEC standard.

Service Bus Premium is fully compliant with the Java/Jakarta EE Java Message Service (JMS) 2.0 API.

# Discover Service Bus queues, topics, and subscriptions

The messaging entities that form the core of the messaging capabilities in Service Bus are queues, topics and subscriptions, and rules/actions.

## Queues

Queues offer First In, First Out (FIFO) message delivery to one or more competing consumers. A related benefit is load-leveling, which enables producers and consumers to send and receive messages at different rates.

## Receive modes

You can specify two different modes in which Service Bus receives messages: `Receive and delete` or `Peek lock`.

## Receive and delete

when Service Bus receives the request from the consumer, it marks the message as being consumed and returns it to the consumer application. This mode is the simplest model. It works best for scenarios in which the application can tolerate not processing a message if a failure occurs.

## Peek lock

the receive operation becomes two-stage, which makes it possible to support applications that can't tolerate missing messages.

Finds the next message to be consumed, locks it to prevent other consumers from receiving it, and then, return the message to the application.

After the application finishes processing the message, it requests the Service Bus service to complete the second stage of the receive process. Then, the service marks the message as being consumed.

If the application is unable to process the message for some reason, it can request the Service Bus service to abandon the message. Service Bus unlocks the message and makes it available to be received again, either by the same consumer or by another competing consumer.  Secondly, there's a timeout associated with the lock

## Topics and subscriptions

topics and subscriptions provide a one-to-many form of communication in a publish and subscribe pattern. It's useful for scaling to large numbers of recipients. Each published message is made available to each subscription registered with the topic.  

Publishers send messages to a topic in the same way that they send messages to a queue. But, consumers don't receive messages directly from the topic. Instead, consumers receive messages from subscriptions of the topic.

A topic subscription resembles a virtual queue that receives copies of the messages that are sent to the topic.

## Rules and actions

# Explore Service Bus message payloads and serialization

Messages carry a payload and metadata. The metadata is in the form of key-value pair properties, and describes the payload, and gives handling instructions to Service Bus and applications. 

A Service Bus message consists of a binary payload section that Service Bus never handles in any form on the service-side, and two sets of properties. The broker properties are predefined by the system. These predefined properties either control message-level functionality inside the broker, or they map to common and standardized metadata items. The user properties are a collection of key-value pairs that can be defined and set by the application.

## Message routing and correlation

A subset of the broker properties described previously, specifically `To`, `ReplyTo`, `ReplyToSessionId`,`MessageId`, `CorrelationId`, and `SessionId`, are used to help applications route messages to particular destinations. To illustrate this, consider a few patterns:

Simple request/reply: A publisher sends a message into a queue and expects a reply from the message consumer. The address of that queue is expressed in the `ReplyTo` property of the outbound message.  When the consumer responds, it copies the `MessageId` of the handled message into the `CorrelationId` property of the reply message and delivers the message to the destination indicated by the ReplyTo property.

Multicast request/reply:  a publisher sends the message into a topic and multiple subscribers become eligible to consume the message. This pattern is used in discovery or roll-call scenarios and the respondent typically identifies itself with a user property or inside the payload

Multiplexing: This session feature enables multiplexing of streams of related messages through a single queue or subscription such that each session (or group) of related messages, identified by matching SessionId value

Multiplexed request/reply: This session feature enables multiplexed replies, allowing several publishers to share a reply queue. By setting ReplyToSessionId, the publisher can instruct the consumer(s) to copy that value into the SessionId property of the reply message.

## Payload serialization

The ContentType property enables applications to describe the payload, with the suggested format for the property values being a MIME content-type description according to IETF RFC2045; for example, application/json;charset=utf-8.

Unlike the Java or .NET Standard variants, the .NET Framework version of the Service Bus API supports creating BrokeredMessage instances by passing arbitrary .NET objects into the constructor.

When using the AMQP protocol, the object is serialized into an AMQP object. The receiver can retrieve those objects with the GetBody<T>() method, supplying the expected type. With AMQP, the objects are serialized into an AMQP graph of ArrayList and IDictionary<string,object> objects, and any AMQP client can decode them.
 

# Exercise: Send and receive message from a Service Bus queue by using .NET.

https://docs.microsoft.com/en-us/learn/modules/discover-azure-message-queue/6-send-receive-messages-service-bus

```
myLocation=<myLocation>
myNameSpaceName=az204svcbus$RANDOM
```
Create a resource group to hold the Azure resources you will be creating.
```
az servicebus namespace create \
    --resource-group az204-svcbus-rg \
    --name $myNameSpaceName \
    --location $myLocation
```
Retrieve the connection string for the Service Bus Namespace
* Open the Azure portal and navigate to the az204-svcbus-rg resource group.
* 
* Select the az204svcbus resource you just created.
* 
* Select Shared access policies in the Settings section, then select the RootManageSharedAccessKey policy.
* 
* Copy the Primary Connection String from the dialog box that opens up and save it to a file, or leave the portal open and copy the key when needed.
## Create console app to send messages to the queue

```
// the client that owns the connection and can be used to create senders and receivers
static ServiceBusClient client;

// the sender used to publish messages to the queue
static ServiceBusSender sender;

// number of messages to be sent to the queue
private const int numOfMessages = 3;
```

```
static async Task Main()
    {
        // Create the clients that we'll use for sending and processing messages.
        client = new ServiceBusClient(connectionString);
        sender = client.CreateSender(queueName);

        // create a batch 
        using ServiceBusMessageBatch messageBatch = await sender.CreateMessageBatchAsync();

        for (int i = 1; i <= 3; i++)
        {
            // try adding a message to the batch
            if (!messageBatch.TryAddMessage(new ServiceBusMessage($"Message {i}")))
            {
                // if an exception occurs
                throw new Exception($"Exception {i} has occurred.");
            }
        }

        try 
        {
            // Use the producer client to send the batch of messages to the Service Bus queue
            await sender.SendMessagesAsync(messageBatch);
            Console.WriteLine($"A batch of {numOfMessages} messages has been published to the queue.");
        }
        finally
        {
            // Calling DisposeAsync on client types is required to ensure that network
            // resources and other unmanaged objects are properly cleaned up.
            await sender.DisposeAsync();
            await client.DisposeAsync();
        }

        Console.WriteLine("Press any key to end the application");
        Console.ReadKey();
    }
```

# Explore Azure Queue Storage

Azure Queue Storage is a service for storing large numbers of messages. 

A queue message can be up to 64 KB in size.

The Queue service contains the following components:
URL format: Queues are addressable using the URL format https://<storage account>.queue.core.windows.net/<queue>. For example, the following URL addresses a queue in the diagram above https://myaccount.queue.core.windows.net/images-to-download

Storage account: All access to Azure Storage is done through a storage account.

Queue: A queue contains a set of messages. All messages must be in a queue. Note that the queue name must be all lowercase.

Message: A message, in any format, of up to 64 KB. For version 2017-07-29 or later, the maximum time-to-live can be any positive number, or -1 indicating that the message doesn't expire. If this parameter is omitted, the default time-to-live is seven days.

# Create and manage Azure Queue Storage queues and messages by using .NET

`Azure.Core` library for .NET: This package provides shared primitives, abstractions, and helpers for modern .NET Azure SDK client libraries.
`Azure.Storage.Common` client library for .NET: This package provides infrastructure shared by the other Azure Storage client libraries.
`Azure.Storage.Queues` client library for .NET: This package enables working with Azure Queue Storage for storing messages that may be accessed by a client.
`System.Configuration.ConfigurationManager` library for .NET: This package provides access to configuration files for client applications.

## Create the Queue service client
`QueueClient queueClient = new QueueClient(connectionString, queueName);`
## Create a queue
```
// Get the connection string from app settings
string connectionString = ConfigurationManager.AppSettings["StorageConnectionString"];

// Instantiate a QueueClient which will be used to create and manipulate the queue
QueueClient queueClient = new QueueClient(connectionString, queueName);

// Create the queue
queueClient.CreateIfNotExists();
```
## Insert a message into a queue
To insert a message into an existing queue, call the `SendMessage` method.
```
// Get the connection string from app settings
string connectionString = ConfigurationManager.AppSettings["StorageConnectionString"];

// Instantiate a QueueClient which will be used to create and manipulate the queue
QueueClient queueClient = new QueueClient(connectionString, queueName);

// Create the queue if it doesn't already exist
queueClient.CreateIfNotExists();

if (queueClient.Exists())
{
    // Send a message to the queue
    queueClient.SendMessage(message);
}
```
## Peek at the next message
You can peek at the messages in the queue without removing them from the queue by calling the `PeekMessages` method.
```
// Get the connection string from app settings
string connectionString = ConfigurationManager.AppSettings["StorageConnectionString"];

// Instantiate a QueueClient which will be used to manipulate the queue
QueueClient queueClient = new QueueClient(connectionString, queueName);

if (queueClient.Exists())
{ 
    // Peek at the next message
    PeekedMessage[] peekedMessage = queueClient.PeekMessages();
}
```
## Change the contents of a queued message
```
// Get the connection string from app settings
string connectionString = ConfigurationManager.AppSettings["StorageConnectionString"];

// Instantiate a QueueClient which will be used to manipulate the queue
QueueClient queueClient = new QueueClient(connectionString, queueName);

if (queueClient.Exists())
{
    // Get the message from the queue
    QueueMessage[] message = queueClient.ReceiveMessages();

    // Update the message contents
    queueClient.UpdateMessage(message[0].MessageId, 
            message[0].PopReceipt, 
            "Updated contents",
            TimeSpan.FromSeconds(60.0)  // Make it invisible for another 60 seconds
        );
}
```
## De-queue the next message
Dequeue a message from a queue in two steps. When you call `ReceiveMessages`, you get the next message in a queue. A message returned from `ReceiveMessages` becomes invisible to any other code reading messages from this queue. By default, this message stays invisible for 30 seconds. To finish removing the message from the queue, you must also call `DeleteMessage`. 

```
// Get the connection string from app settings
string connectionString = ConfigurationManager.AppSettings["StorageConnectionString"];

// Instantiate a QueueClient which will be used to manipulate the queue
QueueClient queueClient = new QueueClient(connectionString, queueName);

if (queueClient.Exists())
{
    // Get the next message
    QueueMessage[] retrievedMessage = queueClient.ReceiveMessages();

    // Process (i.e. print) the message in less than 30 seconds
    Console.WriteLine($"Dequeued message: '{retrievedMessage[0].MessageText}'");

    // Delete the message
    queueClient.DeleteMessage(retrievedMessage[0].MessageId, retrievedMessage[0].PopReceipt);
}
```

## Get the queue length
The `GetProperties` method returns queue properties including the message count. The `ApproximateMessagesCount` property contains the approximate number of messages in the queue. 
```
/// Instantiate a QueueClient which will be used to manipulate the queue
QueueClient queueClient = new QueueClient(connectionString, queueName);

if (queueClient.Exists())
{
    QueueProperties properties = queueClient.GetProperties();

    // Retrieve the cached approximate message count.
    int cachedMessagesCount = properties.ApproximateMessagesCount;

    // Display number of messages.
    Console.WriteLine($"Number of messages in queue: {cachedMessagesCount}");
}
```

## Delete a queue
To delete a queue and all the messages contained in it, call the `Delete` method on the queue object.
```
/// Get the connection string from app settings
string connectionString = ConfigurationManager.AppSettings["StorageConnectionString"];

// Instantiate a QueueClient which will be used to manipulate the queue
QueueClient queueClient = new QueueClient(connectionString, queueName);

if (queueClient.Exists())
{
    // Delete the queue
    queueClient.Delete();
}
```

# Knowledge check
Which of the following advanced features of Azure Service Bus creates a first-in, first-out (FIFO) guarantee? 
Message sessions

In Azure Service Bus messages are durably stored which enables a load-leveling benefit. Which of the below correctly describes the load-leveling benefit relative to a consuming application's performance? Performance needs to handle average load