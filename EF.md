# Entity Framework
## Create Model
```c#
[Table("tblEmployee")] // table name in DB
public class Employee
{
  public int EmployeeId { get; set; }
  public string Name { get; set; }
  public string Gender { get; set; }
  public string City { get; set; }
}
```
## Create Context File
```c#
public class EmployeeContext : DBContext
{
  public DbSet<Employee> Employees { get; set; }
}
```
or
```c#
public class BookStoreContext : DBContext
{
  public BookStoreContext(DbContextOptions<BookStoreContext> options)
    : base(options)
  {

  }

  public DbSet<Book> Books { get; set; }

  protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
  {
      optionsBuilder.UseSqlServer(""Server=.; DataBase=Sample; integrated security=true);
      base.OnConfiguring(optionsBuilder);
  }
}
```
## Add conection string into Web.config file
```xml
<conectionString>
  <add name="EmployeeContext"
        conectionString ="server=.; database=Sample;  integrated security=SSPI"
        providerName="System.Data.SqlClient">
```
## Misrosoft SQL
- SELECT * FROM [Sample][tblEmployee]

## Configuration
Add `DataBase.SetInitializer<MVCDemo.Models.EmployeeContext>(null);` in Aplication_Start()
```c#
public class MvcAplication : System.Web.HttpApplication
{
    protected void Aplication_Start()
    {
        DataBase.SetInitializer<ProjectName.Models.EmployeeContext>(null); // if it doesn't exist create it
        AreaRegistration.RegisterAllAreas();
        WebConfig.Register(GlobalConfiguration.Configuration);
        FilterConfig.RegisterGlobalFilters(GlobalFilters.Filters);
        RouteConfig.RegisterRoutes(RouteTable.Routes);
    }
}
```

## Controller

```c#
public ActionResult Index()
{
  EmployeeContext employeeContext = new EmployeeContext();
  List<Employee> employees = employeeContext.Employees.ToList();
  return View(employees);
}

public ActionResult Details(id)
{
  EmployeeContext employeeContext = new EmployeeContext();
  Employee employee = employeeContext.Employees.Single(emp => emp.EmployeeId = id);
  return View(employee);
}
```

## View

```cshtml
@model Ienumerable<ProjectName.Models.Employee> // the Model

@using ProjectName.Models; // when you want use other class models

<h2>Employees:</h2>
<ul>
@foreach(Employee employee in @model)
{
  <li>@Htm.ActionLink(employee.Name, "Details", new { @id = employee.EmployeeId })</li> // `new { @id = employee.EmployeeId })` is what Details action need as parameter to run
}
</ul>
```
