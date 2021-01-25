vs_setup.exe 安装离线包

Visual Studio 2019 离线安装包下载

首先下载微软在线安装程序，放到下载目录，在目录中，写一个批处理文件（vs.bat），命令格式如下

```bash
<vs引导程序exe> --layout <离线安装包下载的路径> --add <功能模块> --lang <语言> 
```


如果需要全部下载命令如下(一般不建议全部下载)

```bash
call vs_enterprise__1552103481.1603088960.exe --layout "%currentPath%" --lang zh-CN
```

部分下载名列如下

```bash
call vs_enterprise__1552103481.1603088960.exe --layout "%currentPath%" --lang zh-CN --includeOptional
--add Component.GitHub.VisualStudio
--add Microsoft.VisualStudio.Workload.CoreEditor
--add Microsoft.VisualStudio.Workload.ManagedDesktop
--add Microsoft.VisualStudio.Workload.NetCoreTools
--add Microsoft.VisualStudio.Workload.NetWeb 
--add Microsoft.VisualStudio.Workload.Universal

```