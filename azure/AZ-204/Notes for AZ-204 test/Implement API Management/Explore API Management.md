- Describe the components, and their function, of the API Management service.
- Explain how API gateways can help manage calls to your APIs.
- Secure access to APIs by using subscriptions and certificates.
- Create a backend API.

# Discover the API Management service

API Management provides the core functionality to ensure a successful API program through developer engagement, business insights, analytics, security, and protection.

The system is made up of the following components:  

The API gateway is the endpoint that:
- Accepts API calls and routes them to your backend(s).
- Verifies API keys, JWT tokens, certificates, and other credentials.
- Enforces usage quotas and rate limits.
- Transforms your API on the fly without code modifications.
- Caches backend responses where set up.
- Logs call metadata for analytics purposes.

The Azure portal is the administrative interface where you set up your API program. Use it to:
- Define or import API schema.
- Package APIs into products.
- Set up policies like quotas or transformations on the APIs.
- Get insights from analytics.
- Manage users.

The Developer portal serves as the main web presence for developers, where they can:
- Read API documentation.
- Try out an API via the interactive console.
- Create an account and subscribe to get API keys.
- Access analytics on their own usage.

## Products
Products are how APIs are surfaced to developers. Products in API Management have one or more APIs, and are configured with a title, description, and terms of use.

can be open or protected

Protected products must be subscribed to before they can be used.
open products can be used without a subscription.
Subscription approval is configured at the product level and can either require administrator approval, or be auto-approved.

## Groups

API Management has the following immutable system groups:
Administrators - Azure subscription administrators are members of this group.
Developers - Authenticated developer portal users fall into this group. Developers are the customers that build applications using your APIs
Guests - Unauthenticated developer portal users, such as prospective customers visiting the developer portal of an API Management instance fall into this group.

## Developers

Developers can be created or invited to join by administrators, or they can sign up from the Developer portal. 

## Policies

Policies are a powerful capability of API Management that allow the Azure portal to change the behavior of the API through configuration

## Developer portal
The developer portal is where developers can learn about your APIs, view and call operations, and subscribe to products.

# Explore API gateways
An API gateway sits between clients and services. It acts as a reverse proxy, routing requests from clients to services

there are some potential problems with exposing services directly to clients:

It can result in complex client code. The client must keep track of multiple endpoints, and handle failures in a resilient way.
It creates coupling between the client and the backend. The client needs to know how the individual services are decomposed. That makes it harder to maintain the client and also harder to refactor services.
A single operation might require calls to multiple services.
Each public-facing service must handle concerns such as authentication, SSL, and client rate limiting.
Services must expose a client-friendly protocol such as HTTP or WebSocket. This limits the choice of communication protocols.
Services with public endpoints are a potential attack surface, and must be hardened.


A gateway helps to address these issues by decoupling clients from services:

Gateway routing: Use the gateway as a reverse proxy to route requests to one or more backend services, using layer 7 routing. The gateway provides a single endpoint for clients, and helps to decouple clients from services.

Gateway aggregation: Use the gateway to aggregate multiple individual requests into a single request. This pattern applies when a single operation requires calls to multiple backend services.

Gateway Offloading: Use the gateway to offload functionality from individual services to the gateway.  consolidate these functions into one place, rather than making every service responsible for implementing them.

Here are some examples of functionality that could be offloaded to a gateway:
- SSL termination
- Authentication
- IP allow/block list
- Client rate limiting (throttling)
- Logging and monitoring
- Response caching
- GZIP compression
- Servicing static content

# Explore API Management policies

Policies are a collection of Statements that are executed sequentially on the request or response of an API.

a policy can apply changes to both the inbound request and outbound response. 

Policy expressions can be used as attribute values or text values in any of the API Management policies, unless the policy specifies otherwise.

## Understanding policy configuration

The policy definition is a simple XML document that describes a sequence of inbound and outbound statements.

The configuration is divided into `inbound`, `backend`, `outbound`, and `on-error`.

```
<policies>
  <inbound>
    <!-- statements to be applied to the request go here -->
  </inbound>
  <backend>
    <!-- statements to be applied before the request is forwarded to 
         the backend service go here -->
  </backend>
  <outbound>
    <!-- statements to be applied to the response go here -->
  </outbound>
  <on-error>
    <!-- statements to be applied if there is an error condition go here -->
  </on-error>
</policies>
```

## Examples

```
<policies>
    <inbound>
        <cross-domain />
        <base />
        <find-and-replace from="xyz" to="abc" />
    </inbound>
</policies>
```
In the example policy definition above, the cross-domain statement would execute before any higher policies which would in turn, be followed by the find-and-replace policy.

## Filter response content

The snippet assumes that response content is formatted as JSON and contains root-level properties named "minutely", "hourly", "daily", "flags".

```
<policies>
  <inbound>
    <base />
  </inbound>
  <backend>
    <base />
  </backend>
  <outbound>
    <base />
    <choose>
      <when condition="@(context.Response.StatusCode == 200 && context.Product.Name.Equals("Starter"))">
        <!-- NOTE that we are not using preserveContent=true when deserializing response body stream into a JSON object since we don't intend to access it again. See details on https://docs.microsoft.com/azure/api-management/api-management-transformation-policies#SetBody -->
        <set-body>
          @{
            var response = context.Response.Body.As<JObject>();
            foreach (var key in new [] {"minutely", "hourly", "daily", "flags"}) {
            response.Property (key).Remove ();
           }
          return response.ToString();
          }
    </set-body>
      </when>
    </choose>    
  </outbound>
  <on-error>
    <base />
  </on-error>
</policies>
```

# Create advanced policies


- Control flow - Conditionally applies policy statements based on the results of the evaluation of Boolean expressions.
- Forward request - Forwards the request to the backend service.
- Limit concurrency - Prevents enclosed policies from executing by more than the specified number of requests at a time.
- Log to Event Hub - Sends messages in the specified format to an Event Hub defined by a Logger entity.
- Mock response - Aborts pipeline execution and returns a mocked response directly to the caller.
- Retry - Retries execution of the enclosed policy statements, if and until the condition is met. Execution will repeat at the specified time intervals and up to the specified retry count.

## Control flow
The `choose` policy applies enclosed policy statements based on the outcome of evaluation of boolean expressions.
```
<choose>
    <when condition="Boolean expression | Boolean constant">
        <!— one or more policy statements to be applied if the above condition is true  -->
    </when>
    <when condition="Boolean expression | Boolean constant">
        <!— one or more policy statements to be applied if the above condition is true  -->
    </when>
    <otherwise>
        <!— one or more policy statements to be applied if none of the above conditions are true  -->
</otherwise>
</choose>
```

## Forward request
The `forward-request` policy forwards the incoming request to the backend service specified in the request context.
Removing this policy results in the request not being forwarded to the backend service and the policies in the outbound section are evaluated immediately upon the successful completion of the policies in the inbound section.

```
<forward-request timeout="time in seconds" follow-redirects="true | false"/>
```

## Limit concurrency
The `limit-concurrency` policy prevents enclosed policies from executing by more than the specified number of requests at any time. 
After that returns 429 Too Many Requests status code.
```
<limit-concurrency key="expression" max-count="number">
        <!— nested policy statements -->
</limit-concurrency>
```

## Log to Event Hub

The `log-to-eventhub` policy sends messages in the specified format to an Event Hub defined by a Logger entity
```
<log-to-eventhub logger-id="id of the logger entity" partition-id="index of the partition where messages are sent" partition-key="value used for partition assignment">
  Expression returning a string to be logged
</log-to-eventhub>
```

## Mock response
aborts normal pipeline execution and returns a mocked response to the caller.
```
<mock-response status-code="code" content-type="media type"/>
```

## Retry
The retry policy executes its child policies once and then retries their execution until the retry condition becomes false or retry count is exhausted.
```
<retry>
    condition="boolean expression or literal"
    count="number of retry attempts"
    interval="retry interval in seconds"
    max-interval="maximum retry interval in seconds"
    delta="retry interval delta in seconds"
    first-fast-retry="boolean expression or literal">
        <!-- One or more child policies. No restrictions -->
</retry>
```

## Return response

The `return-response` policy aborts pipeline execution and returns either a default or custom response to the caller. Default response is 200 OK with no body

```
<return-response response-variable-name="existing context variable">
  <set-header/>
  <set-body/>
  <set-status/>
</return-response>
```

(https://docs.microsoft.com/en-us/azure/api-management/api-management-policies)
(https://docs.microsoft.com/en-us/azure/api-management/api-management-error-handling-policies)

# Secure APIs by using subscriptions
To get a subscription key for accessing APIs, a subscription is required. A subscription is essentially a named container for a pair of subscription key.
**API Management also supports other mechanisms for securing access to APIs, including: OAuth2.0, Client certificates, and IP allow listing.**


## Subscriptions and Keys
The three main subscription scopes are:
- All APIs	Applies to every API accessible from the gateway
- Single API	This scope applies to a single imported API and all of its endpoints
- Product	A product is a collection of one or more APIs that you configure in API Management. You can assign APIs to more than one product. Products can have different access rules, usage quotas, and terms of use.

Applications that call a protected API must include the key in every request.

Every subscription has two keys, a primary and a secondary.

Having two keys makes it easier when you do need to regenerate a key. For example, if you want to change the primary key and avoid downtime, use the secondary key in your apps.

## Call an API with the subscription key

The default header name is `Ocp-Apim-Subscription-Key`, and the default query string is subscription-key.
To test out your API calls, you can use the developer portal, or command-line tools, such as curl. 

```
curl --header "Ocp-Apim-Subscription-Key: <key string>" https://<apim gateway>.azure-api.net/api/path
```

```
curl https://<apim gateway>.azure-api.net/api/path?subscription-key=<key string>
```

# Secure APIs by using certificates
You can configure the API Management gateway to allow only requests with certificates containing a specific thumbprint. The authorization at the gateway level is handled through inbound policies.

## Transport Layer Security client authentication
The API Management gateway can inspect the certificate contained within the client request and check for properties like:
- `Certificate Authority (CA)`	Only allow certificates signed by a particular CA
- `Thumbprint`	Allow certificates containing a specified thumbprint
- `Subject`	Only allow certificates with a specified subject
- `Expiration Date`	Only allow certificates that have not expired

These properties are not mutually exclusive and they can be mixed together to form your own policy requirements.

There are two common ways to verify a certificate:
- `Check who issued the certificate`. If the issuer was a certificate authority that you trust, you can use the certificate. You can configure the trusted certificate authorities in the Azure portal to automate this process.
- `If the certificate is issued by the partner`, verify that it came from them. For example, if they deliver the certificate in person, you can be sure of its authenticity. These are known as self-signed certificates.

## Accepting client certificates in the Consumption tier

The Consumption tier in API Management is designed to conform with serverless design principals.

In the Consumption tier, you must explicitly enable the use of client certificates, which you can do on the Custom domains page. This step is not necessary in other tiers.

## Certificate Authorization Policies
Create these policies in the inbound processing policy file within the API Management gateway

## Check the thumbprint of a client certificate
The thumbprint ensures that the values in the certificate have not been altered since the certificate was issued by the certificate authority.  The following example checks the thumbprint of the certificate passed in the request:

```
<choose>
    <when condition="@(context.Request.Certificate == null || context.Request.Certificate.Thumbprint != "desired-thumbprint")" >
        <return-response>
            <set-status code="403" reason="Invalid client certificate" />
        </return-response>
    </when>
</choose>
```

## Check the thumbprint against certificates uploaded to API Management
In the previous example, only one thumbprint would work so only one certificate would be validated.
Usually, each customer or partner company would pass a different certificate with a different thumbprint.
To support this scenario, obtain the certificates from your partners and use the Client certificates page in the Azure portal to upload them to the API Management resource. Then add this code to your policy:

```
<choose>
    <when condition="@(context.Request.Certificate == null || !context.Request.Certificate.Verify()  || !context.Deployment.Certificates.Any(c => c.Value.Thumbprint == context.Request.Certificate.Thumbprint))" >
        <return-response>
            <set-status code="403" reason="Invalid client certificate" />
        </return-response>
    </when>
</choose>
```

## Check the issuer and subject of a client certificate
```
<choose>
    <when condition="@(context.Request.Certificate == null || context.Request.Certificate.Issuer != "trusted-issuer" || context.Request.Certificate.SubjectName.Name != "expected-subject-name")" >
        <return-response>
            <set-status code="403" reason="Invalid client certificate" />
        </return-response>
    </when>
</choose>
```

# Exercise: Create a backend API
(https://docs.microsoft.com/en-us/learn/modules/explore-api-management/8-import-api)
## Create an API Management instance
```
myApiName=az204-apim-$RANDOM
myLocation=<myLocation>
myEmail=<myEmail>
```
Create a resource group. 
```
az group create --name az204-apim-rg --location $myLocation
```
Create an APIM instance. The az apim create command is used to create the instance. The --sku-name Consumption option is used to speed up the process for the walkthrough.
```
az apim create -n $myApiName \
    --location $myLocation \
    --publisher-email $myEmail  \
    --resource-group az204-apim-rg \
    --publisher-name AZ204-APIM-Exercise \
    --sku-name Consumption
```
## Import a backend API
1. Azure portal, search for and select API Management services.
2. On the API Management screen, select the API Management instance you created.
3. Select APIs in the API management service navigation pane.
4. Select OpenAPI from the list and select Full in the pop-up.
5. Fill form data
6. Select create

## Configure the backend settings
Select Settings in the blade to the right and enter https://conferenceapi.azurewebsites.net/ in the Web service URL field.
Deselect the Subscription required checkbox.

## Test the API

## Knowledge check
1. Which of the following components of the API Management service would a developer use if they need to create an account and subscribe to get API keys? Developer portal
2. Which of the following API Management policies would one use if one wants to apply a policy based on a condition? 
choose
