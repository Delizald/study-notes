# Learning objectives
After completing this module, you will be able to:

- Explain functional differences between Azure Functions, Azure Logic Apps, and WebJobs
- Describe Azure Functions hosting plan options
- Describe how Azure Functions scale to meet business needs

# Discover Azure Functions
Consider Functions for tasks like image or order processing, file maintenance, or for any tasks that you want to run on a schedule. Functions provides templates to get you started with key scenarios.

Azure Functions supports:
-  triggers (ways to start execution of your code).
- bindings (ways to simplify coding for input and output data).

They can all define input, actions, conditions, and output.

## Compare Azure Functions and Azure Logic Apps

Functions: 
  - enable serverless workloads
  - serverless compute service
  - can create complex orchestrations*.
  - syou develop orchestrations by writing code and using the Durable Functions extension
  - Development: 	Code-first (imperative)
  - Connectivity: About a dozen built-in binding types, write code for custom bindings
  - Actions: Each activity is an Azure function; write code for activity functions
  - Monitoring: Azure Application Insights
  - Management: REST API, Visual Studio
  - Execution context: Can run locally or in the cloud

Logic Apps:
  - enable serverless workloads
  - serverless workflows
  - can create complex orchestrations*.
  - you create orchestrations by using a GUI or editing configuration files
  - Development: Designer-first (declarative)
  - Connectivity: Large collection of connectors, Enterprise Integration Pack for B2B scenarios, build custom connectors
  - Actions: Large collection of ready-made actions
  - Monitoring: Azure portal, Azure Monitor logs
  - Azure portal, REST API, PowerShell, Visual Studio
  - Execution context: Supports run-anywhere scenarios
  

*Orchestrations: An orchestration is a collection of functions or steps, called actions in Logic Apps, that are executed to accomplish a complex task.

## Compare Functions and WebJobs
Both are built on Azure App Service and support features such as source control integration, authentication, and monitoring with Application Insights integration.

Azure Functions is built on the WebJobs SDK,

Here are some factors to consider when you're choosing between Azure Functions and WebJobs with the WebJobs SDK:

Serverless app model with automatic scaling: Functions

Develop and test in browser: Functions

Pay-per-use pricing: Functions

Integration with Logic Apps: Functions

Trigger events: 

  Functions:
  - Timer
  - Azure Storage queues and blobs
  - Azure Service Bus queues and topics
  - Azure Cosmos DB
  - Azure Event Hubs
  - HTTP/WebHook (GitHub
  - Slack)
  - Azure Event Grid

WebJobs:
- Timer
- Azure Storage queues and blobs
- Azure Service Bus queues and topics
- Azure Cosmos DB
- Azure Event Hubs
- File system

For most scenarios, Azure functions it's the best choice.

# Compare Azure Functions hosting options
Three hosting options for Azure Functions (available (GA) on both Linux and Windows virtual machines.):
- Consumption plan:
  This is the default hosting plan. It scales automatically and you only pay for compute resources when your functions are running. Instances of the Functions host are dynamically added and removed based on the number of incoming events.

- Functions Premium plan: 
  Automatically scales based on demand using pre-warmed workers which run applications with no delay after being idle, runs on more powerful instances, and connects to virtual networks.

- App service (Dedicated) plan:
  	Run your functions within an App Service plan at regular App Service plan rates. Best for long-running scenarios where Durable Functions can't be used.

There are two other hosting options which provide the highest amount of control and isolation:

- ASE (App Service Environment): provides a fully isolated and dedicated environment for securely running App Service apps at high scale.
  
- Kubernetes: Kubernetes provides a fully isolated and dedicated environment running on top of the Kubernetes platform. For more information visit Azure Functions on Kubernetes with KEDA(https://docs.microsoft.com/en-us/azure/azure-functions/functions-kubernetes-keda).

## Always on
If you run on an App Service plan, you should enable the Always on setting so that your function app runs correctly.the functions runtime goes idle after a few minutes of inactivity, so only HTTP triggers will "wake up" your functions.

## Storage account requirements
 function app requires a general Azure Storage account, which supports Azure Blob, Queue, Files, and Table storage. This is because Functions relies on Azure Storage for operations such as managing triggers and logging function executions, but some storage accounts do not support queues and tables. 

 The same storage account used by your function app can also be used by your triggers and bindings to store your application data. **However, for storage-intensive operations, you should use a separate storage account.**

 ## Scale Azure Functions
In the Consumption and Premium plans, Azure Functions scales CPU and memory resources by adding additional instances of the Functions host

Each instance of the Functions host in the Consumption plan is limited to 1.5 GB of memory and one CPU.

Function apps that share the same Consumption plan scale independently

In the Premium plan, the plan size determines the available memory and CPU for all apps in that plan on that instance.

 **Function code files are stored on Azure Files shares on the function's main storage account. When you delete the main storage account of the function app, the function code files are deleted and cannot be recovered.**

 ## Runtime scaling
Azure Functions uses a component called the **scale controller** to monitor the rate of events and determine whether to scale out or scale in. 

The unit of scale for Azure Functions is the function app.

The number of instances is eventually "scaled in" to zero when no functions are running within a function app.

After your function app has been idle for a number of minutes, the platform may scale the number of instances on which your app runs down to zero. The next request has the added latency of scaling from zero to one. This latency is referred to as a cold start.

## Scaling behaviors

- Maximum instances: A single function app only scales out to a maximum of 200 instances. A single instance may process more than one message or request at a time though, so there isn't a set limit on number of concurrent executions.
  
- New instance rate: For HTTP triggers, new instances are allocated, at most, once per second. For non-HTTP triggers, new instances are allocated, at most, once every 30 seconds. Scaling is faster when running in a Premium plan.

## Limit scale out

By default, Consumption plan functions scale out to as many as 200 instances, and Premium plan functions will scale out to as many as 100 instances.

The `functionAppScaleLimit` can be set to 0 or null for unrestricted, or a valid value between 1 and the app maximum.

## Azure Functions scaling in an App service plan
Using an App Service plan, you can manually scale out by adding more VM instances. You can also enable autoscale, though autoscale will be slower than the elastic scale of the Premium plan.

## Knowledge check
Which of the following Azure Functions hosting plans is best when predictive scaling and costs are required?

  App service plan

An organization wants to implement a serverless workflow to solve a business problem. One of the requirements is the solution needs to use a designer-first (declarative) development model. Which of the choices below meets the requirements?

  Azure Logic Apps
