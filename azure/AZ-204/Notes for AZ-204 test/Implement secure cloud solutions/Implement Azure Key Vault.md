# Objectives

- Describe the benefits of using Azure Key Vault
- Explain how to authenticate to Azure Key Vault
- Set and retrieve a secret from Azure Key Vault by using the Azure CLI

# Explore AKV

supports two types of containers: vaults and managed hardware security module(HSM) pools.

Azure Key Vault has two service tiers: Standard, which encrypts with a software key, and a Premium tier, which includes hardware security module(HSM)-protected keys. (https://azure.microsoft.com/pricing/details/key-vault/)

helps solve the following problems:

Secrets Management: Azure Key Vault can be used to Securely store and tightly control access to tokens, passwords, certificates, API keys, and other secrets

Key Management: Azure Key Vault can also be used as a Key Management solution. Azure Key Vault makes it easy to create and control the encryption keys used to encrypt your data.

Certificate Management: Azure Key Vault is also a service that lets you easily provision, manage, and deploy public and private Secure Sockets Layer/Transport Layer Security (SSL/TLS) certificates for use with Azure and your internal connected resources.

## Key benefits of using Azure Key Vault

Centralized application secrets: Your applications can securely access the information they need by using URIs. These URIs allow the applications to retrieve specific versions of a secret.

Securely store secrets and keys: Access to a key vault requires proper authentication and authorization before a caller (user or application) can get access. Authentication is done via Azure Active Directory. Authorization may be done via Azure role-based access control (Azure RBAC) or Key Vault access policy.

Monitor access and use: You can monitor activity by enabling logging for your vaults. Key Vault can be configured to:

Archive to a storage account.
Stream to an event hub.
Send the logs to Azure Monitor logs.

Simplified administration of application secrets:
 Removing the need for in-house knowledge of Hardware Security Modules
Scaling up on short notice to meet your organization’s usage spikes.
Replicating the contents of your Key Vault within a region and to a secondary region. Data replication ensures high availability and takes away the need of any action from the administrator to trigger the failover.
Providing standard Azure administration options via the portal, Azure CLI and PowerShell.
Automating certain tasks on certificates that you purchase from Public CAs, such as enrollment and renewal.

## Discover Azure Key Vault best practice

A vault is logical group of secrets.

## Authentication

three ways to authenticate to Key Vault:

Managed identities for Azure resources: When you deploy an app on a virtual machine in Azure, you can assign an identity to your virtual machine that has access to Key Vault. You can also assign identities to other Azure resources.

Service principal and certificate: You can use a service principal and an associated certificate that has access to Key Vault. **We don't recommend this approach because the application owner or developer must rotate the certificate.**

Service principal and secret: Although you can use a service principal and a secret to authenticate to Key Vault, we don't recommend it. It's hard to automatically rotate the bootstrap secret that's used to authenticate to Key Vault.

## Encryption of data in transit

Perfect Forward Secrecy (PFS) protects connections between customers’ client systems and Microsoft cloud services by unique keys. Connections also use RSA-based 2,048-bit encryption key lengths. This combination makes it difficult for someone to intercept and access data that is in transit.

## Azure Key Vault best practices

Use separate key vaults: Recommended to use a vault per application per environment (Development, Pre-Production and Production). This helps you not share secrets across environments and also reduces the threat in case of a breach.

Control access to your vault: Key Vault data is sensitive and business critical, you need to secure access to your key vaults by allowing only authorized applications and users.

Backup: Create regular back ups of your vault on update/delete/create of objects within a Vault.

Logging: Be sure to turn on logging and alerts.

Recovery options: Turn on soft-delete (https://docs.microsoft.com/en-us/azure/key-vault/general/soft-delete-overview) and purge protection if you want to guard against force deletion of the secret.

## Authenticate to Azure Key Vault
**It is recommended to use a system-assigned managed identity.**
For applications, there are two ways to obtain a service principal:

Enable a system-assigned managed identity for the application. With managed identity, Azure internally manages the application's service principal and automatically authenticates the application with other Azure services.

If you cannot use managed identity, you instead register the application with your Azure AD tenant. Registration also creates a second application object that identifies the app across all tenants.

## Authentication to Key Vault in application code
Key Vault SDK is using Azure Identity client library, which allows seamless authentication to Key Vault across environments
https://docs.microsoft.com/en-us/dotnet/api/overview/azure/identity-readme
https://docs.microsoft.com/en-us/python/api/overview/azure/identity-readme
https://docs.microsoft.com/en-us/java/api/overview/azure/identity-readme
https://docs.microsoft.com/en-us/javascript/api/overview/azure/identity-readme

## Authentication to Key Vault with REST

Access tokens must be sent to the service using the HTTP Authorization header:
```
PUT /keys/MYKEY?api-version=<api_version>  HTTP/1.1  
Authorization: Bearer <access_token>
```
When an access token is not supplied, or when a token is not accepted by the service, an HTTP 401 error will be returned to the client and will include the WWW-Authenticate header, for example:
```
401 Not Authorized  
WWW-Authenticate: Bearer authorization="…", resource="…"
```

The parameters on the WWW-Authenticate header are:
authorization: The address of the OAuth2 authorization service that may be used to obtain an access token for the request.

resource: The name of the resource (https://vault.azure.net) to use in the authorization request.

(https://docs.microsoft.com/en-us/azure/key-vault/general/developers-guide)
(https://docs.microsoft.com/en-us/azure/key-vault/general/disaster-recovery-guidance)

# Exercise: Set and retrieve a secret from Azure Key Vault by using Azure CLI

(https://docs.microsoft.com/en-us/learn/modules/implement-azure-key-vault/5-set-retrieve-secret-azure-key-vault)

## Create a Key Vault
```
myKeyVault=az204vault-$RANDOM
myLocation=<myLocation>
```
Create a resource group.
```
az group create --name az204-vault-rg --location $myLocation
```
Create a Key Vault by using the az keyvault create command.
```
az keyvault create --name $myKeyVault --resource-group az204-vault-rg --location $myLocation
```
## Add and retrieve a secret
Create a secret. 
```
az keyvault secret set --vault-name $myKeyVault --name "ExamplePassword" --value "hVFkk965BuUv"
```
Use the az keyvault secret show command to retrieve the secret.
```
az keyvault secret show --name "ExamplePassword" --vault-name $myKeyVault
```

# Knowledge check
Which of the below methods of authenticating to Azure Key Vault is recommended for most scenarios? Managed identities

Azure Key Vault protects data when it is traveling between Azure Key Vault and clients. What protocol does it use for encryption?
Transport Layer Security