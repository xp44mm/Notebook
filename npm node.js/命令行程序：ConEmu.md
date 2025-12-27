# 命令行程序：ConEmu

## ConEmu

这是一个命令行程序，可以从[ConEmu](https://github.com/Maximus5/ConEmu/releases)下载安装。

安装完成后，用管理员启动这个程序，从右上角的菜单中选择`setting...`, 在右侧的树选择`General`，如下设置：

Choose your startup task or even a shell with arguments:

```
{Shells::cmd(Admin)}
```
命令行以`#`提示，表示是管理员模式，以`$`提示则是非管理员模式，欲进入管理员模式:

在命令行窗口的标签上右击, [restart or duplicate], [restart as Adimin], 会以管理员身份重新打开当前目录。

确保添加路径

#### 打开系统环境变量设置

- 按 `Win + S`，搜索 "编辑系统环境变量"

- 选择环境变量

- 在 **"系统变量"** 部分，找到 `Path`，点击 **"编辑"**。

- 点击 **"新建"**，然后粘贴 ConEmu 的安装路径。

  右击桌面快捷方式，打开文件所在位置并将其复制：`C:\Program Files\ConEmu`

- 点击 **"确定"** 保存所有更改。



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



### `cls` 命令的帮助文档

在 Windows 命令行（CMD）中，`cls` 命令用于清除当前命令提示符窗口的所有输出内容，并将光标重置到窗口左上角。  

在 CMD 中，你可以使用 `help cls` 或 `cls /?` 查看其帮助信息：
```cmd
C:\> help cls
清除屏幕。

CLS
```
（Windows 的 `cls` 命令非常简单，没有额外参数。）

### 功能说明
- **作用**：清空当前 CMD 窗口的所有文本，只保留命令提示符（如 `C:\>`）。
- **特点**：
  - 不会影响命令历史记录（仍可用 `↑`/`↓` 查看之前执行的命令）。
  - 不会关闭或重启 CMD 窗口，仅清理显示内容。
  - 不支持参数（如 `cls /a` 会报错）。

### 使用示例
```cmd
C:\> echo Hello World
Hello World

C:\> cls  （执行后屏幕清空，仅显示 `C:\>`）
```
