本文来自Expert Asp.net web api 2 for mvc dev第14\~16章

# 准备框架

新建名为`ExampleApp`的项目。

![](ExampleApp-新建项目.png)

选择空模板，选中MVC和Web API，按确定生成项目。

![](ExampleApp-新建应用程序.png)

`update-package`，并编译。

#### \\Views\\\_ViewStart.cshtml

```c#
@{
	Layout = "\~/Views/Shared/\_Layout.cshtml";
}
```
#### \\Views\\Shared\\\_Layout.cshtml
```php+HTML
<!DOCTYPE html>

<html>
<head>
    <meta name="viewport" content="width=device-width" />
    <link href="https://cdn.bootcss.com/bootstrap/3.3.7/css/bootstrap.css" rel="stylesheet">
    <script src="https://cdn.bootcss.com/jquery/2.2.4/jquery.js"></script>
    <script src="https://cdn.bootcss.com/bootstrap/3.3.7/js/bootstrap.js"></script>
    <script src="https://cdn.bootcss.com/knockout/3.4.2/knockout-debug.js"></script>
    <title>@ViewBag.Title</title>

    @RenderSection("scripts", required: false)
</head>
<body>
    @RenderBody()
</body>
</html>
```

#### \\App\_Start\\WebApiConfig.cs

```C#
using System.Net.Http;
using System.Web.Http;
using System.Web.Http.Controllers;

namespace ExampleApp
{
    public static class WebApiConfig
    {
        public static void Register(HttpConfiguration config)
        {

            // Web API 路由
            config.MapHttpAttributeRoutes();

            config.Routes.MapHttpRoute(
                name: "DefaultApi",
                routeTemplate: "api/{controller}/{action}"
            );

            config.ParameterBindingRules.Insert(0, x =>
            {
                //非get方法，且只有一个参数
                if (!x.ActionDescriptor.SupportedHttpMethods.Contains(HttpMethod.Get)
                && x.ActionDescriptor.GetParameters().Count == 1)
                {
                    return x.BindWithAttribute(new FromBodyAttribute());
                }
                else
                {
                    return x.BindWithAttribute(new FromUriAttribute());
                }

            });
        }
    }
}

```

#### \\Views\\Home\\bindings.cshtml

```html
@{ ViewBag.Title = "Bindings"; }

<div class="alert alert-success" data-bind="css: { 'alert-danger': gotError }">
    <span data-bind="text: response"></span>
</div>

<div class="form-group">
    <label>First Number</label>
    <input class="form-control" data-bind="value: first" />
</div>

<div class="form-group">
    <label>Second Number</label>
    <input class="form-control" data-bind="value: second" />
</div>

<div class="form-group">
    <label>Third Number</label>
    <input class="form-control" data-bind="value: third" />
</div>

<div class="form-group">
    <label>Fourth Number</label>
    <input class="form-control" data-bind="value: fourth" />
</div>

<div class="form-group">
    <label>add or minus</label>
    <input type="checkbox" class="form-control" data-bind="checked: add" />
</div>

<div class="form-group">
    <label>is double or single</label>
    <input type="checkbox" class="form-control" data-bind="checked: double" />
</div>

<button class="btn btn-primary" data-bind="click: SumNumbers">SumNumbers</button>
<button class="btn btn-primary" data-bind="click: singleObject">singleObject</button>
<button class="btn btn-primary" data-bind="click: multipleObjects">multipleObjects</button>
<button class="btn btn-primary" data-bind="click: nestedObjects">nestedObjects</button>
<button class="btn btn-primary" data-bind="click: simpleArray">simpleArray</button>
<button class="btn btn-primary" data-bind="click: complexArray">complexArray</button>
<button class="btn btn-primary" data-bind="click: kvpairs">kvpairs</button>

<script src="~/Scripts/bindings.js"></script>

```

#### \\Scripts\\bindings.js

```javascript
let response = ko.observable("Ready")
let gotError = ko.observable(false)

function request(url, data) {
    $.getJSON(url, data)
        .done((data) => {
            gotError(false)
            response(JSON.stringify(data))
        })
        .fail((jqxhr, textStatus, error) => {
            gotError(true)
            response(`Request Failed: ${textStatus}, ${error}`)
        })
}

function BindingsViewModel() {
    this.first = ko.observable(2)
    this.second = ko.observable(5)
    this.third = ko.observable(10)
    this.fourth = ko.observable(100)
    this.add = ko.observable(true)
    this.double = ko.observable(false)
}

const vm = new BindingsViewModel()

vm.SumNumbers = function () {
    request(
        "/api/bindings/SumNumbers",
        {
            first: this.first(),
            second: this.second(),
        })
}

vm.singleObject = function () {
    request(
        "/api/bindings/singleObject",
        {
            numbers: {
                first: this.first(),
                second: this.second(),
            }
        })
}

vm.multipleObjects = function () {
    request(
        "/api/bindings/multipleObjects",
        {
            numbers1: {
                first: this.first(), second: this.second(),
            },
            numbers2: {
                first: this.third(), second: this.fourth(),
            },
        })
}

vm.nestedObjects = function () {
    request(
        "/api/bindings/nestedObjects",
        {
            calc: {
                first: this.first(),
                second: this.second(),
                op: {
                    add: this.add(),
                    double: this.double(),
                },
            }
        })
}

vm.simpleArray = function () {
    request(
        "/api/bindings/simpleArray",
        {
            numbers: [this.first(), this.second(), this.third(), this.fourth(),]
        })
}

vm.complexArray = function () {
    request(
        "/api/bindings/complexArray",
        {
            numbers: [
                { first: this.first(), second: this.second(), },
                { first: this.third(), second: this.fourth(), },]
        })
}

vm.kvpairs = function () {
    request(
        "/api/bindings/kvpairs",
        {
            numbers: [
                {
                    key: "one", value: { first: this.first(), second: this.second() }
                },
                {
                    key: "two", value: { first: this.third(), second: this.fourth() }
                }]
        })
}

ko.applyBindings(vm)
```

## WebApi项目

项目名称：ExampleApp.Ground

项目类型：Visual F\#$\Rightarrow$Windows$\Rightarrow$库

为本项目balance.webApi安装NuGet程序包：

```powershell
Install-package Microsoft.Aspnet.webApi
Install-package Fsharp.core
```

添加项目引用：程序集$\Rightarrow$框架$\Rightarrow$System.Web

从ExampleApp项目添加对本项目ExampleApp.Ground的引用。

#### \\ExampleApp.Ground\\BindingsController.fs

```F#
namespace ExampleApp.Ground
open ExampleApp.Ground.Models

open System.Web.Http
open System.Collections.Generic

type BindingsController() =
    inherit ApiController()
    member private this.tojson result = base.Json(result)

    [<HttpGet>]
    member this.SumNumbers(first:int,  second:int)= first + second |> this.tojson

    [<HttpGet>]
    member this.singleObject(numbers:Numbers) = numbers |> this.tojson

    [<HttpGet>]
    member this.multipleObjects(numbers1:Numbers,  numbers2:Numbers) = [|numbers1;numbers2|] |> this.tojson

    [<HttpGet>]
    member this.nestedObjects(calc:Calculation) = calc |> this.tojson

    [<HttpGet>]
    member this.simpleArray(numbers:int[]) = numbers |> this.tojson

    [<HttpGet>]
    member this.complexArray(numbers:Numbers[]) = numbers |> this.tojson

    [<HttpGet>]
    member this.kvpairs(numbers:Dictionary<string, Numbers>) = numbers |> this.tojson
```

#### \\ExampleApp.Ground\\Models.fs

```F#
namespace ExampleApp.Ground.Models

type Numbers() =
    member val First = 0 with get,set
    member val Second = 0 with get,set

type Operation() =
    member val Add = false with get,set
    member val Double = false with get,set

type Calculation() =
    member val First = 0 with get,set
    member val Second = 0 with get,set
    member val Op = new Operation() with get,set
```

# 客户端请求

## 参数绑定

添加按钮点击代码：

vm.SumNumbers = function () {

request(

"/api/bindings/SumNumbers",

{

first: this.first(),

second: this.second(),

})

}

客户端请求发送的查询字符串：

![](media/image3.png){width="4.1875in" height="0.7604166666666666in"}

## Binding Objects

添加javascript代码

vm.singleObject = function () {

request(

"/api/bindings/singleObject",

{

numbers: {

first: this.first(),

second: this.second(),

}

})

}

客户端请求发送的查询字符串

![](media/image4.png){width="4.260416666666667in" height="0.78125in"}

## Binding Multiple Objects

添加javascript代码

vm.multipleObjects = function () {

request(

"/api/bindings/multipleObjects",

{

numbers1: {

first: this.first(), second: this.second(),

},

numbers2: {

first: this.third(), second: this.fourth(),

},

})

}

客户端请求发送的查询字符串

![](media/image5.png){width="4.510416666666667in" height="1.0833333333333333in"}

## Binding Nested Objects

添加javascript代码

vm.nestedObjects = function () {

request(

"/api/bindings/nestedObjects",

{

calc: {

first: this.first(),

second: this.second(),

op: {

add: this.add(),

double: this.double(),

},

}

})

}

客户端请求发送的查询字符串

![](media/image6.png){width="4.4375in" height="1.1145833333333333in"}

## Binding Collections and Arrays

you can also bind to 强类型的 Enumerable, such as Enumerable&lt;T&gt;, by changing操作方法参数的类型，这里以数组为例

添加javascript代码

vm.simpleArray = function () {

request(

"/api/bindings/simpleArray",

{

numbers: \[this.first(), this.second(), this.third(), this.fourth(),\]

})

}

客户端请求发送的查询字符串

![](media/image7.png){width="4.375in" height="1.0625in"}

## Binding Arrays and Lists of Complex Types

添加javascript代码

vm.complexArray = function () {

request(

"/api/bindings/complexArray",

{

numbers: \[

{ first: this.first(), second: this.second(), },

{ first: this.third(), second: this.fourth(), },\]

})

}

客户端请求发送的查询字符串

![](media/image8.png){width="4.4375in" height="1.0625in"}

## Binding Key-Value Pairs

添加javascript代码

vm.kvpairs = function () {

request(

"/api/bindings/kvpairs",

{

numbers: \[

{

key: "one", value: { first: this.first(), second: this.second() }

},

{

key: "two", value: { first: this.third(), second: this.fourth() }

}\]

})

}

客户端请求发送的查询字符串

![](media/image9.png){width="4.71875in" height="1.4375in"}

# 总结

参数绑定、模型绑定不区分大小写

模型绑定可以省略最顶层的变量名。

# 原来的笔记

# Understanding the Default Binding Behavior

参数绑定和模型绑定都是从请求中提取数据，以用作操作方法的参数。这样，你就可以定义接受.net类型的操作方法，而将如何获取参数值交由WebApi处理。

参数和模型绑定使操作方法参数的提取方式一致。使代码在程序中通用。不限制使用任何一种绑定方式。

如果需要也可以直接使用HttpRequestMessage。

参数绑定从URL中定位数值到简单.net类型。模型绑定负责创建复杂.net类型，只从请求体获取。

## Understanding Parameter Binding

从Url中，两个来源：路由片段、查询字符串。一般建议使用后者，因为jQuery等工具负责将javaScript对象拼接成查询字符串。

### Understanding the Parameter Binding Pitfall

参数绑定默认从Url中获取数据。

jQuery会将非Get请求的数据放到请求体中，可以用下面的方法将数据放置到查询字符串中：

\$.ajax("/api/bindings/sumnumbers?" + \$.param(data), {

## Understanding Model Binding

模型绑定用于复杂类型，模型绑定只工作在请求体内。

WebApi首先根据url找到相应的操作方法，然后，根据操作方法获取其参数表。

控制器操作方法定义：

\[HttpPost\]

public int SumNumbers(Numbers calc)

客户端提供的数据：

\$.ajax("/api/bindings/sumnumbers", {

method: "POST",

data: { first: 2, second: 5 },

})

WebApi负责综合以上信息为操作方法提供实际参数：

calc = new Numbers() { First = 2, Second = 5 };

由此可见，客户端不必提供形式参数的命名和数据类型。只需提供属性对应的值，属性名称忽略大小写。

### Understanding the Model Binding Pitfall

模型绑定默认从请求体中提取数据。

模型绑定只能从请求体中提取一个对象。

请求中的数据是名值对集合，可以创造同样数据项目的不同种对象。

# Performing Binding Customizations

## Binding Complex Types from the Request URL

\[FromUri\]

## Binding Simple Types from the Request Body

\[FromBody\]

## Defining a Binding Rule

绑定规则：告诉WebApi如何贯穿应用程序绑定给定类型的参数。

config.ParameterBindingRules.Insert(0, typeof(Numbers),

x =&gt; x.BindWithAttribute(new FromUriAttribute()));

# Working with Value Providers and Value Provider Factories

值提供器负责获得一单个的简单的数据值。给值提供器一个数据项目的名字，返回此数据项目的值。数据项目的名字依赖于上下文。对于参数绑定，将是参数的名字。对于模型绑定，将是参数属性的名字。

值提供器工厂会根据操作方法的参数创造一个合适的值提供器的实例。

# 模型绑定实例

本项目演示asp.net内置的参数绑定与模型绑定，分为参数绑定，绑定对象，绑定多个对象，绑定嵌套对象，绑定数组或集合，绑定复杂类型的数组，绑定词典。

## 新建项目

![](media/image10.png){width="8.1875in" height="6.385416666666667in"}

Install-package knockout

BundleConfig.cs

bundles.Add(new ScriptBundle("\~/bundles/knockout").Include(

"\~/Scripts/knockout-{version}.js"));

\_Layout.cshtml

&lt;!DOCTYPE html&gt;

&lt;html&gt;

&lt;head&gt;

&lt;meta http-equiv="Content-Type" content="text/html; charset=utf-8" /&gt;

&lt;meta charset="utf-8" /&gt;

&lt;meta name="viewport" content="width=device-width" /&gt;

&lt;title&gt;@ViewBag.Title&lt;/title&gt;

@Styles.Render("\~/Content/css")

@Scripts.Render("\~/bundles/modernizr")

@Scripts.Render("\~/bundles/jquery")

@Scripts.Render("\~/bundles/bootstrap")

@Scripts.Render("\~/bundles/knockout")

@RenderSection("scripts", required: false)

&lt;/head&gt;

&lt;body&gt;

@RenderBody()

&lt;script&gt;

\$(document).ready(function () {

ko.applyBindings();

});

&lt;/script&gt;

&lt;/body&gt;

&lt;/html&gt;

模型类：

namespace ModelBinders.Models

{

public class Calculation

{

public Numbers Nums { get; set; }

public bool Add { get; set; }

public bool Double { get; set; }

}

public class Numbers

{

public int First { get; set; }

public int Second { get; set; }

}

}

## 参数绑定

HomeController.cs

public ActionResult ParameterBinders() =&gt; View();

BindingsController.cs

\[HttpGet\]

public Numbers ParameterBinders(int first,int second)

{

return new Numbers { First = first, Second = second };

}

ParameterBinders.cshtml

&lt;div class="alert alert-success" data-bind="css: { 'alert-danger': gotError }"&gt;

&lt;span data-bind="text: response"&gt;&lt;/span&gt;

&lt;/div&gt;

&lt;button class="btn btn-primary" onclick="sendRequest()"&gt;Send Request&lt;/button&gt;

&lt;script&gt;

var response = ko.observable("Ready");

var gotError = ko.observable(false);

var sendRequest = function () {

\$.ajax("/api/bindings/@ViewBag.Title", {

data: {

first: 2, second: 5

},

})

.done(function (data) {

gotError(false);

response("Total: " + JSON.stringify(data));

})

.fail(function (jqXHR) {

gotError(true);

response(jqXHR.status + " (" + jqXHR.statusText + ")");

})

};

&lt;/script&gt;

运行项目：

/Home/ParameterBinders

点击发送按钮：

WebForms:

  Name     Value
  -------- -------
  first    2
  second   5

返回的数据：{"First":2,"Second":5}

以下的演示为了简便起见，只显示数据。

## 绑定对象

HomeController.cs

public ActionResult Objects() =&gt; View();

BindingsController.cs

\[HttpGet\]

public Numbers Objects(Numbers nums)

{

return nums;

}

客户端数据：

data: {

first: 2, second: 5

},

WebForms:

  Name     Value
  -------- -------
  first    2
  second   5

返回的数据: {"First":2,"Second":5}

## 绑定多个对象

HomeController.cs

public ActionResult MultiObjects() =&gt; View();

BindingsController.cs

\[HttpGet\]

public Numbers\[\] MultiObjects(Numbers numbers1, Numbers numbers2)

{

return new\[\] { numbers1, numbers2 };

}

客户端数据（两种格式是等价的）：

var data1 = {

"numbers1.first": 2,

"numbers1.second": 5,

"numbers2.first": 10,

"numbers2.second": 100

}

var data2 = {

numbers1: { first: 2, second: 5 },

numbers2: { first: 10, second: 100 },

}

WebForms:

Data1对应：

  Name              Value
  ----------------- -------
  numbers1.first    2
  numbers1.second   5
  numbers2.first    10
  numbers2.second   100

Data2对应：

  Name                 Value
  -------------------- -------
  numbers1\[first\]    2
  numbers1\[second\]   5
  numbers2\[first\]    10
  numbers2\[second\]   100

返回的数据: \[{"First":2,"Second":5},{"First":10,"Second":100}\]

## 绑定嵌套对象

HomeController.cs

public ActionResult NestedObjects() =&gt; View();

BindingsController.cs

\[HttpGet\]

public Calculation NestedObjects(Calculation calc)

{

return calc;

}

客户端数据（两种格式是等价的）：

var data1 = {

"calc.nums.first": 2,

"calc.nums.second": 5,

"calc.add": true,

"calc.double": true

}

var data2 = {

calc: {

nums: { first: 2, second: 5 },

add: true,

double: true,

}

}

WebForms:

Data1对应：

  Name               Value
  ------------------ -------
  calc.add           TRUE
  calc.double        TRUE
  calc.nums.first    2
  calc.nums.second   5

Data2对应：

  Name                     Value
  ------------------------ -------
  calc\[add\]              TRUE
  calc\[double\]           TRUE
  calc\[nums\]\[first\]    2
  calc\[nums\]\[second\]   5

返回的数据: {"Nums":{"First":2,"Second":5},"Add":true,"Double":true}

## 绑定数组（集合）

HomeController.cs

public ActionResult Arrays() =&gt; View();

BindingsController.cs

\[HttpGet\]

public int\[\] Arrays(int\[\] numbers)

{

return numbers;

}

客户端数据：

var data1 = {

numbers: \[2, 5, 100\]

}

WebForms:

  Name          Value
  ------------- -------
  numbers\[\]   2
  numbers\[\]   5
  numbers\[\]   100

返回的数据: \[2,5,100\]

模型绑定将各种泛型集合都当作数组对待。比如Enumerable&lt;T&gt;、List&lt;T&gt;都等价于T\[\]

## 绑定复杂类型的数组

HomeController.cs

public ActionResult Dictionaries() =&gt; View();

BindingsController.cs

\[HttpGet\]

public Dictionary&lt;string, Numbers&gt; Dictionaries(Dictionary&lt;string, Numbers&gt; numbers)

{

return numbers;

}

客户端数据：

var data1 = {

numbers: \[

{ first: 2, second: 5 },

{ first: 100, second: 200 }\]

}

WebForms:

  Name                     Value
  ------------------------ -------
  numbers\[0\]\[first\]    2
  numbers\[0\]\[second\]   5
  numbers\[1\]\[first\]    100
  numbers\[1\]\[second\]   200

返回的数据: \[{"First":2,"Second":5},{"First":100,"Second":200}\]

## 绑定词典

HomeController.cs

public ActionResult Dictionaries() =&gt; View();

BindingsController.cs

\[HttpGet\]

public Dictionary&lt;string, Numbers&gt; Dictionaries(Dictionary&lt;string, Numbers&gt; numbers)

{

return numbers;

}

客户端数据：

var data1 = {

numbers: \[

{ key: "one", value: { first: 2, second: 5 } },

{ key: "two", value: { first: 100, second: 200 } }\]

}

WebForms:

  Name                              Value
  --------------------------------- -------
  numbers\[0\]\[key\]               one
  numbers\[0\]\[value\]\[first\]    2
  numbers\[0\]\[value\]\[second\]   5
  numbers\[1\]\[key\]               two
  numbers\[1\]\[value\]\[first\]    100
  numbers\[1\]\[value\]\[second\]   200

返回的数据: {"one":{"First":2,"Second":5},"two":{"First":100,"Second":200}}
