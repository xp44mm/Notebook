# DOT 语言 GUIDE
## 第一部分  设置点和线的形状和颜色
### 例1：
先来看一个例子，我们创建一个文档graph1.dot：
digraph是有向图，graph是无向图，要注意，`->`用在有向图中，`--`用在无向图中表示一条边，不能混用。 

```dot
 digraph G {
 main -> parse -> execute; 
 main -> init;
 main -> cleanup;
 execute -> make_string;
 execute -> printf
 init -> make_string;
 main -> printf; 
 execute -> compare; 
 } 
```
第一行给出了图的类型和名字，
当一个点第一次出现，它就被创建了
用->标示符创建一条边 

菜单$Graph\to Settings$，然后在弹出对话框点Ok，将输出输出图像文件。

画出以下图形： 

![graph1](graph1.png)
图1：Drawing of small graph

###  例2：

来看下一个稍微复杂点的例子，我们开始手动的设置一下图的属性。可以给点设置属性，也可以给边设置属性。先来讲讲怎么设置边的属性，在每条边后面的双括号里设置边的属性。也可以在用edge设置边的默认值。 

而给点设置属性就必须给每个点单独的设置一个属性，node表示点的默认值。点的默认参数是shape=ellipse, width=.75, height=.5 并且labeled by the node name. 一些点的形状在附录H中，常用的形状有 bos, circle, record, plaintext。 

```dot
digraph G {     
size ="4,4";  
main [shape=box]; /* this is a comment */ 
main -> parse [weight=8];
parse -> execute; 
main -> init [style=dotted];
main -> cleanup; 
execute -> { make_string; printf}
init -> make_string; 
edge [color=red]; // so is this
main -> printf [style=bold,label="100 times"]; 
make_string [label="make a\nstring"]; 
node [shape=box,style=filled,color=".7 .3 1.0"];
execute -> compare; 
}
```
把图的尺寸设为4 inch，4 inch
把main点的形状设为方形 
weight是设置了这条边的重要程度，默认是1
让这条线是点状的
这条语句一次连了两条线
把边的默认颜色设为了red
label就是在边上写了一行字
让make_string变成了一个两行的字符串（注意那个\n）
设置了一下点的默认参数，蓝色，这个被用在了compare中

画出以下图形： 

![graph2](graph2.png)

图2：Drawing of fancy graph

### 例3：

可以设置每条边箭头的方向，用 dir，有 forward(default)，back，both，none 四种。

```dot
digraph html { 
A -> B[dir = both]; 
B -> C[dir = none]; 
C -> D[dir = back]; 
D -> A[dir = forward]; 
}
```

![graph3](graph3.png)

### 例4：

点的 shape 除了 record 和 Mrecord 这两种之外，其他的形状都是多边形，而我们可以对多边形进行一下属性上的设置，shape = polygon。Sides 用于设置它的边数，peripheries 用于设置多边形的外框的层数，regular = true 可以让你的多边形是一个规则的多边形，orientation = * ，可以让你的多边形旋转一个角度，如 orientation = 15 就是转了 15 度。Skew 后面跟一个（-1.0~1.0）的小数，能让你的图形斜切一个角度，distortion 是让你的图形产生透视效果。 

```dot
digraph G { 
a -> b -> c; 
b -> d; 
a [shape=polygon,sides=5,peripheries=3,color=lightblue,style=filled]; 
c [shape=polygon,sides=4,skew=.4,label="hello world"] 
d [shape=invtriangle]; 
e [shape=polygon,sides=4,distortion=.7]; 
} 
```

![graph4](graph4.png)

### 例5：
```dot
digraph A{ 
A -> B; 
A[orientation = 15, regular = true, shape = polygon, sides = 8, peripheries = 4, color = red style = filled]; 
B[shape = polygon, sides = 4, skew = 0.5, color = blue]; 
}
```
![graph5](graph5.png)

### 例6：
record 和 Mrecord 的区别就是 Mrecord 的角是圆的。Record 就是由横的和竖的矩形组成的图形。 

```dot
digraph structs { 
node [shape=record]; 
struct1 [shape=record,label="<f0> left|<f1> mid\ dle|<f2> right"]; 
struct2 [shape=record,label="<f0> one|<f1> two"]; 
struct3 [shape=record,label="hello\nworld |{ b |{c|<here> d|e}| f}| g | h"]; 
struct1 -> struct2; 
struct1 -> struct3; 
}
```
![graph6](graph6.png)

### 例7：
当你的线和线 label 比较多时，可以给线的属性 decorate = true，使得每条线的 label 与所属线之间连线。你还可以给每条线加上 headlabel 和 taillabel，给每条线的起始点和终点加上label，他们的颜色由 labelfontcolor 来决定，而 label 的颜色由 fontcolor 来决定。 

```dot
graph A{ 
label = "I love you"; //给这幅图设置，名字 
labelloc = b;         //图名字的位置在 bottom，也可以是 t 
labeljust = l;        //图名字的位置在 left，也可以是 r 
 
edge[decorate = true]; 
C -- D[label = "s1"]; 
C -- E[label = "s2"]; 
C -- F[label = "s3"]; 
D -- E[label = "s4"]; 
D -- F[label = "s5"]; 
edge[decorate = false, labelfontcolor = blue, fontcolor = red]; 
C1 -- D1[headlabel = "c1", taillabel = "d1", label = "c1 - d1"]; 
} 
```
![graph7](graph7.png)

### 例8：
在 dot 中我们可以用 html 语言写一个 table。在 label 后用`< >`而不是`""`就能引入 html 语言。
```dot
digraph html { 
abc [shape=none, margin=0, label=< 
<table border="0" cellborder="1" cellspacing="0" cellpadding="4"> 
<tr><td rowspan="3"><font color="red">hello</font><br/>world</td> 
<td colspan="3">b</td> 
<td rowspan="3" bgcolor="lightgrey">g</td> 
<td rowspan="3">h</td> 
</tr> 
<tr><td>c</td> 
<td port="here">d</td> 
<td>e</td>
</tr> 
<tr><td colspan="3">f</td> 
</tr> 
</table>>]; 
}
```

![graph8](graph8.png)

### 例9：
这样创造了一个 5 行 5 列的表格，我们可以在表格中打字。 
```dot
digraph html { 
abc [shape=none, margin=0, label=< 
<table border="0" cellborder="1" cellspacing="0" cellpadding="4"> 
<tr><td>0</td><td>1</td><td>2</td><td>3</td><td>4</td> 
</tr> 
<tr><td>1</td><td></td><td></td><td></td><td></td> 
</tr> 
<tr><td>2</td><td></td><td></td><td></td><td></td> 
</tr> 
<tr><td>3</td><td></td><td></td><td></td><td></td> 
</tr> 
<tr><td>4</td><td></td><td></td><td></td><td></td> 
</tr> 
</table>>]; 
} 
```

![graph9](graph9.png)

## 第二部分  设置点和线的位置，子图的概念

### 例10：
默认时图中的线都是从上到下的，我们可以将其改为从左到右，在文件的最上层打入rankdir=LR 就是从左到右，默认是 TB(top -> bottom)，也可以是 RL，BT。 

当图中时间表之类的东西时，我们会需要点能排在一行（列），这时要用到 rank，用花括号把 rank=same，然后把需要并排的点一次输入。 
```dot
digraph html { 
rankdir = LR; 
{ 
    node[shape = plaintext]; 
    1995 -> 1996 -> 1997 -> 1998 -> 1999 -> 2000 -> 2001; 
}
{ 
    node[shape = box, style = filled]; 
    WAR3 -> Xhero -> Footman -> DOTA; 
    WAR3 -> Battleship; 
}
{rank = same; 1996; WAR3;} 
{rank = same; 1998; Xhero; Battleship;} 
{rank = same; 1999; Footman;} 
{rank = same; 2001; DOTA;} 
} 
```
![graph10](graph10.png)

### 例11：
设立一条边时，我们可以制定这条边从起点的那个位置射出和从哪个位置结束。控制符有"n", "ne", "e",  "se", "s", "sw", "w"  和  "nw"，具体效果见下： 

```dot
digraph html { 
node[shape = box]; 
c:n -> d[label = n]; 
c1:ne -> d1[label = ne]; 
c2:e -> d2[label = e]; 
b:se -> a[label = se]; 
c3:s -> d3[label = s]; 
c4:sw -> d4[label = sw]; 
c5:w -> d5[label = w]; 
c6:nw -> d6[label = nw]; 
} 
```
![graph11](graph11.png)

### 例12：
我们也可以在 record 中给点定义一些 port，因为 record 类型中都是一个个格子。

```dot
digraph html { 
label = "Binary search tree"; 
node[shape = record]; 
A[label = "<f0> | <f1> A |<f2> "]; 
B[label = "<f0> | <f1> B |<f2> "]; 
C[label = "<f0> | <f1> C |<f2> "]; 
D[label = "<f0> | <f1> D |<f2> "]; 
E[label = "<f0> | <f1> E |<f2> "]; 
A:f0:sw -> B:f1; 
A:f2:se -> C:f1; 
B:f0:sw -> D:f1; 
B:f2:se -> E:f1; 
} 
```
![graph12](graph12.png)

### 例13：
构造一个 HASH 表。

```dot
digraph G { 
nodesep=.05;
rankdir=LR;
node [shape=record,width=.1,height=.1];
node0 [label = "<f0> |<f1> |<f2> |<f3> |<f4> |<f5> |<f6> | ",height=2.5]; 
node [width = 1.5];
node1 [label = "{<n> n14 | 719 |<p> }"];
node2 [label = "{<n> a1 | 805 |<p> }"];
node3 [label = "{<n> i9 | 718 |<p> }"];
node4 [label = "{<n> e5 | 989 |<p> }"];
node5 [label = "{<n> t20 | 959 |<p> }"] ;
node6 [label = "{<n> o15 | 794 |<p> }"] ;
node7 [label = "{<n> s19 | 659 |<p> }"] ;

node0:f0 -> node1:n;
node0:f1 -> node2:n;
node0:f2 -> node3:n;
node0:f5 -> node4:n;
node0:f6 -> node5:n;
node2:p -> node6:n;
node4:p -> node7:n;
} 
```
![graph13](graph13.png)
### 例14：
画一个子图就是 subgraph cluster#，必须有 cluster 前缀。 

```dot
digraph G {
subgraph cluster0 {
node [style=filled,color=white];
style=filled;
color=lightgrey;
a0 -> a1 -> a2 -> a3;
label = "process #1";
}
subgraph cluster1 {
node [style=filled];
b0 -> b1 -> b2 -> b3;
label = "process #2";
color=blue
}
start -> a0;
start -> b0;
a1 -> b3;
b2 -> a3;
a3 -> a0;
a3 -> end;
b3 -> end;
start [shape=Mdiamond];
end [shape=Msquare];
}
```
![graph14](graph14.png)

### 例15：
当你想把一条边连到一个子图的边界上，先输入 compound = true，然后就能用 lhead 和 ltail 来设置连接的子图了。 

```dot
digraph G {
compound=true;
subgraph cluster0 {
a -> b;
a -> c;
b -> d;
c -> d;
}
subgraph cluster1 {
e -> g;
e -> f;
}
b -> f [lhead=cluster1];
d -> e;
c -> g [ltail=cluster0, lhead=cluster1];
c -> e [ltail=cluster0];
d -> h;
}
```
![graph15](graph15.png)


