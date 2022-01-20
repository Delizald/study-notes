# Azure App Service

- Azure App Service is an HTTP-based service for hosting web applications, REST APIs, and mobile back ends. You can develop in your favorite language. Applications run and scale with ease on both Windows and Linux-based environments.

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