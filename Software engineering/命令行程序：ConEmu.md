# 命令行程序：ConEmu

## ConEmu

这是一个命令行程序，可以从[ConEmu](https://www.fosshub.com/ConEmu.html)下载安装。

安装完成后，用管理员启动这个程序，从右上角的菜单中选择`setting...`, 在右侧的树选择`General`，如下设置：

Choose your startup task or even a shell with arguments:

```
{Shells::cmd(Admin)}
```
命令行以`#`提示，表示是管理员模式，以`$`提示则是非管理员模式，欲进入管理员模式:

在命令行窗口的标签上右击, [restart or duplicate], [restart as Adimin], 会以管理员身份重新打开当前目录。

## 命令行程序与文件资源管理器的交互

### 文件资源管理器启动命令行

文件资源管理器，在地址栏输入:

```bash
ConEmu64.exe
```

可以打开命令行程序，且命令行的当前目录是文件资源管理器的当前目录。

### 命令行启动文件资源管理器

在命令行输入以下命令，可以在文件资源管理器中打开命令行的当前目录

```bash
explorer .
```

在命令行输入以下命令，可以在文件资源管理器中打开命令行的当前目录的上级目录

```bash
explorer ..
```
## 命令行打印树状目录结构

Displays the directory structure of a path or of the disk in a drive graphically. The structure displayed by this command depends upon the parameters that you specify at the command prompt. If you don't specify a drive or path, this command displays the tree structure beginning with the current directory of the current drive.

### Syntax

```bash
tree [<drive>:][<path>] [/f] [/a]
```

*Parameters*

| Parameter  | Description                                                  |
| :--------- | :----------------------------------------------------------- |
| `<drive>:` | Specifies the drive that contains the disk for which you want to display the directory structure. |
| `<path>`   | Specifies the directory for which you want to display the directory structure. |
| /f         | Displays the names of the files in each directory.           |
| /a         | Specifies to use text characters instead of graphic characters to show the lines that link subdirectories. |
| /?         | Displays help at the command prompt.                         |

补充内容：

1. 命令行的命令和开关不区分大小写。

2. `/A` - Specifies that alternative characters (plus signs, hyphens, and vertical bars) be used to draw the tree diagram so that it can be printed by printers that don't support the line-drawing and box-drawing characters.

3. 打印出来的内容，文件夹前面一定有横线，文件前面一定没有横线。

### Examples

To display the names of all the subdirectories on the disk in your current drive, type:

```bash
tree \
```

显示当前目录的树：

```bash
tree .
```

把打印出的树状目录结构写入到tree.txt文件中

```bash
tree /f > tree.txt
```

打印当前目录下的文件夹和文件，输出：

```bash
$ tree . /f                                    
卷 xxx 的文件夹 PATH 列表                             
卷序列号为 xxxxxx                       
D:\XP44MM\SHAPES\SHAPES.CLIENT\CLIENTAPP       
│  .gitignore                                  
│  babel.config.js                             
│  index.css                                   
│  index.js                                    
│  jest.config.js                              
│  jest.setup.js                               
│  package-lock.json                           
│  package.json                                
│  prettier.config.js                          
│  webpack.common.js                           
│  webpack.dev.js                              
│  webpack.prod.js                             
│                                              
├─assets                                       
│      favicon.ico                             
│      index.html                              
│                                              
```
