正常使用node执行整个JavaScript文件的方法：

如果要用node直接执行一整个JavaScript文件的话，那么就不能进入node的编辑模式，而应该直接在命令框里面输入：

```bash
node <JavaScript文件名>
```

如下：

```bash
node hello.js
```

或者

```bash
node hello
```

对于跟在node后面的JavaScript文件，`.js`后缀可加可不加。如果JavaScript文件不在当前命令行所在目录的话，你需要先将命令行定位到相应目录下，

```bash
C:\Users\administrator\Desktop\
$ node hello.js
```

或者当前命令行在任一目录下，node后面加上js文件名的具体路径：

```bash
in any path
$ node C:\Users\administrator\Desktop\hello.js
```

这时问题来了，这个文件的全路径名称很难写。我们可以用拖拽的方法，在任意目录下，先打出命令的前半部分

```bash
in any path
$ node 
```

用鼠标按住JavaScript文件，拖放到命令行：

```bash
in any path
$ node C:\Users\administrator\Desktop\hello.js
```

即可得到想要的命令。按回车执行即可。我们写一个打印淘宝地址的`taobao.js`文件，并保存到桌面：

```js
console.log('--registry=https://registry.npm.taobao.org');
```

用鼠标按住JavaScript文件，拖放到命令行，按回车执行命令：

```bash
in any path
$ node C:\Users\administrator\Desktop\taobao.js
--registry=https://registry.npm.taobao.org
```

全文完，谢谢阅读。