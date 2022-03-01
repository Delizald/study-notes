# Azure App Service

Azure App Service is an HTTP-based service for hosting web applications, REST APIs, and mobile back ends. You can develop in your favorite language. Applications run and scale with ease on both Windows and Linux-based environments.

## Built-in auto scale support
Depending on the usage of an app you can scale your app up/down the resources.
Resources include the number of cores or the amount of RAM available.
Scaling out/in is the ability to increase, or decrease, the number of machine instances that are running your web app.

## Continuous integration/deployment support
Continuous integration and deployment with Azure DevOps, GitHub, Bitbucket, FTP, or a local Git repository on your development machine.

## Deployment slots
For instance, you can create a staging deployment slot where you can push your code to test on Azure. Once you are happy
you can easily swap the staging deployment slot with the production slot. You do all this with a few simple mouse clicks in the Azure portal.

**Deployment slots are only available in the Standard and Premium plan tiers.**

## App Service on Linux
App Service can also host web apps natively on Linux for supported application stacks. It can also run custom Linux containers (also known as Web App for Containers).
Supported languages include: 
- Node.js, 
- Java (JRE 8 & JRE 11), 
- PHP, 
- Python, 
- .NET Core,
- Ruby. 
If the runtime your application requires is not supported in the built-in images, you can deploy it with a custom container.

**The languages, and their supported versions, are updated on a regular basis. You can retrieve the current list by using the following command in the Cloud Shell.**

`az webapp list-runtimes --linux`

## App Service on Linux Limitations
- App Service on Linux is not supported on Shared pricing tier.
- You can't mix Windows and Linux apps in the same App Service plan.
Historically, you could not mix Windows and Linux apps in the same resource group. However, all resource groups created on or after January 21, 2021 do support this scenario. Support for resource groups created before January 21, 2021 will be rolled out across Azure regions (including National cloud regions) soon.
- The Azure portal shows only features that currently work for Linux apps. As features are enabled, they're activated on the portal.

# Azure App Service plans

- An App Service plan defines a set of compute resources for a web app to run.
- One or more apps can be configured to run on the same computing resources (or in the same App Service plan). In addition, Azure Functions also has the option of running in an App Service plan.

Each App Service plan defines:
- Region (West US, East US, etc.)
- Number of VM instances
- Size of VM instances (Small, Medium, Large)
- Pricing tier (Free, Shared, Basic, Standard, Premium, PremiumV2, PremiumV3, Isolated)

The pricing tier determines what App Service features you get and how much you pay:

- Shared compute: Both Free and Shared share the resource pools of your apps with the apps of other customers. These tiers allocate CPU quotas to each app that runs on the shared resources, and the resources can't scale out.
- Dedicated compute: The Basic, Standard, Premium, PremiumV2, and PremiumV3 tiers run apps on dedicated Azure VMs. Only apps in the same App Service plan share the same compute resources. The higher the tier, the more VM instances are available to you for scale-out.
- Isolated: This tier runs dedicated Azure VMs on dedicated Azure Virtual Networks. It provides network isolation on top of compute isolation to your apps. It provides the maximum scale-out capabilities.
- Consumption: This tier is only available to function apps. It scales the functions dynamically depending on workload.

**App Service Free and Shared (preview) hosting plans are base tiers that run on the same Azure virtual machines as other App Service apps. Some apps might belong to other customers. These tiers are intended to be used only for development and testing purposes.**

## How does my app run and scale?
- In the **Free** and **Shared tiers**, an app can't scale out.
- An app runs on all the VM instances configured in the App Service plan.
- If multiple apps are in the same App Service plan, they all share the same VM instances.
- If you have multiple deployment slots for an app, all deployment slots also run on the same VM instances.
- If you enable diagnostic logs, perform backups, or run WebJobs, they also use CPU cycles and memory on these VM instances.

- the App Service plan is the scale unit of the App Service apps. 

## What if my app needs more capabilities or features?
Isolate your app into a new App Service plan when:

- The app is resource-intensive.
- You want to scale the app independently from the other apps in the existing plan.
- The app needs resource in a different geographical region.

# Deploy to App Service
## Automated deployment
A process used to push out new features and bug fixes in a fast and repetitive pattern with minimal impact on end users.

The following options are available:
- Azure DevOps
- GitHub
- Bitbucket

## Manual deployment
- Git
- CLI: `webapp` up is a feature of the `az` command-line interface that packages your app and deploys it. Unlike other deployment methods, **`az webapp up`** can create a new App Service web app for you if you haven't already created one.
- Zip deploy: Use curl or a similar HTTP utility to send a ZIP of your application files to App Service.
- FTP/S.
## Use deployment slots
When using a Standard App Service Plan tier or better, you can deploy your app to a staging environment and then swap your staging and production slots. The swap operation warms up the necessary worker instances to match your production scale, thus eliminating downtime.

# Explore authentication and authorization in App Service
The built-in authentication feature for App Service and Azure Functions can save you time and effort by providing out-of-the-box authentication with federated identity provider. 
You can integrate with multiple login providers. For example, Azure AD, Facebook, Google, Twitter.

## Identity providers
App Service uses federated identity: a third-party identity provider manages the user identities and authentication flow for you

## Provider	Sign-in endpoint	How-To guidance
- Microsoft Identity Platform	`/.auth/login/aad`	App Service Microsoft Identity Platform login
- Facebook	`/.auth/login/facebook`	App Service Facebook login
- Google	`/.auth/login/google`	App Service Google login
- Twitter	`/.auth/login/twitter`	App Service Twitter login

## How it works

The authentication and authorization module runs in the same sandbox as your application code. When it's enabled, every incoming HTTP request passes through it before being handled by your application code.

- Authenticates users with the specified provider
- Validates, stores, and refreshes tokens
- Manages the authenticated session
- Injects identity information into request headers

**In Linux and containers the authentication and authorization module runs in a separate container, isolated from your application code. Because it does not run in-process, no direct integration with specific language frameworks is possible.**

## Authentication flow

Authentication flow

- Without provider SDK: The application delegates federated sign-in to App Service. This is typically the case with browser apps, which can present the provider's login page to the user. The server code manages the sign-in process, so it is also called server-directed flow or server flow.

- With provider SDK: The application signs users in to the provider manually and then submits the authentication token to App Service for validation. This is typically the case with browser-less apps, which can't present the provider's sign-in page to the user. The application code manages the sign-in process, so it is also called client-directed flow or client flow. This applies to REST APIs, Azure Functions, JavaScript browser clients, and native mobile apps that sign users in using the provider's SDK.

## authentication flow: 
Sign user in -> Redirects to `/.auth/login/<provider>` 
Client code signs user in directly with provider's SDK and receives an authentication token.

Post-authentication -> Provider redirects  to `/.auth/login/<provider>/callback`.
Client code posts token from provider to /.auth/login/<provider> for validation.

Establish authenticated session: App Service adds authenticated cookie to response.
App Service returns its own authentication token to client code.

Serve authenticated content: Client includes authentication cookie in subsequent requests (automatically handled by browser).
Client code presents authentication token in X-ZUMO-AUTH header (automatically handled by Mobile Apps client SDKs).

## Authorization behavior.
You can configure App Service with a number of behaviors when an incoming request is not authenticated.

- Allow unauthenticated requests: This option defers authorization of unauthenticated traffic to your application code. For authenticated requests, App Service also passes along authentication information in the HTTP headers.This option provides more flexibility in handling anonymous requests. It lets you present multiple sign-in providers to your users.

- Require authentication: This option will reject any unauthenticated traffic to your application. This rejection can be a redirect action to one of the configured identity providers. In these cases, a browser client is redirected to `/.auth/login/<provider>` for the provider you choose. If the anonymous request comes from a native mobile app, the returned response is an `HTTP 401 Unauthorized`. You can also configure the rejection to be an `HTTP 401 Unauthorized` or `HTTP 403 Forbidden` for all requests.


# Discover App Service networking features.
There are two main deployment types for Azure App Service:

- The multitenant public service hosts App Service plans in the Free, Shared, Basic, Standard, Premium, PremiumV2, and PremiumV3 pricing SKUs. 

- single-tenant App Service Environment (ASE) hosts Isolated SKU App Service plans directly in your Azure virtual network.

## Multitenant App Service networking features
- The roles that handle incoming HTTP or HTTPS requests are called front ends.
- The roles that host the customer workload are called workers. 

Inbound features | Outbound features
App-assigned address | 	Hybrid Connections
Access restrictions | 	Gateway-required VNet Integration
Service endpoints	| VNet Integration
Private endpoints

The following inbound use cases are examples of how to use App Service networking features to control traffic inbound to your app:

Inbound use case	| Feature
Support IP-based SSL needs for your app |   App-assigned address
Support unshared dedicated inbound address for your app |   App-assigned address
Restrict access to your app from a set of well-defined |   addresses	Access restrictions

## Default networking behavior
If you have a Standard App Service plan, all the apps in that plan will run on the same worker. If you scale out the worker, all the apps in that App Service plan will be replicated on a new worker for each instance in your App Service plan.

## Outbound addresses

The worker VMs are broken down in large part by the App Service plans. The Free, Shared, Basic, Standard, and Premium plans all use the same worker VM type. The PremiumV2 plan uses another VM type. PremiumV3 uses yet another VM type. When you change the VM family, you get a different set of outbound addresses.

If you want to see all the addresses that your app might use in a scale unit, there's property called `possibleOutboundAddresses` that will list them.

## Find outbound IPs
To find the outbound IP addresses currently used by your app in the Azure portal, click Properties in your app's left-hand navigation.

You can find the same information by running the following command in the Cloud Shell. They are listed in the Additional Outbound IP Addresses field.

```
az webapp show \
    --resource-group <group_name> \
    --name <app_name> \ 
    --query outboundIpAddresses \
    --output tsv
```

To find all possible outbound IP addresses for your app, regardless of pricing tiers, run the following command in the Cloud Shell.

```
`az webapp show \
    --resource-group <group_name> \ 
    --name <app_name> \ 
    --query possibleOutboundIpAddresses \
    --output tsv
```

##  Create a static HTML web app by using Azure Cloud Shell
The `az webapp up` command makes it easy to create and update web apps. When executed it performs the following actions:

- Create a default resource group.
- Create a default app service plan.
- Create an app with the specified name.
- Zip deploy files from the current working directory to the web app.

### Create the web app
`az webapp up --location <myLocation> --name <myAppName> --html`

This command may take a few minutes to run. While running, it displays information similar to the example below. Make a note of the **resourceGroup value**. You need it for the Clean up resources section later.

- `mkdir app`
- `cd app`
- `git clone https://github.com/app`
- `az webapp up --location <myLocation> --name <myAppName> --html`

```
{
"app_url": "https://<myAppName>.azurewebsites.net",
"location": "westeurope",
"name": "<app_name>",
"os": "Windows",
"resourcegroup": "<resource_group_name>",
"serverfarm": "appsvc_asp_Windows_westeurope",
"sku": "FREE",
"src_path": "/home/<username>/demoHTML/html-docs-hello-world ",
< JSON data removed for brevity. >
}
````

Open a browser and go to http://<myAppName>.azurewebsites.net

### Update and redeploy the app
- Edit any file
- `az webapp up --location <myLocation> --name <myAppName> --html`

### Clean up resources
`az group delete --name <resource_group> --no-wait`

## Knowledge check
- Which of the following App Service plans supports only function apps? Consumption
- Which of the following networking features of App Service can be used to control outbound network traffic? Hybrid Connections

