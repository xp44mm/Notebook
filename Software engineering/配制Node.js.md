# 配制Node.js和命令行

## 安装或升级node.js

从[Node.js (nodejs.org)](https://nodejs.org/en/)下载最新版安装文件。

查看npm全局包有哪些：

```bash
npm list -g --depth=0
```

查看npm版本：

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

以管理员身份打开命令行，执行如下命令

```bash
npm i -g npm
```

如果有错误，错误信息为文件已经存在，或文件不能重命名，可以手动删除再次执行安装命令。

可能的文件是node.js安装目录`C:\Program Files\nodejs`下的文件：

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

npm-check-updates工具用于更新`package.json`中的包版本到最新版。

安装插件:

```bash
npm install -g npm-check-updates
```

另一个中国常用插件，淘宝镜像：

```bash
npm install -g cnpm --registry=https://registry.npm.taobao.org
```

### 升级全局的本地包

```
npm install -g <packagename>
```

例如，我们发现全局包`@angular/cli`需要更新，执行:

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

- `-P, --save-prod`: Package will appear in your `dependencies`. This is the default unless `-D` or `-O` are present.

- `-D, --save-dev`: Package will appear in your `devDependencies`.

- `-O, --save-optional`: Package will appear in your `optionalDependencies`.

- `--no-save`: Prevents saving to `dependencies`.

### npm-check-updates

npm-check-updates插件用于检测`package.json`的引用程序包为最新版本。简写命令为`ncu`。

```bash
ncu -u
```
此选项为升级`package.json`文件，中引用包的版本号为最新，但不执行具体的安装。检查更新也可以使用淘宝镜像。

```bash
ncu -u --registry=https://registry.npm.taobao.org
```

当需要升级项目的依赖包时，先删除锁定文件（`package-lock.json`），然后执行：

```bash
npm i --registry=https://registry.npm.taobao.org
```

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

这条命令会临时安装`create-react-app`包，命令完成后`create-react-app`会删掉，不会出现在`global`中。下次再执行，还是会重新临时安装。

### 发布npm包

npm naming restrictions:

- name can no longer contain capital letters

不管升级降级，在配置文件中，修改完版本号，或增加包，或删除包，执行`npm i`，即可达成配置文件所配置的目标状态。

发包需要登陆npm网站账户。

```bash
> npm login
Username: xp44m
Password:
Email: (this IS public) 34696643@qq.com
Logged in as xp44m on https://registry.npmjs.org/.
```

按密码时，屏幕没有任何打字的响应，输入密码完成回车即可提示账户的信息。

在包的目录下，输入命令：

```bash
npm publish
```

升级只需提升版本号，在`package.json`文件中，然后重复发布，即可。