# npm 编写cli

## 编写程序包

新建文件夹，进入目录

```bash
npm init -y
```

创建 package.json

在package.json里，配置bin：

```javascript
"bin": {
    "hmt": "./index.js"
  }
```

键名就是命令名，值是入口文件名。

示例`./index.js`文件的内容：

```js
#!/usr/bin/env node
const { inspect } = require('util')
console.log(inspect(process.argv))
```

文件第一行的代码表示用node来执行这个文件。命令行的参数用`process.argv`来提取，用`util.inspect`检查提取出来的数据。

## 本地安装和测试

在项目根目录下执行

```bash
npm link
```

会把命令链接到全局

> 注：删除链接的命令`npm unlink hmt`，不必在项目根目录下。在任何目录下都可以。

在任何目录下键入命令：

```bash
# hmt xdafsdf
[
  'C:\\Program Files\\nodejs\\node.exe',
  'C:\\Program Files\\nodejs\\node_modules\\hmt\\index.js',
  'xdafsdf'
]
```

会打印出`process.argv`。argv 属性返回一个数组，由命令行执行脚本时的各个参数组成。它的第一个成员总是node，第二个成员是脚本文件名，其余成员是脚本文件的参数。



## 下一步

更复杂的cli需要实现更多功能，现有包支持，比如这几个很常用：

```json
 "dependencies": {
    "chalk": "^3.0.0",
    "commander": "^4.0.1",
    "inquirer": "^7.0.1",
  },
```