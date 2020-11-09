## 配置visual studio

### 选项配置

欲进行配置，通过点击：菜单[工具]->[选项...]，打开选项对话框：

左侧列表框选择[项目和解决方案]，[Web包管理器(Web Package Management)]，[程序包还原]。然后见右侧，NPM分组下面的所有选项都设为`False`

```
保存时还原：False
打开项目时还原：False
```

左侧列表框选择[项目和解决方案]，[Web包管理器(Web Package Management)]，[外部Web工具]。然后见右侧，外部工具的位置：`$(PATH)`向上至顶部，以获得最高优先级。


### ESLint设置

选项，文本编辑器，javascript/typescript，Linting，常规。

项目文件夹下的配置文件将完全覆盖默认配置文件。

visual studio 自带ESLint功能，只需修改默认配置文件：

```js
"rules": {
    'semi': ['off'],
    'no-console': "warn",
}
```

规则可以用数字或字符串配置：

| 数字 | 字符串  | 含义 | 动作       |
| ---- | ------- | ---- | ---------- |
| 0    | 'off'   | 关闭 | 不检查     |
| 1    | 'warn'  | 警告 | 检查可通过 |
| 2    | 'error' | 错误 | 检查不通过 |

如果有额外选项，可以用数组常量，依次排列选项。

### 常用扩展工具

visual studio工具->扩展和更新...

[ ] CodeMaid
[ ] Markdown Editor
[ ] EF Core Power Tools
[ ] Add New File

### Asp.net core的配置

引用安装包的版本号是根据目标框架版本号推算出的，不要加版本号：

```xml
<PackageReference Include="Microsoft.AspNetCore.All"/>
```

详见网址：
https://docs.microsoft.com/zh-cn/dotnet/core/tools/csproj#implicit-package-references

`Microsoft.AspNetCore.App`已经取代了`Microsoft.AspNetCore.All`，详见网址：

https://docs.microsoft.com/en-us/aspnet/core/fundamentals/metapackage?view=aspnetcore-2.1#migrate

### 如何重新安装和更新NuGet包

[何时重新安装包](https://docs.microsoft.com/zh-cn/nuget/consume-packages/reinstalling-and-updating-packages#when-to-reinstall-a-package)中描述了大量有关对包的引用可能在 Visual Studio 项目中损坏的情况。 在这些情况下，卸载并重新安装同一版本的包会将这些应用还原为正常工作状态。 更新包仅意味着安装更新版本，这常常将包还原为正常工作状态。

更新和重新安装包按以下方式实现：

| 方法           | 更新                                           | 重新安装                                                     |
| -------------- | ---------------------------------------------- | ------------------------------------------------------------ |
| 包管理器控制台 | `Update-Package`命令                           | `Update-Package -reinstall` 命令                             |
| 包管理器 UI    | 在“更新”选项卡上，选择一个或多个包并选择“更新” | 在“已安装”选项卡上，选择一个包，记录其名称，然后选择“卸载”。 切换到“浏览”选项卡，搜索包名称并选中，然后选择“安装”。 |
| nuget.exe CLI  | `nuget update` 命令                            | 对于所有包，删除包文件夹，然后运行 `nuget install`。对于单个包，删除包文件夹并使用 `nuget install <id>`再安装一个。 |

### 字体或颜色配置

常见需要改变颜色的显示项是：

- 标点
- 大括号匹配
- 运算符

vs 2019 不支持管道操作符`|>`，会在错误列表中给出警告，可以关闭他：

~~选项，IntelliCode，常规，IntelliSense完成项，JavaScript/TypeScript基础模型，禁用~~

选项，文本编辑器，JavaScript/TypeScript，代码验证，JavaScript错误，

JavaScript错误，启用JavaScript错误，False

如果遇到打开解决方案长时间没有反应时，可以尝试删除`.vs`文件夹，（或其他辅助型文件夹），因为里面没有源代码，可以放心删除，系统再打开时会从头建一个新的。