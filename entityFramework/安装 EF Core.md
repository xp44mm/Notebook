## 安装 EF Core

### 创建新项目

* Open Visual Studio
* File > New > Project...
* From the left menu select Templates > Visual C# > .NET Standard
* Select the **类库 (.NET Standard)** project template
* Ensure you are targeting **.NET Framework 4.6.1** or later
* Give the project a name and click **OK**

### 安装Entity Framework

Tools > NuGet Package Manager > Package Manager Console

```
Install-Package Microsoft.EntityFrameworkCore.SqlServer
```

或命令行

```
dotnet add package Microsoft.EntityFrameworkCore.SqlServer
```

### 创建模型

模型文件的内容：

```csharp
using Microsoft.EntityFrameworkCore;
using System.Collections.Generic;

namespace LakeEf
{
    public class BloggingContext : DbContext
    {
        public DbSet<Blog> Blogs { get; set; }
        public DbSet<Post> Posts { get; set; }

        protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
        {
            optionsBuilder.UseSqlServer(@"Server=.;Database=LakeEf;Trusted_Connection=True;");
        }
    }

    public class Blog
    {
        public int BlogId { get; set; }
        public string Url { get; set; }

        public List<Post> Posts { get; set; }
    }

    public class Post
    {
        public int PostId { get; set; }
        public string Title { get; set; }
        public string Content { get; set; }

        public int BlogId { get; set; }
        public Blog Blog { get; set; }
    }
}
```

### 新建测试项目

新建测试项目，【visual F#】【.NET Core】【xUnit 测试项目(.NET Core)】。命名为`LakeEf.Tests`。

设为启动项目

引用.net standard项目

打开本项目的`.csproj`，添加程序包，这种方式将直接访问本地缓存，不访问网络。添加命令行工具：

```xml
  <ItemGroup>
    <PackageReference Include="Microsoft.EntityFrameworkCore.SqlServer" Version="2.0.3" />
    <PackageReference Include="Microsoft.EntityFrameworkCore.Design" Version="2.0.3" />    
    <DotNetCliToolReference Include="Microsoft.EntityFrameworkCore.Tools.DotNet" Version="2.0.3" />
    
    <ProjectReference Include="..\LakeEf\LakeEf.csproj" />
  </ItemGroup>
```

编译解决方案。下一步是生成数据库。

从网站https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.Tools.DotNet/查找最新版本号。

### 创建数据库

迁移(migration):根据模型代码(C#)修改数据库，不使用SQL语句。

在测试项目中打开命令行，执行：

```
dotnet ef migrations add Initial -p ..\LakeEf\LakeEf.csproj
```

项目路径不带引号，可以从`.csproj`中找到。

应用迁移到数据库：

```
dotnet ef database update -p ..\LakeEf\LakeEf.csproj
```

用Microsoft Sql Server Management Studio打开服务器，查看数据库已经生成。

### 使用模型

在测试项目中，添加文件`Test.fs`：

```F#
namespace Tests

open System
open Xunit
open Xunit.Abstractions
open LakeEf

type Test(output : ITestOutputHelper) =
    [<Fact>]
    member this.seedData() =
        use db = new BloggingContext()
        db.Blogs.Add(new Blog ( Url = "http://blogs.msdn.com/adonet" ))|> ignore
        let count = db.SaveChanges()

        output.WriteLine("{0} records saved to database", count)
        output.WriteLine("")

    [<Fact>]
    member this.readData() =
        output.WriteLine("All blogs in database:")
        use db = new BloggingContext()
        for blog in db.Blogs do
            output.WriteLine(" - {0}", blog.Url)

```

运行测试，查看输出，并浏览数据库，看看数据是否被添加。

参考文献：https://docs.microsoft.com/en-us/ef/core/get-started/aspnetcore/new-db

### 配置连接字符串

安装包：

```
System.Configuration.ConfigurationManager
```

EF Core Power Tools:

https://marketplace.visualstudio.com/items?itemName=ErikEJ.EFCorePowerTools