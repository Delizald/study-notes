## Eager, Lazy and Explicit loading

Eager Loading

Eager loading is the process whereby a query for one type of entity also loads related entities as part of the query, so that we don't need to execute a separate query for related entities.s

Lazy Loading

It is the default behavior of an Entity Framework, where a child entity is loaded only when it is accessed for the first time. It simply delays the loading of the related data, until you ask for it.

Explicit Loading
with lazy loading disabled (in EF 6), it is still possible to lazily load related entities, but it must be done with an explicit call. Use the Load() method to load related entities explicitly. 

using (var context = new SchoolContext())
{
    var student = context.Students
                        .Where(s => s.FirstName == "Bill")
                        .FirstOrDefault<Student>();

    context.Entry(student).Reference(s => s.StudentAddress).Load(); // loads StudentAddress
    context.Entry(student).Collection(s => s.StudentCourses).Load(); // loads Courses collection 
}     

When to use what

Use Eager Loading when the relations are not too much. Thus, Eager Loading is a good practice to reduce further queries on the Server.
Use Eager Loading when you are sure that you will be using related entities with the main entity everywhere.
Use Lazy Loading when you are using one-to-many collections.
Use Lazy Loading when you are sure that you are not using related entities instantly.
When you have turned off Lazy Loading, use Explicit loading when you are not sure whether or not you will be using an entity beforehand.
