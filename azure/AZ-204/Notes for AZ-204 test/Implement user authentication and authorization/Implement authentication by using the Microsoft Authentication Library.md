# Learning objectives
- Explain the benefits of using Microsoft Authentication Library and the application types and scenarios it supports.
- Instantiate both public and confidential client apps from code.
- Register an app with the Microsoft identity platform.
- Create an app that retrieves a token by using the MSAL.NET (http://msal.net/) library.

## Explore the Microsoft Authentication Library

Using MSAL provides the following benefits:

- No need to directly use the OAuth libraries or code against the protocol in your application.
- Acquires tokens on behalf of a user or on behalf of an application (when applicable to the platform).
- Maintains a token cache and refreshes tokens for you when they are close to expire. You don't need to handle token expiration on your own.
- Helps you specify which audience you want your application to sign in.
- Helps you set up your application from configuration files.
- Helps you troubleshoot your app by exposing actionable exceptions, logging, and telemetry.

## Application types and scenarios
(https://docs.microsoft.com/en-us/learn/modules/implement-authentication-by-using-microsoft-authentication-library/2-microsoft-authentication-library-overview)

## Authentication flows

Authorization code:	Native and web apps securely obtain tokens in the name of the user

Client credentials: Service applications run without user interaction

On-behalf-of: The application calls a service/web API, which in turns calls Microsoft Graph

Implicit: Used in browser-based applications

Device code: Enables sign-in to a device by using another device that has a browser

Integrated Windows: Windows computers silently acquire an access token when they are domain joined

Interactive: Mobile and desktops applications call Microsoft Graph in the name of a user

Username/password:	The application signs in a user by using their username and password

## Public client, and confidential client applications

- Public client applications: Are apps that run on devices or desktop computers or in a web browser. They're not trusted to safely keep application secrets, so they only access web APIs on behalf of the user. (They support only public client flows.) Public clients can't hold configuration-time secrets, so they don't have client secrets.

- Confidential client applications: Are apps that run on servers (web apps, web API apps, or even service/daemon apps). They're considered difficult to access, and for that reason capable of keeping an application secret. Confidential clients can hold configuration-time secrets. Each instance of the client has a distinct configuration (including client ID and client secret).

## Initialize client applications

With MSAL.NET 3.x, the recommended way to instantiate an application is by using the application builders: `PublicClientApplicationBuilder` and `ConfidentialClientApplicationBuilder`.

Before initializing an application, you first need to register it so that your app can be integrated with the Microsoft identity platform. After registration, you may need the following information (which can be found in the Azure portal):

- The client ID (a string representing a GUID)
- The identity provider URL (named the instance) and the sign-in audience for your application. These two parameters are collectively known as the authority.
- The tenant ID if you are writing a line of business application solely for your organization (also named single-tenant application)
- The application secret (client secret string) or certificate (of type X509Certificate2) if it's a confidential client app.
- For web apps, and sometimes for public client apps (in particular when your app needs to use a broker), you'll have also set the redirectUri where the identity provider will contact back your application with the security tokens.

## Initializing public and confidential client applications from code

The following code instantiates a public client application, signing-in users in the Microsoft Azure public cloud, with their work and school accounts, or their personal Microsoft accounts.
```
IPublicClientApplication app = PublicClientApplicationBuilder.Create(clientId).Build();
```

The application is identified with the identity provider by sharing a client secret:

```
string redirectUri = "https://myapp.azurewebsites.net";
IConfidentialClientApplication app = ConfidentialClientApplicationBuilder.Create(clientId)
    .WithClientSecret(clientSecret)
    .WithRedirectUri(redirectUri )
    .Build();
```

## Builder modifiers

In the code snippets using application builders, a number of `.With` methods can be applied as modifiers (for example, .`WithAuthority` and `.WithRedirectUri`).

.WithAuthority modifier: The .WithAuthority modifier sets the application default authority to an Azure Active Directory authority
```
var clientApp = PublicClientApplicationBuilder.Create(client_id)
    .WithAuthority(AzureCloudInstance.AzurePublic, tenant_id)
    .Build();
```

.WithRedirectUri modifier: The .WithRedirectUri modifier overrides the default redirect URI. In the case of public client applications, this will be useful for scenarios which require a broker.
```
var clientApp = PublicClientApplicationBuilder.Create(client_id)
    .WithAuthority(AzureCloudInstance.AzurePublic, tenant_id)
    .WithRedirectUri("http://localhost")
    .Build();
```

## Modifiers common to public and confidential client applications
.WithAuthority():  Sets the application default authority to an Azure Active Directory authority, with the possibility of choosing the Azure Cloud, the audience, the tenant (tenant ID or domain name), or providing directly the authority URI.

.WithTenantId(string tenantId): Overrides the tenant ID, or the tenant description.

.WithClientId(string):	Overrides the client ID.

.WithRedirectUri(string redirectUri):	Overrides the default redirect URI. In the case of public client applications, this will be useful for scenarios requiring a broker.

.WithComponent(string):	Sets the name of the library using MSAL.NET (for telemetry reasons).

.WithDebugLoggingCallback():	If called, the application will call Debug.Write simply enabling debugging traces.

.WithLogging():	If called, the application will call a callback with debugging traces.

.WithTelemetry(TelemetryCallback telemetryCallback):	Sets the delegate used to send telemetry.

## Modifiers specific to confidential client applications
.WithCertificate(X509Certificate2 certificate):	Sets the certificate identifying the application with Azure Active Director
.WithClientSecret(string clientSecret):	Sets the client secret (app password) identifying the application with Azure Active Directory.

# Exercise: Implement interactive authentication by using MSAL.NET
(https://docs.microsoft.com/en-us/learn/modules/implement-authentication-by-using-microsoft-authentication-library/4-interactive-authentication-msal)

# Knowledge check
Which of the following MSAL libraries supports single-page web apps?  
MSAL.js

