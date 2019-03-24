## Entity Framework入门(F\#)

新建F\#普通类库，名为entityframeworks

工具/NuGet包管理器/程序包管理器控制台

确认控制台中的默认项目

安装以下：
```PowerShell
install-package fsharp.core
install-Package xunit
install-Package xunit.runner.visualstudio
install-package entityFramework
```

添加引用： 程序集$\rightarrow$框架$\rightarrow$ `System.Data, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089'`。

在项目根目录添加`App.config`文件，并配置连接字符串：

```xml
<connectionStrings>
<add name="Recipe"
connectionString="data source=.;initial catalog=Recipe;integrated security=True;MultipleActiveResultSets=True;App=EntityFramework"
providerName="System.Data.SqlClient" />
</connectionStrings>
</configuration>
```



在项目根目录下添加POCO.fs文件：
```fsharp
namespace entityframeworks.POCO
open entityframeworks
open System

[<AllowNullLiteral>]
type Person() =
//member val {1}:{0} = {2} with get, set
member val PersonId:int = 0 with get, set
member val FirstName:string = "" with get, set
member val LastName:string = "" with get, set
member val MiddleName:string = "" with get, set
member val BirthDate:Nullable<DateTime> = Nullable() with get, set
member val HeightInFeet:decimal = 0M with get, set
member val Photo:byte[] = [||] with get, set
member val IsActive:bool = true with get, set
member val NumberOfCars:int = 0 with get, set
open System.Data.Entity.ModelConfiguration

type PersonMap() =
inherit EntityTypeConfiguration<Person>()
do
base.Property(fun p -> p.FirstName).HasMaxLength(Nullable 30)
|> ignore
base.Property(fun p -> p.LastName).HasMaxLength(Nullable 30)
|> ignore

base.Property(fun p -> p.MiddleName).HasMaxLength(Nullable 1).IsFixedLength().IsUnicode(Nullable false)
|> ignore

[<AllowNullLiteral>]
type Company() =
member val CompanyId = 0 with get, set
member val Name = "" with get, set
```

在项目根目录最后添加RecipeContext.fs文件：
```fsharp

namespace entityframeworks
open entityframeworks.POCO
open System.Data.Entity

type RecipeContext() =
inherit DbContext("name=Recipe")
static do Database.SetInitializer(new RecipeInitializer())

//Turn off DB Initializer
//static do Database.SetInitializer null

[<DefaultValue(true)>]
val mutable people : DbSet<Person>

[<DefaultValue(true)>]
val mutable companies : DbSet<Company>

member x.People
with get () = x.people
and set v = x.people <- v

member x.Companies
with get () = x.companies
and set v = x.companies <- v

override this.OnModelCreating(modelBuilder : DbModelBuilder) =
modelBuilder.Configurations.Add(new PersonMap()) |> ignore

and RecipeInitializer() =
inherit DropCreateDatabaseAlways<RecipeContext>()

// inherit DropCreateDatabaseIfModelChanges<RecipeContext>()
// inherit CreateDatabaseIfNotExists<RecipeContext>()

override this.Seed(context : RecipeContext) =
new Person(FirstName = "Robert", LastName = "Doe")
|> context.People.Add
|> ignore

new Person(FirstName = "John", LastName = "Smith")
|> context.People.Add
|> ignore

new Person(FirstName = "Billy", LastName = "Minor")
|> context.People.Add
|> ignore

new Person(FirstName = "Kathy", LastName = "Ryan")
|> context.People.Add
|> ignore

//可选
context.SaveChanges() |> ignore
```

在项目根目录最后添加`Tester.fs`文件：
```fsharp
namespace entityframeworks

open Xunit
open Xunit.Abstractions
open System.Linq

type Tester(output : ITestOutputHelper) =
  // Querying data in a database
  [<Fact>]
  member this.read() =
    use context = new RecipeContext()
    for person in context.People do
    output.WriteLine
    ("{0} {1} {2}",person.PersonId, person.FirstName, person.LastName)
  //Updating a record
  [<Fact>]
  member this.Updating() =
    use context = new RecipeContext()
    let savedPeople = context.People
    if savedPeople.Any() then
    let person = savedPeople.First()
    person.FirstName - "Johnny"
    person.LastName - "Benson"
    context.SaveChanges()
    | ignore

    let person = context.People.First()
    Assert.Equal("Johnny",person.FirstName)
    Assert.Equal("Benson",person.LastName)
    
  //Deleting a row from the database
  [<Fact>]
  member this.Deleting() =
    use context = new RecipeContext()
    let personId = 2
    let person = context.People.Find(personId)
    if person > null then
    context.People.Remove(person)
    | ignore
    context.SaveChanges()
    | ignore
    Assert.Null(context.People.Find(personId))
```