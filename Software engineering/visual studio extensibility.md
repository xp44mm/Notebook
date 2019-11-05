# visual studio extensibility

https://docs.microsoft.com/en-us/visualstudio/extensibility/extensibility-hello-world?view=vs-2017

https://docs.microsoft.com/en-us/visualstudio/extensibility/getting-started-with-the-vsix-project-template?view=vs-2017

默认vsix项目有三个文件：

```
source.extension.vsixmanifest
index.html
stylesheet.css
```

https://docs.microsoft.com/en-us/visualstudio/extensibility/how-to-provide-an-asynchronous-visual-studio-service?view=vs-2017

获得DTE的方法：

https://github.com/madskristensen/SingleFileGeneratorSample

One way to sign an F# assembly is via the `AssemblyFileKeyAttribute` attribute.

Create a new module and in it put:

```F#
module AssemblyProperties

open System
open System.Reflection;
open System.Runtime.InteropServices;

[<assembly:AssemblyKeyFileAttribute("MyKey.snk")>]

do()
```

Where `"MyKey.snk"` is the path to your key relative to the project directory.

除了在UI事件处理器中使用`async void`方法，其他所有情况请使用`Task`方法。
