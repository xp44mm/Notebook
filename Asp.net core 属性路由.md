# 属性路由

## 路由模板参考

如果路由找到匹配项，大括号 (`{ }`) 内的令牌定义将绑定的路由参数。 可在路由段中定义多个路由参数，但必须由文本值隔开。 例如，`{controller=Home}{action=Index}` 不是有效的路由，因为 `{controller}` 和 `{action}` 之间没有文本值。 这些路由参数必须具有名称，且可能指定了其他属性。

路由参数以外的文本（例如 `{id}`）和路径分隔符 `/` 必须匹配 URL 中的文本。 文本匹配区分大小写，并且基于 URL 路径已解码的表示形式。 若要匹配文本路由参数分隔符 `{` 或 `}`，可通过重复该字符（`{{` 或 `}}`）对其进行转义。

尝试捕获具有可选文件扩展名的文件名的 URL 模式还有其他注意事项。 例如，使用模板 `files/{filename}.{ext?}`，如果 `filename` 和 `ext` 同时存在，将同时填充这两个值。 如果 URL 中仅存在 `filename`，则路由匹配，因为尾随句点 `.` 是可选的。 以下 URL 与此路由相匹配：

* `/files/myFile.txt`
* `/files/myFile`

你可以使用 `*` 字符作为路由参数的前缀，以绑定到 URI 的其余部分，这称之为调用全方位参数。 例如，`blog/{*slug}` 将匹配以 `/blog` 开头且其后带有任何值（将分配给 `slug` 路由值）的 URI。 全方位参数还可以匹配空字符串。

路由参数可能具有指定的默认值，方法是在参数名称后使用 `=` 隔开以指定默认值。 例如，`{controller=Home}` 将 `Home` 定义为 `controller` 的默认值。 如果参数的 URL 中不存在任何值，则使用默认值。 除默认值外，路由参数可能是可选的（如 `id?` 中所示，通过将 `?` 追加到参数名称末尾）。 可选和“具有默认值”的区别在于具有默认值的路由参数始终会生成一个值，而可选参数仅当提供值时才会具有一个值。

路由参数也可能具有约束，必须匹配从 URL 中绑定的路由值。 在路由参数后面添加一个冒号 `:` 和约束名称可指定路由参数上的内联约束。 如果约束需要参数，将以在约束名称后括在括号中的形式 (`( )`) 提供。 通过追加另一个冒号 `:` 和约束名称，可指定多个内联约束。 约束名称将传递给 `IInlineConstraintResolver` 服务，以创建 `IRouteConstraint` 的实例，用于处理 URL。 例如，路由模板 `blog/{article:minlength(10)}` 使用参数 `10` 指定 `minlength` 约束。 有关路由约束的详细说明以及框架提供的约束列表，请参阅 [route-constraint-reference](https://docs.microsoft.com/zh-cn/aspnet/core/fundamentals/routing?view=aspnetcore-2.1#route-constraint-reference)。

下表演示某些路由模板及其行为。

| 路由模板                               | 匹配 URL 示例         | 说明                                                         |
| -------------------------------------- | --------------------- | ------------------------------------------------------------ |
| hello                                  | /hello                | 仅匹配单个路径 `/hello`                                      |
| {Page=Home}                            | /                     | 匹配并将 `Page` 设置为 `Home`                                |
| {Page=Home}                            | /Contact              | 匹配并将 `Page` 设置为 `Contact`                             |
| {controller}/{action}/{id?}            | /Products/List        | 映射到 `Products` 控制器和 `List`操作                        |
| {controller}/{action}/{id?}            | /Products/Details/123 | 映射到 `Products` 控制器和 `Details` 操作。 将 `id` 设置为 123 |
| {controller=Home}/{action=Index}/{id?} | /                     | 映射到 `Home` 控制器和 `Index` 方法；将忽略 `id`。           |

使用模板通常是进行路由最简单的方法。 还可在路由模板外指定约束和默认值。

提示：启用[日志记录](https://docs.microsoft.com/zh-cn/aspnet/core/fundamentals/logging/index?view=aspnetcore-2.1)查看内置路由实现（如 `Route`）如何匹配请求。

## 保留的路由名称

以下关键字是保留的名称，它们不能用作路由名称或参数：

* `action`
* `area`
* `controller`
* `handler`
* `page`

## 使用 Http[Verb] 属性的属性路由

属性路由还可以使用 `Http[Verb]` 属性，比如 `HttpPostAttribute`。 所有这些属性都可采用路由模板。此示例展示与同一路由模板匹配的两项操作：

```C#
[HttpGet("/products")]
public IActionResult ListProducts()
{
   // ...
}

[HttpPost("/products")]
public IActionResult CreateProduct(...)
{
   // ...
}
```

对于诸如 `/products` 之类的 URL 路径，当 Http 谓词为 `GET` 时将执行 `ProductsApi.ListProducts` 操作，当 Http 谓词为 `POST` 时将执行 `ProductsApi.CreateProduct`。 属性路由首先将 URL 与路由属性定义的路由模板集进行匹配。 一旦某个路由模板匹配，就会应用 `IActionConstraint` 约束来确定可以执行的操作。

---

提示

生成 REST API 时，很少会在操作方法上使用 `[Route(...)]`。 建议使用更特定的 `Http*Verb*Attributes` 来明确 API 所支持的操作。 REST API 的客户端需要知道映射到特定逻辑操作的路径和 Http 谓词。

---

由于属性路由适用于特定操作，因此，使参数变成路由模板定义中的必需参数很简单。 在此示例中，`id` 是 URL 路径中的必需参数。

```C#
public class ProductsApiController : Controller
{
   [HttpGet("/products/{id}")]
   public IActionResult GetProduct(int id) { ... }
}
```

将针对诸如 `/products/3`（而非 `/products`）之类的 URL 路径执行 `ProductsApi.GetProduct(int)` 操作。 请参阅[路由](https://docs.microsoft.com/zh-cn/aspnet/core/fundamentals/routing?view=aspnetcore-2.1)了解路由模板和相关选项的完整说明。

### 合并路由

若要使属性路由减少重复，可将控制器上的路由属性与各个操作上的路由属性合并。 控制器上定义的所有路由模板均作为操作上路由模板的前缀。 在控制器上放置路由属性会使控制器中的**所有**操作都使用属性路由。

```C#
[Route("products")]
public class ProductsApiController : Controller
{
   [HttpGet]
   public IActionResult ListProducts() { ... }

   [HttpGet("{id}")]
   public ActionResult GetProduct(int id) { ... }
}
```

在此示例中，URL 路径 `/products` 可以匹配 `ProductsApi.ListProducts`，URL 路径 `/products/5` 可以匹配 `ProductsApi.GetProduct(int)`。 这两项操作仅匹配 HTTP GET，因为它们用 `HttpGetAttribute`修饰。

应用于操作的以 `/` 开头的路由模板不与应用于控制器的路由模板合并。 此示例匹配一组与*默认路由*类似的 URL 路径。

```C#
[Route("Home")]
public class HomeController : Controller
{
    [Route("")]      // Combines to define the route template "Home"
    [Route("Index")] // Combines to define the route template "Home/Index"
    [Route("/")]     // Doesn't combine, defines the route template ""
    public IActionResult Index()
    {
    }

    [Route("About")] // Combines to define the route template "Home/About"
    public IActionResult About()
    {
    }
}
```

## 路由模板中的标记替换（[controller]、[action]、[area]）

为方便起见，属性路由支持*标记替换*，方法是将标记用方括号（`[`、`]`）括起来。 标记 `[action]`、`[area]` 和 `[controller]` 将替换为定义了路由的操作中的操作名称值、区域名称值和控制器名称值。在此示例中，操作可以与注释中所述的 URL 路径匹配：

```C#
[Route("[controller]/[action]")]
public class ProductsController : Controller
{
    [HttpGet] // Matches '/Products/List'
    public IActionResult List() {
        // ...
    }

    [HttpGet("{id}")] // Matches '/Products/Edit/{id}'
    public IActionResult Edit(int id) {
        // ...
    }
}
```

标记替换发生在属性路由生成的最后一步。 上述示例的行为方式将与以下代码相同：

```C#
public class ProductsController : Controller
{
    [HttpGet("[controller]/[action]")] // Matches '/Products/List'
    public IActionResult List() {
        // ...
    }

    [HttpGet("[controller]/[action]/{id}")] // Matches '/Products/Edit/{id}'
    public IActionResult Edit(int id) {
        // ...
    }
}
```

属性路由还可以与继承结合使用。 与标记替换结合使用时尤为强大。

```C#
[Route("api/[controller]")]
public abstract class MyBaseController : Controller { ... }

public class ProductsController : MyBaseController
{
   [HttpGet] // Matches '/api/Products'
   public IActionResult List() { ... }

   [HttpPut("{id}")] // Matches '/api/Products/{id}'
   public IActionResult Edit(int id) { ... }
}
```

若要匹配文本标记替换分隔符 `[` 或 `]`，可通过重复该字符（`[[` 或 `]]`）对其进行转义。

### 绑定源参数推理

绑定源特性定义可找到操作参数值的位置。 存在以下绑定源特性：

| 特性               | 绑定源                     |
| ------------------ | -------------------------- |
| **[FromBody]**     | 请求正文                   |
| **[FromForm]**     | 请求正文中的表单数据       |
| **[FromHeader]**   | 请求标头                   |
| **[FromQuery]**    | 请求查询字符串参数         |
| **[FromRoute]**    | 当前请求中的路由数据       |
| **[FromServices]** | 作为操作参数插入的请求服务 |

---

警告

当值可能包含 `%2f`（即 `/`）时，请勿使用 `[FromRoute]`。 `%2f` 不会转换为 `/`（非转移形式）。 如果值可能包含 `%2f`，则使用 `[FromQuery]`。

---

没有 `[ApiController]` 特性时，将显式定义绑定源特性。 在下面的示例中，`[FromQuery]` 特性指示 `discontinuedOnly` 参数值在请求 URL 的查询字符串中提供：

C#复制

```
[HttpGet]
public async Task<ActionResult<List<Product>>> GetAsync(
    [FromQuery] bool discontinuedOnly = false)
{
    List<Product> products = null;

    if (discontinuedOnly)
    {
        products = await _repository.GetDiscontinuedProductsAsync();
    }
    else
    {
        products = await _repository.GetProductsAsync();
    }

    return products;
}
```

推理规则应用于操作参数的默认数据源。 这些规则将配置绑定资源，否则你可以手动应用操作参数。绑定源特性的行为如下：

* **[FromBody]**，针对复杂类型参数进行推断。 此规则不适用于具有特殊含义的任何复杂的内置类型，如 [IFormCollection](https://docs.microsoft.com/zh-cn/dotnet/api/microsoft.aspnetcore.http.iformcollection) 和 [CancellationToken](https://docs.microsoft.com/zh-cn/dotnet/api/system.threading.cancellationtoken)。 绑定源推理代码将忽略这些特殊类型。 当操作中的多个参数为显式指定（通过 `[FromBody]`）或在请求正文中作为绑定进行推断时，将会引发异常。 例如，下面的操作签名会导致异常：

C#复制

```
// Don't do this. All of the following actions result in an exception.
[HttpPost]
public IActionResult Action1(Product product, 
                             Order order) => null;

[HttpPost]
public IActionResult Action2(Product product, 
                             [FromBody] Order order) => null;

[HttpPost]
public IActionResult Action3([FromBody] Product product, 
                             [FromBody] Order order) => null;
```

* **[FromForm]**，针对 [IFormFile](https://docs.microsoft.com/zh-cn/dotnet/api/microsoft.aspnetcore.http.iformfile) 和 [IFormFileCollection](https://docs.microsoft.com/zh-cn/dotnet/api/microsoft.aspnetcore.http.iformfilecollection) 类型的操作参数进行推断。 该特性不针对任何简单类型或用户定义类型进行推断。
* **[FromRoute]**，针对与路由模板中的参数相匹配的任何操作参数名称进行推断。 当多个路由与一个操作参数匹配时，任何路由值都视为 `[FromRoute]`。
* **[FromQuery]**，针对任何其他操作参数进行推断。

当 [SuppressInferBindingSourcesForParameters](https://docs.microsoft.com/zh-cn/dotnet/api/microsoft.aspnetcore.mvc.apibehavioroptions.suppressinferbindingsourcesforparameters) 属性设为 `true` 时，会禁用默认推理规则。 将以下代码添加至 Startup.ConfigureServices 中的 `services.AddMvc().SetCompatibilityVersion(CompatibilityVersion.Version_2_1);` 后：

C#复制

```
services.Configure<ApiBehaviorOptions>(options =>
{
    options.SuppressConsumesConstraintForFormFileParameters = true;
    options.SuppressInferBindingSourcesForParameters = true;
    options.SuppressModelStateInvalidFilter = true;
});
```