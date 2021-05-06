> 没有魔术，显式而不使用默认值。
> 

所有结构只有一种形式，不可以采用简写形式。

数据不完整将引发错误。

# 数据类型与构造函数

原生支持JSON

数据与类型分离，因为有类型信息，数据序列化用JSON格式。

基本不可变类型为：`unit`，数组，元组，记录（F#中的匿名记录）。数组用`list`语法。

构造函数表示为通用类型绑定一组功能，构造函数用法如下:

```F#
let rgx = new Regex()
```

运算符`new`优先级最高。

元组的小括号不可以省略。

不支持函数的参数表。

# 缩进与语句块

不支持缩进代表语句块，显式的语句开闭括号。优点是可以表示空语句，空块。可以写二维表格对齐的程序。



## 代码漂亮的缩进：

闭括号开始的行，对齐闭括号配对的开始符号所在的行的缩进位置。

其他符号开始的行，所在语句块开括号所在行的缩进位置，比后者缩进一次。


# 语句

语句块返回值为最后一个语句的返回值。像F#一样，不显式使用`return`关键字。

使用JavaScript的选择`if`语句，`if`的语句块、`else if`的语句块、`else`的语句块大括号不可以省略。

不使用循环语句，循环语句用`Array.iter`库函数实现。



基本库中有可变的结构类型：比如词典，队列，集合，栈。

语句块的大括号不能省略。

用一对空的大括号`{}`表示空语句块，这是因为F#没有空对象。一对空大括号`{}`是`{()}`的简写。

```F#
let fn x = if(x == null) {0} else if(x == true){1} else {-1}

```


# 函数定义

curried函数是一个输入参数，返回一个输出参数，函数表达式定义方法与JavaScript的函数定义方法相同，修改为：

```
let fn x y z = ....
let fn = x => y => z => {....}
```
输入参数是某个数据结构的模式匹配，一定是单个树节点。

不等号`!=`。

函数一定只有一个输入参数，一定只有一个输出参数。

函数体的语句块两边的大括号不可以省略。

定义的函数要带有名称，泛型时，不能用函数表达式定义，用`let`语句定义，或者全写的函数表达式。

```js
let foo<''> a b = ...

let fn = function<'K'> (obj:'K') {...}

let foo = function bar<'K'> (obj:'K') {...}
```

We could consider allowing named arguments within immediate applications of curried functions, but where correct curried ordering is still required, e.g.

```F#
let sum (m,km) dm cm mm = 1000000*km + 1000*m + 100*dm+10*cm+mm
let res = sum (?m=2,?km=1) (?dm=3) (?cm=4) (?mm=5)
```

Here named arguments mainly serve for code clarity. I'm not in favour of allowing changed ordering of curried parameters.

# 泛型

泛型类型，与泛型方法，同C#泛型类型与泛型方法定义不变：

```F#
let empty x = ..
let single<'t> (x:'t) = ...
let map<'k,'v> (pairs:('k*'v)[]) = ...
```

泛型形参的前缀？

# 模式匹配与数据解构

同F#，不使用活动模式。

# 面向对象与可变性

原生支持reactive，

删除面向对象中的属性支持，属性通知用`BehaviorSubject`实现。

删除可变量

F#的状态，可变量归纳到可变的数据结构中。删除多余的关于改变状态的操作符，比如`<-`

```F#
let i = stack<int>()
do i.push(0)
while i.top < 5 do
    print "done!"
    i.push(i.top+1)
```

任何副作用，以及对状态的修改，必须放在`do`语句中，或者`Array.iter`，`Array.do`函数中。

不支持面向对象中的参数表。

可选参数，

默认参数。

引用参数。

# 类型与反射

类型有一个语义标签。匿名函数、元组分量应该带有提示性标签。

功能与数据分离，类与数据分离。比如三大基本功能render，type，comparison抽象为接口，可以配置。

```F#
interface render =
  member filter: type -> obj -> bool
  member print: (type -> obj -> bool) -> type -> obj -> bool
```

# 源代码的管理

模块化，`import`/`export` 文件打包工具。删除F#模块，函数，类型，类定义直接放置在文件中。

删除作用域访问修饰符 `private`，`public`，用缓存函数的闭包代替私有成员。举例。

删除嵌套类型。类型直接定义在顶级。

删除名字空间。

导入名称支持重载（overload），编译器根据名称代表对象的类型，函数参数等上下文信息确定引用的实例，必须显式标注`overload`为每个将重载的名称。

When two or more different declarations are specified for a single name in the same scope, that name is said to *overloaded*. By extension, two declarations in the same scope that declare the same name but with different types are called *overloaded declarations*. Only function declarations can be overloaded; object and type declarations cannot be overloaded.

# 类型推导

类型推导，顶级函数的参数表必须显式定义输入参数的类型。同一名称的函数可以根据输入参数类型不同进行区分。

```F#
let x (a:int,b:int,c:int) = .....
let x (a:int,b:int) = ....
```





