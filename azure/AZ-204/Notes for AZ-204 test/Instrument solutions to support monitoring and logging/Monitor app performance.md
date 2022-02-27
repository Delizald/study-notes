- Explain how Azure Monitor operates as the center of monitoring in Azure.
- Describe how Application Insights works and how it collects events and metrics.
- Instrument an app for monitoring, perform availability tests, and use Application Map to help you monitor performance and troubleshoot issues.

# Explore Azure Monitor
delivers a comprehensive solution for collecting, analyzing, and acting on telemetry from your cloud and on-premises environments.

## What data does Azure Monitor collect?
`Application monitoring data`: Data about the performance and functionality of the code you have written, regardless of its platform.
`Guest OS monitoring data`: Data about the operating system on which your application is running. This could be running in Azure, another cloud, or on-premises.
`Azure resource monitoring data`: Data about the operation of an Azure resource. For a complete list of the resources that have metrics or logs, (https://docs.microsoft.com/en-us/azure/azure-monitor/monitor-reference#list-of-azure-monitor-supported-services)
`Azure subscription monitoring data`: Data about the operation and management of an Azure subscription, as well as data about the health and operation of Azure itself.
`Azure tenant monitoring data`: Data about the operation of tenant-level Azure services, such as Azure Active Directory.

## Monitoring data platform
All data collected by Azure Monitor fits into one of two fundamental types, `metrics` and `logs`

Metrics are numerical values that describe some aspect of a system at a particular point in time. 

Logs contain different kinds of data organized into records with different sets of properties for each type. Telemetry such as events and traces are stored as logs in addition to performance data so that it can all be combined for analysis.

## Insights and curated visualizations
Some Azure resource providers have a "curated visualization" which gives you a customized monitoring experience for that particular service or set of services. . Larger scalable curated visualizations are known at "insights" and marked with that name in the documentation and Azure portal. Some examples are:

Application Insights: Application Insights monitors the availability, performance, and usage of your web applications whether they're hosted in the cloud or on-premises. 

Container Insights: Container Insights monitors the performance of container workloads that are deployed to managed Kubernetes clusters hosted on Azure Kubernetes Service (AKS) and Azure Container Instances. 

VM Insights: VM Insights monitors your Azure virtual machines (VM) at scale. It analyzes the performance and health of your Windows and Linux VMs and identifies their different processes and interconnected dependencies on external processes.

## Explore Application Insights

is an extensible Application Performance Management (APM) service for developers and DevOps professionals. Use it to monitor your live applications. It will automatically detect performance anomalies

## How Application Insights works
You install a small instrumentation package (SDK) in your application or enable Application Insights using the Application Insights Agent (https://docs.microsoft.com/en-us/azure/azure-monitor/app/platforms)

You can instrument not only the web service application, but also any background components, and the JavaScript in the web pages themselves. The application and its components can run anywhere - it doesn't have to be hosted in Azure

## What Application Insights monitors
`Request rates, response times, and failure rates` - Find out which pages are most popular, at what times of day, and where your users are. See which pages perform best. If your response times and failure rates go high when there are more requests, then perhaps you have a resourcing problem.
`Dependency rates, response times, and failure rates` - Find out whether external services are slowing you down.
`Exceptions` - Analyze the aggregated statistics, or pick specific instances and drill into the stack trace and related requests. Both server and browser exceptions are reported.
`Page views and load performance` - reported by your users' browsers.
`AJAX calls from web pages` - rates, response times, and failure rates.
`User and session counts`.
`Performance counters` from your Windows or Linux server machines, such as CPU, memory, and network usage.
`Host diagnostics from Docker or Azure`.
`Diagnostic trace logs from your app` - so that you can correlate trace events with requests.
`Custom events and metrics` that you write yourself in the client or server code, to track business events such as items sold or games won.

## Use Application Insights
There are several ways to get started monitoring and analyzing app performance:

`At run time`: instrument your web app on the server. Ideal for applications already deployed. Avoids any update to the code.
`At development time`: add Application Insights to your code. Allows you to customize telemetry collection and send additional telemetry.
`Instrument your web pages for page view, AJAX, and other client-side telemetry`.
`Analyze mobile app usage` by integrating with Visual Studio App Center.
`Availability tests` - ping your website regularly from our servers.

# Discover log-based metrics

Log-based metrics behind the scene are translated into `Kusto queries` (https://docs.microsoft.com/en-us/azure/kusto/query/) from stored events.
Standard metrics are stored as pre-aggregated time series.

Since standard metrics are pre-aggregated during collection, they have better performance at query time.
log-based metrics have more dimensions, which makes them the superior option for data analysis and ad-hoc diagnostics
(https://docs.microsoft.com/en-us/azure/azure-monitor/essentials/metrics-getting-started#create-your-first-metric-chart)

## Log-based metrics
Using logs to retain a complete set of events can bring great analytical and diagnostic value. At the same time, collecting a complete set of events may be impractical (or even impossible) for applications that generate a large volume of telemetry. For situations when the volume of events is too high, Application Insights implements several telemetry volume reduction techniques, such as sampling and filtering that reduce the number of collected and stored events.

## Pre-aggregated metrics
The pre-aggregated metrics are not stored as individual events with lots of properties. Instead, they are stored as pre-aggregated time series, and only with key dimensions

**Both, log-based and pre-aggregated metrics coexist in Application Insights. To differentiate the two, in the Application Insights UX the pre-aggregated metrics are now called "Standard metrics (preview)", while the traditional metrics from the events were renamed to "Log-based metrics".**

# Instrument an app for monitoring

## Auto-instrumentation
Auto-instrumentation allows you to enable application monitoring with Application Insights without changing your code.
The list of services that are supported by auto-instrumentation changes rapidly, visit this page for a list of what is currently supported.
(https://docs.microsoft.com/en-us/azure/azure-monitor/app/codeless-overview#supported-environments-languages-and-resource-providers)

## Instrumenting for distributed tracing

Distributed tracing is the equivalent of call stacks for modern cloud and microservices architectures, with the addition of a simplistic performance profiler thrown in. zure Monitor, we provide two experiences for consuming distributed trace data. The first is our transaction diagnostics view, which is like a call stack with a time dimension added in. The transaction diagnostics view provides visibility into one single transaction/request, and is helpful for finding the root cause of reliability issues and performance bottlenecks on a per request basis.

## How to enable distributed tracing
Enabling distributed tracing across the services in an application is as simple as adding the proper SDK or library to each service, based on the language the service was implemented in.

## Enabling via Application Insights SDKs
The Application Insights SDKs for .NET, .NET Core, Java, Node.js, and JavaScript all support distributed tracing natively.
Additionally, any technology can be tracked manually with a call to `TrackDependency` on the `TelemetryClient`.

## Enable via OpenCensus
OpenCensus is an open source, vendor-agnostic, single distribution of libraries to provide metrics collection and distributed tracing for services. It also enables the open source community to enable distributed tracing with popular technologies like Redis, Memcached, or MongoDB.

## Select an availability test
You can create up to 100 availability tests per Application Insights resource, and there are three types of availability tests:
- URL ping test (classic): You can create this simple test through the portal to validate whether an endpoint is responding and measure performance associated with that response. You can also set custom success criteria coupled with more advanced features, like parsing dependent requests and allowing for retries.
- Standard test (Preview): This single request test is similar to the URL ping test. It includes SSL certificate validity, proactive lifetime check, HTTP request verb (for example GET, HEAD, or POST), custom headers, and custom data associated with your HTTP request.
- Custom TrackAvailability test: If you decide to create a custom application to run availability tests, you can use the TrackAvailability() method to send the results to Application Insights.

**Multi-step test is a fourth type of availability test, however that is only available through Visual Studio 2019. Custom TrackAvailability test is the long term supported solution for multi request or authentication test scenarios.**

**The URL ping test relies on the DNS infrastructure of the public internet to resolve the domain names of the tested endpoints. If you're using private DNS, you must ensure that the public domain name servers can resolve every domain name of your test. When that's not possible, you can use custom TrackAvailability tests instead.**

# Troubleshoot app performance by using Application Map
Application Map helps you spot performance bottlenecks or failure hotspots across all components of your distributed application. 

Components are independently deployable parts of your distributed/microservices application. Developers and operations teams have code-level visibility or access to telemetry generated by these application components.

- Components are different from "observed" external dependencies such as SQL, Event Hubs, etc. which your team/organization may not have access to (code or telemetry).
- Components run on any number of server/role/container instances.
- Components can be separate Application Insights instrumentation keys (even if subscriptions are different) or different roles reporting to a single Application Insights instrumentation key. The preview map experience shows the components regardless of how they are set up.

# Knowledge check
1. Which of the following availability tests is recommended for authentication tests? Custom TrackAvailability
2. Which of the following metric collection types below provides near real-time querying and alerting on dimensions of metrics, and more responsive dashboards? Pre-aggregated