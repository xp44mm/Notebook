# VSIX 项目总结

一个`VSIXProject`对应一个`*.vsct`文件，这个文件是Visual Studio command table。

命令表内容见：
https://docs.microsoft.com/en-us/visualstudio/extensibility/internals/how-vspackages-add-user-interface-elements?view=vs-2017

### 项目创建

选择C#、VSIXProject项目，创建一个。

完成后，项目包括两个文件

* source.extension.vsixmanifest

  包括发布项目的一些信息。双击可以打开图形界面，可以填写扩展包的名称，描述，等等。

* VSIXProject1Package.cs

  扩展包的初始化文件。或者说启动文件。

### DTE

保存一个静态DTE到扩展包类中，所有命令都可以访问，赋值是异步方式的，其他代码省略：

```C#
public sealed class VSIXProject1Package : AsyncPackage
{
    public static EnvDTE80.DTE2 _dte;

    protected override async Task InitializeAsync(CancellationToken cancellationToken, IProgress<ServiceProgressData> progress)
    {
        await this.JoinableTaskFactory.SwitchToMainThreadAsync(cancellationToken);

        VSIXProject1Package._dte = await this.GetServiceAsync(typeof(EnvDTE.DTE)) as EnvDTE80.DTE2;
        Microsoft.Assumes.Present(VSIXProject1Package._dte);

    }
}
```

### 添加命令

为项目添加新建项，选择C#、vspackage，custom command，当为项目首次添加自定义命令时，模板会为项目自动新建三个文件，形如`command1.cs`，`VSIXProject1Package.vsct`，还会添加一个文件夹`Resources`，里面包括定义命令界面所需要的图片。第一个是命令类。第二个命令表文件(vs command table)。命令类用于定义命令本身。命令表描述扩展包的所有命令在VS界面中的布局。文件夹资源是用于命令表的。

除了添加了文件，模板还会修改扩展包的`VSIXProject1Package.cs`启动文件，向其初始化方法中添加初始化语句：

```C#
public sealed class VSIXProject1Package : AsyncPackage
{
    protected override async Task InitializeAsync(CancellationToken cancellationToken, IProgress<ServiceProgressData> progress)
    {
        await this.JoinableTaskFactory.SwitchToMainThreadAsync(cancellationToken);

        //初始化dte
        VSIXProject1Package._dte = await this.GetServiceAsync(typeof(EnvDTE.DTE)) as EnvDTE80.DTE2;
        Microsoft.Assumes.Present(VSIXProject1Package._dte);

        //每次添加一个自定义命令，模板都会自动添加一条语句。
        await Command1.InitializeAsync(this);
    }
}
```

命令类`command1.cs`需要修改的就只是`Execute`方法，位于类的末尾：

```C#
internal sealed class Command1
{
    private async void Execute(Object sender, EventArgs e)
    {
        await ThreadHelper.JoinableTaskFactory.SwitchToMainThreadAsync();
        
        //添加功能代码。。。
    }
}
```

命令表`VSIXProject1Package.vsct`需要修改的部分如下：

```xml
<CommandTable xmlns="http://schemas.microsoft.com/VisualStudio/2005-10-18/CommandTable" xmlns:xs="http://www.w3.org/2001/XMLSchema">
    <Buttons>
      <Button guid="guidVSIXProject1PackageCmdSet" id="Command1Id" priority="0x0100" type="Button">
        <Parent guid="guidVSIXProject1PackageCmdSet" id="MyMenuGroup" />
        <Icon guid="guidImages" id="bmpPicArrows" />
        <Strings>
          <ButtonText>垂直对齐代码</ButtonText>
        </Strings>
      </Button>

    </Buttons>

    <Bitmaps>
      <Bitmap guid="guidImages" href="Resources\Command1.png" usedList="bmpPic1, bmpPic2, bmpPicSearch, bmpPicX, bmpPicArrows, bmpPicStrikethrough" />
    </Bitmaps>
  </Commands>

  <Symbols>
    <!-- This is the package guid. -->
    <GuidSymbol name="guidVSIXProject1Package" value="{73b532a5-177b-4666-856a-113986e2b95b}" />

    <!-- This is the guid used to group the menu commands together -->
    <GuidSymbol name="guidVSIXProject1PackageCmdSet" value="{96c53cf9-0a03-4b0b-986c-75c9b6e9aecc}">
      <IDSymbol name="MyMenuGroup" value="0x1020" />
      <IDSymbol name="Command1Id" value="0x0100" />
    </GuidSymbol>

    <GuidSymbol name="guidImages" value="{811231b3-5e29-46b1-9ddb-a531d04b469a}">
      <IDSymbol name="bmpPic1" value="1" />
      <IDSymbol name="bmpPic2" value="2" />
      <IDSymbol name="bmpPicSearch" value="3" />
      <IDSymbol name="bmpPicX" value="4" />
      <IDSymbol name="bmpPicArrows" value="5" />
      <IDSymbol name="bmpPicStrikethrough" value="6" />
    </GuidSymbol>
  
  </Symbols>

</CommandTable>

```

每个按钮对应一个命令。看符号`<Symbols/>`部分，第一个符号`<GuidSymbol/>`是包guid，第二个符号是菜单命令符号，菜单和所有的命令共用这个符号。第三个符号对应一张图片，每张图片一个符号。

用模板添加第二个命令时，会新建命令`Command2.cs`类文件，添加一张图片`Command2.png`到`Resources`文件夹。

向`VSIXProject1Package.cs`启动文件，添加命令初始化代码：

```C#
protected override async Task InitializeAsync(CancellationToken cancellationToken, IProgress<ServiceProgressData> progress)
{
    await this.JoinableTaskFactory.SwitchToMainThreadAsync(cancellationToken);

    //初始化dte
    VSIXProject1Package._dte = await this.GetServiceAsync(typeof(EnvDTE.DTE)) as EnvDTE80.DTE2;
    Microsoft.Assumes.Present(VSIXProject1Package._dte);

    //每次添加一个自定义命令，模板都会自动添加一条语句。
    await Command1.InitializeAsync(this);
    await Command2.InitializeAsync(this);
}
```

浏览到相应位置，查看确认一下即可，初始化代码是模板自动添加的。

下一步是修改命令表`VSIXProject1Package.vsct`文件。修改了三部分`<Buttons/>`,`<Bitmaps/>`,`<Symbols/>`

```xml
    <Buttons>
      <Button guid="guidVSIXProject1PackageCmdSet" id="Command1Id" priority="0x0100" type="Button">
        <Parent guid="guidVSIXProject1PackageCmdSet" id="MyMenuGroup" />
        <Icon guid="guidImages" id="bmpPicArrows" />
        <Strings>
          <ButtonText>垂直对齐代码</ButtonText>
        </Strings>
      </Button>
+      <Button guid="guidVSIXProject1PackageCmdSet" id="cmdidCommand2" priority="0x0100" type="Button">
+        <Parent guid="guidVSIXProject1PackageCmdSet" id="MyMenuGroup" />
+        <Icon guid="guidImages2" id="bmpPicX" />
+        <Strings>
+          <ButtonText>合并空格</ButtonText>
+        </Strings>
+      </Button>
    </Buttons>

    <!--The bitmaps section is used to define the bitmaps that are used for the commands.-->
    <Bitmaps>
      <Bitmap guid="guidImages" href="Resources\Command1.png" usedList="bmpPic1, bmpPic2, bmpPicSearch, bmpPicX, bmpPicArrows, bmpPicStrikethrough" />
-      <Bitmap guid="guidImages1" href="Resources\Command2.png" usedList="bmpPic1, bmpPic2, bmpPicSearch, bmpPicX, bmpPicArrows, bmpPicStrikethrough" />        
    </Bitmaps>
  </Commands>

  <Symbols>
    <!-- This is the package guid. -->
    <GuidSymbol name="guidVSIXProject1Package" value="{73b532a5-177b-4666-856a-113986e2b95b}" />

    <!-- This is the guid used to group the menu commands together -->
    <GuidSymbol name="guidVSIXProject1PackageCmdSet" value="{96c53cf9-0a03-4b0b-986c-75c9b6e9aecc}">
      <IDSymbol name="MyMenuGroup" value="0x1020" />
      <IDSymbol name="Command1Id" value="0x0100" />
+      <IDSymbol name="cmdidCommand2" value="4129" />
    </GuidSymbol>

    <GuidSymbol name="guidImages" value="{811231b3-5e29-46b1-9ddb-a531d04b469a}">
      <IDSymbol name="bmpPic1" value="1" />
      <IDSymbol name="bmpPic2" value="2" />
      <IDSymbol name="bmpPicSearch" value="3" />
      <IDSymbol name="bmpPicX" value="4" />
      <IDSymbol name="bmpPicArrows" value="5" />
      <IDSymbol name="bmpPicStrikethrough" value="6" />
    </GuidSymbol>
  
-    <GuidSymbol name="guidImages2" value="{811231b3-5e29-46b1-9ddb-a531d04b469a}">
-      <IDSymbol name="bmpPic1" value="1" />
-      <IDSymbol name="bmpPic2" value="2" />
-      <IDSymbol name="bmpPicSearch" value="3" />
-      <IDSymbol name="bmpPicX" value="4" />
-      <IDSymbol name="bmpPicArrows" value="5" />
-      <IDSymbol name="bmpPicStrikethrough" value="6" />
-    </GuidSymbol>      
  </Symbols>

```

第二个`<GuidSymbol/>`符号是表示菜单命令的，每个`<IDSymbol/>`代表一个命令。这个条目是自动添加的，查看确认一下即可，无需手动修改。图片使用一个即可，所以删除bitmpas,和guidImages2对应的GuidSymbol，要删除的行行首是减号。修改命令2的Icon。最终命令表代码如下：

```xml
<Buttons>
  <Button guid="guidVSIXProject1PackageCmdSet" id="Command1Id" priority="0x0100" type="Button">
    <Parent guid="guidVSIXProject1PackageCmdSet" id="MyMenuGroup" />
    <Icon guid="guidImages" id="bmpPicArrows" />
    <Strings>
      <ButtonText>垂直对齐代码</ButtonText>
    </Strings>
  </Button>
  <Button guid="guidVSIXProject1PackageCmdSet" id="cmdidCommand2" priority="0x0100" type="Button">
    <Parent guid="guidVSIXProject1PackageCmdSet" id="MyMenuGroup" />
    <Icon guid="guidImages" id="bmpPicX" />
    <Strings>
      <ButtonText>合并空格</ButtonText>
    </Strings>
  </Button>
</Buttons>

<Bitmaps>
  <Bitmap guid="guidImages" href="Resources\Command1.png" usedList="bmpPic1, bmpPic2, bmpPicSearch, bmpPicX, bmpPicArrows, bmpPicStrikethrough" />
</Bitmaps>

<Symbols>
  <!-- This is the package guid. -->
  <GuidSymbol name="guidVSIXProject1Package" value="{73b532a5-177b-4666-856a-113986e2b95b}" />

  <!-- This is the guid used to group the menu commands together -->
  <GuidSymbol name="guidVSIXProject1PackageCmdSet" value="{96c53cf9-0a03-4b0b-986c-75c9b6e9aecc}">
    <IDSymbol name="MyMenuGroup" value="0x1020" />
    <IDSymbol name="Command1Id" value="0x0100" />
    <IDSymbol name="cmdidCommand2" value="4129" />
  </GuidSymbol>

  <GuidSymbol name="guidImages" value="{811231b3-5e29-46b1-9ddb-a531d04b469a}">
    <IDSymbol name="bmpPic1" value="1" />
    <IDSymbol name="bmpPic2" value="2" />
    <IDSymbol name="bmpPicSearch" value="3" />
    <IDSymbol name="bmpPicX" value="4" />
    <IDSymbol name="bmpPicArrows" value="5" />
    <IDSymbol name="bmpPicStrikethrough" value="6" />
  </GuidSymbol>

</Symbols>
```

配置好命令以后，就可以先运行一下，确认修改正确。可以先使用模板自动生成的执行方法，暂时不用修改命令的执行方法。

### 程序集的创建

~~项目属性、签名、为程序集签名，取消勾选。~~

~~删除`key.snk`的文件。~~

如果不需要，删除引用：System.Windows.Form

### 升级NuGet程序包

注意选择相应的VisualStudio版本，不要变更扩展包相关NuGet项的主要版本号。

https://docs.microsoft.com/en-us/nuget/tools/ps-ref-update-package

NuGet版本号由三部分组成:`Major.Minor.Patch`，中文名为主要版本，次要版本，补丁版本

打开NuGet界面，先选择性升级可以更新到最新版的包。然后用命令行升级所有包到次要版最新。

ToHighestMinor: Constrains upgrades to only versions with the same Major version as the currently installed package.

```
Update-Package -ToHighestMinor
```

升级所有包到次要版最新，`Major`相同，`Minor.Patch`最新。

参考：

```
Update-Package
```

升级所有包到最新版。`Major.Minor.Patch`都是最新。

```
Update-Package -Safe
```

升级所有包的第三部分到最新版，`Major.Minor`相同，`Patch`最新。 `-Safe` is an alias of `-ToHighestPatch`.

### WPF相关依赖

如果项目需要wpf，添加下面引用：

```
PresentationCore
PresentationFramework
System.Xaml
WindowsBase
```

### NuGet找不到资产文件 project.assets.json

解决办法：
使用 cmd cd 目前 .net core 项目文件夹中
运行 dotnet build