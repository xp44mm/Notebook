# SQL Server命令行



## 启动及停止SQL服务的方法

在SQL Server中，想要启动或停止SQL Server服务，通过SQL Server命令行操作就可以实现了。

操作步骤如下：

（1）在操作系统的任务栏中单击“开始”菜单，选择“运行”命令，在下拉列表框中输入“cmd”命令，单击“确定”按钮。

（2）输入如下命令，即可通过SQL Server命令行启动、停止或暂停的服务。

SQL Server命令行如下：

```powershell
# 停止SQL Server
NET STOP MSSQLSERVER

# 启动SQL Server
NET START MSSQLSERVER

# 暂停SQL Server
NET PAUSE MSSQLSERVER

# 重新启动暂停的SQL Server
NET CONTINUE MSSQLSERVER
```



