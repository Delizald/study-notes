## **Dependency Injection in .net core**

DI is a technique for achieving [Inversion of Control (IoC)](https://docs.microsoft.com/en-us/dotnet/architecture/modern-web-apps-azure/architectural-principles#dependency-inversion) between classes and their dependencies.

It allows the creation of dependent objects outside of a class and provides those objects to a class through different ways. Using DI, we move the creation and binding of the dependent objects outside of the class that depends on them.

**DI - Service lifetimes**

-   **Transient**: Transient lifetime services are created each time they're requested from the service container. This lifetime works best for lightweight, stateless services. Register transient services with [AddTransient](https://docs.microsoft.com/en-us/dotnet/api/microsoft.extensions.dependencyinjection.servicecollectionserviceextensions.addtransient).

-   **Scoped**: For web applications, a scoped lifetime indicates that services are created once per client request (connection). Register scoped services with  [**AddScoped**](https://docs.microsoft.com/en-us/dotnet/api/microsoft.extensions.dependencyinjection.servicecollectionserviceextensions.addscoped). In apps that process requests, scoped services are disposed at the end of the request.

-   **Singleton**: Singleton lifetime services are created either:
	-   The first time they're requested.
	-   By the developer, when providing an implementation instance directly to the container. This approach is rarely needed. Every subsequent request of the service implementation from the dependency injection container uses the same instance.  **AddSingleton**
	
**[Which one to use](https://stackoverflow.com/questions/38138100/addtransient-addscoped-and-addsingleton-services-differences) **
Use Singletons where you need to maintain application wide state. Application configuration or parameters, Logging Service, caching of data is some of the examples where you can use singletons.

Injecting service with different lifetimes into another

1.  **Never inject Scoped & Transient services into Singleton service.**  ( This effectively converts the transient or scoped service into the singleton.)
    
2.  **Never inject Transient services into scoped service**  ( This converts the transient service into the scoped.)

## The Repository and Unit of Work Patterns
The repository and unit of work patterns are intended to create an abstraction layer between the data access layer and the business logic layer of an application. Implementing these patterns can help insulate your application from changes in the data store and can facilitate automated unit testing or test-driven development (TDD).

![Repository_pattern_diagram](https://asp.net/media/2578149/Windows-Live-Writer_8c4963ba1fa3_CE3B_Repository_pattern_diagram_1df790d3-bdf2-4c11-9098-946ddd9cd884.png)

**The Repository pattern**
Use a repository to separate the logic that retrieves the data and maps it to the entity model from the business logic that acts on the model. The business logic should be agnostic to the type of data that comprises the data source layer. For example, the data source layer can be a database, a SharePoint list, or a Web service.

The repository mediates between the data source layer and the business layers of the application. It queries the data source for the data, maps the data from the data source to a business entity, and persists changes in the business entity to the data source.

**Unit of work TODO**