# Learning objectives

- Identify the components of the Microsoft identity platform.
- Describe the three types of service principals and how they relate to application objects.
- Explain how permissions and user consent operate, and how conditional access impacts your application.

# Explore the Microsoft identity platform
There are several components that make up the Microsoft identity platform:

OAuth 2.0 and OpenID Connect standard-compliant authentication service enabling developers to authenticate several identity types, including:

  Work or school accounts, provisioned through Azure Active Directory
  Personal Microsoft account, like Skype, Xbox, and Outlook.com
  Social or local accounts, by using Azure Active Directory B2C
  Open-source libraries: Microsoft Authentication Libraries (MSAL) and support for other standards-compliant libraries

Application management portal: A registration and configuration experience in the Azure portal, along with the other Azure management capabilities.

Application configuration API and PowerShell: Programmatic configuration of your applications through the Microsoft Graph API and PowerShell so you can automate your DevOps tasks.

# Explore service principals
To delegate Identity and Access Management functions to Azure Active Directory, an application must be registered with an Azure Active Directory tenant. When you register an app in the Azure portal, you choose whether it is:

- Single tenant: only accessible in your tenant
- Multi-tenant: accessible in other tenants

If you register an application in the portal, an application object (the globally unique instance of the app) as well as a service principal object are automatically created in your home tenant. You also have a globally unique ID for your app

## Application object

An Azure Active Directory application is defined by its one and only application object, which resides in the Azure Active Directory tenant where the application was registered.

The application object describes three aspects of an application: how the service can issue tokens in order to access the application, resources that the application might need to access, and the actions that the application can take.

The Microsoft Graph `Application entity` (https://docs.microsoft.com/en-us/graph/api/resources/application) defines the schema for an application object's properties.

## Service principal object
The security principal defines the access policy and permissions for the user/application in the Azure Active Directory tenant. 

There are three types of service principal:

Application - The type of service principal is the local representation, or application instance, of a global application object in a single tenant or directory. A service principal is created in each tenant where the application is used and references the globally unique app object.

Managed identity - This type of service principal is used to represent a managed identity. https://docs.microsoft.com/en-us/azure/active-directory/managed-identities-azure-resources/overview managed identities provide an identity for applications to use when connecting to resources that support Azure Active Directory authentication.

Legacy - This type of service principal represents a legacy app, which is an app created before app registrations were introduced or an app created through legacy experiences. 

## Relationship between application objects and service principals

The application object is the global representation of your application for use across all tenants

the service principal is the local representation for use in a specific tenant.

An application object has:

- A 1:1 relationship with the software application, and
- A 1:many relationship with its corresponding service principal object(s).

A service principal must be created in each tenant where the application is used, enabling it to establish an identity for sign-in and/or access to resources being secured by the tenant. 

# Discover permissions and consent

The Microsoft identity platform implements the OAuth 2.0 authorization protocol. OAuth 2.0 is a method through which a third-party app can access web-hosted resources on behalf of a user.

Here are some examples of Microsoft web-hosted resources:

- Microsoft Graph: https://graph.microsoft.com
- Microsoft 365 Mail API: https://outlook.office.com
- Azure Key Vault: https://vault.azure.net

When a resource's functionality is chunked into small permission sets, third-party apps can be built to request only the permissions that they need to perform their function. Users and administrators can know what data the app can access.

In OAuth 2.0, these types of permission sets are called scopes.  often referred to as permissions.

. An app requests the permissions it needs by specifying the permission in the scope query parameter. (https://docs.microsoft.com/en-us/azure/active-directory/develop/v2-permissions-and-consent#openid-connect-scopes)


some high-privilege permissions can be granted only through administrator consent. They can be requested or granted by using the administrator consent endpoint. (https://docs.microsoft.com/en-us/azure/active-directory/develop/v2-permissions-and-consent#admin-restricted-permissions)

**In requests to the authorization, token or consent endpoints for the Microsoft Identity platform, if the resource identifier is omitted in the scope parameter, the resource is assumed to be Microsoft Graph. For example, scope=User.Read is equivalent to https://graph.microsoft.com/User.Read.**

## Permission types

- Delegated permissions are used by apps that have a signed-in user present. For these apps, either the user or an administrator consents to the permissions that the app requests. The app is delegated with the permission to act as a signed-in user when it makes calls to the target resource.

- Application permissions are used by apps that run without a signed-in user present, for example, apps that run as background services or daemons. Only an administrator can consent to application permissions.

## Consent types

Applications in Microsoft identity platform rely on consent in order to gain access to necessary resources or APIs

There are three consent types: static user consent,
incremental and dynamic user consent, 
and admin consent.

## Static user consent

you must specify all the permissions it needs in the app's configuration in the Azure portal. If the user (or administrator, as appropriate) has not granted consent for this app.

it presents some possible issues for developers:

- The app needs to request all the permissions it would ever need upon the user's first sign-in. This can lead to a long list of permissions that discourages end users from approving the app's access on initial sign-in.

- The app needs to know all of the resources it would ever access ahead of time. It is difficult to create apps that could access an arbitrary number of resources.

## Incremental and dynamic user consent

You can ask for a minimum set of permissions upfront and request more over time as the customer uses additional app features.
To do so, you can specify the scopes your app needs at any time by including the new scopes in the `scope` parameter when requesting an access token 

**Dynamic consent can be convenient, but presents a big challenge for permissions that require admin consent, since the admin consent experience doesn't know about those permissions at consent time. If you require admin privileged permissions or if your app uses dynamic consent, you must register all of the permissions in the Azure portal (not just the subset of permissions that require admin consent). This enables tenant admins to consent on behalf of all their users.**

## Admin consent

Admin consent ensures that administrators have some additional controls before authorizing apps or users to access highly privileged data from the organization. Set those permissions for apps in the app registration portal if you need an admin to give consent on behalf of the entire organization. This reduces the cycles required by the organization admin to set up the application.

## Requesting individual user consent

In an OpenID Connect or OAuth 2.0 authorization request, an app can request the permissions it needs by using the scope query parameter. For example, when a user signs in to an app, the app sends a request like the following example. Line breaks are added for legibility.

```
GET https://login.microsoftonline.com/common/oauth2/v2.0/authorize?
client_id=6731de76-14a6-49ae-97bc-6eba6914391e
&response_type=code
&redirect_uri=http%3A%2F%2Flocalhost%2Fmyapp%2F
&response_mode=query
&scope=
https%3A%2F%2Fgraph.microsoft.com%2Fcalendars.read%20
https%3A%2F%2Fgraph.microsoft.com%2Fmail.send
&state=12345
```

# Discover conditional access

- Multifactor authentication
- Allowing only Intune enrolled devices to access specific services
- Restricting user locations and IP ranges

## How does Conditional Access impact an app?

Specifically, the following scenarios require code to handle Conditional Access challenges:
- Apps performing the on-behalf-of flow
- Apps accessing multiple services/resources
- Single-page apps using MSAL.js
- Web apps calling a resource

## Conditional Access examples

You are building a single-tenant iOS app and apply a Conditional Access policy. The app signs in a user and doesn't request access to an API. When the user signs in, the policy is automatically invoked and the user needs to perform multifactor authentication.

You are building a native app that uses a middle tier service to access a downstream API. An enterprise customer at the company using this app applies a policy to the downstream API. When an end user signs in, the native app requests access to the middle tier and sends the token. The middle tier performs on-behalf-of flow to request access to the downstream API. At this point, a claims "challenge" is presented to the middle tier. The middle tier sends the challenge back to the native app, which needs to comply with the Conditional Access policy.

## Knowledge check

Which of the types of permissions supported by the Microsoft identity platform is used by apps that have a signed-in user present? Delegated permissions

Which of the following app scenarios require code to handle Conditional Access challenges? Apps performing the on-behalf-of flow