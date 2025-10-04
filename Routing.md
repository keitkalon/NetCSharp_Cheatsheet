# ASP.NET Core MVC Controller Routing Cheat Sheet

**Applies to:** ASP.NET **Core MVC** on **.NET 6–8** (minimal hosting).

> For .NET Core 3.1/5, see the legacy `UseRouting()/UseEndpoints()` example below.

---

## 1) Minimal Hosting Setup (Program.cs, .NET 6–8)

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllersWithViews();

var app = builder.Build();

if (!app.Environment.IsDevelopment())
{
    app.UseExceptionHandler("/Home/Error");
    app.UseHsts();
}

app.UseHttpsRedirection();
app.UseStaticFiles();

app.UseRouting();
app.UseAuthentication();   // if using auth
app.UseAuthorization();

// Conventional routing
app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");

// Example of an additional conventional route
app.MapControllerRoute(
    name: "about-us",
    pattern: "about-us/{id?}",
    defaults: new { controller = "Landpage", action = "AboutUs" });

app.Run();
```

### Legacy style (ASP.NET Core 3.1/5)

```csharp
app.UseRouting();

app.UseAuthentication();
app.UseAuthorization();

app.UseEndpoints(endpoints =>
{
    endpoints.MapControllerRoute(
        name: "default",
        pattern: "{controller=Home}/{action=Index}/{id?}");

    endpoints.MapControllerRoute(
        name: "about-us",
        pattern: "about-us/{id?}",
        defaults: new { controller = "Landpage", action = "AboutUs" });

    // endpoints.MapControllers(); // for pure attribute routing only
});
```

> **Tip:** Route order matters. The first matching route wins.

---

## 2) Conventional Routing (controller/action pattern)

```csharp
// Default route
app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");

// Slug route to a specific controller/action
app.MapControllerRoute(
    name: "product-details",
    pattern: "products/{id:int}",
    defaults: new { controller = "Products", action = "Details" });

// Optional segments, constraints
app.MapControllerRoute(
    name: "blog",
    pattern: "blog/{year:int:min(2000)}/{month:int:range(1,12)}/{slug?}",
    defaults: new { controller = "Blog", action = "Post" });
```

**Common constraints:**

* `int`, `bool`, `datetime`, `decimal`, `double`, `float`, `long`, `guid`
* `min(n)`, `max(n)`, `range(min,max)`
* `alpha`, `regex(<pattern>)`, `length(n)`, `minlength(n)`, `maxlength(n)`

---

## 3) Attribute Routing

Enable both conventional and attribute routing together, or go attribute-only (`app.MapControllers()` legacy) if desired.

```csharp
[Route("[controller]/[action]")] // controller-level prefix
public class HomeController : Controller
{
    // Rooted route (~) ignores the controller-level prefix
    [HttpGet("~/")]
    public IActionResult Index() => View();

    // Full custom template from the app root
    [HttpGet("~/about-us/{id:int:min(1)}/test/{name:alpha:minlength(5):maxlength(20)}")]
    public IActionResult AboutUs(int id, string name, string? region) // /about-us/17/test/john?region=Americas
        => View();

    // Named route for link generation
    [HttpGet("contact-us", Name = "contact-us", Order = 1)]
    public IActionResult ContactUs() => View();

    // Another rooted route with regex
    [HttpGet("~/otherpage/{id:int:min(1)}/test/{name:regex(^[a-zA-Z]{5,20}$)}")]
    public IActionResult OtherPage(int id, string name, string? region) => View();
}
```

**Tokens available:** `[controller]`, `[action]`, `[area]`
**Rooted routes:** prefix with `~/` to bypass controller/area templates.

---

## 4) Compound Attribute Routing Examples

```csharp
[Route("[controller]/[action]")]
public class TestController : Controller
{
    // Starts with 'a' (regex)
    [HttpGet("~/test/{a:regex(^a.*$)}")]
    public IActionResult A(string a) => View();

    // Starts with 'b'
    [HttpGet("~/test/{a:regex(^b.*$)}")]
    public IActionResult B(string a) => View();
}
```

---

## 5) Areas (optional)

```csharp
// Program.cs
app.MapControllerRoute(
    name: "areas",
    pattern: "{area:exists}/{controller=Home}/{action=Index}/{id?}");
```

Controller:

```csharp
[Area("Admin")]
[Route("Admin/[controller]/[action]")]
public class UsersController : Controller
{
    public IActionResult Index() => View();
}
```

---

## 6) Link Generation (route names & Url helpers)

**Named routes** let you generate stable links:

```csharp
// In a controller
var url = Url.RouteUrl("contact-us");
return Redirect(url);
```

Razor:

```razor
<a asp-route="contact-us">Contact</a>
```

---

## 7) Common Pitfalls & Notes

* **Quotes & braces:** watch for missing quotes/parentheses in route templates.
* **Typos:** `Conventional`, not "Convetional"; `HomeController`, not `homeController`.
* **Rooted routes:** must start with `~/`.
* **Constraints:** `regex(<pattern>)` must not be empty; escape braces properly in C# strings.
* **Order:** Place more specific routes **before** generic ones.
* **Attribute + Conventional:** they can coexist; attribute routes are matched **before** conventional ones for the same controller.
* **HTTP verbs:** use `[HttpGet]`, `[HttpPost]`, `[HttpPut]`, `[HttpDelete]`, `[HttpPatch]` as appropriate.
* **Optional parameters:** use `{id?}` and nullable types (`int?`) in action parameters.

---

## 8) Quick Reference

* Minimal hosting: `app.MapControllerRoute("default", "{controller=Home}/{action=Index}/{id?}");`
* Attribute route rooted: `[HttpGet("~/path")]`
* Constraint samples: `{id:int}`, `{slug:alpha}`, `{page:int:min(1)}`, `{name:regex(^[A-Za-z]+$)}`
* Named route: `[HttpGet("contact", Name = "contact-us")]`
* Areas: `app.MapControllerRoute("areas", "{area:exists}/{controller=Home}/{action=Index}/{id?}");`
