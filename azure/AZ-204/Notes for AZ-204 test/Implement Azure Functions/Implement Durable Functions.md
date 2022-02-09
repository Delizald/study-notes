# Learning objectives
- Describe the app patterns typically used in durable functions
- Describe the four durable function types
- Explain the function Task Hubs perform in durable functions
- Describe the use of durable orchestrations, timers, and events.

# Intro
Durable Functions is an extension of Azure Functions that lets you write stateful functions in a serverless compute environment.

# Explore Durable Functions app patterns

The durable functions extension lets you define stateful workflows by writing orchestrator functions and stateful entities by writing entity functions using the Azure Functions programming model. Behind the scenes, the extension manages state, checkpoints, and restarts for you, allowing you to focus on your business logic.

## Supported languages

- C#: both precompiled class libraries and C# script.
- JavaScript: supported only for version 2.x of the Azure Functions runtime. Requires version 1.7.0 of the Durable Functions extension, or a later version.
- Python: requires version 2.3.1 of the Durable Functions extension, or a later version.
- F#: precompiled class libraries and F# script. F# script is only supported for version 1.x of the Azure Functions runtime.
- PowerShell: Supported only for version 3.x of the Azure Functions runtime and PowerShell7. Requires version 2.x of the bundle extensions.

## Application patterns

**The primary use case for Durable Functions is simplifying complex, stateful coordination requirements in serverless applications.**
Typical application patterns:

- Function chaining:  a sequence of functions executes in a specific order. In this pattern, the output of one function is applied to the input of another function.
  
In the code example below, the values `F1`, `F2`, `F3`, and `F4` are the names of other functions in the function app.

```
[FunctionName("Chaining")]
public static async Task<object> Run(
    [OrchestrationTrigger] IDurableOrchestrationContext context)
{
    try
    {
        var x = await context.CallActivityAsync<object>("F1", null);
        var y = await context.CallActivityAsync<object>("F2", x);
        var z = await context.CallActivityAsync<object>("F3", y);
        return  await context.CallActivityAsync<object>("F4", z);
    }
    catch (Exception)
    {
        // Error handling or compensation goes here.
    }
}
```


- Fan-out/fan-in: you execute multiple functions in parallel and then wait for all functions to finish. Often, some aggregation work is done on the results that are returned from the functions.

In the code example below, the fan-out work is distributed to multiple instances of the `F2` function. The work is tracked by using a dynamic list of tasks. The .NET `Task.WhenAll` API or JavaScript `context.df.Task.all` API is called, to wait for all the called functions to finish. Then, the F2 function outputs are aggregated from the dynamic task list and passed to the `F3` function.

```
[FunctionName("FanOutFanIn")]
public static async Task Run(
    [OrchestrationTrigger] IDurableOrchestrationContext context)
{
    var parallelTasks = new List<Task<int>>();

    // Get a list of N work items to process in parallel.
    object[] workBatch = await context.CallActivityAsync<object[]>("F1", null);
    for (int i = 0; i < workBatch.Length; i++)
    {
        Task<int> task = context.CallActivityAsync<int>("F2", workBatch[i]);
        parallelTasks.Add(task);
    }

    await Task.WhenAll(parallelTasks);

    // Aggregate all N outputs and send the result to F3.
    int sum = parallelTasks.Sum(t => t.Result);
    await context.CallActivityAsync("F3", sum);
}
```

- Async HTTP APIs: addresses the problem of coordinating the state of long-running operations with external clients. A common way to implement this pattern is by having an HTTP endpoint trigger the long-running action. Then, redirect the client to a status endpoint that the client polls to learn when the operation is finished.

After an instance starts, the extension exposes webhook HTTP APIs that query the orchestrator function status.

The following example shows REST commands that start an orchestrator and query its status. For clarity, some protocol details are omitted from the example.

```
> curl -X POST https://myfunc.azurewebsites.net/orchestrators/DoWork -H "Content-Length: 0" -i
HTTP/1.1 202 Accepted
Content-Type: application/json
Location: https://myfunc.azurewebsites.net/runtime/webhooks/durabletask/b79baf67f717453ca9e86c5da21e03ec

{"id":"b79baf67f717453ca9e86c5da21e03ec", ...}

> curl https://myfunc.azurewebsites.net/runtime/webhooks/durabletask/b79baf67f717453ca9e86c5da21e03ec -i
HTTP/1.1 202 Accepted
Content-Type: application/json
Location: https://myfunc.azurewebsites.net/runtime/webhooks/durabletask/b79baf67f717453ca9e86c5da21e03ec

{"runtimeStatus":"Running","lastUpdatedTime":"2019-03-16T21:20:47Z", ...}

> curl https://myfunc.azurewebsites.net/runtime/webhooks/durabletask/b79baf67f717453ca9e86c5da21e03ec -i
HTTP/1.1 200 OK
Content-Length: 175
Content-Type: application/json

{"runtimeStatus":"Completed","lastUpdatedTime":"2019-03-16T21:20:57Z", ...}
```

You can use the `HttpStart` triggered function to start instances of an orchestrator function using a client function. To interact with orchestrators, the function must include a `DurableClient` input binding

```
public static class HttpStart
{
    [FunctionName("HttpStart")]
    public static async Task<HttpResponseMessage> Run(
        [HttpTrigger(AuthorizationLevel.Function, methods: "post", Route = "orchestrators/{functionName}")] HttpRequestMessage req,
        [DurableClient] IDurableClient starter,
        string functionName,
        ILogger log)
    {
        // Function input comes from the request content.
        object eventData = await req.Content.ReadAsAsync<object>();
        string instanceId = await starter.StartNewAsync(functionName, eventData);

        log.LogInformation($"Started orchestration with ID = '{instanceId}'.");

        return starter.CreateCheckStatusResponse(req, instanceId);
    }
}
```

- Monitor: refers to a flexible, recurring process in a workflow. An example is polling until specific conditions are met (such as a periodic cleanup job)

```
[FunctionName("MonitorJobStatus")]
public static async Task Run(
    [OrchestrationTrigger] IDurableOrchestrationContext context)
{
    int jobId = context.GetInput<int>();
    int pollingInterval = GetPollingInterval();
    DateTime expiryTime = GetExpiryTime();

    while (context.CurrentUtcDateTime < expiryTime)
    {
        var jobStatus = await context.CallActivityAsync<string>("GetJobStatus", jobId);
        if (jobStatus == "Completed")
        {
            // Perform an action when a condition is met.
            await context.CallActivityAsync("SendAlert", machineId);
            break;
        }

        // Orchestration sleeps until this time.
        var nextCheck = context.CurrentUtcDateTime.AddSeconds(pollingInterval);
        await context.CreateTimer(nextCheck, CancellationToken.None);
    }

    // Perform more work here, or let the orchestration end.
}
```

- Human interaction: Many automated processes involve some kind of human interaction.  An automated process might allow for this interaction by using timeouts and compensation logic.

```
[FunctionName("ApprovalWorkflow")]
public static async Task Run(
    [OrchestrationTrigger] IDurableOrchestrationContext context)
{
    await context.CallActivityAsync("RequestApproval", null);
    using (var timeoutCts = new CancellationTokenSource())
    {
        DateTime dueTime = context.CurrentUtcDateTime.AddHours(72);
        Task durableTimeout = context.CreateTimer(dueTime, timeoutCts.Token);

        Task<bool> approvalEvent = context.WaitForExternalEvent<bool>("ApprovalEvent");
        if (approvalEvent == await Task.WhenAny(approvalEvent, durableTimeout))
        {
            timeoutCts.Cancel();
            await context.CallActivityAsync("ProcessApproval", approvalEvent.Result);
        }
        else
        {
            await context.CallActivityAsync("Escalate", null);
        }
    }
}
```

## Differences between 1.x and 2.x Durable functions

https://docs.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-versions

# Discover the four function types

There are currently four durable function types in Azure: 
- orchestrator
- activity
- entity
- client

## Orchestrator functions
describe how actions are executed and the order in which actions are executed.
Orchestrator functions can also interact with entity functions.
orchestrator function code must be deterministic. Failing to follow these determinism requirements can cause orchestrator functions to fail to run correctly. (https://docs.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-code-constraints)

## Activity functions
**are the basic unit of work in a durable function orchestration.** 
For example, you might create an orchestrator function to process an order. These activity functions may be executed serially, in parallel, or some combination of both.
aren't restricted in the type of work you can do in them. Activity functions are frequently used to make network calls or run CPU intensive operations.
An activity trigger is used to define an activity function. 
.NET functions receive a `DurableActivityContext` as a parameter.
In JavaScript, you can access an input via the <activity trigger binding name> property on the context.bindings 
Activity functions can only have a single value passed to them. To pass multiple values, you must use tuples, arrays, or complex types.

## Entity functions
define operations for reading and updating small pieces of state.
We often refer to these stateful entities as durable entities
entity functions are functions with a special trigger type, `entity trigger`.
do not have any specific code constraints.
Entities are accessed via a unique identifier, the `entity ID`. An entity ID is simply a pair of strings that uniquely identifies an entity instance.
Operations on entities require that you specify the `Entity ID` of the target entity, and the `Operation name`, which is a string that specifies the operation to perform.

## Client functions

Orchestrator and entity functions are triggered by their bindings and both of these triggers work by reacting to messages that are enqueued in a task hub.
The primary way to deliver these messages is by using an orchestrator client from within a client function.

**Orchestrator and entity functions cannot be triggered directly using the buttons in the Azure portal.**
If you want to test an orchestrator or entity function in the Azure portal, you must instead run a client function that starts an orchestrator or entity function as part of its implementation. For the simplest testing experience, a manual trigger function is recommended.

# Explore task hubs
is a logical container for durable storage resources that are used for orchestrations and entities. Orchestrator, activity, and entity functions can only directly interact with each other when they belong to the same task hub.

If multiple function apps share a storage account, each function app must be configured with a separate task hub name.A storage account can contain multiple task hubs.

## Azure Storage resources
A task hub consists of the following:

- One or more control queues.
- One work-item queue.
- One history table.
- One instances table.
- One storage container containing one or more lease blobs.
- A storage container containing large message payloads, if applicable.

## Task hub names
- Contains only alphanumeric characters
- Starts with a letter
- Has a minimum length of 3 characters, maximum length of 45 characters

**The task hub name is declared in the host.json**

```
{
  "version": "2.0",
  "extensions": {
    "durableTask": {
      "hubName": "MyTaskHub"
    }
  }
}
```

**If you have multiple function apps sharing a shared storage account, you must explicitly configure different names for each task hub in the host.json**

# Explore durable orchestrations
Orchestrator functions have the following characteristics:
- Orchestrator functions define function workflows using procedural code. No declarative schemas or designers are needed.
- Orchestrator functions can call other durable functions synchronously and asynchronously. Output from called functions can be reliably saved to local variables.
- Orchestrator functions are durable and reliable. Execution progress is automatically checkpointed when the function "awaits" or "yields". Local state is never lost when the process recycles or the VM reboots.
- Orchestrator functions can be long-running. The total lifespan of an orchestration instance can be seconds, days, months, or never-ending.

## Orchestration identity
Each instance of an orchestration has an instance identifier (also known as an instance ID). 
By default, each instance ID is an autogenerated GUID.
instance IDs can also be any user-generated string value. 
An orchestration's instance ID is a required parameter for most instance management operations.

**It is generally recommended to use autogenerated instance IDs whenever possible. User-generated instance IDs are intended for scenarios where there is a one-to-one mapping between an orchestration instance and some external application-specific entity.**

## Reliability
Orchestrator functions reliably maintain their execution state by using the `event sourcing design pattern` (https://microservices.io/patterns/data/event-sourcing.html)
the Durable Task Framework uses an append-only store to record the full series of actions the function orchestration takes.
the `await (C#)` or `yield (JavaScript)` operator in an orchestrator function yields control of the orchestrator thread back to the Durable Task Framework dispatcher.

When an orchestration function is given more work to do, the orchestrator wakes up and re-executes the entire function from the start to rebuild the local state. During the replay, if the code tries to call a function (or do any other async work), the Durable Task Framework consults the execution history of the current orchestration. If it finds that the activity function has already executed and yielded a result, it replays that function's result and the orchestrator code continues to run.

## Features and patterns

- Sub-orchestrations: Orchestrator functions can call activity functions, but also other orchestrator functions.

- Durable timers: to implement delays or to set up timeout handling on async actions. Use durable timers in orchestrator functions instead of `Thread.Sleep` and `Task.Delay` (C#) or `setTimeout()` and `setInterval()` (JavaScript).

- External events: 	Orchestrator functions can wait for external events to update an orchestration instance. Useful for human interactions or other external callbacks.

- Error handling: Orchestrator functions can use the error-handling features of the programming language.

- Critical sections: 	Orchestration instances are single-threaded so it isn't necessary to worry about race conditions. To mitigate race conditions when interacting with external systems, orchestrator functions can define critical sections using a `LockAsync` method in .NET.

- Calling HTTP endpoints: Orchestrator functions aren't permitted to do I/O. The typical workaround for this limitation is to wrap any code that needs to do I/O in an activity function.

- Passing multiple parameters: It isn't possible to pass multiple parameters directly (use an array of objects or  ValueTuples objects in .NET)

# Control timing in Durable Functions
Durable Functions provides durable timers for use in orchestrator functions to implement delays or to set up timeouts on async actions.

You create a durable timer by calling the `CreateTimer` (.NET) method or the `createTimer` (JavaScript) method of the orchestration trigger binding. The method returns a task that completes on a specified date and time.

## Timer limitations
When you create a timer that expires at 4:30 pm, the underlying Durable Task Framework enqueues a message that becomes visible only at 4:30 pm. When running in the Azure Functions Consumption plan, the newly visible timer message will ensure that the function app gets activated on an appropriate VM.

## Usage for delay
The following example is issuing a billing notification every day for 10 days.

```
[FunctionName("BillingIssuer")]
public static async Task Run(
    [OrchestrationTrigger] IDurableOrchestrationContext context)
{
    for (int i = 0; i < 10; i++)
    {
        DateTime deadline = context.CurrentUtcDateTime.Add(TimeSpan.FromDays(1));
        await context.CreateTimer(deadline, CancellationToken.None);
        await context.CallActivityAsync("SendBillingEvent");
    }
}
```

## Usage for timeout
```
[FunctionName("TryGetQuote")]
public static async Task<bool> Run(
    [OrchestrationTrigger] IDurableOrchestrationContext context)
{
    TimeSpan timeout = TimeSpan.FromSeconds(30);
    DateTime deadline = context.CurrentUtcDateTime.Add(timeout);

    using (var cts = new CancellationTokenSource())
    {
        Task activityTask = context.CallActivityAsync("GetQuote");
        Task timeoutTask = context.CreateTimer(deadline, cts.Token);

        Task winner = await Task.WhenAny(activityTask, timeoutTask);
        if (winner == activityTask)
        {
            // success case
            cts.Cancel();
            return true;
        }
        else
        {
            // timeout case
            return false;
        }
    }
}
```

***Use a `CancellationTokenSource` to cancel a durable timer (.NET) or call `cancel()`on the returned TimerTask (JavaScript) if your code will not wait for it to complete.**

This cancellation mechanism doesn't terminate in-progress activity function or sub-orchestration executions. It simply allows the orchestrator function to ignore the result and move on.

# Send and wait for events
## Wait for events
- `WaitForExternalEvent` (.NET)
- `waitForExternalEvent` (JavaScript)
- `wait_for_external_event` (Python)

The listening orchestrator function declares the name of the event and the shape of the data it expects to receive.

```
[FunctionName("BudgetApproval")]
public static async Task Run(
    [OrchestrationTrigger] IDurableOrchestrationContext context)
{
    bool approved = await context.WaitForExternalEvent<bool>("Approval");
    if (approved)
    {
        // approval granted - do the approved action
    }
    else
    {
        // approval denied - send a notification
    }
}
```

## Send events
The `RaiseEventAsync` (.NET) or `raiseEvent` (JavaScript) method of the orchestration client binding sends the events that `WaitForExternalEvent` (.NET) or `waitForExternalEvent` (JavaScript) wait for.

Internally, RaiseEventAsync (.NET) or raiseEvent (JavaScript) enqueues a message that gets picked up by the waiting orchestrator function. If the instance is not waiting on the specified event name, the event message is added to an in-memory queue.

The RaiseEventAsync method takes eventName and eventData as parameters. The event data must be JSON-serializable.

```
[FunctionName("ApprovalQueueProcessor")]
public static async Task Run(
    [QueueTrigger("approval-queue")] string instanceId,
    [DurableClient] IDurableOrchestrationClient client)
{
    await client.RaiseEventAsync(instanceId, "Approval", true);
}
```

# Knowledge check
Which of the following durable function types is used to read and update small pieces of state?

Entity

Which application pattern would you use for a durable function that is polling a resource until a specific condition is met?

Monitor