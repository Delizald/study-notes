The languages, and their supported versions, are updated on a regular basis. You can retrieve the current list by using the following command in the Cloud Shell.

az webapp list-runtimes --linux

Limitations
App Service on Linux does have some limitations:

App Service on Linux is not supported on Shared pricing tier.
You can't mix Windows and Linux apps in the same App Service plan.
Historically, you could not mix Windows and Linux apps in the same resource group. However, all resource groups created on or after January 21, 2021 do support this scenario. Support for resource groups created before January 21, 2021 will be rolled out across Azure regions (including National cloud regions) soon.
The Azure portal shows only features that currently work for Linux apps. As features are enabled, they're activated on the portal


Apps) always runs in an App Service plan. An App Service plan defines a set of compute resources for a web app to run. One or more apps can be configured to run on the same computing resources (or in the same App Service plan). In addition, Azure Functions also has the option of running in an App Service plan.

When you create an App Service plan in a certain region (for example, West Europe), a set of compute resources is created for that plan in that region. Whatever apps you put into this App Service plan run on these compute resources as defined by your App Service plan. Each App Service plan defines:

Region (West US, East US, etc.)
Number of VM instances
Size of VM instances (Small, Medium, Large)
Pricing tier (Free, Shared, Basic, Standard, Premium, PremiumV2, PremiumV3, Isolated)

The pricing tier of an App Service plan determines what App Service features you get and how much you pay for the plan. There are a few categories of pricing tiers:

Shared compute: Both Free and Shared share the resource pools of your apps with the apps of other customers. These tiers allocate CPU quotas to each app that runs on the shared resources, and the resources can't scale out.
Dedicated compute: The Basic, Standard, Premium, PremiumV2, and PremiumV3 tiers run apps on dedicated Azure VMs. Only apps in the same App Service plan share the same compute resources. The higher the tier, the more VM instances are available to you for scale-out.
Isolated: This tier runs dedicated Azure VMs on dedicated Azure Virtual Networks. It provides network isolation on top of compute isolation to your apps. It provides the maximum scale-out capabilities.
Consumption: This tier is only available to function apps. It scales the functions dynamically depending on workload.

How does my app run and scale?
In the Free and Shared tiers, an app receives CPU minutes on a shared VM instance and can't scale out. In other tiers, an app runs and scales as follows:

An app runs on all the VM instances configured in the App Service plan.
If multiple apps are in the same App Service plan, they all share the same VM instances.
If you have multiple deployment slots for an app, all deployment slots also run on the same VM instances.
If you enable diagnostic logs, perform backups, or run WebJobs, they also use CPU cycles and memory on these VM instances.

the App Service plan is the scale unit of the App Service apps

Isolate your app into a new App Service plan when:

The app is resource-intensive.
You want to scale the app independently from the other apps the existing plan.
The app needs resource in a different geographical region.

Automated deployment

Azure supports automated deployment directly from several sources. The following options are available:

Azure DevOps: You can push your code to Azure DevOps, build your code in the cloud, run the tests, generate a release from the code, and finally, push your code to an Azure Web App.
GitHub: Azure supports automated deployment directly from GitHub. When you connect your GitHub repository to Azure for automated deployment, any changes you push to your production branch on GitHub will be automatically deployed for you.
Bitbucket: With its similarities to GitHub, you can configure an automated deployment with Bitbucket.

Manual deployment

There are a few options that you can use to manually push your code to Azure:

Git: App Service web apps feature a Git URL that you can add as a remote repository. Pushing to the remote repository will deploy your app.
CLI: webapp up is a feature of the az command-line interface that packages your app and deploys it. Unlike other deployment methods, az webapp up can create a new App Service web app for you if you haven't already created one.
Zip deploy: Use curl or a similar HTTP utility to send a ZIP of your application files to App Service.
FTP/S: FTP or FTPS is a traditional way of pushing your code to many hosting environments, including App Service.

Use deployment slots
Whenever possible, use deployment slots when deploying a new production build. When using a Standard App Service Plan tier or better, you can deploy your app to a staging environment and then swap your staging and production slots. The swap operation warms up the necessary worker instances to match your production scale, thus eliminating downtime.

Explore authentication and authorization in App Service

The built-in authentication feature for App Service and Azure Functions can save you time and effort by providing out-of-the-box authentication with federated identity providers, allowing you to focus on the rest of your application.

Azure App Service allows you to integrate a variety of auth capabilities into your web app or API without implementing them yourself.
It’s built directly into the platform and doesn’t require any particular language, SDK, security expertise, or even any code to utilize.
You can integrate with multiple login providers. For example, Azure AD, Facebook, Google, Twitter.


Identity providers
App Service uses federated identity, in which a third-party identity provider manages the user identities and authentication flow for you. The following identity providers are available by default:

IDENTITY PROVIDERS
Provider	Sign-in endpoint	How-To guidance
Microsoft Identity Platform	/.auth/login/aad	App Service Microsoft Identity Platform login
Facebook	/.auth/login/facebook	App Service Facebook login
Google	/.auth/login/google	App Service Google login
Twitter	/.auth/login/twitter	App Service Twitter login


The authentication and authorization module runs in the same sandbox as your application code:

Authenticates users with the specified provider
Validates, stores, and refreshes tokens
Manages the authenticated session
Injects identity information into request headers


Authentication flow

Without provider SDK: The application delegates federated sign-in to App Service. This is typically the case with browser apps, which can present the provider's login page to the user. The server code manages the sign-in process, so it is also called server-directed flow or server flow.

With provider SDK: The application signs users in to the provider manually and then submits the authentication token to App Service for validation. This is typically the case with browser-less apps, which can't present the provider's sign-in page to the user. The application code manages the sign-in process, so it is also called client-directed flow or client flow. This applies to REST APIs, Azure Functions, JavaScript browser clients, and native mobile apps that sign users in using the provider's SDK.

TODO: Authentication flow table.

Authorization behavior

Allow unauthenticated requests: This option defers authorization of unauthenticated traffic to your application code. For authenticated requests, App Service also passes along authentication information in the HTTP headers.This option provides more flexibility in handling anonymous requests. It lets you present multiple sign-in providers to your users.

Require authentication: This option will reject any unauthenticated traffic to your application. This rejection can be a redirect action to one of the configured identity providers. In these cases, a browser client is redirected to /.auth/login/<provider> for the provider you choose. If the anonymous request comes from a native mobile app, the returned response is an HTTP 401 Unauthorized. You can also configure the rejection to be an HTTP 401 Unauthorized or HTTP 403 Forbidden for all requests.


Discover App Service networking features

Multitenant App Service networking features

The roles that handle incoming HTTP or HTTPS requests are called front ends.

The roles that host the customer workload are called workers. 

 All the roles in an App Service deployment exist in a multitenant network. Because there are many different customers in the same App Service scale unit, you can't connect the App Service network directly to your network.

 #TODO TABLES

 Default networking behavior

 if you have a Standard App Service plan, all the apps in that plan will run on the same worker. If you scale out the worker, all the apps in that App Service plan will be replicated on a new worker for each instance in your App Service plan.

 Outbound addresses

When you change the VM family, you get a different set of outbound addresses. If you change Plans the addresses wull change. In some older scale units, both the inbound and outbound addresses will change when you scale from Standard to PremiumV2.

 If you want to see all the addresses that your app might use in a scale unit, there's property called possibleOutboundAddresses that will list them.

 Find outbound IPs
 To find the outbound IP addresses currently used by your app in the Azure portal, click Properties in your app's left-hand navigation.

You can find the same information by running the following command in the Cloud Shell. They are listed in the Additional Outbound IP Addresses field.

az webapp show \
    --resource-group <group_name> \
    --name <app_name> \ 
    --query outboundIpAddresses \
    --output tsv

To find all possible outbound IP addresses for your app, regardless of pricing tiers, run the following command in the Cloud Shell.

az webapp show \
    --resource-group <group_name> \ 
    --name <app_name> \ 
    --query possibleOutboundIpAddresses \
    --output tsv

The az webapp up command makes it easy to create and update web apps. When executed it performs the following actions:

The az webapp up command makes it easy to create and update web apps. When executed it performs the following actions:

Create a default resource group.
Create a default app service plan.
Create an app with the specified name.
Zip deploy files from the current working directory to the web app.


Clean up resources
```bash
az group delete --name <resource_group> --no-wait
```



Which of the following networking features of App Service can be used to control outbound network traffic? Hybrid Connections