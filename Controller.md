# ASP.NET MVC Controllers Cheat Sheet

**Applies to:** ASP.NET MVC 5 (System.Web.Mvc) on .NET Framework 4.x.

> Not for ASP.NET Core — in Core, controller APIs and data passing differ.

---

## Retrieve QueryString Values

* **Legacy (WebForms style, avoid if possible):**

```csharp
var name = Request.QueryString["name"];
```

* **Preferred (MVC model binding):**

```csharp
public ActionResult Index(string id, string name)
{
    // automatically bound from ?id=1&name=John
    return View();
}
```

---

## Sending Data to Views

### 1. ViewBag

Dynamic property — works but **no compile-time safety**.

```csharp
public ViewResult Index()
{
    ViewBag.Countries = new List<string>() { "Andorra", "Liechtenstein", "Luxembourg", "Monaco", "Vatican" };
    return View();
}
```

View:

```razor
<h3>Countries</h3>
<ul>
@foreach (string country in ViewBag.Countries)
{
    <li>@country</li>
}
</ul>
```

### 2. ViewData

Dictionary-based — slightly better, but still no strong typing.

```csharp
public ViewResult Index()
{
    ViewData["Countries"] = new List<string>() { "Andorra", "Liechtenstein", "Luxembourg", "Monaco", "Vatican" };
    return View();
}
```

View:

```razor
<h3>Countries</h3>
<ul>
@foreach (var country in ViewData["Countries"] as List<string>)
{
    <li>@country</li>
}
</ul>
```

### 3. ViewModels (Recommended)

Strongly typed class passed to the View.

```csharp
public ViewResult Index()
{
    var viewModel = new CountryViewModel
    {
        Countries = new List<string>() { "Andorra", "Liechtenstein", "Luxembourg", "Monaco", "Vatican" }
    };
    return View(viewModel);
}
```

ViewModel:

```csharp
public class CountryViewModel
{
    public List<string> Countries { get; set; }
}
```

View:

```razor
@model YourNamespace.CountryViewModel
<h3>Countries</h3>
<ul>
@foreach (var country in Model.Countries)
{
    <li>@country</li>
}
</ul>
```

### 4. Dynamic Decoration (ExpandoObject)

Provides flexibility but **no compile-time safety**.

```csharp
public ViewResult GetBook(int id)
{
    dynamic data = new System.Dynamic.ExpandoObject();
    var book = _bookRepository.GetBookById(id);
    data.Book = book;
    data.Author = _authorRepository.GetAuthor(book.AuthorId);
    return View(data);
}
```

---

## Useful Extras (What You Might Have Missed)

### TempData

* Persists data between requests (uses session under the hood).
* Useful for **Redirect → View** scenarios.

```csharp
public ActionResult Save()
{
    TempData["Message"] = "Saved successfully!";
    return RedirectToAction("Index");
}

public ActionResult Index()
{
    ViewBag.Message = TempData["Message"];
    return View();
}
```

### ActionResult Return Types

* `ViewResult` → returns a view
* `RedirectToRouteResult` → `RedirectToAction("Index")`
* `RedirectResult` → `Redirect("/Home/About")`
* `JsonResult` → `return Json(data, JsonRequestBehavior.AllowGet)`
* `FileResult` → return files (File, FilePathResult, FileStreamResult)
* `HttpStatusCodeResult` → e.g. `return new HttpNotFoundResult();`

### Attribute Routing

```csharp
public class ProductsController : Controller
{
    [Route("products/{id:int}")]
    public ActionResult Details(int id)
    {
        // matched from /products/5
        return View();
    }
}
```

### Model Binding

* Query string, form data, and route values automatically bind to action parameters.
* For complex types:

```csharp
public ActionResult Register(UserViewModel model)
{
    if (ModelState.IsValid)
    {
        // model populated from form
    }
    return View(model);
}
```

### Model Validation

* Decorate ViewModel with DataAnnotations:

```csharp
public class UserViewModel
{
    [Required]
    public string Name { get; set; }

    [EmailAddress]
    public string Email { get; set; }
}
```

View:

```razor
@Html.ValidationMessageFor(m => m.Email)
```

---

## Best Practices

* Prefer **ViewModels** for strongly typed safety.
* Use **TempData** only for short-lived data (redirect scenarios).
* Avoid **ExpandoObject** in production unless truly dynamic.
* Add **[ValidateAntiForgeryToken]** on POST actions to prevent CSRF.
* Use **async/await** in controllers for long-running IO (with `Task<ActionResult>`).
* Keep controllers thin — business logic belongs in services, not controllers.
