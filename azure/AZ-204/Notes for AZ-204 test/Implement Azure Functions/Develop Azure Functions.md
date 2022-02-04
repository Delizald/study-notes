## Learning objectives
After completing this module you will be able to:

- Explain the key components of a function and how they are structured
- Create triggers and bindings to control when a function runs and where the output is directed
- Connect a function to services in Azure
- Create a function by using Visual Studio Code and the Azure Functions Core Tools

## Explore Azure Functions development

A function contains two important pieces: your code, which can be written in a variety of languages, and some config, the function.json file.

The function.json file defines the function's trigger, bindings, and other configuration settings. Every function has one and only one trigger. 

The following is an example function.json file:

```
{
    "disabled":false,
    "bindings":[
        // ... bindings here
        {
            "type": "bindingType",
            "direction": "in",
            "name": "myParamName",
            // ... more depending on binding
        }
    ]
}
``` 

The `bindings` property is where you configure both triggers and bindings. Every binding requires the following settings:

- `type`: Name of binding. For example, queueTrigger.
- `direction` Indicates whether the binding is for receiving data into the function or sending data from the function. For example, `in` or `out`.
- `name` The name that is used for the bound data in the function. For example, `myQueue`.

## Function app

it is the unit of deployment and management for your functions.  All of the functions in a function app share the same pricing plan, deployment method, and runtime version. Think of a function app as a way to organize and collectively manage your functions.


**In Functions 2.x all functions in a function app must be authored in the same language. In previous versions of the Azure Functions runtime, this wasn't required.**

## Folder structure

The host.json file contains runtime-specific configurations and is in the root folder of the function app. A bin folder contains packages and other library files that the function app requires. Specific folder structures required by the function app depend on language:

- C# compiled (.csproj) (https://docs.microsoft.com/en-us/azure/azure-functions/functions-dotnet-class-library#functions-class-library-project)
- C# script (.csx) (https://docs.microsoft.com/en-us/azure/azure-functions/functions-reference-csharp#folder-structure)
- F# script (https://docs.microsoft.com/en-us/azure/azure-functions/functions-reference-fsharp#folder-structure)
- Java (https://docs.microsoft.com/en-us/azure/azure-functions/functions-reference-java#folder-structure)
- JavaScript (https://docs.microsoft.com/en-us/azure/azure-functions/functions-reference-node#folder-structure)
- Python (https://docs.microsoft.com/en-us/azure/azure-functions/functions-reference-python#folder-structure)

## Local development environments
The way in which you develop functions on your local computer depends on your language and tooling preferences. See Code and test Azure Functions locally for more information. (https://docs.microsoft.com/en-us/azure/azure-functions/functions-develop-local)

Do not mix local development with portal development in the same function app. When you create and publish functions from a local project, you should not try to maintain or modify project code in the portal.

## Create triggers and bindings

Triggers are what cause a function to run. A trigger defines how a function is invoked and a function must have exactly one trigger.

Binding to a function is a way of declaratively connecting another resource to the function; bindings may be connected as input bindings, output bindings, or both. Data from bindings is provided to the function as parameters.

Triggers and bindings let you avoid hardcoding access to other services.

## Trigger and binding definitions
- C# class library: 	decorating methods and parameters with C# attributes
- Java: decorating methods and parameters with Java annotations
- JavaScript/PowerShell/Python/TypeScript: updating function.json schema.

For languages that rely on function.json, the portal provides a UI for adding bindings in the Integration tab.(you can also edit the file directly in the portal in the Code + test tab)

. Since .NET class library functions and Java functions don't rely on function.json for binding definitions, they can't be created and edited in the portal. C# portal editing is based on C# script, which uses function.json instead of attributes.

For languages that are dynamically typed such as JavaScript, use the dataType property in the function.json file. For example, to read the content of an HTTP request in binary format, set dataType to binary:

```
{
    "dataType": "binary",
    "type": "httpTrigger",
    "name": "req",
    "direction": "in"
}
```
Other options for dataType are **stream** and **string**.


## Binding direction

All triggers and bindings have a direction property in the function.json file.

- For triggers, the direction is always **in**
- Input and output bindings use **in** and **out**
- Some bindings support a special direction **inout**. If you use inout, only the
Advanced editor is available via the Integrate tab in the portal.

## Azure Functions trigger and binding example
Suppose you want to write a new row to Azure Table storage whenever a new message appears in Azure Queue storage.

Here's a function.json file for this scenario.

```
{
  "bindings": [
    {
      "type": "queueTrigger",
      "direction": "in",
      "name": "order",
      "queueName": "myqueue-items",
      "connection": "MY_STORAGE_ACCT_APP_SETTING"
    },
    {
      "type": "table",
      "direction": "out",
      "name": "$return",
      "tableName": "outTable",
      "connection": "MY_TABLE_STORAGE_ACCT_APP_SETTING"
    }
  ]
}
```

- The first element in the array is the `Queue storage trigger`:
-  `type` and `direction` identify the trigger
- `name` identifies the function parameter that receives the queue message content.
- `queueName` is the name of the queue to monitor
- `connection` is the connection string .
- The second element in the bindings array is the `Azure Table Storage output binding`.
- `type` and `direction` identify the binding
- `name` specifies how the function provides the new table row, in this case by using the function return value. 
- `tableName` is the table name, 
- `connection` is the connection string.

To view and edit the contents of function.json in the Azure portal, click the **Advanced editor** option on the Integrate tab of your function.