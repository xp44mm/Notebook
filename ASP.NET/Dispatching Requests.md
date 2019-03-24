# Dispatching Requests

## Preparing the Example Project

新建项目$\rightarrow$ASP.NET Web 应用程序，名称：`Dispatch`，确定。

使用空模板，选中引用：MVC，和Web API，确定。

在程序包管理器控制台，输入命令：

```PowerShell
Update-Package
```

### Creating the Model Class

\Models\Product.cs

```C#
namespace Dispatch.Models
{
    public class Product
    {
        public int ProductID { get; set; }
        public string Name { get; set; }
        public decimal Price { get; set; }
    }
}
```

### Creating the Web API Web Service

\Controllers\ProductsController.cs

```C#
using System.Collections.Generic;
using System.Linq;
using System.Web.Http;
using Dispatch.Models;

namespace Dispatch.Controllers
{
    public class ProductsController : ApiController
    {
        private static List<Product> products = new List<Product> {
                new Product {ProductID = 1, Name = "Kayak", Price = 275M },
                new Product {ProductID = 2, Name = "Lifejacket", Price = 48.95M },
                new Product {ProductID = 3, Name = "Soccer Ball", Price = 19.50M },
                new Product {ProductID = 4, Name = "Thinking Cap", Price = 16M },
            };


        public IEnumerable<Product> Get()
        {
            return products;
        }

        public Product Get(int id)
        {
            return products.Where(x => x.ProductID == id).FirstOrDefault();
        }

        public Product Post(Product product)
        {
            product.ProductID = products.Count + 1;
            products.Add(product);
            return product;
        }
    }
}
```

### Creating the MVC Controller and View

\Controllers\HomeController.cs

```C#
using System.Web.Mvc;

namespace Dispatch.Controllers
{
    public class HomeController : Controller
    {
        // GET: Home
        public ActionResult Index()
        {
            return View();
        }
    }
}
```

\Views\Home\Index.cshtml

```HTML
@{ Layout = null;}
<!DOCTYPE html>
<html>
<head>
    <meta name="viewport" content="width=device-width" />
    <link href="https://cdn.bootcss.com/bootstrap/3.3.7/css/bootstrap.min.css" rel="stylesheet">
    <script src="https://cdn.bootcss.com/jquery/3.2.1/jquery.min.js"></script>
    <script src="https://cdn.bootcss.com/knockout/3.4.2/knockout-min.js"></script>

    <title>Index</title>

    <script src="~/Scripts/dispatch.js"></script>
    <style>
        body {
            padding-top: 10px;
        }
    </style>
</head>
<body class="container">
    <div class="alert alert-success" data-bind="css: { 'alert-danger': gotError }">
        <span data-bind="text: response()"></span>
    </div>

    <div class="panel panel-primary">
        <div class="panel-heading">Products</div>
        <table class="table table-striped">
            <thead>
                <tr><th>ID</th><th>Name</th><th>Price</th></tr>
            </thead>
            <tbody data-bind="foreach: products">
                <tr>
                    <td data-bind="text: ProductID"></td>
                    <td data-bind="text: Name"></td>
                    <td data-bind="text: Price"></td>
                </tr>
            </tbody>
        </table>
    </div>

    <button class="btn btn-primary" data-bind="click: getAll">Get All</button>
    <button class="btn btn-primary" data-bind="click: getOne">Get One</button>
    <button class="btn btn-primary" data-bind="click: postOne">Post</button>
</body>
</html>
```

\Scripts\dispatch.js

```javascript
var viewModel = ko.observable({
    productId: 100, name: "Bananas", price: 12.34
});

var products = ko.observableArray();
var response = ko.observable("Ready");
var gotError = ko.observable(false);

var getAll = function () {
    sendRequest("GET");
}

var getOne = function () {
    sendRequest("GET", 2);
}

var postOne = function () {
    sendRequest("POST");
}

var sendRequest = function (verb, id) {
    var url = "/api/products/" + (id || "");

    var config = {
        type: verb || "GET",
        data: verb == "POST" ? viewModel() : null,
        success: function (data) {
            gotError(false);
            response("Success");
            products.removeAll();
            if (Array.isArray(data)) {
                data.forEach(function (product) {
                    products.push(product);
                });
            } else {
                products.push(data);
            }
        },
        error: function (jqXHR) {
            gotError(true);
            products.removeAll();
            response(jqXHR.status + " (" + jqXHR.statusText + ")");
        }
    }

    $.ajax(url, config);
};
$(document).ready(function () {
    ko.applyBindings();
});
```

### Testing the Example Application

## Understanding Request Dispatching

