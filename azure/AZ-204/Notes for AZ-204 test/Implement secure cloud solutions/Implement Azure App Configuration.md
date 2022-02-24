- Explain the benefits of using Azure App Configuration
- Describe how Azure App Configuration stores information
- Implement feature management
- Securely access your app configuration information

# Explore

App Configuration offers the following benefits:

- A fully managed service that can be set up in minutes
- Flexible key representations and mappings
- Tagging with labels
- Point-in-time replay of settings
- Dedicated UI for feature flag management
- Comparison of two sets of configurations on custom-defined dimensions
- Enhanced security through Azure-managed identities
- Complete data encryptions, at rest or in transit
- Native integration with popular frameworks

App Configuration makes it easier to implement the following scenarios:

- Centralize management and distribution of hierarchical configuration data for different environments and geographies
- Dynamically change application settings without the need to redeploy or restart an application
- Control feature availability in real-time


## Use App Configuration
.NET Core and ASP.NET Core	App Configuration provider for .NET Core
.NET Framework and ASP.NET	App Configuration builder for .NET
Java Spring	App Configuration client for Spring Cloud
Others	App Configuration REST API

# Create paired keys and values

Azure App Configuration stores configuration data as key-value pairs.

## Keys
it's a common practice to organize keys into a hierarchical namespace by using a character delimiter, such as / or :
Keys stored in App Configuration are case-sensitive, unicode-based strings. 
You can use any unicode character in key names entered into App Configuration except for *, ,, and \. 

## Design key namespaces

two general approaches to naming keys used for configuration data: flat or hierarchical. 

hierarchical naming offers a number of advantages:

Easier to read. Instead of one long sequence of characters, delimiters in a hierarchical key name function as spaces in a sentence.
Easier to manage. A key name hierarchy represents logical groups of configuration data.
Easier to use. It's simpler to write a query that pattern-matches keys in a hierarchical structure and retrieves only a portion of configuration data.

Below are some examples of how you can structure your key names into a hierarchy:

on component services:
```
AppName:Service1:ApiEndpoint
AppName:Service2:ApiEndpoint
```
on deployment regions:
```
AppName:Region1:DbEndpoint
AppName:Region2:DbEndpoint
```

## Label keys

Key values in App Configuration can optionally have a label attribute. Labels are used to differentiate key values with the same key. A key app1 with labels A and B forms two separate keys in an App Configuration store. 

By default, the label for a key value is empty, or null.

```
Key = AppName:DbEndpoint & Label = Test
Key = AppName:DbEndpoint & Label = Staging
Key = AppName:DbEndpoint & Label = Production
```

## Version key values

App Configuration doesn't version key values automatically as they're modified. Use labels as a way to create multiple versions of a key value. . For example, you can input an application version number or a Git commit ID.

## Query key values

Each key value is uniquely identified by its key plus a label that can be null. You query an App Configuration store for key values by specifying a pattern. T

## Values

Values assigned to keys are also unicode strings. You can use all unicode characters for values. There's an optional user-defined content type associated with each value.

Configuration data stored in an App Configuration store, which includes all keys and values, is encrypted at rest and in transit. App Configuration isn't a replacement solution for Azure Key Vault. Don't store application secrets in it.

# Manage application features

Feature management is a modern software-development practice that decouples feature release from code deployment and enables quick changes to feature availability on demand. uses a technique called feature flags.

## Basic concepts
Feature flag: : A feature flag is a variable with a binary state of on or off.
Feature manager: A feature manager is an application package that handles the lifecycle of all the feature flags in an application. 
Filter: A filter is a rule for evaluating the state of a feature flag. A user group, a device or browser type, a geographic location, and a time window.

## Feature flag usage in code

```
if (featureFlag) {
    // Run the following code
}
```

## Feature flag declaration

Each feature flag has two parts: a name and a list of one or more filters that are used to evaluate if a feature's state is on (that is, when its value is True). 

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

To use feature flags effectively, you need to externalize all the feature flags used in an application. Azure App Configuration is designed to be a centralized repository for feature flags. You can use it to define different kinds of feature flags and manipulate their states quickly and confidently.


# Secure app configuration data

## Encrypt configuration data by using customer-managed keys
Azure App Configuration encrypts sensitive information at rest using a 256-bit AES encryption key provided by Microsoft.

## Enable customer-managed key capability

The following components are required to successfully enable the customer-managed key capability for Azure App Configuration:
- Standard tier Azure App Configuration instance
- Azure Key Vault with soft-delete and purge-protection features enabled
- An RSA or RSA-HSM key within the Key Vault: The key must not be expired, it must be enabled, and it must have both wrap and unwrap capabilities enabled.

two steps remain to allow Azure App Configuration to use the Key Vault key:
- Assign a managed identity to the Azure App Configuration instance
- Grant the identity `GET`, `WRAP`, and `UNWRAP`   permissions in the target Key Vault's access policy.

## Use private endpoints for Azure App Configuration

You can use private endpoints for Azure App Configuration.  The private endpoint uses an IP address from the VNet address space for your App Configuration store. Network traffic between the clients on the VNet and the App Configuration store traverses over the VNet using a private link on the Microsoft backbone network, eliminating exposure to the public internet.

This enables to:

- Secure your application configuration details by configuring the firewall to block all connections to App Configuration on the public endpoint.
- Increase security for the virtual network (VNet) ensuring data doesn't escape from the VNet.
- Securely connect to the App Configuration store from on-premises networks that connect to the VNet using VPN or ExpressRoutes with private-peering.

## Private endpoints for App Configuration

When creating a private endpoint, you must specify the App Configuration store to which it connects. If you have multiple App Configuration stores, you need a separate private endpoint for each store.

## DNS changes for private endpoints

When you create a private endpoint, the DNS CNAME resource record for the configuration store is updated to an alias in a subdomain with the prefix privatelink. Azure also creates a private DNS zone corresponding to the privatelink subdomain, with the DNS A resource records for the private endpoints. (https://docs.microsoft.com/en-us/azure/dns/private-dns-overview)

## Managed identities

A managed identity from Azure Active Directory (AAD) allows Azure App Configuration to easily access other AAD-protected resources, such as Azure Key Vault.

Your application can be granted two types of identities:

- A system-assigned identity is tied to your configuration store. It's deleted if your configuration store is deleted. A configuration store can only have one system-assigned identity.
- A user-assigned identity is a standalone Azure resource that can be assigned to your configuration store. A configuration store can have multiple user-assigned identities.

## Add a system-assigned identity

using the Azure CLI, use the `az appconfig identity assign`
```
az appconfig identity assign \ 
    --name myTestAppConfigStore \ 
    --resource-group myResourceGroup
```

## Add a user-assigned identity

Create an identity using the `az identity create` command:
```
az identity create -resource-group myResourceGroup --name myUserAssignedIdentity
```
Assign the new user-assigned identity to the myTestAppConfigStore configuration store:
```
az appconfig identity assign --name myTestAppConfigStore \ 
    --resource-group myResourceGroup \ 
    --identities /subscriptions/[subscription id]/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/myUserAssignedIdentity
```

# Knowledge check

1. Which type of encryption does Azure App Configuration use to encrypt data at rest? 
256-bit AES
2. Which of the below evaluates the state of a feature flag? Filter