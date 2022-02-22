- Explain the benefits of using Microsoft Graph.
- Perform operations on Microsoft Graph by using REST and SDKs.
- Apply best practices to help your applications get the most out of Microsoft Graph.

# Discover Microsoft Graph
Microsoft Graph is the gateway to data and intelligence in Microsoft 365. It provides a unified programmability model that you can use to access the tremendous amount of data in Microsoft 365, Windows 10, and Enterprise Mobility + Security.

In the Microsoft 365 platform, three main components facilitate the access and flow of data:

The Microsoft Graph API offers a single endpoint, https://graph.microsoft.com. You can use REST APIs or SDKs to access the endpoint. Microsoft Graph also includes a powerful set of services that manage user and device identity, access, compliance, security, and help protect organizations from data leakage or loss.

Microsoft Graph connectors work in the incoming direction, delivering data external to the Microsoft cloud into Microsoft Graph services and applications, to enhance Microsoft 365 experiences such as Microsoft Search. Connectors exist for many commonly used data sources such as Box, Google Drive, Jira, and Salesforce.

Microsoft Graph Data Connect provides a set of tools to streamline secure and scalable delivery of Microsoft Graph data to popular Azure data stores. The cached data serves as data sources for Azure development tools that you can use to build intelligent applications.

# Query Microsoft Graph by using REST

The Microsoft Graph API defines most of its resources, methods, and enumerations in the OData namespace,

`microsoft.graph`, in the Microsoft Graph metadata. A small number of API sets are defined in their sub-namespaces, such as the call records API which defines resources like callRecord in microsoft.graph.callRecords.

Unless explicitly specified in the corresponding topic, assume types, methods, and enumerations are part of the microsoft.graph namespace.

## Call a REST API method
`{HTTP method} https://graph.microsoft.com/{version}/{resource}?{query-parameters}`

## Version
- `v1.0` includes generally available APIs. Use the v1.0 version for all production apps.
- `beta` includes APIs that are currently in preview. Because we might introduce breaking changes to our beta APIs, we recommend that you use the beta version only to test apps that are in development; do not use beta APIs in your production apps.

## Resource

resource can be an entity or complex type, commonly defined with properties. Entities differ from complex types by always including an id property.

Your URL will include the resource you are interacting with in the request, such as me, user, group, drive, and site

Often, top-level resources also include relationships, which you can use to access additional resources

You can also interact with resources using methods; for example, to send an email

## Query parameters

Query parameters can be OData system query options, or other strings that a method accepts to customize its response.

# Query Microsoft Graph by using SDKs

## Install the Microsoft Graph .NET SDK

- Microsoft.Graph - Contains the models and request builders for accessing the v1.0 endpoint with the fluent API. Microsoft.Graph has a dependency on Microsoft.Graph.Core.
- Microsoft.Graph.Beta - Contains the models and request builders for accessing the beta endpoint with the fluent API. Microsoft.Graph.Beta has a dependency on Microsoft.Graph.Core.
- Microsoft.Graph.Core - The core library for making calls to Microsoft Graph.
- Microsoft.Graph.Auth - Provides an authentication scenario-based wrapper of the Microsoft Authentication Library (MSAL) for use with the Microsoft Graph SDK. Microsoft.Graph.Auth has a dependency 

## Create a Microsoft Graph client

For details about which provider and options are appropriate for your scenario, see Choose an Authentication Provider.
(https://docs.microsoft.com/en-us/graph/sdks/choose-authentication-providers)

```
// Build a client application.
IPublicClientApplication publicClientApplication = PublicClientApplicationBuilder
            .Create("INSERT-CLIENT-APP-ID")
            .Build();
// Create an authentication provider by passing in a client application and graph scopes.
DeviceCodeProvider authProvider = new DeviceCodeProvider(publicClientApplication, graphScopes);
// Create a new instance of GraphServiceClient with the authentication provider.
GraphServiceClient graphClient = new GraphServiceClient(authProvider);
```

## Read information from Microsoft Graph

you first need to create a request object and then run the GET method on the request.

```
// GET https://graph.microsoft.com/v1.0/me

var user = await graphClient.Me
    .Request()
    .GetAsync();
```

## Retrieve a list of entities

```
// GET https://graph.microsoft.com/v1.0/me/messages?$select=subject,sender&$filter=<some condition>&orderBy=receivedDateTime

var messages = await graphClient.Me.Messages
    .Request()
    .Select(m => new {
        m.Subject,
        m.Sender
    })
    .Filter("<filter condition>")
    .OrderBy("receivedDateTime")
    .GetAsync();
```

The $filter query parameter can be used to reduce the result set to only those rows that match the provided

The $orderBy query parameter will request that the server provide the list of entities sorted by the specified properties.

## Delete an entity

Delete requests are constructed in the same way as requests to retrieve an entity, but use a DELETE request instead of a GET.

```
// DELETE https://graph.microsoft.com/v1.0/me/messages/{message-id}

string messageId = "AQMkAGUy...";
var message = await graphClient.Me.Messages[messageId]
    .Request()
    .DeleteAsync();
```

## Create a new entity
new items can be added to collections with an Add method. For template-based SDKs, the request object exposes a post method.

```
// POST https://graph.microsoft.com/v1.0/me/calendars

var calendar = new Calendar
{
    Name = "Volunteer"
};

var newCalendar = await graphClient.Me.Calendars
    .Request()
    .AddAsync(calendar);
```

(https://docs.microsoft.com/en-us/graph/api/overview)