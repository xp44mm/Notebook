## Create your local repo

Create a local Git repo for your code. If your code is already in a local Git repo, you can skip this step.

1. Navigate to the folder where your code is on the command line:

   ```
   cd /home/fabrikam/fiber
   ```

2. Create a Git repo on your machine to store your code. 

   ```
   git init .
   ```

3. 在VS中，打开文件夹，执行其他git操作。如果已经打开，将文件夹关闭，重新打开。



### 删除未提交的推送

本地历史记录

要删除的项目，上下文菜单，重置，删除更改。



## 忽略文件

在git中如果想忽略掉某个文件，不让这个文件提交到版本库中，可以使用修改根目录中`.gitignore`文件的方法（如无，则需自己手工建立此文件）。这个文件每一行保存了一个匹配的规则，例如：

```Python
# 此为注释 - 将被Git忽略
*.a       # 忽略所有 .a 结尾的文件
!lib.a    # 但 lib.a 除外
/TODO     # 仅仅忽略项目根目录下的TODO文件，不包括子目录下的/TODO
build/    # 忽略 build/ 目录下的所有文件
doc/*.txt # 会忽略 doc/notes.txt 但不包括 doc/server/arch.txt
```

规则很简单，不做过多解释，但是有时候在项目开发过程中，突然心血来潮想把某些目录或文件加入忽略规则，按照上述方法定义后发现并未生效，原因是`.gitignore`只能忽略那些原来没有被track的文件，如果文件以及被纳入了版本管理中，则修改`.gitignore`是无效的。那么解决方法就是先把本地缓存删除（改变成未track状态），然后再提交。

```
git rm -r --cached .
git add .
git commit -m 'update .gitignore'
```

其中：

```
git rm [path_and_filename] --cached
```

该命令将永远不想签入的文件从Git索引（index）中移除，如果不想这个文件存在磁盘上，可以使用

```
git rm [path_and_filename] --force
```

将操作后的文件（此时已经是staged）提交就好了



1. `.gitignore`是最先想到的方法，但是也不行，不过SO上有人貌似是可以的，如下：

```python
 # Visual Studio 2015 cache/options directory
 .vs/
```



微软官档：

## Permanently ignore changes to a file

If a file is already tracked by Git, adding that file to your `.gitignore` is not enough to ignore changes to the file. You also need to remove the information about the file from Git's index

These steps will not delete the file from your system. They just tell Git to ignore future updates to the file.

1. Add the file in your `.gitignore`.

2. Run the following:

   > git rm --cached

3. Commit the removal of the file and the updated `.gitignore` to your repo.



### 1. 关于将已经ignore的文件(夹)重新加入跟踪里面

重新添加已经被忽略过的文件（夹）

重新添加已经被忽略过的文件时，我们仅仅使用`git add`是不行的，因为git仓库中根本没有那个文件，这时候我们需要加上`-f`参数来强制添加到仓库中，然后再提交。比如上面设置了忽略排除的文件，我们需要重新加入

```
git add -f [文件(夹)名]
```

然后再`commit`和`push`就行了

------

### 2. 关于VS中经常出现的error: Your local changes to the following files would be overwritten by merge:

> error: Your local changes to the following files would be overwritten by merge:

意思是我本地上新修改的代码的文件，将会被git服务器上的代码覆盖；我当然不想刚刚写的代码被覆盖掉，看了git的手册，发现可以这样解决：

##### 方法1

如果你想保留刚才本地修改的代码，并把git服务器上的代码pull到本地（本地刚才修改的代码将会被暂时封存起来）

```
 git stash  
 git pull origin master  
 git stash pop  
```

如此一来，服务器上的代码更新到了本地，而且你本地修改的代码也没有被覆盖，之后使用add，commit，push 命令即可更新本地代码到服务器了。

##### 方法2

如果你想完全地覆盖本地的代码，只保留服务器端代码，则直接回退到上一个版本，再进行pull：

```
 git reset --hard
 git pull origin master
```

注：其中origin master表示git的主分支。