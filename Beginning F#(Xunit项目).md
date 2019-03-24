# Beginning F#

新建F# Xunit项目

更改项目类型为类库

使用.net core 2.1

更新NuGet程序包为最新

替换`Tests.fs`文件：

```F#
namespace beginningFsharp

open Xunit
open Xunit.Abstractions

type Tests(output : ITestOutputHelper) =
  [<Fact>]
  member this.first() =
    output.WriteLine(sprintf "%A" "Hello World!")
```

`output`的用法，与`Console`基本一样。



