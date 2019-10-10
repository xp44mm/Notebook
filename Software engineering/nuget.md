# nuget

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



### NuGet找不到资产文件 project.assets.json

解决办法：
使用 cmd cd 目前 .net core 项目文件夹中
运行 dotnet build



# NuGet File Locations

NuGet directory and file locations across Linux, Mac and Windows operating systems.

## NuGet Cache

Stores downloaded NuGet packages (.nupkg).

**Windows**

- %LocalAppData%\NuGet\Cache
- %UserProfile%\.nuget\packages

## NuGet Configuration

The [NuGet.Config](http://docs.nuget.org/docs/reference/nuget-config-file) file stores user defined NuGet package sources, credentials and proxy settings. The default location for this file is:

**Windows** %AppData%\NuGet\NuGet.Config

## Machine Wide NuGet Configuration

Machine wide configurations are used to define NuGet package sources specific to a machine or a particular IDE, such as Visual Studio.

**Windows** %ProgramData%\NuGet\Config

来源：https://lastexitcode.com/projects/NuGet/FileLocations/

机器范围配置文件内容：

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <packageSources>
    <add key="nuget.org" value="https://api.nuget.org/v3/index.json" protocolVersion="3" />
    <add key="cnblog" value="https://nuget.cnblogs.com/v3/index.json" />
  </packageSources>
  <packageRestore>
    <add key="enabled" value="True" />
    <add key="automatic" value="True" />
  </packageRestore>
  <bindingRedirects>
    <add key="skip" value="False" />
  </bindingRedirects>
  <packageManagement>
    <add key="format" value="0" />
    <add key="disabled" value="False" />
  </packageManagement>
  <disabledPackageSources>
    <add key="Microsoft Visual Studio Offline Packages" value="true" />
  </disabledPackageSources>
  <config>
    <add key="globalPackagesFolder" value="D:\packages" />
  </config>
</configuration>
```

来源：https://blog.csdn.net/lindexi_gd/article/details/79399744