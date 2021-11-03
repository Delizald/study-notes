In App Service, app settings are variables passed as environment variables to the application code.

 For Linux apps and custom containers, App Service passes app settings to the container using the --env flag to set the environment variable in the container.

 Application settings can be accessed by navigating to your app's management page and selecting Configuration > Application Settings.

 For ASP.NET and ASP.NET Core developers, setting app settings in App Service are like setting them in <appSettings> in Web.config or appsettings.json, but the values in App Service override the ones in Web.config or appsettings.json

 App settings are always encrypted when stored (encrypted-at-rest).

 To add a new app setting, click New application setting. If you are using deployment slots you can specify if your setting is swappable or not. When finished, click Update. Don't forget to click Save back in the Configuration page.

 In a default, or custom, Linux container any nested JSON key structure in the app setting name like ApplicationInsights:InstrumentationKey needs to be configured in App Service as ApplicationInsights__InstrumentationKey for the key name. In other words, any : should be replaced by __ (double underscore).

 To add or edit app settings in bulk, click the Advanced edit button.


 There is one case where you may want to use connection strings instead of app settings for non-.NET languages: certain Azure database types are backed up along with the app only if you configure a connection string for the database in your App Service app

 Adding and editing connection strings follow the same principles as other app settings and they can also be tied to deployment slots. Below is an example of connection strings in JSON formatting that you would use for bulk adding or editing.


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



## Configure general settings

Stack settings: The software stack to run the app, including the language and SDK versions. For Linux apps and custom container apps, you can also set an optional start-up command or file.

Platform settings: Lets you configure settings for the hosting platform, including:
  Bitness: 32-bit or 64-bit.

  WebSocket protocol: For ASP.NET SignalR or socket.io, for example.

  Always On: Keep the app loaded even when there's no traffic. By default, Always On is not enabled and the app is unloaded after 20 minutes without any incoming requests. It's required for continuous WebJobs or for WebJobs that are triggered using a CRON expression.

  Managed pipeline version: The IIS pipeline mode. Set it to Classic if you have a legacy app that requires an older version of IIS.

  HTTP version: Set to 2.0 to enable support for HTTPS/2 protocol.

  ARR affinity: In a multi-instance deployment, ensure that the client is routed to the same instance for the life of the session. You can set this option to Off for stateless applications.

  Debugging: Enable remote debugging for ASP.NET, ASP.NET Core, or Node.js apps. This option turns off automatically after 48 hours.

  Incoming client certificates: require client certificates in mutual authentication. TLS mutual authentication is used to restrict access to your app by enabling different types of authentication for it.

  ## Configure path mappings

  Configuration => Path mappings section you can configure handler mappings, and virtual application and directory mappings. The Path mappings page will display different options based on the OS type.

### Windows apps (uncontainerized)

Handler mappings let you add custom script processors to handle requests for specific file extensions. To add a custom handler, click New handler. Configure the handler as follows:

  Extension: The file extension you want to handle, such as *.php or handler.fcgi.
  Script processor: The absolute path of the script processor to you. Requests to files that match the file extension are processed by the script processor. Use the path D:\home\site\wwwroot to refer to your app's root directory.
  Arguments: Optional command-line arguments for the script processor.

Each app has the default root path (/) mapped to D:\home\site\wwwroot

To configure virtual applications and directories, specify each virtual directory and its corresponding physical path relative to the website root (D:\home). To mark a virtual directory as a web application, clear the Directory check box.

### Linux and containerized apps

Linux and containerized apps
You can add custom storage for your containerized app. Containerized apps include all Linux apps and also the Windows and Linux custom containers running on App Service. Click New Azure Storage Mount and configure your custom storage as follows:

Name: The display name.
Configuration options: Basic or Advanced.
Storage accounts: The storage account with the container you want.
Storage type: Azure Blobs or Azure Files. Windows container apps only support Azure Files.
Storage container: For basic configuration, the container you want.
Share name: For advanced configuration, the file share name.
Access key: For advanced configuration, the access key.
Mount path: The absolute path in your container to mount the custom storage.

#  Enable diagnostic logging

TODO: Resume diagnostig logging table

## Enable application logging (Windows)

To enable application logging for Windows apps in the Azure portal, navigate to your app and select App Service logs.

Select On for either Application Logging (Filesystem) or Application Logging (Blob), or both. The Filesystem option is for temporary debugging purposes, and turns itself off in 12 hours. The Blob option is for long-term logging, and needs a blob storage container to write logs to.


Log level:
  Disabled: Nine
  Error: Error, Critical
  Warning. Warning, Error, Critical
  Information: Info, Warning, Error, Critical
  Verbose: Trace, Debug, Info, Warning. Error: Critical (all categories)


## Enable application logging (Linux/Container)

1) In Application logging, select File System.
2) In Quota (MB), specify the disk quota for the application logs. In Retention Period (Days), set the number of days the logs should be retained.
3) When finished, select Save.

## Enable web server logging

1) For Web server logging, select Storage to store logs on blob storage, or File System to store logs on the App Service file system.

2) In Retention Period (Days), set the number of days the logs should be retained.

3) When finished, select Save.

## Add log messages in code

ASP.NET applications can use the System.Diagnostics.Trace class to log information to the application diagnostics log. For example:

System.Diagnostics.Trace.TraceError("If you're seeing this, something bad happened");

By default, ASP.NET Core uses the Microsoft.Extensions.Logging.AzureAppServices logging provider.

## Stream logs

Before you stream logs in real time, enable the log type that you want. Any information written to files ending in .txt, .log, or .htm that are stored in the /LogFiles directory (d:/home/logfiles) is streamed by App Service.

Azure portal - To stream logs in the Azure portal, navigate to your app and select Log stream.

Azure CLI - To stream logs live in Cloud Shell, use the following command:

  az webapp log tail --name appname --resource-group myResourceGroup

Local console - To stream logs in the local console, install Azure CLI and sign in to your account. Once signed in, follow the instructions for Azure CLI above.

## Access log files

If you configure the Azure Storage blobs option for a log type, you need a client tool that works with Azure Storage.

For logs stored in the App Service file system, the easiest way is to download the ZIP file in the browser at:

Linux/container apps: https://<app-name>.scm.azurewebsites.net/api/logs/docker/zip
Windows apps: https://<app-name>.scm.azurewebsites.net/api/dump

For Linux/container apps, the ZIP file contains console output logs for both the docker host and the docker container.
For a scaled-out app, the ZIP file contains one set of logs for each instance.
n the App Service file system, these log files are the contents of the /home/LogFiles directory.

## Configure security certificates

A certificate uploaded into an app is stored in a deployment unit that is bound to the app service plan's resource group and region combination (internally called a webspace). This makes the certificate accessible to other apps in the same resource group and region combination.