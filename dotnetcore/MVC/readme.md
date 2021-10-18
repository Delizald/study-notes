The default URL routing logic used by MVC, uses a format like this to determine what code to invoke:

/[Controller]/[ActionName]/[Parameters]

The routing format is set in the Configure method in Startup.cs file.

app.UseEndpoints(endpoints =>
{
    endpoints.MapControllerRoute(
        name: "default",
        pattern: "{controller=Home}/{action=Index}/{id?}");
});


using System.Text.Encodings.Web; ( HtmlEncoder.Default.Encode) to protect the app from malicious input, such as through JavaScript.


To make a controller return a view:
public IActionResult Index()
{
    return View();
}

Controller methods:

Are referred to as action methods. For example, the Index action method in the preceding code.
Generally return an IActionResult or a class derived from ActionResult, not a type like string.


The Views/_ViewStart.cshtml file brings in the Views/Shared/_Layout.cshtml file to each view. The Layout property can be used to set a different layout view, or set it to null so no layout file will be used.


Controllers are responsible for providing the data required in order for a view template to render a response.

View templates should not:
  Do business logic
  Interact with a database directly.

A view template should work only with the data that's provided to it by the controller. Maintaining this "separation of concerns" helps keep the code:

Clean.
Testable.
Maintainable.

The ViewData dictionary is a dynamic object, which means any type can be used. The ViewData object has no defined properties until something is added. The ViewData dictionary object contains data that will be passed to the view.

a context object represents a session with the database. 

MVC provides the ability to pass strongly typed model objects to a view. This strongly typed approach enables compile time code checking.

// GET: Movies/Details/5
public async Task<IActionResult> Details(int? id)
{
    if (id == null)
    {
        return NotFound();
    }

    var movie = await _context.Movie
        .FirstOrDefaultAsync(m => m.Id == id);
    if (movie == null)
    {
        return NotFound();
    }

    return View(movie);
}



he ASP.NET Core Configuration system reads the ConnectionString key. For local development, it gets the connection string from the appsettings.json file. When the app is deployed to a test or production server, an environment variable can be used to set the connection string to a production SQL Server. 


The [Bind] attribute is one way to protect against over-posting. 
Mass assignment, also known as over-posting, is an attack used on websites that involve some sort of model-binding to a request. It is used to set values on the server that a developer did not expect to be set. (https://andrewlock.net/preventing-mass-assignment-or-over-posting-in-asp-net-core/)

[ValidateAntiForgeryToken] attribute is used to prevent forgery of a request