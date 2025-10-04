# ASP.NET MVC Configuration Cheat Sheet

**Applies to:** **ASP.NET MVC 5** on **.NET Framework 4.6–4.8.x** (System.Web‑based MVC).

> Not for ASP.NET Core. For Core, Global.asax, RouteConfig, Web.config, etc. do not apply.

---

## Identify MVC Version & Runtime

**Solution Explorer → References → System.Web.Mvc → Properties → Version**

Or at runtime:

```csharp
using System.Web.Mvc;

public class HomeController : Controller
{
    public string Index()
    {
        // System.Web.Mvc assembly (MVC 5.x typically: 5.2.x)
        return typeof(Controller).Assembly.GetName().Version.ToString();
    }
}
```

**Project → Properties → Web**

* **IIS Express** (recommended for local dev in modern VS)
* **Local IIS** (requires admin; click **Create Virtual Directory**)
* App will be hosted under **IIS** site (e.g., *Default Web Site*) using an **App Pool** targeting **.NET CLR v4.0** (Integrated pipeline).

**URL format:**

```
https://server/app/Controller/Action/routeParam?param1=value1&param2=value2
```

---

## Application Startup (Global.asax)

**Global.asax.cs**

```csharp
using System.Web;
using System.Web.Mvc;
using System.Web.Routing;
using System.Web.Optimization; // if using bundling/minification
using System.Web.Http;        // only if you also use Web API

public class MvcApplication : HttpApplication
{
    protected void Application_Start()
    {
        AreaRegistration.RegisterAllAreas();

        // Web API (optional; remove if not using Web API)
        GlobalConfiguration.Configure(WebApiConfig.Register);

        FilterConfig.RegisterGlobalFilters(GlobalFilters.Filters);
        RouteConfig.RegisterRoutes(RouteTable.Routes);

        // Bundling/minification (optional; requires Microsoft.AspNet.Web.Optimization)
        BundleConfig.RegisterBundles(BundleTable.Bundles);
    }
}
```

---

## Routing (App_Start/RouteConfig.cs)

**Conventional routing**

```csharp
using System.Web.Mvc;
using System.Web.Routing;

public class RouteConfig
{
    public static void RegisterRoutes(RouteCollection routes)
    {
        routes.IgnoreRoute("{resource}.axd/{*pathInfo}");

        // Enable attribute routing (optional but recommended)
        routes.MapMvcAttributeRoutes();

        routes.MapRoute(
            name: "Default",
            url: "{controller}/{action}/{id}",
            defaults: new { controller = "Home", action = "Index", id = UrlParameter.Optional }
        );
    }
}
```

**Attribute routing example**

```csharp
public class ProductsController : Controller
{
    [Route("products/{id:int}")]
    public ActionResult Details(int id) => View();
}
```

---

## Filters (App_Start/FilterConfig.cs)

```csharp
using System.Web.Mvc;

public class FilterConfig
{
    public static void RegisterGlobalFilters(GlobalFilterCollection filters)
    {
        filters.Add(new HandleErrorAttribute());
        // e.g., filters.Add(new AuthorizeAttribute());
    }
}
```

---

## (Optional) Web API Config (App_Start/WebApiConfig.cs)

> Only if your project also hosts **ASP.NET Web API 2**.

```csharp
using System.Web.Http;

public static class WebApiConfig
{
    public static void Register(HttpConfiguration config)
    {
        config.MapHttpAttributeRoutes();

        config.Routes.MapHttpRoute(
            name: "DefaultApi",
            routeTemplate: "api/{controller}/{id}",
            defaults: new { id = RouteParameter.Optional }
        );
    }
}
```

---

## Bundling/Minification (optional)

```csharp
using System.Web.Optimization;

public class BundleConfig
{
    public static void RegisterBundles(BundleCollection bundles)
    {
        bundles.Add(new ScriptBundle("~/bundles/jquery").Include(
                    "~/Scripts/jquery-{version}.js"));

        bundles.Add(new StyleBundle("~/Content/css").Include(
                  "~/Content/site.css"));

        // BundleTable.EnableOptimizations = true; // force in Debug
    }
}
```

---

## Web.config Essentials (production hardening)

> Located at project root. Consider **Web.Release.config** transform for prod.

* **Compilation**

```xml
<compilation debug="false" targetFramework="4.8" />
```

* **Custom Errors**

```xml
<customErrors mode="On" defaultRedirect="~/Error">
  <error statusCode="404" redirect="~/Error/NotFound" />
</customErrors>
```

* **Authentication/Authorization** (forms/windows as needed)

```xml
<system.web>
  <authentication mode="None" />
  <authorization>
    <deny users="?" /> <!-- deny anonymous -->
  </authorization>
</system.web>
```

* **HTTP Runtime / Request Limits** (uploads, etc.)

```xml
<system.web>
  <httpRuntime maxRequestLength="51200" /> <!-- ~50 MB -->
</system.web>
```

* **IIS Integration** (root Web.config or applicationHost.config)

  * App Pool: **.NET CLR v4.0**, **Integrated**
  * **32‑bit** only if native deps require it

---

## NuGet Packages (typical MVC 5 app)

* **Microsoft.AspNet.Mvc (5.2.x)**
* Microsoft.AspNet.Razor (3.2.x)
* Microsoft.AspNet.WebPages (3.2.x)
* Microsoft.AspNet.Web.Optimization (if using bundling)
* Microsoft.AspNet.WebApi.Core / WebApi.WebHost (if using Web API)
* jQuery / Bootstrap (if using default templates)

---

## Common Pitfalls (Urgent Mentions)

* **Spelling matters**: `MvcApplication`, `Application_Start`, `RegisterRoutes`, `MapRoute` (not `Aplication`, `registerRoutes`, `routesMapRoute`).
* **Ignore AXD**: `routes.IgnoreRoute("{resource}.axd/{*pathInfo}")` to skip trace/ELMAH resources.
* **Do not mix** MVC and Web API config calls unless you actually reference Web API; remove `GlobalConfiguration.Configure(...)` if not used.
* **Set `debug=false` in production** to avoid leaking stack traces and to enable optimizations.
* **Use `MapMvcAttributeRoutes()`** if you decorate actions with `[Route]`.
* **IIS App Pool** must target **.NET Framework v4.x** (not No Managed Code).
* **Web.config transforms** (`Web.Debug.config`, `Web.Release.config`) are your friend—store prod‑only secrets in secure config stores, not in source.
* **Prefer IIS Express** over the legacy *Visual Studio Development Server* which is deprecated.

---

## Quick Checklist

* [ ] `Application_Start` wires **Areas**, **Filters**, **Routes**, **Bundles** (as used)
* [ ] `RouteConfig.RegisterRoutes` has `IgnoreRoute` + default route (and attribute routing if needed)
* [ ] **Web.config**: `compilation debug="false"` for prod; `customErrors` on
* [ ] Using **IIS Express** (dev) or **Local IIS** with correct App Pool (.NET CLR v4.0, Integrated)
* [ ] NuGet packages align with **ASP.NET MVC 5**
* [ ] If using Web API, `GlobalConfiguration.Configure(WebApiConfig.Register)` present and routes set
