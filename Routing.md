## Configuration
- UseRouting();
- UseEndpoints();
```C#
//- Attribut Routing;
app.UseEndpoints(endpoints =>
{
   //endpoints.MapControllers(); // this if you want to use [Route("")] on the HomeController over Index action  for default. And [Route("[controller]/[action]")] over HomeController class
  //endpoints.MapDefaultControllerRoute()
  endpoints.MapControllerRoute(
    name: "Default",
    pattern: "{controller=Home}/{action=Index}/{id?});
  )
})
```
or
```C#
//- Convetional Routing;
app.UseEndpoints(endpoints =>
{
  
  endpoints.MapControllerRoute(
    name: "AboutUs",
    pattern: "about-us/{id?}"
    default: new {controller= "Landpage"}/{action= "AboutUs"});
  )
})// you can call about-us with /aboutus
```                                   
 ## Attribute Routing 

```C#
[Route("[controller]/[action]")]
public homeController : Controller
{
  [Route("~")] // ~ override other controller routing 
  public ViewResult  Index()
  {
    return View();
  }
  [HttpGet("~/about-us/{id:int:min(1)}/test/{name:string}")]     // [Route("~/about-us/{id}/test/{name}")]  "~" override other controller routing 
  public ViewResult  AboutUs(int id, string name, string region)   //about-us/17/test/john?region=Americas
  {
    return View();
  }
  [Route("contact-us", Name = "contact-us", Order = 1)]
  public ViewResult  ContactUs()
  {
    return View();
  }

  [HttpGet("~/otherpage/{id:int:min(1)}/test/{name:alpha:minlength(5):maxlength(20):regex()")]     // id must be int not lower then 1, name only alphabetical winth minimum 5 characters, 
  public ViewResult  OtherPage(int id, string name, string region)   //about-us/17/test/john?region=Americas
  {
    return View();
  }
}
```                   
 ## Compound Attribute Routing 


```C#
[Route("[controller]/[action]")]
public homeController : Controller
{
  [Route("~")] // ~ override other controller routing 
  public ViewResult  Index()
  {
    return View();
  }

  [HttpGet("~/test/a{a}]     // starts with a 
  public ViewResult  A( string a)   //about-us/17/test/john?region=Americas
  {
    return View();
  }

  [HttpGet("~/test/b{a}]     // starts with b 
  public ViewResult  B( string a)   //about-us/17/test/john?region=Americas
  {
    return View();
  }
 
}
```          
