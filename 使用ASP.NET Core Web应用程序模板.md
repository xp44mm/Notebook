# 使用ASP.NET Core Web应用程序模板

## 配制Visual Studio和Node.js

开始本篇之前,需要配置visual studio和node.js,见单独文件.

## 创建项目

使用[ASP.NET Core Web应用程序]模板,新建项目,名为`aspnetcoreng`,选中为解决方案创建目录.点击确定.

项目名称不能有减号`-`，使用npm工具会失败。

在第二页依次选择[.net core],[asp.net core 2.0],[Angular],点击确定.

### 安装程序包

#### 修改`package.json`

项目根目录`$(ProjectDir)`下有一个`package.json`文件，首先删除我们不用的程序包：

```json
{
  "bootstrap":"",
  "jquery":"",
}
```

保存并关闭文件。

以管理员身份在项目文件夹`$(ProjectDir)`打开命令行

只更新`package.json`文件,不升级实际的包:

```
ncu -u
```

执行完命令，`package.json`依赖包的版本号就被更新到最新版了。 



删除同目录下的`npm-shrinkwrap.json`文件. 以及`package-lock.json`, 

然后可以为项目安装包:

```
npm install
```

安装成功



#### 修改项目文件

此前，我们删除了依赖项`bootstrap`和`jquery`，所以要去掉相应的代码。

当`wwwroot/dist`文件夹的内容过时，或者`ClientApp/dist`文件夹中的内容过时，可能会引起网页无法打开的错误。要解决这个问题，删除两个文件夹，选择【生成】【重新生成解决方案】，注意输出窗口，VS将重建两个文件夹。

打开文件`webpack.config.vendor.js`，删除配置中的相关配置：

```js
const nonTreeShakableModules = [
    //'bootstrap',
    //'bootstrap/dist/css/bootstrap.css',
    //'jquery',
];
```


打开文件`/ClientApp/boot.browser.ts`，删除如下：

```typescript
//import 'bootstrap';
```

打开文件`/Views/Shared/_Layout.cshtml`，删除如下：

```html
@*<link rel="stylesheet" href="~/dist/vendor.css" asp-append-version="true" />*@
```

文件`\tsconfig.json`, 改变以下项目:

```json
{
  "compilerOptions": {
    "target": "es2017",
    "lib": [ "es2017", "dom" ],
  },
}

```

这样typescript文件可以充分利用最新的语法特性。

添加文件`bower.json`

```json
{
  "name": "asp.net",
  "private": true,
  "dependencies": {
    "jquery": "3.2.1",
    "popper.js": "1.12.9",
    "bootstrap": "4.0.0-beta.3",
    "font-awesome": "4.7.0"
  }
}

```



文件`$(ProjectDir)\ClientApp\app\app.module.shared.ts`, 为服务器和浏览器的共享根文件.修改配置在此根文件中修改.



**特征模块**,特征模块相比主模块主要不同有:

```typescript
/*1*/
//import { BrowserModule } from '@angular/platform-browser';
import { CommonModule } from '@angular/common'; 

const routes: Routes = [];

@NgModule({
    imports: [
        /*2*/
        //BrowserModule,
        CommonModule,
      
        
        RouterModule.forChild(routes),/*3*/
    ],

    exports: [],/*4*/
  
    /*5*/
    //bootstrap: [AppComponent], 
})
export class StoreModule { }
```





执行`Ctrl+F5`,或[调试],[开始执行(不调试)].



