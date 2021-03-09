# 配制Node.js和命令行

## ConEmu

这是一个命令行程序，可以从 https://www.fosshub.com/ConEmu.html 下载安装。

安装完成后，用管理员启动这个程序，从右上角的菜单中选择`setting...`, 在右侧的树选择`General`，如下设置：

Choose your startup task or even a shell with arguments:

```
{Shells::cmd(Admin)}
```

### 命令行程序与文件资源管理器的交互

文件资源管理器，在地址栏输入:

```
ConEmu64.exe
```
可以打开命令行程序，且当前目录是文件资源管理器的当前目录。

Windows命令行打开文件夹图形界面

在命令行输入以下命令，可以在文件资源管理器中打开当前目录

```bash
explorer .
```

在文件资源管理器中打开命令行当前目录的上级目录

```bash
explorer ..
```



## 安装或升级node.js

从网站 https://nodejs.org/en/download/ 下载最新版安装文件.

查看npm全局包有哪些：

```bash
npm list -g --depth=0
```

查看npm版本:

```bash
npm -v
```
查看哪些包有更新：

```bash
npm outdated -g
```

查看参数配置：

```bash
npm config ls
```



### 安装或升级npm

以管理员身份打开命令行,执行如下命令

```bash
npm i -g npm
```

如果有错误,错误信息为文件已经存在,或文件不能重命名,可以手动删除再次执行安装命令,

可能的文件是node.js安装目录`C:\Program Files\nodejs`下的文件:

```bash
npm
npm.cmd
npx
npx.cmd
```


Try the latest stable version of npm

```
npm install -g npm@latest
npm install -g npm@next
```
版本号默认为latest

npm网站是国外的，连接速度太慢有时会超时，可以使用淘宝镜像：

```bash
npm i -g npm --registry=https://registry.npm.taobao.org
```




### npm插件

npm-check-updates工具用于更新`package.json`中的包版本到最新版,参考:http://blog.csdn.net/wkl305268748/article/details/76641323

安装插件:


```bash
npm install -g npm-check-updates
```

另一个中国常用插件，淘宝网站：

```bash
npm install -g cnpm --registry=https://registry.npm.taobao.org
```



### 升级全局的本地包

```
npm install -g <packagename>
```

例如,我们发现全局包`@angular/cli`需要更新, 就执行:

```
 npm i -g @angular/cli@latest
```

### 使用NPM查找包的所有版本

```
npm view react-hot-loader versions
```

此命令查找包`react-hot-loader`的所有版本。屏幕打印出该包所有版本的数组。也可以使用简写：

```
npm v immutable versions
```

### 本地安装位置

`npm install` saves any specified packages into `dependencies` by default. Additionally, you can control where and how they get saved with some additional flags:

* `-P, --save-prod`: Package will appear in your `dependencies`. This is the default unless `-D` or `-O` are present.
* `-D, --save-dev`: Package will appear in your `devDependencies`.
* `-O, --save-optional`: Package will appear in your `optionalDependencies`.
* `--no-save`: Prevents saving to `dependencies`.

### npm-check-updates
npm-check-updates插件用于检测`package.json`的引用程序包为最新版本。简写命令为`ncu`。

```bash
ncu -u
```
为升级`package.json`文件，中引用包的版本号为最新，但不执行具体的安装。检查更新也可以使用淘宝镜像。

```bash
ncu -u --registry=https://registry.npm.taobao.org
```

当需要升级项目的依赖包时，先删除锁定文件，然后执行：

```bash
npm i --registry=https://registry.npm.taobao.org
```


### 置命令行于项目文件夹

在解决方案资源管理器中,单击选中击ASP.NET项目,右击[在文件资源管理器中打开文件夹],

文件资源管理器，在地址栏输入:

```
ConEmu64.exe
```

命令行以`#`提示，表示是管理员模式，以`$`提示则是非管理员模式，欲进入管理员模式:

在命令行窗口的标签上右击,[restart or duplicate],[restart as Adimin],以置命令行于项目文件夹。

### NPM Unexpected end of JSON input while parsing near

解决办法：

这是服务器连接的问题，多换几个站点，重试。

first:

```bash
npm i --registry=https://registry.npm.taobao.org --loglevel=silly
```

then:

```bash
npm cache clean --force
```

验证缓存：
```bash
> npm cache verify
Cache verified and compressed (~\AppData\Roaming\npm-cache\_cacache):
Content verified: 2503 (45460300 bytes)
Content garbage-collected: 1 (87138 bytes)
Index entries: 4304
Finished in 14.909s
```
### npx

这个命令的目的是为了提升开发者使用包内提供的命令行工具的体验。

npx 允许我们单次执行命令而不需要安装，例如：

```bash
npx create-react-app my-app
```

这条命令会临时安装 create-react-app 包，命令完成后 create-react-app 会删掉，不会出现在 global 中。下次再执行，还是会重新临时安装。

npm naming restrictions:

  *  name can no longer contain capital letters

不管升级降级，在配置文件中，修改完版本，或者增加包，删除包，执行`npm i`，即可自动添加和删除。



发包需要登陆npm

```bash
> npm login
Username: xp44m
Password:
Email: (this IS public) 34696643@qq.com
Logged in as xp44m on https://registry.npmjs.org/.
```

按密码时，屏幕没有任何提示，输入完成回车即可。

在包的目录下，输入命令：

```bash
npm publish
```

升级只需提升版本号，在`package.json`文件中，然后重复发布，即可。