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
C# class library 	decorating methods and parameters with C# attributes
Java decorating methods and parameters with Java annotations
JavaScript/PowerShell/Python/TypeScript updating function.json schema