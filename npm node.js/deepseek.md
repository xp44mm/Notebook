# 配制Node.js和命令行

## 安装或升级node.js

从[Node.js (nodejs.org)](https://nodejs.org/en/)下载最新版安装文件。

以管理员身份执行安装。

查看node.js版本，查看npm版本：

```bash
node -v
npm -v
```

全局安装 pnpm，在终端执行：

```bash
npm install -g pnpm
```

安装完成后，执行 pnpm -v 验证是否成功。如果依然提示 'pnpm' 不是内部或外部命令，请关闭当前终端重新打开，让新加入的环境变量生效。

执行一次命令，以后所有 pnpm 操作都会自动使用淘宝镜像。

```bash
pnpm config set registry https://registry.npmmirror.com
```

配置完成后，可以运行以下命令验证是否成功：

```bash
pnpm config get registry
```

如果输出 https://registry.npmmirror.com，就说明配置成功了。

查看npm全局包有哪些：

```bash
npm list -g --depth=0
```

查看哪些包有更新：

```bash
npm outdated -g
```

npm outdated -g 有时候对全局包显示不完整，可以试试这个更可靠的命令：

```bash
npm list -g --depth=0  # 查看当前版本
npm view @deepseek-ai/dsh version  # 查看最新版本
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

```bash
npm install -g npm@latest
npm install -g npm@next
```

版本号默认为latest


## @deepseek-ai/dsh

在终端执行一次全局安装，之后就可以在任何目录下直接使用 dsh 命令了。

```bash
npm install -g @deepseek-ai/dsh
```

启动服务：
```bash
dsh web  # 以后直接用这个命令启动
```

用浏览器复制网址，打开操作界面。

```bash
dsh plugin --profile web add dshmarket
```

运行 dsh plugin list --profile web 查看已安装的插件列表，确认 dshmarket 在其中。


```bash
dsh plugin --profile web add @linxin666/dsh-web-ui-all
```





### 升级全局的本地包

```bash
npm install -g @deepseek-ai/dsh
```

例如:

```bash
 npm i -g @deepseek-ai/dsh
```

### 使用NPM查找包的所有版本

```npm
npm view @deepseek-ai/dsh version  # 查看最新版本
```

此命令查找包`@deepseek-ai/dsh`的版本。

## NPM Unexpected end of JSON input while parsing near

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

## npx

这个命令的目的是为了提升开发者使用包内提供的命令行工具的体验。

npx 允许我们单次执行命令而不需要安装，例如：

```bash
npx create-react-app my-app
```

这条命令会临时安装`create-react-app`包到my-app目录，命令完成后`create-react-app`会删掉，不会出现在`global`中。下次再执行，还是会重新临时安装。
