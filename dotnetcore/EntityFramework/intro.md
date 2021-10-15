##  **Intro**

EF Core can serve as an object-relational mapper (O/RM), which:

- Enables .NET developers to work with a database using .NET objects.

- Eliminates the need for most of the data-access code that typically needs to be written.

  

**The model**

  

Data access is performed using a **model**. A **model** is made up of entity **classes** and a **context object** that represents a session with the database.

  

using System.Collections.Generic;

**using Microsoft.EntityFrameworkCore;**

    namespace Intro
    
    {
    
    public class BloggingContext **: DbContext**
    
    {
    
    public DbSet<Blog> Blogs { get; set; }
    
    public DbSet<Post> Posts { get; set; }
    
    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    
    {
    
    optionsBuilder.UseSqlServer(
    
    @"Server=(localdb)\mssqllocaldb;Database=Blogging;Trusted_Connection=True");
    
    }
    
    }
    
    public class Blog
    
    {
    
    public int BlogId { get; set; }
    
    public string Url { get; set; }
    
    public int Rating { get; set; }
    
    public List<Post> Posts { get; set; }
    
    }
    
    public class Post
    
    {
    
    public int PostId { get; set; }
    
    public string Title { get; set; }
    
    public string Content { get; set; }
    
    public int BlogId { get; set; }
    
    public Blog Blog { get; set; }
    
    }
    
    }

  

**LINQ Example:**

  

    using (var db = new BloggingContext())
    
    {
    
    var blogs = db.Blogs
    
    .Where(b => b.Rating > 3)
    
    .OrderBy(b => b.Url)
    
    .ToList();
    
    }

**Saving data**

  

    using (var db = new BloggingContext())
    
    {
    
    var blog = new Blog { Url = "http://sample.com" };
    
    db.Blogs.Add(blog);
    
    db.SaveChanges();
    
    }

**Installation **

 1. Install provider: https://docs.microsoft.com/en-us/ef/core/what-is-new/nuget-packages#database-providers
 2. install [Microsoft.EntityFrameworkCore](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore/), and [Microsoft.EntityFrameworkCore.Relational](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.Relational/) **if using a relational database provider.**
 3. Make sure to install the same version of all EF Core packages shipped by Microsoft.