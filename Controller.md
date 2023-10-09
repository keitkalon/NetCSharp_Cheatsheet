# Retrieve QueryString name:
- `Request.QueryString["name"];` //an old method use also in webforms
- add the parameter at the controller method `Index(string id, string name)` // actulize way

# Send data to View 
## ViewBag 
dynamic property // don't use because it doesn't provide compile error checking
- Ex in Controller:
```c#
public ViewResult Index()
{
  ViewBag.Countries = new List<string>() { "Andora", "Liechtenstein", "Luxemburg", "Monaco", "Vatican" };
  return View();
}
```
- Ex in View:
```razor
<h3>Contries</h3>
<ul>
@foreach(string contry in ViewBag.Countries)
{
  <li>@contry</li>
}
</ul>
```
## ViewData
dictionary property // second best
- Ex in Controller:
```c#
public ViewResult Index()
{
  ViewData["Countries"] = new List<string>() { "Andora", "Liechtenstein", "Luxemburg", "Monaco", "Vatican" };
  return View();
}
```
- Ex in View:
```razor
<h3>Contries</h3>
<ul>
@foreach (var country in ViewData["Countries"] as List<string>)
{
    <li>@country</li>
}
</ul>
```
## ViewModels
strongly typed classes // this is the best to use 
- Ex in Controller:
```c#
public ViewResult Index()
{
  var viewModel = new CountryViewModel
  {
      Countries = new List<string>() { "Andora", "Liechtenstein", "Luxemburg", "Monaco", "Vatican" }
  };
  return View(viewModel); // add model to View
}
```
- Ex ViewModel class:
```c#
public class CountryViewModel
{
    public List<string> Countries { get; set; }
}
```
- Ex in View:
```razor

@model YourNamespace.CountryViewModel
<h3>Contries</h3>
<ul>
@foreach (var country in Model.Countries)
{
    <li>@country</li>
}
</ul>
```
## Dynamic Decoration with ExpandoObject
Do to dynamic nature of the pattern this variant do not provide compile error checking
- Ex in Controller:
```c#
public ViewResult GetBook(int id)
{
    dynamic data = new System.Dynamic.ExpandoObject();
    data.book = _bookRepository.GetBookById(Id);
    data.Author = _authorRepository.GetAuthor(book.AuthorId);
    return View(data);
}
```
