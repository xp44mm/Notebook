# Testing npm packages before publishing

[参考来源](https://blog.vcarl.com/testing-packages-before-publishing/)
Now, I use npm pack.

## npm pack

The pack command creates a `.tgz` file exactly the way it would if you were going to publish the package to npm. It pulls the name and version from `package.json`, resulting in a file like `package-name-0.0.0.tgz`. This can be copied, uploaded, or sent to a coworker. It's exactly the file that would be uploaded to npm if you published it.

```sh
~/workspace/package-name $ npm pack
```

Once you have the file, you can install it. npm install can install from a lot more sources than just npm, and I highly suggest [skimming through the docs](https://docs.npmjs.com/cli/install). We have to specify the full path to the file, so I usually copy it to my home directory first for convenience.

```sh
~/workspace/package-name $ npm pack
~/workspace/package-name $ cp package-name-0.0.0.tgz ~
~/workspace/some-application $ npm install ~/package-name-0.0.0.tgz
```

You could probably set up an alias or function in your terminal to automate this, but I don't do it frequently enough to bother. `npm pack | tail -n 1` will output just the filename to standard out.

This runs through a complete publish cycle—it even runs the publish npm script and the associated pre-scripts and post-scripts. Packing it up and installing it is an excellent way to simulate publishing a package, and it avoids all of the quirks and problems of symlinking.

I know when I was first trying to publish packages to npm, one of the hurdles I faced was figuring out if it would actually work. Publishing feels so final; you put it up for the world to see and that version number can never be used again. `npm pack` helps me be more confident that it's going to work the way I expect it to.

## 实战

我们从零构建一个项目，用来测试一个名为`structural-comparison`的npm包。

### 打包npm包的tgz压缩文件

开始建立测试项目前，在主包的位置，运行命令行，生成当地tgz文件。

```bash
structural-comparison $ npm pack
```

此命令会在`structural-comparison`文件夹生成一个压缩文件，名为`structural-comparison-1.1.0.tgz`。

### 初始化测试项目

在你希望的位置建立一个文件夹，名为`structural-comparison-test`。

在`structural-comparison-test`根目录下，使用`package.json`配置项目，其内容为：

```json
{
  "dependencies": {
  },
  "devDependencies": {
    "@babel/core": "7.15.8",
    "@babel/preset-env": "7.15.8",
    "babel-loader": "8.2.3",
    "core-js": "3.18.3",
    "jest": "27.3.1"
  },
  "scripts": {
    "test": "jest"
  },
  "name": "structural-comparison-test",
  "private": true,
  "license": "LGPL-3.0-or-later"
}
```
注：`"name"`，`"private"`，`"license"`字段是安装本地tgz包必须的字段，尽管它们似乎没有用处。

命令行，执行安装程序。

```bash
structural-comparison-test $ npm i
```

### 安装本地tgz包到测试项目

在测试项目根目录的命令行，执行安装程序。

```bash
structural-comparison-test $ npm install .../structural-comparison/structural-comparison-1.1.0.tgz
```

给出的文件来源参数用了省略号(...)，实际上键入的是文件绝对路径。一个小技巧，我们可以用鼠标拖动tgz文件，释放到当前命令行，即可代替手动打字，生成文件的绝对路径。

命令执行完成，重新用编辑器打开`package.json`文件查看。

```json
{
  "dependencies": {
    "structural-comparison": "file:../../../../structural-comparison/structural-comparison-1.1.0.tgz"
  },
}
```

`"file:"`是本地文件安装包的前缀。这是因为，`npm i`命令中的文件路径 will be normalized to a relative path and added to your package.json.

### 验证

另外，再配置好babel，如果需要的话。

我们添加一个使用`structural-comparison`项目的测试文件：

```js
import { thunk } from 'structural-comparison'

test('thunk test', () => {
    let lazyadd = thunk((a, b) => a + b)
    let y = lazyadd(1, 2)()
    expect(y).toEqual(3)
})
```

命令行执行测试命令：

```bash
structural-comparison-test $ npm test
```

如果正常运行，说明测试项目已经正确依赖了npm项目的本地包。

### 更新

当我们修改了本地包，需要更新测试项目中安装的包，只需再次执行安装命令。并且无需更改版本号，本地包将自动更新。

```bash
structural-comparison-test $ npm install .../structural-comparison/structural-comparison-1.1.0.tgz
```

执行命令最后的总结是移出了一个包，并添加了一个包。

当更新`.tgz`文件后，执行`npm i`是否会自动更新包，我还没有尝试，等有机会再来尝试补充。