# 如何制作并上传自己的Nuget包

Nuget gallery 可以托管dll库,我们制作dll打包之后上传到https://www.nuget.org/

推荐使用微软账户注册Nuget gallery,非常方便。不要使用其他关联账号登陆，一定要使用微软账号。

If you have forgotten which Microsoft account is associated with your NuGet.org account, please follow the steps below to get assistance.

1. Go to [NuGet.org login page](https://www.nuget.org/users/account/LogOn) and click on **Need assistance signing in?** link.
2. This will show you the pop-up dialog box for assistance. Follow the steps in this dialog box to understand the associated Microsoft account(s) for your NuGet.org account.

就算不改变账号也请先登出账号，再重新用邮件中的微软账号登陆。



打包方法：

项目，属性中，打包标签，就是NuGet打包。尽可能详细的填写打包标签中的内容。

可以每次编译时都生成打包文件，也可以在项目管理器中，右键项目，选择打包，会生成一个文件，`libname.1.0.0.nupkg`，版本号变化文件名将随着变化。

记住这个文件，将通过NuGet网站上传的就是它。

到网站页面上，确保是正确的用户名登陆，然后`upload`，选择刚生成的`.nupkg`文件，确定提交后，等待审核通过，可能会几分钟的时间。



几个注意事项:

上传package更新方法非常简约,直接上传同名称高一个版本的新包即可替换掉旧版本的package

无法删除已上传的package,只能隐藏(unlist).因为要考虑老用户兼容为题,情有可原,反正不在我自己的服务器上。

