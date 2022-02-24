- Explain the differences between the two types of managed identities
- Describe the flows for user- and system-assigned managed identities
- Configure managed identities
- Acquire access tokens by using REST and code

# Explore managed identities
Managed identities provide an identity for applications to use when connecting to resources that support Azure Active Directory (Azure AD) authentication. 

## Types of managed identities

two types of managed identities:

A system-assigned managed identity is enabled directly on an Azure service instance. When the identity is enabled, Azure creates an identity for the instance in the Azure AD tenant that's trusted by the subscription of the instance. The lifecycle of a system-assigned identity is directly tied to the Azure service instance that it's enabled on. If the instance is deleted, Azure automatically cleans up the credentials and the identity in Azure AD. 

A user-assigned managed identity is created as a standalone Azure resource. Through a create process, Azure creates an identity in the Azure AD tenant that's trusted by the subscription in use. The lifecycle of a user-assigned identity is managed separately from the lifecycle of the Azure service instances to which it's assigned.

## When to use managed identities

(see pic)

## What Azure services support managed identities?

https://docs.microsoft.com/en-us/azure/active-directory/managed-identities-azure-resources/services-support-msi

# Discover the managed identities authentication flow

## How a system-assigned managed identity works with an Azure virtual machine

Azure Resource Manager receives a request to enable the system-assigned managed identity on a virtual machine.

Azure Resource Manager creates a service principal in Azure Active Directory for the identity of the virtual machine. The service principal is created in the Azure Active Directory tenant that's trusted by the subscription.

Azure Resource Manager configures the identity on the virtual machine by updating the Azure Instance Metadata Service identity endpoint with the service principal client ID and certificate.

After the virtual machine has an identity, use the service principal information to grant the virtual machine access to Azure resources. To call Azure Resource Manager, use role-based access control in Azure Active Directory to assign the appropriate role to the virtual machine service principal. To call Key Vault, grant your code access to the specific secret or key in Key Vault.

Your code that's running on the virtual machine can request a token from the Azure Instance Metadata service endpoint, accessible only from within the virtual machine: http://169.254.169.254/metadata/identity/oauth2/token

A call is made to Azure Active Directory to request an access token (as specified in step 5) by using the client ID and certificate configured in step 3. Azure Active Directory returns a JSON Web Token (JWT) access token.

Your code sends the access token on a call to a service that supports Azure Active Directory authentication.

## How a user-assigned managed identity works with an Azure virtual machine

Azure Resource Manager receives a request to create a user-assigned managed identity.

Azure Resource Manager creates a service principal in Azure Active Directory for the user-assigned managed identity. The service principal is created in the Azure Active Directory tenant that's trusted by the subscription.

Azure Resource Manager receives a request to configure the user-assigned managed identity on a virtual machine and updates the Azure Instance Metadata Service identity endpoint with the user-assigned managed identity service principal client ID and certificate.

After the user-assigned managed identity is created, use the service principal information to grant the identity access to Azure resources. To call Azure Resource Manager, use role-based access control in Azure Active Directory to assign the appropriate role to the service principal of the user-assigned identity. To call Key Vault, grant your code access to the specific secret or key in Key Vault.

Your code that's running on the virtual machine can request a token from the Azure Instance Metadata Service identity endpoint, accessible only from within the virtual machine: http://169.254.169.254/metadata/identity/oauth2/token

A call is made to Azure Active Directory to request an access token (as specified in step 5) by using the client ID and certificate configured in step 3. Azure Active Directory returns a JSON Web Token (JWT) access token.

Your code sends the access token on a call to a service that supports Azure Active Directory authe

## Configure managed identities

You can configure an Azure virtual machine with a managed identity during, or after, the creation of the virtual machine.

## Enable system-assigned managed identity during creation of an Azure virtual machine

```
az vm create --resource-group myResourceGroup \ 
    --name myVM --image win2016datacenter \ 
    --generate-ssh-keys \ 
    --assign-identity \ 
    --admin-username azureuser \ 
    --admin-password myPassword12
```

## Enable system-assigned managed identity on an existing Azure virtual machine

Use `az vm identity assign` command enable the system-assigned identity to an existing virtual machine:
`az vm identity assign -g myResourceGroup -n myVm`

## User-assigned managed identity
Create a user-assigned managed identity using az identity create. The -g parameter specifies the resource group where the user-assigned managed identity is created, and the -n parameter specifies its name.

`az identity create -g myResourceGroup -n myUserAssignedIdentity`

## Assign a user-assigned managed identity during the creation of an Azure virtual machine

```
az vm create \
--resource-group <RESOURCE GROUP> \
--name <VM NAME> \
--image UbuntuLTS \
--admin-username <USER NAME> \
--admin-password <PASSWORD> \
--assign-identity <USER ASSIGNED IDENTITY NAME>
```

## Assign a user-assigned managed identity to an existing Azure virtual machine

```
az vm identity assign \
    -g <RESOURCE GROUP> \
    -n <VM NAME> \
    --identities <USER ASSIGNED IDENTITY>
```

## Azure SDKs with managed identities for Azure resources support

https://github.com/Azure-Samples/aad-dotnet-manage-resources-from-vm-with-msi
https://github.com/Azure-Samples/compute-java-manage-resources-from-vm-with-msi-in-aad-group
https://azure.microsoft.com/resources/samples/compute-node-msi-vm/
https://azure.microsoft.com/resources/samples/compute-python-msi-vm/
https://github.com/Azure-Samples/compute-ruby-msi-vm/

## Acquire an access token

The token is based on the managed identities for Azure resources service principal. As such, there is no need for the client to register itself to obtain an access token under its own service principal. The token is suitable for use as a bearer token in service-to-service calls requiring client credentials.

**All sample code/script in this unit assumes the client is running on a virtual machine with managed identities for Azure resources.**

## Acquire a token
Based on REST
```
GET 'http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https://management.azure.com/' HTTP/1.1 Metadata: true
```

Metadata: An HTTP request header field, required by managed identities for Azure resources as a mitigation against Server Side Request Forgery (SSRF) attack. This value must be set to "true", in all lower case.

Samples response:

```
HTTP/1.1 200 OK
Content-Type: application/json
{
  "access_token": "eyJ0eXAi...",
  "refresh_token": "",
  "expires_in": "3599",
  "expires_on": "1506484173",
  "not_before": "1506480273",
  "resource": "https://management.azure.com/",
  "token_type": "Bearer"
}
```

## Get a token by using C#
```
using System;
using System.Collections.Generic;
using System.IO;
using System.Net;
using System.Web.Script.Serialization; 

// Build request to acquire managed identities for Azure resources token
HttpWebRequest request = (HttpWebRequest)WebRequest.Create("http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https://management.azure.com/");
request.Headers["Metadata"] = "true";
request.Method = "GET";

try
{
    // Call /token endpoint
    HttpWebResponse response = (HttpWebResponse)request.GetResponse();

    // Pipe response Stream to a StreamReader, and extract access token
    StreamReader streamResponse = new StreamReader(response.GetResponseStream()); 
    string stringResponse = streamResponse.ReadToEnd();
    JavaScriptSerializer j = new JavaScriptSerializer();
    Dictionary<string, string> list = (Dictionary<string, string>) j.Deserialize(stringResponse, typeof(Dictionary<string, string>));
    string accessToken = list["access_token"];
}
catch (Exception e)
{
    string errorText = String.Format("{0} \n\n{1}", e.Message, e.InnerException != null ? e.InnerException.Message : "Acquire token failed");
}
```

## Token caching
While the managed identities for Azure resources subsystem does cache tokens, we also recommend to implement token caching in your code. As a result, you should prepare for scenarios where the resource indicates that the token is expired.

On-the-wire calls to Azure Active Directory result only when:

Cache miss occurs due to no token in the managed identities for Azure resources subsystem cache.
The cached token is expired.

## Retry guidance

It is recommended to retry if you receive a 404, 429, or 5xx error code. Throttling limits apply to the number of calls made to the IMDS endpoint. When the throttling threshold is exceeded, IMDS endpoint limits any further requests while the throttle is in effect. During this period, the IMDS endpoint will return the HTTP status code 429 ("Too many requests"), and the requests fail.

## Knowledge check

Which of the following characteristics is indicative of user-assigned identities? 
Independent life-cycle

A client app requests managed identities for an access token for a given resource. Which of the below is the basis for the token? Service principal

