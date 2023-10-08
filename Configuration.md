## Idetification
SolutionExplorer/References/System.Web.MVC   right click select properties and look at Version
```C#
public class HomeController: Controller
{
    public string Index()
    {
        typeof(Controller).Assembly.GetName().Version.ToString() // Version identification 
    }
}
```
Projectname/right click select properties look at Web / Servers
- Use VisualStudio Development Server
- Use Local IIS Web Server / Create Virtual Directory/ go to IIS application and in Sites/ Default Web Site/NameOfYourProject
  
`https//: Server/Project/Controller/Action/RouteParameter?MethodParameterName=MethodParameterValue&MethodParameterName=MethodParameterValue`

## Configuration
Application Starts:
Global.asax.cs
```c#
public class MvcAplication : System.Web.HttpApplication
{
    protected void Aplication_Start()
    {
        AreaRegistration.RegisterAllAreas();
        WebConfig.Register(GlobalConfiguration.Configuration);
        FilterConfig.RegisterGlobalFilters(GlobalFilters.Filters);
        RouteConfig.RegisterRoutes(RouteTable.Routes); // Go to definition into App_Start/RouteConfig.cs
    }
}
```
App_Start/RouteConfig.cs
```c#
public class RouteConfig
{
    public static void registerRoutes(RouteCollection routes)
    {
        routes.IgnoreRoute("{resource}.axd/{pathInfo}");
        routesMapRoute(
            name: "Default",
            url: "{controller}/{action}/{id}",
            defaults: new { controller = "Home", action = "Index",  id = UrlParameter.Optional } // Add your own default Controller and Action  
        );
    }
}
```
