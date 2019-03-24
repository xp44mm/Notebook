# Pro ASP.NET core MVC 2.0

## 2



One way to pass data from the controller to the view is by using the `ViewBag` object, which is a member of the `Controller` base class. `ViewBag` is a dynamic object to which you can assign arbitrary properties, making those values available in whatever view is subsequently rendered.

控制器:

```C#
public ViewResult Index() {
    base.ViewBag.Greeting = ...;
    return View("MyView");
}

```

视图:

```asp
<div>
     @ViewBag.Greeting ...
</div>
```

