- Describe how Event Grid operates and how it connects to services and event handlers.
- Explain how Event Grid delivers events and how it handles errors.
- Implement authentication and authorization.
- Route custom events to web endpoint by using Azure CLI.

# Explore Azure Event Grid

Azure Event Grid is an eventing backplane that enables event-driven, reactive programming. It uses the publish-subscribe model. Publishers emit events, but have no expectation about how the events are handled. Subscribers decide on which events they want to handle.

 First, select the Azure resource you would like to subscribe to, and then give the event handler or WebHook endpoint to send the event to. 

 You can use filters to route specific events to different endpoints, multicast to multiple endpoints, and make sure your events are reliably delivered.

 ## Concepts in Azure Event Grid

There are five concepts in Azure Event Grid you need to understand to help you get started:

- Events - What happened.
- Event sources - Where the event took place.
- Topics - The endpoint where publishers send events.
- Event subscriptions - The endpoint or built-in mechanism to route events, sometimes to more than one handler. Subscriptions are also used by handlers to intelligently filter incoming events.
- Event handlers - The app or service reacting to the event.

## Events
An event is the smallest amount of information that fully describes something that happened in the system.

An event of size up to 64 KB is covered by General Availability (GA) Service Level Agreement (SLA). The support for an event of size up to 1 MB is currently in preview. Events over 64 KB are charged in 64-KB increments.

## Event sources

An event source is where the event happens. Each event source is related to one or more event types. . Your application is the event source for custom events that you define. Event sources are responsible for sending events to Event Grid.

## Topics

A topic is used for a collection of related events. To respond to certain types of events, subscribers decide which topics to subscribe to. 

System topics are built-in topics provided by Azure services. You don't see system topics in your Azure subscription because the publisher owns the topics, but you can subscribe to them.  As long as you have access to the resource, you can subscribe to its events.

Custom topics are application and third-party topics. When you create or are assigned access to a custom topic, you see that custom topic in your subscription.

## Event subscriptions

A subscription tells Event Grid which events on a topic you're interested in receiving.

## Event handlers

an event handler is the place where the event is sent.

# Discover event schemas
Events consist of a set of four required string properties.
Event sources send events to Azure Event Grid in an array, which can have several event objects.
When posting events to an event grid topic, the array can have a total size of up to 1 MB. Each event in the array is limited to 1 MB.
 If an event or the array is greater than the size limits, you receive the response 413 Payload Too Large.

You can find the JSON schema for the Event Grid event and each Azure publisher's data payload in (https://github.com/Azure/azure-rest-api-specs/tree/master/specification/eventgrid/data-plane)

## Event schema
```
[
  {
    "topic": string,
    "subject": string,
    "id": string,
    "eventType": string,
    "eventTime": string,
    "data":{
      object-unique-to-each-publisher
    },
    "dataVersion": string,
    "metadataVersion": string
  }
]
```

## Event properties
Property	Type	Required	Description
topic	string	No. If not included, Event Grid will stamp onto the event. If included it must match the event grid topic Azure Resource Manager ID exactly.	Full resource path to the event source. This field isn't writeable. Event Grid provides this value.

subject	string	Yes	Publisher-defined path to the event subject.

eventType	string	Yes	One of the registered event types for this event source.

eventTime	string	Yes	The time the event is generated based on the provider's UTC time.

id	string	Yes	Unique identifier for the event.

data	object	No	Event data specific to the resource provider.

dataVersion	string	No. If not included it will be stamped with an empty value.	The schema version of the data object. The publisher defines the schema version.

metadataVersion	string	No. If not included, Event Grid will stamp onto the event. If included, must match the Event Grid Schema metadataVersion exactly (currently, only 1).	The schema version of the event metadata. Event Grid defines the schema of the top-level properties. Event Grid provides this value.

## CloudEvents v1.0 schema

Azure Event Grid natively supports events in the JSON implementation of CloudEvents v1.0 and HTTP protocol binding.  CloudEvents is an open specification for describing event data.

CloudEvents simplifies interoperability by providing a common event schema for publishing, and consuming cloud based events.

Here is an example of an Azure Blob Storage event in CloudEvents format:
{
    "specversion": "1.0",
    "type": "Microsoft.Storage.BlobCreated",  
    "source": "/subscriptions/{subscription-id}/resourceGroups/{resource-group}/providers/Microsoft.Storage/storageAccounts/{storage-account}",
    "id": "9aeb0fdf-c01e-0131-0922-9eb54906e209",
    "time": "2019-11-18T15:13:39.4589254Z",
    "subject": "blobServices/default/containers/{storage-container}/blobs/{new-file}",
    "dataschema": "#",
    "data": {
        "api": "PutBlockList",
        "clientRequestId": "4c5dd7fb-2c48-4a27-bb30-5361b5de920a",
        "requestId": "9aeb0fdf-c01e-0131-0922-9eb549000000",
        "eTag": "0x8D76C39E4407333",
        "contentType": "image/png",
        "contentLength": 30699,
        "blobType": "BlockBlob",
        "url": "https://gridtesting.blob.core.windows.net/testcontainer/{new-file}",
        "sequencer": "000000000000000000000000000099240000000000c41c18",
        "storageDiagnostics": {
            "batchId": "681fe319-3006-00a8-0022-9e7cde000000"
        }
    }
}

description for fields: https://github.com/cloudevents/spec/blob/v1.0/spec.md#required-attributes

The headers values for events delivered in the CloudEvents schema and the Event Grid schema are the same except for `content-type`.

For CloudEvents schema, that header value is "`content-type`":"`application/cloudevents+json; charset=utf-8`".

For Event Grid schema, that header value is `"content-type`":"`application/json; charset=utf-8`".

You can use Event Grid for both input and output of events in CloudEvents schema.

You can use CloudEvents for system events, like Blob Storage events and IoT Hub events, and custom events.

# Explore event delivery durability

Event Grid retries delivery based on a fixed retry schedule and retry policy. By default, Event Grid delivers one event at a time to the subscriber, and the payload is an array with a single event.

## Retry schedule

If the error returned by the subscribed endpoint is a configuration-related error that can't be fixed with retries (for example, if the endpoint is deleted), EventGrid will either perform dead-lettering on the event or drop the event if dead-letter isn't configured.

The following table describes the types of endpoints and errors for which retry doesn't happen:
- Azure Resources	400 Bad Request: 413 Request Entity Too Large, 403 Forbidden
- Webhook	400 Bad Request, 413 Request Entity Too Large, 403 Forbidden, 404 Not Found, 401 Unauthorized

Event Grid waits `30 seconds` for a response after delivering a message. After 30 seconds, if the endpoint hasn’t responded, the message is queued for retry. Event Grid uses an exponential backoff retry policy for event delivery. 

If the endpoint responds within 3 minutes, Event Grid will attempt to remove the event from the retry queue on a best effort basis but duplicates may still be received

## Retry policy

- Maximum number of attempts - The value must be an integer between 1 and 30. The default value is 30.
- Event time-to-live (TTL) - The value must be an integer between 1 and 1440. The default value is 1440 minutes

```
az eventgrid event-subscription create \
  -g gridResourceGroup \
  --topic-name <topic_name> \
  --name <event_subscription_name> \
  --endpoint <endpoint_URL> \
  --max-delivery-attempts 18 
```

## Output batching

Batched delivery has two settings:

Max events per batch - Maximum number of events Event Grid will deliver per batch. This number will never be exceeded, however fewer events may be delivered if no other events are available at the time of publish. Event Grid doesn't delay events to create a batch if fewer events are available. Must be between 1 and 5,000.

Preferred batch size in kilobytes - Target ceiling for batch size in kilobytes. Similar to max events, the batch size may be smaller if more events aren't available at the time of publish. It's possible that a batch is larger than the preferred batch size if a single event is larger than the preferred size. For example, if the preferred size is 4 KB and a 10-KB event is pushed to Event Grid, the 10-KB event will still be delivered in its own batch rather than being dropped.

## Delayed delivery

For example, if the first 10 events published to an endpoint fail, Event Grid will assume that the endpoint is experiencing issues and will delay all subsequent retries, and new deliveries, for some time - in some cases up to several hours.

The functional purpose of delayed delivery is to protect unhealthy endpoints and the Event Grid system. Without back-off and delay of delivery to unhealthy endpoints, Event Grid's retry policy and volume capabilities can easily overwhelm a system.

## Dead-letter events

When Event Grid can't deliver an event within a certain time period or after trying to deliver the event a certain number of times, it can send the undelivered event to a storage account. This process is known as `dead-lettering`.

Event Grid dead-letters an event when one of the following conditions is met.
- Event isn't delivered within the time-to-live period.
- The number of tries to deliver the event exceeds the limit.

By default, Event Grid doesn't turn on dead-lettering. To enable it, you must specify a storage account to hold undelivered events when creating the event subscription. You pull events from this storage account to resolve deliveries.

If Event Grid receives a 400 (Bad Request) or 413 (Request Entity Too Large) response code, it immediately schedules the event for dead-lettering.

There is a five-minute delay between the last attempt to deliver an event and when it is delivered to the dead-letter location. This delay is intended to reduce the number of Blob storage operation

## Custom delivery properties

You can set custom headers on the events that are delivered to the following destinations:

- Webhooks
- Azure Service Bus topics and queues
- Azure Event Hubs
- Relay Hybrid Connections

# Control access to events

Event Grid uses Azure role-based access control (Azure RBAC).

**EventGrid doesn't support Azure RBAC for publishing events to Event Grid topics or domains. Use a Shared Access Signature (SAS) key or token to authenticate clients that publish events.**

## Built-in roles

- Event Grid Subscription Reader:	Lets you read Event Grid event subscriptions.
- Event Grid Subscription Contributor:	Lets you manage Event Grid event subscription operations.
- Event Grid Contributor:	Lets you create and manage Event Grid resources.
- Event Grid Data Sender:	Lets you send events to Event Grid topics.

## Permissions for event subscriptions

You must have the `Microsoft.EventGrid/EventSubscriptions/Write` permission on the resource that is the event source.
You need this permission because you're writing a new subscription at the scope of the resource. The required resource differs based on whether you're subscribing to a system topic or custom topic. Both types are described in this section.

System topics:	Need permission to write a new event subscription at the scope of the resource publishing the event. The format of the resource is: `/subscriptions/{subscription-id}/resourceGroups/{resource-group-name}/providers/{resource-provider}/{resource-type}/{resource-name}`

Custom topics: Need permission to write a new event subscription at the scope of the event grid topic. The format of the resource is: `/subscriptions/{subscription-id}/resourceGroups/{resource-group-name}/providers/Microsoft.EventGrid/topics/{topic-name}`

# Receive events by using webhooks

Event Grid requires you to prove ownership of your Webhook endpoint before it starts delivering events to that endpoint. This requirement prevents a malicious user from flooding your endpoint with events.

When you use any of the three Azure services listed below, the Azure infrastructure automatically handles this validation:

- Azure Logic Apps with Event Grid Connector
- Azure Automation via webhook
- Azure Functions with Event Grid Trigger

## Endpoint validation with Event Grid events

Event Grid supports two ways of validating the subscription.

`Synchronous handshake`: At the time of event subscription creation, Event Grid sends a subscription validation event to your endpoint. The schema of this event is similar to any other Event Grid event. The data portion of this event includes a validationCode property. Your application verifies that the validation request is for an expected event subscription, and returns the validation code in the response synchronously. This handshake mechanism is supported in all Event Grid versions.

`Asynchronous handshake`: In certain cases, you can't return the ValidationCode in response synchronously. For example, if you use a third-party service (like Zapier or IFTTT), you can't programmatically respond with the validation code.

Starting with version 2018-05-01-preview, Event Grid supports a manual validation handshake. If you're creating an event

If you're creating an event subscription with an SDK or tool that uses API version 2018-05-01-preview or later, Event Grid sends a `validationUrl` property in the data portion of the subscription validation event. To complete the handshake, find that URL in the event data and do a GET request to it. 

The provided URL is valid for 5 minutes. During that time, the provisioning state of the event subscription is `AwaitingManualAction`. If you don't complete the manual validation within 5 minutes, the provisioning state is set to `Failed`. You'll have to create the event subscription again before starting the manual validation

**Using self-signed certificates for validation isn't supported. Use a signed certificate from a commercial certificate authority (CA) instead.**

# Filter events

three options for filtering:

- Event types
- Subject begins with or ends with
- Advanced fields and operators

## Event type filtering
filter by the `Microsoft.Resources.ResourceWriteSuccess` event type. Provide an array with the event types, or specify All to get all event types for the event source.

The JSON syntax for filtering by event type is:

```
"filter": {
  "includedEventTypes": [
    "Microsoft.Resources.ResourceWriteFailure",
    "Microsoft.Resources.ResourceWriteSuccess"
  ]
}
```

## Subject filtering

For simple filtering by subject, specify a starting or ending value for the subject.
The JSON syntax for filtering by subject is:
```
"filter": {
  "subjectBeginsWith": "/blobServices/default/containers/mycontainer/log",
  "subjectEndsWith": ".jpg"
}
```

## Advanced filtering

In advanced filtering, you specify the:

- operator type - The type of comparison.
- key - The field in the event data that you're using for filtering. It can be a number, boolean, or string.
- value or values - The value or values to compare to the key.

```
"filter": {
  "advancedFilters": [
    {
      "operatorType": "NumberGreaterThanOrEquals",
      "key": "Data.Key1",
      "value": 5
    },
    {
      "operatorType": "StringContains",
      "key": "Subject",
      "values": ["container1", "container2"]
    }
  ]
}
```

# Exercise: Route custom events to web endpoint by using Azure CLI
## Create a resource group
Launch the Cloud Shell: https://shell.azure.com
```
let rNum=$RANDOM*$RANDOM
myLocation=<myLocation>
myTopicName="az204-egtopic-${rNum}"
mySiteName="az204-egsite-${rNum}"
mySiteURL="https://${mySiteName}.azurewebsites.net"
```
```
az group create --name az204-evgrid-rg --location $myLocation
```
## Enable an Event Grid resource provider
Register the Event Grid resource provider by using the `az provider register` command.
`az provider register --namespace Microsoft.EventGrid`
To check the status run the command below.
`az provider show --namespace Microsoft.EventGrid --query "registrationState"`
## Create a custom topic
Create a custom topic by using the `az eventgrid topic create` command. The name must be unique because it is part of the DNS entry.
```
az eventgrid topic create --name $myTopicName \
    --location $myLocation \
    --resource-group az204-evgrid-rg
```
## Create a message endpoint
Create a message endpoint. The deployment may take a few minutes to complete.
```
az deployment group create \
    --resource-group az204-evgrid-rg \
    --template-uri "https://raw.githubusercontent.com/Azure-Samples/azure-event-grid-viewer/main/azuredeploy.json" \
    --parameters siteName=$mySiteName hostingPlanName=viewerhost

echo "Your web app URL: ${mySiteURL}"
```
## Subscribe to a custom topic
Subscribe to a custom topic by using the `az eventgrid event-subscription create` command.
```
endpoint="${mySiteURL}/api/updates"
subId=$(az account show --subscription "" | jq -r '.id')

az eventgrid event-subscription create \
    --source-resource-id "/subscriptions/$subId/resourceGroups/az204-evgrid-rg/providers/Microsoft.EventGrid/topics/$myTopicName" \
    --name az204ViewerSub \
    --endpoint $endpoints
```
## Send an event to your custom topic
Retrieve URL and key for the custom topic.
```
topicEndpoint=$(az eventgrid topic show --name $myTopicName -g az204-evgrid-rg --query "endpoint" --output tsv)
key=$(az eventgrid topic key list --name $myTopicName -g az204-evgrid-rg --query "key1" --output tsv)
```
Create event data to send. Typically, an application or Azure service would send the event data, we're creating data for the purposes of the exercise.
```
event='[ {"id": "'"$RANDOM"'", "eventType": "recordInserted", "subject": "myapp/vehicles/motorcycles", "eventTime": "'`date +%Y-%m-%dT%H:%M:%S%z`'", "data":{ "make": "Contoso", "model": "Monster"},"dataVersion": "1.0"} ]'
```
Use curl to send the event to the topic.
```
curl -X POST -H "aeg-sas-key: $key" -d "$event" $topicEndpoint
```

# Knowledge check
Which of the following event schema properties requires a value? Subject
Which of the following Event Grid built-in roles is appropriate for managing Event Grid resources? Event Grid Contributor