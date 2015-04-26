Hyprlinkr
=========
Hyprlinkr is a small and very focused helper library for the ASP.NET Web API. It does one thing only: it creates URIs according to the application's route configuration in a type-safe manner.

As of version 2 it works with ASP.NET Web 2. Look for version 1 for compatibility with ASP.NET Web API 1.

Example
-------
Imagine that you're using the standard route configuration created by the Visual Studio project template:
```C#
name: "API Default",
routeTemplate: "api/{controller}/{id}",
defaults: new { id = RouteParameter.Optional }
```
In that case, you can create an URI to a the resource handled by the FooController.GetById Action Method like this:
```C#
var uri = linker.GetUri<FooController>(r => r.GetById(1337));
```
This will create a URI like this:
```
http://localhost/api/foo/1337
```
assuming that the current host is http://localhost.
Creating a RouteLinker instance
-------------------------------
The RouteLinker class has two constructor overloads:
```C#
public RouteLinker(HttpRequestMessage request)

public RouteLinker(HttpRequestMessage request, IRouteDispatcher dispatcher)
```
In both cases it requires an instance of the HttpRequestMessage class, which is provided by the ASP.NET Web API for each request. The preferred way to get this instance is to implement a custom IHttpControllerActivator and create the RouteLinker instance from there.
```C#
public IHttpController Create(
    HttpRequestMessage request,
    HttpControllerDescriptor controllerDescriptor,
    Type controllerType)
{
    var linker = new RouteLinker(request);

    // Use linker and other services to create the appropriate Controller.
    // If desired, a DI Container can be used for this task.
}
```
Such a custom IHttpControllerActivator can be registered in Global.asax like this:
```C#
GlobalConfiguration.Configuration.Services.Replace(
    typeof(IHttpControllerActivator),
    new MyCustomControllerActivator());
```
This approach enables the use of Dependency Injection (DI) because the request can be injected into the services which require it.
### Without Dependency Injection ###
As an alternative to Dependency Injection (DI), the request can also be pulled directly from the ApiController instance. This requires that Controllers derive from ApiController. If this is the case, a RouteLinker instance can be created easily:
```C#
var linker = new RouteLinker(this.Request);
```
The example code includes a NoDIController class that demonstrates this approach.
### From an API Controller ###
If you want to use Hyprlinkr directly from an `ApiController`, you can use extension methods to create links:
```C#
// Inside an ApiController
var uri = this.Url.GetLink<FooController>(a => a.GetById(1337));
```
This will create a URI like this:
```
http://localhost/api/foo/1337
```
assuming that the current host is http://localhost.
Custom route dispatching
------------------------
The default behavior for RouteLinker is:
* If the method being linked to has a [Route] attribute with a route name defined, use that route. Note that you cannot link to a method with an unnamed [Route] attribute
* Otherwise assume that there's only a single configured route, and that route is named "API Default"

This behavior is implemented by the DefaultRouteDispatcher class. If you require different dispatching behavior, you can implement a custom IRouteDispatcher and inject it into the RouteLinker instances.
Nuget
-----
Hyprlinkr is [available via nuget](https://nuget.org/packages/Hyprlinkr)
Versioning
----------
Hyprlinkr follows [Semantic Versioning 2.0.0](http://semver.org/spec/v2.0.0.html).
Example code
------------
The *ExampleService* project, included in the source code, provides a very simple example of how to wire and use Hyprlinkr. As an example, in HomeController.cs you can see that links are added to a model instance like this:
```C#
public HomeModel Get(string id)
{
    return new HomeModel
    {
        Name = id,
        Links = new[]
        {
            new AtomLinkModel
            {
                Href = this.linker.GetUri<HomeController>(r =>
                    r.Get(id)).ToString(),
                Rel = "self"
            },
            new AtomLinkModel
            {
                Href = this.linker.GetUri<HomeController>(r =>
                    r.Get()).ToString(),
                Rel = "http://sample.ploeh.dk/rels/home"
            }
        }
    };
}
```
This produces a representation equivalent to this:
```XML
<home xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xmlns:xsd="http://www.w3.org/2001/XMLSchema"
      xmlns="http://www.ploeh.dk/hyprlinkr/sample/2012">
    <links>
        <link xmlns="http://www.w3.org/2005/Atom"
              href="http://localhost:6788/home/ploeh"
              rel="self"/>
        <link xmlns="http://www.w3.org/2005/Atom"
              href="http://localhost:6788/"
              rel="http://sample.ploeh.dk/rels/home"/>
    </links>
    <name>ploeh</name>
</home>
```
In order to run the sample application, open the Hyprlinkr.sln solution and set *ExampleService* as the startup project, then run the application by hitting F5 (or Ctrl+F5).
Credits
-------
The strongly typed Resource Linker idea was [originally presented by José F. Romaniello](http://joseoncode.com/2011/03/18/wcf-web-api-strongly-typed-resource-linker/).