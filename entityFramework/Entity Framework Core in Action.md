# Entity Framework Core in Action

源代码地址：

### 1.6.2  Creating your own .NET Core console app with EF Core

Creating a .net Core Console application

1  In the top menu of VS 2017, click File > New > Project to open the New Project form.
2  From  the  installed  templates,  select  Visual  F#  >  .NET  Core  >  Console  App (.NET Core).
3  Type in the name of your program (in this case, *MyFirstEfCoreApp*) and make sure the location is sensible.

安装NuGet程序包`Microsoft.EntityFrameworkCore.SqlServer`

### 1.8.2  The application's DbContext

```C#
public class AppDbContext : DbContext
{
    private const System.String ConnectionString =
        @"Server=.;Database=MyFirstEfCoreDb;Trusted_Connection=True";

    protected override void OnConfiguring(
        DbContextOptionsBuilder optionsBuilder)
    {
        optionsBuilder.UseSqlServer(ConnectionString);
    }

    public DbSet<Book> Books { get; set; }
}
```

In a console application, you configure EF Core's database options by overriding the `OnConfiguring` method. In this case you tell it you're using an SQL Server database by using the `UseSqlServer` method. 

用法：

```F#
use db = new AppDbContext()
```

### 1.9.1  Modeling the database

从代码生成数据库：

```F#
/// <summary>
/// This will wipe and create a new database - which takes some time
/// </summary>
/// <param name="onlyIfNoDatabase">If true it will not do anything if the database exists</param>
/// <returns>returns true if database database was created</returns>
let WipeCreateSeed(onlyIfNoDatabase:bool) =
    use db = new AppDbContext()

    let dbExists () =
        let creator = db.GetService<IDatabaseCreator>() :?> RelationalDatabaseCreator
        creator.Exists()

    if onlyIfNoDatabase && dbExists() then
        false
    else
        db.Database.EnsureDeleted() |> ignore
        db.Database.EnsureCreated() |> ignore
        if not <| db.Books.Any() then
            WriteTestData(db)
            printfn "Seeded database"
        true
```

这个方法有两种用法，第一种强制重建数据库，不管数据库是否存在。第二种只有数据库不存在时，才建立数据库。前者通过设置`onlyIfNoDatabase`输入参数为假，后者为真。

这是命令行摘要：

```F#
[<EntryPoint>]
let main argv =
    Console.WriteLine(
        if Commands.WipeCreateSeed(true) then 
            "created database and seeded it." 
        else "it exists.")

    let mutable looping = true
    while looping do
        Console.Write("> ");
        let command = Console.ReadLine();
        match command with
        | "r" ->
                Commands.WipeCreateSeed(false) |> ignore
        | "e" ->
                looping <- false
        | _ ->
                Console.WriteLine("Unknown command.")
    0
```

程序一启动，就检查数据库是否存在，如果存在就略过，否则就创建数据库。

命令行等待命令输入，当输入`r`时，会删除现有数据库，强制重新创建数据库。

### 1.9.2  Reading data from the database

`AsNoTracking`

只读访问数据库，即，读取数据后，不需要使用`SaveChanges`。

`Include`

读取的内容包括外键对应的实体，生成`JOIN`连接语句的查询。

顺便提一下，MyFirstEfCoreApp包含`ILoggerProvider`的示例代码。

### 2.2.2  Creating an instance of the application's DbContext

数据库上下文的构造函数：

```C#
public class EfCoreContext : DbContext
{
    public EfCoreContext(                           
        DbContextOptions<EfCoreContext> options)
        : base(options) { }                           
}
```

This constructor is how the ASP.NET creates an instance of `EfCoreContext`

 

构造`options`的方法：

```F#
open Microsoft.EntityFrameworkCore

let buildOptions<'t when 't:> DbContext> (connection:string) =
    let optionsBuilder = new DbContextOptionsBuilder<'t>()
    optionsBuilder.UseSqlServer(connection) |> ignore
    optionsBuilder.Options
```

Creating an instance of the application's `DbContext` to access the database

源代码位于@EfCoreInAction\Test\UnitTests\DataLayer\Ch02_CreateDbContext.cs

测试`buildOptions`用法：

```F#
let Ch02_CreateDbContext() =
    let connection = @"Server=.;Database=MyFirstEfCoreDb;Trusted_Connection=True";
    let options = buildOptions<EfCoreContext> connection
    use context = new EfCoreContext(options)
```

### 在单元测试中使用配置文件

假设提供了`appsettings.json`配置文件，位于源代码目录下。

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=.;Database=Test.EfCoreInActionDb;Trusted_Connection=True;MultipleActiveResultSets=true"
  }
}
```

首先是如何读取配置文件，并从配置文件读取连接字符串。可以参考文件 [配置.md](..\配置.md) 。

为了便于测试，我们可以为每一个测试类，甚至每一个测试方法，在每一个分支下创建自己专属的数据库，这些数据库除了数据库的名称外，其他配置完全相同。

下面的正则表达式匹配数据库连接字符串中的数据库名部分。

```F#
let rgxDbName = Regex(@"\bDatabase=([^;]+)", RegexOptions.IgnoreCase)
```

然后利用正则表达式，修改数据库名称，给现有名称加扩展名。

```F#
let uniqueConnectionString = rgxDbName.Replace(connectionString, "$&." + suffix)
```

正则表达式参考：https://docs.microsoft.com/zh-cn/dotnet/standard/base-types/regular-expression-language-quick-reference

有了连接字符串，下一步是`DbContext`构造所需的输入参数`DbContextOptions<DbContext>`：

```F#
// private
let getOptionsBuilder<'t when 't :> DbContext> (connectString:string) =
    let optionsBuilder = new DbContextOptionsBuilder<'t>()
    optionsBuilder.UseSqlServer(connectString) |> ignore
    optionsBuilder

// 
let getOptions<'t when 't :> DbContext> (connectString:string) =
    let optionsBuilder = getOptionsBuilder<'t> connectString
    optionsBuilder.Options
```

获取连接字符串，以及获取数据库上下文选项的方法：

```F#
type Test(output:ITestOutputHelper) =
    member this.MyConnectionString = 
        uniqueConnectionString connectionString (this.GetType().Name)
    
    member this.MyOptions =
        let builder = getOptionsBuilder<EfCoreContext> this.MyConnectionString
        let options = builder.Options
        options
```

附加测试类的类型名作为其专用的数据库名称，然后利用这个名称创建上下文。

有了选项，就可以创建数据库上下文了：

```F#
type Test(output:ITestOutputHelper) =
    member this.MyConnectionString = 
        uniqueConnectionString connectionString (this.GetType().Name)
    
    member this.MyOptions =
        let builder = getOptionsBuilder<EfCoreContext> this.MyConnectionString
        let options = builder.Options
        options
        
    [<Fact>]
    member this.TestOk()=
        let options = this.MyOptions
        use context = new EfCoreContext(options)
```

注意数据库上下文占用数据库连接，声明需要使用`use`代替`let`。

### 数据库初始化

有上下文，就要有数据库，需要填充一下初始化的数据等，常用两种方法，一种是无论是否存在数据库，都从头开始建立数据库并填充数据。也就是先删除后创建。

```F#
let resetSeed seedDatabase (context:#DbContext)=
    context.Database.EnsureDeleted() |> ignore
    context.Database.EnsureCreated() |> ignore
    seedDatabase()
```

这会每次执行方法，都会先删除数据库，再创建它，会导致不必要的性能问题。

另一种方法，当数据库已经存在时，就当它已经初始化了。

首先要判断物理数据库是否已经存在。一条语句：

```F#
let exists = context.Database.CanConnect()
```

如果物理数据库存在返回真，否则返回假。

下面方法，如果数据库不存在，就创建它并填充数据。如果存在就什么也不做。

```F#
let ensureSeed seedDatabase (context:#DbContext)=
    if context.Database.CanConnect() then
        ()
    else
        context.Database.EnsureCreated() |> ignore
        seedDatabase()
```

这里用到两个关键的数据库方法：

`EnsureDeleted`

如果数据库存在就删除它；不存在就算了，什么也不做。

`EnsureCreated `

如果数据库不存在，就创建它和上下文的模式；存在就算了，什么也不做。它的返回值没有什么用，一般都返回真。还没有遇到返回假的时候。

### 日志数据库上下文

首先定义一个日志记录器：

```F#
type private XunitLogger(output:ITestOutputHelper, logLevel:LogLevel) =
    let _logLevel = logLevel
    interface ILogger with
        member this.BeginScope<'TState>(state:'TState) = null
        member this.IsEnabled(logLevel:LogLevel) = logLevel >= _logLevel
        member this.Log<'TState>(logLevel:LogLevel, eventId:EventId, state:'TState, e:Exception, formatter:Func<'TState, Exception, string>)=
            let res = formatter.Invoke(state, e)
            output.WriteLine(res)
```

注意`Log`成员，调用`output`输出到Xunit，将副作用对象通过参数表输入挂靠到类型。副作用对象是头不变的，而不可变对象是尾不变的。头不变意思是无论如何修改对象的内容，对象引用（头）是不变的。尾不变指的是新生成一个头部，指向或引用原有对象作为新对象的尾部，构成一个新对象。这个结构的尾部始终是不变的。

头不变的例子：

```F#
let arr = new ResizeArray<int>()
arr.Add 10
arr.Add 8
someEffect arr
```

头指`arr`，不管如何操作它，它仍然保持整个副作用的最终结果。

尾不变的例子：

```F#
let lst = []
let lst = 0::lst
let lst = 1:4:lst
```

每次`let`都一个新的引用，充分利用现有数据。这是尾不变的。

再定义一个提供器，将日志记录器提供给数据库上下文：

```F#
type XunitLoggerProvider(output:ITestOutputHelper, ?logLevel:LogLevel) =
    let logLevel = defaultArg logLevel LogLevel.Information
    interface ILoggerProvider with
        member this.Dispose() = ()    
        member this.CreateLogger(categoryName) = 
        	new XunitLogger(output, logLevel) :> ILogger
```

下面是设置数据库上下文的方法：

```F#
let logToXunit (output:ITestOutputHelper) (context:#DbContext) =
    let loggerFactory = context.GetService<ILoggerFactory>()
    loggerFactory.AddProvider(new XunitLoggerProvider(output))
```

单元测试方法：

```F#
type Test(output:ITestOutputHelper) =
    [<Fact>]
    member this.TestOk()=
        use context = new EfCoreContext(options)        
        context |> logToXunit output
```

连接字符串可以提取数据库名称，通过`SqlConnectionStringBuilder`类实现的。

```F#

```


