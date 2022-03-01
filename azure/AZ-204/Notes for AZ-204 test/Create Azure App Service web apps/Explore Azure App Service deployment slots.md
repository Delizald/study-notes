# Learning objectives
After completing this module, you'll be able to:

- Describe the benefits of using deployment slots.
- Understand how slot swapping operates in App Service.
- Perform manual swaps and enable auto swap.
- Route traffic manually and automatically.

# Explore staging environments

Deploying your application to a non-production slot has the following benefits:

- You can validate app changes in a staging deployment slot before swapping it with the production slot.
- Deploying an app to a slot first and swapping it into production makes sure that all instances of the slot are warmed up before being swapped into production. This eliminates downtime when you deploy your app. The traffic redirection is seamless, and no requests are dropped because of swap operations. You can automate this entire workflow by configuring auto swap when pre-swap validation isn't needed.
- After a swap, the slot with previously staged app now has the previous production app. If the changes swapped into the production slot aren't as you expect, you can perform the same swap immediately to get your "last known good site" back.

Each App Service plan tier supports a different number of deployment slots. There's no additional charge for using deployment slots. 

Standard tier supports only five deployment slots.

When you create a new slot the new deployment slot has no content, even if you clone the settings from a different slot. You can deploy to the slot from a different repository branch or a different repository.

# Examine slot swapping
When swapping a lot, App Service does the following to ensure that the target slot doesn't experience downtime:

1. Apply the following settings from the target slot (for example, the production slot) to all instances of the source slot:
  - Slot-specific app settings and connection strings, if applicable.
  - Continuous deployment settings, if enabled.
  - App Service authentication settings, if enabled.
2. Wait for every instance in the source slot to complete its restart. If any instance fails to restart, the swap operation reverts all changes to the source slot and stops the operation.
3. If local cache is enabled, trigger local cache initialization by making an HTTP request to the application root ("/") on each instance of the source slot. Wait until each instance returns any HTTP response. Local cache initialization causes another restart on each instance.
4. If auto swap is enabled with custom warm-up, trigger Application Initiation by making an HTTP request to the application root ("/") on each instance of the source slot.
   - If `applicationInitialization` isn't specified, trigger an HTTP request to the application root of the source slot on each instance.
   - If an instance returns any HTTP response, it's considered to be warmed up.
5. swap the two slots by switching the routing rules for the two slots. After this step, the target slot (for example, the production slot) has the app that's previously warmed up in the source slot.
6. Now that the source slot has the pre-swap app previously in the target slot, perform the same operation by applying all settings and restarting the instances.

**Settings that are swapped**:
- General settings, such as framework version, 32/64-bit, web sockets
- App settings (can be configured to stick to a slot)
- Connection strings (can be configured to stick to a slot)
- Handler mappings
- Public certificates
- WebJobs content
- Hybrid connections *
- Service endpoints *
- Azure Content Delivery Network *
- Path mappings
Features marked with an asterisk (*) are planned to be unswapped.

**Settings that aren't swapped**:
- Publishing endpoints
- Custom domain names
- Non-public certificates and TLS/SSL settings
- Scale settings
- WebJobs schedulers
- IP restrictions
- Always On
- Diagnostic settings
- Cross-origin resource sharing (CORS)
- Virtual network integration
- Managed identities
- Settings that end with the suffix _EXTENSION_VERSION

**To make settings swappable, add the app setting WEBSITE_OVERRIDE_PRESERVE_DEFAULT_STICKY_SLOT_SETTINGS in every slot of the app and set its value to 0 or false. These settings are either all swappable or not at all.**

**To configure an app setting or connection string to stick to a specific slot (not swapped), go to the Configuration page for that slot. Add or edit a setting, and then select Deployment slot setting. Selecting this check box tells App Service that the setting is not swappable.**

# Swap Deployment Slots
You can swap deployment slots on your app's Deployment slots page and the Overview page.

## Manually swapping deployment slots
- Go to your app's Deployment slots page and select Swap. The Swap dialog box shows settings in the selected source and target slots that will be changed.

- Select the desired Source and Target slots. Usually, the target is the production slot. Also, select the Source Changes and Target Changes tabs and verify that the configuration changes are expected. When you're finished, you can swap the slots immediately by selecting Swap.

- To see how your target slot would run with the new settings before the swap actually happens, don't select Swap, but follow the instructions in Swap with preview.

- When you're finished, close the dialog box by selecting Close.

## Swap with preview (multi-phase swap)
When you perform a swap with preview, App Service performs the same swap operation but pauses after the first step. You can then verify the result on the staging slot before completing the swap.
If you cancel the swap, App Service reapplies configuration elements to the source slot.

- Follow the steps above in Swap deployment slots but select `Perform swap with preview`. The dialog box shows you how the configuration in the source slot changes in phase 1, and how the source and target slot change in phase 2.

- When you're ready to start the swap, select Start Swap.

- When phase 1 finishes, you're notified in the dialog box. Preview the swap in the source slot by going to `https://<app_name>-<source-slot-name>.azurewebsites.net.`

- When you're ready to complete the pending swap, select Complete Swap in Swap action and select Complete Swap. To cancel a pending swap, select Cancel Swap.

When you're finished, close the dialog box by selecting Close.

## Configure auto swap
Auto swap streamlines Azure DevOps scenarios where you want to deploy your app continuously with zero cold starts and zero downtime for customers of the app

**Auto swap isn't currently supported in web apps on Linux.** (2022-01-30)
To configure auto swap:
- Go to your app's resource page and select the deployment slot you want to configure to auto swap. The setting is on the `Configuration > General` settings page.

- Set Auto swap enabled to On. Then select the desired target slot for Auto swap deployment slot, and select Save on the command bar.

- Execute a code push to the source slot. Auto swap happens after a short time, and the update is reflected at your target slot's URL.

## Specify custom warm-up
The `applicationInitialization` configuration element in web.config lets you specify custom initialization actions. The swap operation waits for this custom warm-up to finish before swapping with the target slot
Sample:
```
<system.webServer>
    <applicationInitialization>
        <add initializationPage="/" hostName="[app hostname]" />
        <add initializationPage="/Home/About" hostName="[app hostname]" />
    </applicationInitialization>
</system.webServer>
```

For more information on customizing the applicationInitialization element, see **Most common deployment slot swap failures and how to fix them (https://ruslany.net/2017/11/most-common-deployment-slot-swap-failures-and-how-to-fix-them/)**.

You can also customize the warm-up behavior with one or both of the following app settings:

- `WEBSITE_SWAP_WARMUP_PING_PATH`: The path to ping to warm up your site. Add this app setting by specifying a custom path that begins with a slash as the value. An example is /statuscheck. The default value is /.
- `WEBSITE_SWAP_WARMUP_PING_STATUSES`: Valid HTTP response codes for the warm-up operation. Add this app setting with a comma-separated list of HTTP codes. An example is 200,202 . If the returned status code isn't in the list, the warmup and swap operations are stopped. By default, all response codes are valid.

## Roll back and monitor a swap
If any errors occur in the target slot (for example, the production slot) after a slot swap, restore the slots to their pre-swap states by swapping the same two slots immediately.

you can get information on the swap operation in the activity log ->On your app's resource page in the portal, in the left pane, select `Activity log`.
A swap operation appears in the log query as Swap Web App Slots.

# Route traffic in App Service

By default, all client requests to the app's production URL (http://<app_name>.azurewebsites.net) are routed to the production slot.

## Route production traffic automatically
- Go to your app's resource page and select **Deployment slots**.
- In the **Traffic** % column of the slot you want to route to, specify a percentage (between 0 and 100) to represent the amount of total traffic you want to route. Select **Save**.

the specified percentage of clients is randomly routed to the non-production slot.

After a client is automatically routed to a specific slot, it's "pinned" to that slot for the life of that client session.

On the client browser, you can see which slot your session is pinned to by looking at the `x-ms-routing-name` cookie in your HTTP headers.

staging slot -> x-ms-routing-name=staging
production slot -> x-ms-routing-name=self

## Route production traffic manually
This is useful when you want your users to be able to opt in to or opt out of your beta app.

To route production traffic manually, you use the `x-ms-routing-name` query parameter.

To let users opt out of your beta app, for example, you can put this link on your webpage:

```<a href="<webappname>.azurewebsites.net/?x-ms-routing-name=self">Go back to production app</a>```

After the client browser accesses the link every subsequent request has the `x-ms-routing-name=self` cookie that pins the session to the production slot.

To let users opt in to your beta app, set the same query parameter to the name of the non-production slot. Here's an example:

```<webappname>.azurewebsites.net/?x-ms-routing-name=staging```

By default, new slots are given a routing rule of 0%.

# Knowledge check

1. By default, all client requests to the app's production URL (http://<app_name>.azurewebsites.net) are routed to the production slot. One can automatically route a portion of the traffic to another slot. What is the default routing rule applied to new deployment slots?  0%

2. Some configuration elements follow the content across a swap (not slot specific), whereas other configuration elements stay in the same slot after a swap (slot specific). Which of the settings below are swapped? WebJobs content