The lifetime of a **DbContext** begins when the instance is created and ends when the instance is disposed.

  **A typical unit-of-work** when using Entity Framework Core (EF Core) involves:
  
-   Creation of a  `DbContext`  instance
-   Tracking of entity instances by the context. Entities become tracked by
    -   Being  [returned from a query](https://docs.microsoft.com/en-us/ef/core/querying/tracking)
    -   Being  [added or attached to the context](https://docs.microsoft.com/en-us/ef/core/saving/disconnected-entities)
-   Changes are made to the tracked entities as needed to implement the business rule
-   [SaveChanges](https://docs.microsoft.com/en-us/dotnet/api/microsoft.entityframeworkcore.dbcontext.savechanges)  or  [SaveChangesAsync](https://docs.microsoft.com/en-us/dotnet/api/microsoft.entityframeworkcore.dbcontext.savechangesasync)  is called. EF Core detects the changes made and writes them to the database.
-   The  `DbContext`  instance is disposed



-   It is very important to dispose the  [DbContext](https://docs.microsoft.com/en-us/dotnet/api/microsoft.entityframeworkcore.dbcontext)  after use. This ensures both that any unmanaged resources are freed, and that any events or other hooks are unregistered so as to prevent memory leaks in case the instance remains referenced.
-   [DbContext is  **not thread-safe**](https://docs.microsoft.com/en-us/ef/core/dbcontext-configuration/#avoiding-dbcontext-threading-issues). Do not share contexts between threads. Make sure to  [await](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/operators/await)  all async calls before continuing to use the context instance.
-   An  [InvalidOperationException](https://docs.microsoft.com/en-us/dotnet/api/system.invalidoperationexception)  thrown by EF Core code can put the context into an unrecoverable state. Such exceptions indicate a program error and are not designed to be recovered from.


**How to add EF in .NET Core**

** in Startup.cs:**

    public void ConfigureServices(IServiceCollection services)
    {
        services.AddControllers();
    
        services.AddDbContext<ApplicationDbContext>(
            options => options.UseSqlServer("name=ConnectionStrings:DefaultConnection"));
    }

**Create a DbContext class:**

    public class ApplicationDbContext : DbContext
    {
        protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
        {
            optionsBuilder.UseSqlServer(@"Server=(localdb)\mssqllocaldb;Database=Test");
        }
    }

**Using  DbContextOptionsBuilder**
This allows a DbContext configured for dependency injection to also be constructed explicitly.

    public class ApplicationDbContext : DbContext
    {
        public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options)
            : base(options)
        {
        }
    }

    var contextOptions = new DbContextOptionsBuilder<ApplicationDbContext>()
        .UseSqlServer(@"Server=(localdb)\mssqllocaldb;Database=Test")
        .Options;
    
    using var context = new ApplicationDbContext(contextOptions);


**Your DbContext does not need to be sealed, but sealing is best practice to do so for classes not designed to be inherited from.**

**Always await EF Core asynchronous methods immediately.** (if a caller does not await the completion of one of these methods, and proceeds to perform other operations on the DbContext, the state of the DbContext can be, (and very likely will be) corrupted.)

The **AddDbContext** extension method registers **DbContext** with a scoped lifetime by default.