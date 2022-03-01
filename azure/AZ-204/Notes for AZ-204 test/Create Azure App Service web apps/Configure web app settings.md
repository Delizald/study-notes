# Objetives
After completing this module, you'll be able to:

- Create application settings that are bound to deployment slots.
- Explain the options for installing SSL/TLS certificates for your app.
- Enable diagnostic logging for your app to aid in monitoring and debugging.
- Create virtual app to directory mappings.

# Configure application settings
app settings are variables passed as environment variables to the application code. 

For Linux apps and custom containers, App Service passes app settings to the container using the --env flag to set the environment variable in the container.

Application settings can be accessed by navigating to your app's management page and selecting **Configuration > Application Settings**.

For ASP.NET and ASP.NET Core developers, setting app settings in App Service are like setting them in <appSettings> in Web.config or appsettings.json

values in App Service override the ones in Web.config or appsettings.json

You can keep development settings (for example, local MySQL password) in Web.config or appsettings.json, but production secrets (for example, Azure MySQL database password) safe in App Service.

## Adding and editing settings

To add a new app setting, click New application setting. If you are using deployment slots you can specify if your setting is swappable or no. To edit a setting, click the Edit button on the right side. When finished, click Update. Don't forget to click Save back in the Configuration page.

**In a default, or custom, Linux container any nested JSON key structure in the app setting name like ApplicationInsights:InstrumentationKey needs to be configured in App Service as ApplicationInsights__InstrumentationKey for the key name. In other words, any : should be replaced by __ (double underscore).**

## Editing application settings in bulk

To add or edit app settings in bulk, click the Advanced edit button. When finished, click Update. App settings have the following JSON formatting:
```
[
  {
    "name": "<key-1>",
    "value": "<value-1>",
    "slotSetting": false
  },
  {
    "name": "<key-2>",
    "value": "<value-2>",
    "slotSetting": false
  },
  ...
]
```
## Configure connection strings

**For ASP.NET and ASP.NET Core developers the values you set in App Service override the ones in Web.config.**

Connection strings are always encrypted when stored (encrypted-at-rest).

**There is one case where you may want to use connection strings instead of app settings for non-.NET languages: certain Azure database types are backed up along with the app only if you configure a connection string for the database in your App Service app.**

Below is an example of connection strings in JSON formatting that you would use for bulk adding or editing.
```
[
  {
    "name": "name-1",
    "value": "conn-string-1",
    "type": "SQLServer",
    "slotSetting": false
  },
  {
    "name": "name-2",
    "value": "conn-string-2",
    "type": "PostgreSQL",
    "slotSetting": false
  },
  ...
]
```

## Configure general settings
In the **Configuration > General** settings section you can configure some common settings for your app. Some settings require you to scale up to higher pricing tiers.

- Stack settings: The software stack to run the app, including the language and SDK versions. For Linux apps and custom container apps, you can also set an optional start-up command or file.

- Platform settings: Lets you configure settings for the hosting platform, including:
  - Bitness: 32-bit or 64-bit.
  - WebSocket protocol: For ASP.NET SignalR or socket.io, for example.
  - Always On: Keep the app loaded even when there's no traffic. By default, Always On is not enabled and the app is unloaded after 20 minutes without any incoming requests. It's required for continuous WebJobs or for WebJobs that are triggered using a CRON expression.
  - Managed pipeline version: The IIS pipeline mode. Set it to Classic if you have a legacy app that requires an older version of IIS.
  - HTTP version: Set to 2.0 to enable support for HTTPS/2 protocol.
  - ARR affinity: In a multi-instance deployment, ensure that the client is routed to the same instance for the life of the session. You can set this option to Off for stateless applications.
  
- Debugging: Enable remote debugging for ASP.NET, ASP.NET Core, or Node.js apps. This option turns off automatically after 48 hours.

- Incoming client certificates: require client certificates in mutual authentication. TLS mutual authentication is used to restrict access to your app by enabling different types of authentication for it.

## Configure path mappings
In the **Configuration > Path mappings** section you can configure handler mappings, and virtual application and directory mappings. The Path mappings page will display different options based on the OS type

### Windows apps (uncontainerized)
you can customize the IIS handler mappings and virtual applications and directories.

Handler mappings let you add custom script processors to handle requests for specific file extensions.

- Extension: The file extension you want to handle, such as *.php or handler.fcgi.
- Script processor: The absolute path of the script processor. Requests to files that match the file extension are processed by the script processor. Use the path **D:\home\site\wwwroot** to refer to your app's root directory.
- Arguments: Optional command-line arguments for the script processor.

You can configure virtual applications and directories by specifying each virtual directory and its corresponding physical path relative to the website root (D:\home). To mark a virtual directory as a web application, clear the **Directory** check box.

## Linux and containerized apps
Containerized apps include all Linux apps and also the Windows and Linux custom containers running on App Service. Click **New Azure Storage Mount** and configure your custom storage as follows:

- Name: The display name.
- Configuration options: Basic or Advanced.
- Storage accounts: The storage account with the container you want.
- Storage type: Azure Blobs or Azure Files. Windows container apps only support Azure Files.
- Storage container: For basic configuration, the container you want.
- Share name: For advanced configuration, the file share name.
- Access key: For advanced configuration, the access key.
- Mount path: The absolute path in your container to mount the custom storage.

## Enable diagnostic logging

### logging types.
- Application logging: (Windows, Linux). Located at App Service file system and/or Azure Storage blobs. Logs messages generated by your application code. Each message is assigned one of the following categories: Critical, Error, Warning, Info, Debug, and Trace.
- Web server logging: (Windows).Location:  App Service file system or Azure Storage blobs. Raw HTTP request data in the W3C extended log file format ( HTTP method, resource URI, client IP, client port, etc).
- Detailed error logging (Windows): App Service file system. Copies of the .htm error pages that would have been sent to the client browser. App Service can save the error page each time an application error occurs that has HTTP code 400 or greater.
- Failed request tracing (Windows): App Service file system. Detailed tracing information on failed requests. One folder is generated for each failed request, which contains the XML log file, and the XSL stylesheet to view the log file with.
- Deployment logging: (Windows, Linux): App Service file system. Helps determine why a deployment failed. Happens automatically.

### Enable application logging (Windows).

- navigate to your app and select **App Service logs**.
- Select On for either (or both): 
  - Application Logging (Filesystem) (for temporary debugging purposes, and turns itself off in 12 hours)
  - Application Logging (Blob) (for long-term logging, and needs a blob storage container to write logs to.)

### Log level

- Disabled: None
- Error: Error, Critical
- Warning: 	Warning, Error, Critical
- Information: Info, Warning, Error, Critical
- Verbose: Trace, Debug, Info, Warning, Error, Critical (all categories)


### Enable application logging (Linux/Container)
- In App Service logs set the Application logging option to File System.
- In Quota (MB), specify the disk quota for the application logs. In Retention Period (Days), set the number of days the logs should be retained.

### Enable web server logging
- For Web server logging, select Storage to store logs on blob storage, or File System to store logs on the App Service file system.

- In Retention Period (Days), set the number of days the logs should be retained.

### Add log messages in code

By default, ASP.NET Core uses the `Microsoft.Extensions.Logging.AzureAppServices` logging provider.

ASP.NET applications can use the `System.Diagnostics.Trace` class to log information to the application diagnostics log. For example:
`System.Diagnostics.Trace.TraceError("If you're seeing this, something bad happened");`

### Stream logs
Any information written to files ending in .txt, .log, or .htm that are stored in the `/LogFiles` directory `(d:/home/logfiles)` is streamed by App Service.

**Some types of logging buffer write to the log file, which can result in out of order events in the stream. For example, an application log entry that occurs when a user visits a page may be displayed in the stream before the corresponding HTTP log entry for the page request.**

- **Azure portal**: To stream logs in the Azure portal, navigate to your app and select **Log stream**.
- **Azure CLI**: `az webapp log tail --name appname --resource-group myResourceGroup`

- **Local console**: To stream logs in the local console, install Azure CLI and sign in to your account. Once signed in, follow the instructions for Azure CLI above.

### Access log files
For logs stored in the App Service file system, the easiest way is to download the ZIP file in the browser at:

- **Linux/container**: https://<app-name>.scm.azurewebsites.net/api/logs/docker/zip
- **Windows apps** https://<app-name>.scm.azurewebsites.net/api/dump

For Linux/container apps, the ZIP file contains console output logs for both the docker host and the docker container. 

For a scaled-out app, the ZIP file contains one set of logs for each instance.
In the App Service file system, these log files are the contents of the /home/LogFiles directory.


## Configure security certificates

Options you have for adding certificates in App Service:

- **Create a free App Service managed certificate**: A private certificate that's free of charge and easy to use if you just need to secure your custom domain in App Service.
- **Purchase an App Service certificate**: A private certificate that's managed by Azure. It combines the simplicity of automated certificate management and the flexibility of renewal and export options.
- **Import a certificate from Key Vault**: Useful if you use Azure Key Vault to manage your certificates. 
- **Upload a private certificate**: If you already have a private certificate from a third-party provider, you can upload it.
- **Upload a public certificate**: Public certificates are not used to secure custom domains, but you can load them into your code if you need them to access remote resources.

## Private certificate requirements
- Exported as a password-protected PFX file, encrypted using triple DES.
- Contains private key at least 2048 bits long.
- Contains all intermediate certificates in the certificate chain.

To secure a custom domain in a TLS binding, the certificate has additional requirements:

- Contains an Extended Key Usage for server authentication (OID = 1.3.6.1.5.5.7.3.1).
- Signed by a trusted certificate authority.

## Creating a free managed certificate
Custom SSL is not supported in the F1 or D1 tier.

To create custom SSL your App Service plan must be in the Basic, Standard, Premium, or Isolated tier.

The free App Service managed certificate it is fully managed by App Service and renewed continuously and automatically in six-month increments, 45 days before expiration. You create the certificate and bind it to a custom domain, and let App Service do the rest.

## Free cert limitations:

Does not support wildcard certificates.
Does not support usage as a client certificate by certificate thumbprint.
Is not exportable.
Is not supported on App Service Environment (ASE).
Is not supported with root domains that are integrated with Traffic Manager.
If a certificate is for a CNAME-mapped domain, the CNAME must be mapped directly to `app-name.azurewebsites.net`.

## Import an App Service Certificate
If you purchase an App Service Certificate from Azure, Azure manages the following tasks:

- Takes care of the purchase process from GoDaddy.
- Performs domain verification of the certificate.
- Maintains the certificate in Azure Key Vault.
- Manages certificate renewal.
- Synchronize the certificate automatically with the imported copies in App Service apps.

## Upload a private certificate

If you generated your certificate request using OpenSSL, then you have created a private key file. To export your certificate to PFX, run the following command:

`openssl pkcs12 -export -out myserver.pfx -inkey <private-key-file> -in <merged-certificate-file>`

When prompted, define an export password. You'll use this password when uploading your TLS/SSL certificate to App Service.

## Enforce HTTPS
navigate to your app page and, in the left navigation, select **TLS/SSL** settings. Then, in **HTTPS Only**, select **On**.

# Manage app features
Feature management uses a technique called feature flags (also known as feature toggles, feature switches, and so on) to dynamically administer a feature's lifecycle.

## Basic concepts
- **Feature flag**: A feature flag is a variable with a binary state of on or off. The feature flag also has an associated code block. The state of the feature flag triggers whether the code block runs or not.
- **Feature manager**: A feature manager is an application package that handles the lifecycle of all the feature flags in an application. The feature manager typically provides additional functionality, such as caching feature flags and updating their states.
- **Filter**: A filter is a rule for evaluating the state of a feature flag. A user group, a device or browser type, a geographic location, and a time window are all examples of what a filter can represent.

An effective implementation of feature management consists of at least:
- An application that makes use of feature flags.
- A separate repository that stores the feature flags and their current states.

## Feature flag usage in code
```
`if (featureFlag) {
  // Run the following code
}`
```

## Feature flag declaration
When a feature flag has multiple filters, the filter list is traversed in order until one of the filters determines the feature should be enabled. 
The feature manager supports appsettings.json as a configuration source for feature flags. The following example shows how to set up feature flags in a JSON file:

```
"FeatureManagement": {
    "FeatureA": true, // Feature flag set to on
    "FeatureB": false, // Feature flag set to off
    "FeatureC": {
        "EnabledFor": [
            {
                "Name": "Percentage",
                "Parameters": {
                    "Value": 50
                }
            }
        ]
    }
}
```

## Feature flag repository
To use feature flags effectively, you need to externalize all the feature flags used in an application.

## Knowledge check
- In which of the app configuration settings categories below would you set the language and SDK version? General settings (This category is used to configure stack, platform, debugging, and incoming client certificate settings.)
- Which of the following types of application logging is supported on the Linux platform? Deployment logging
- Which of the following choices correctly lists the two parts of a feature flag?
- Name, one or more filters


