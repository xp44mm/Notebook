# ExcelDNA UDF

## Getting Started

office是64位的，编程工具是Visual Studio 2019

1. Create a new **Class Library (.NET Framework)** project in C#.
2. Use the **Manage NuGet Packages** dialog or the **Package Manager Console** to install the **Excel-DNA** package:

```
PM> Install-Package ExcelDna.AddIn
```

3. Add your code (C#):

```C#
using ExcelDna.Integration;

namespace ExcelDNA
{
    public static class MyFunctions
    {
        [ExcelFunction(Description = "My first .NET function", Category = "STRING")]
        public static string SayHello([ExcelArgument(Name = "TEXT", Description = "text to split")] string name)
        {
            return "Hello " + name;
        }
    }
}
```

4. Compile, 在生成目录里会出现许多文件，生成目录下有4个xll文件，其中有一个是我们所用的：

   ```
   ExcelDNA-AddIn64.xll
   ```

   看名字应该用在64位office中。其他还不知到有什么用，放在那里，不管他。

5. 打开Excel，加载步骤如下：

   开发工具，Excel加载项，浏览，选择生成目录的`ExcelDNA-AddIn64.xll`文件，可用加载宏，勾选。
   
   > 如果开发工具选项卡不在功能区，选文件，选项，自定义功能区，主选项卡，开发工具，勾选。

   > 清除多余的加载宏项目：删除加载宏项目对应的文件，勾选要清除的项目，会出现对话框，问是否清除加载宏项目，确认清除。

6. load and use your function in Excel:

   ```
   =SayHello("World!")
   ```

当你在Excel手动输入函数时，函数名会出现在智能提示框中。

公式，插入函数，选择类别`STRING`，可以找到`SayHello`函数，并且有描述信息。



获得应用

```C#
var xlApp = (Application)ExcelDnaUtil.Application;
```



参考文献：

https://excel-dna.net/

https://jingyan.baidu.com/article/f71d6037c911351ab741d150.html



# How to access range selected in excel through ExcelDNA in C#?



From your Ribbon event handlers, you can access the Excel COM API via `ExcelDnaUtil.Application`, get a reference to the active sheet, and get the selected range and inspect its values.

Install the ExcelDna.Interop NuGet package to make it easier (so you can get IntelliSense), and use the `ExcelDnaUtil.Application` instance to access the COM API.

e.g.

```C#
try
{
    var excelApp = (Microsoft.Office.Interop.Excel.Application)ExcelDnaUtil.Application;

    // Use excelApp to access the selected sheet, range, etc.
    // the same way you would do with VBA
}
catch (Exception ex)
{
    MessageBox.Show(ex.Message, "Error", MessageBoxButtons.OK, MessageBoxIcon.Error);
}
```

# How do I detect a reference to an empty cell using Excel-DNA?

By declaring the arguments of the function as `double` (or `double[]`) you're only allowed to have valid `double` values.

If you need to check for special values such as `ExcelEmpty` you need to declare the arguments of your function as `object` (or `object[]`).

e.g.

```C#
[ExcelFunction(IsMacroType = true)]
public static object getVal([ExcelArgument(AllowReference = true)] object[] dataRange)
{
    if (dataRange[0] == ExcelEmpty.Value)
    {
        // ...
    }

    // ...
}
```

------

If you prefer to get the values via an `ExcelReference` then declare your input argument as `object` and check if it's an `ExcelReference`.

e.g.

```c#
[ExcelFunction(IsMacroType = true)]
public static object getVal([ExcelArgument(AllowReference = true)] object input)
{
    if (input is ExcelReference reference)
    {
        // ...

        var value = reference.GetValue();

        // ...
    }

    // ...
}
```



编译不报错，也应该安装所有的依赖程序集。如果，公式计算结果出现#Value错误，可以检查是否有依赖的程序集没有在xll项目中安装。