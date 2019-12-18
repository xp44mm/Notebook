# HTML CSS整理笔记



点击链接后退页面：

```html
<a href="javascript:history.go(-1)">回到上一个网页</a>
```

——修改placeholder提示的样式：

```
类名或标签名::placeholder {color: red;}
```


css超出一行显示省略号：

给定宽度(width:100px)、

超出隐藏（overflow:hidden）、

强制在同一行显示（white-space: nowrap）、

省略号（text-overflow:ellipsis）

——常见字体单位——

**1.em**

相当于“倍”，比如设置当前的div的字体大小为1.5em，则当前的div的字体大小为：该div继承的父级字体大小*1.5。（根据他爸）

**2.rem**

r即root，始终相对于根节点html的font-size进行缩放。（根据祖先html）

**3.vh**

vh指当前屏幕可见高度的1%，即 height:100vh == height:100%;

它的好处：当元素没有内容时候，若设置height:100%该元素高度不会被撑开。但设置height:100vh，该元素会被撑开和屏幕高度一致。

**4.vw**

viewpoint width，视窗宽度，1vw = 视窗宽度的1%。

vw就是当前屏幕宽度的1%，

对比：

width:100%; 设置元素宽度占父元素的宽度100%。

width:100vw; 相对于屏幕可见宽度来设置，所以会出现50vw 比50%大的情况。

# **从这里往下是分类整理**

——1.HTML5基础——

5.在网页中，HTML决定结构和内容，CSS设定网页的表现样式，JavaScript控制网页的行为。

6.<!DOCTYPE html>必须位于HTML文档第一行。

7.<meta>标签：用于方便浏览器解析或搜索引擎搜索，一般放置于<head>中，用"名称/值"方式：

(1)表示文档内容类型、字符串编码信息 如：<meta charset="UTF-8">

(2)为搜索引擎定义关键词:

<meta name="keywords" content="HTML,CSS,XML,XHTML,JavaScript">

(3)为网页定义描述内容:

<meta name="description" content="Free Web tutorials on HTML and CSS">

(4)定义网页作者:

<meta name="author" content="Hege Refsnes">

(5)每30秒中刷新当前页面:

<meta http-equiv="refresh" content="30">

8.字体样式标签：<strong>字体变粗、<em>字体倾斜

9.注释 <!--内容-->

10.特殊符号：空格 >大于号> <小于号< "引号" 版权符号©

11.常用图片格式：JPG、GIF、PNG、BMP

12.图片标签，必须要有src和alt属性：

<img src="图片地址" alt="图片的替代文字" title="鼠标悬停提示文字" width="图片宽度" height="图片高度" />

13.超链接标签(target指定在哪个窗口打开 值有_self自身窗口、_blank新建窗口)

<a href="链接地址" target="目标打开窗口位置">附连接的文本或图像</a>

14.链接地址

(1)绝对路径(指向目标地址的完整描述 多指向本站点外的文件

如<a href="http://www.baidu.com/index.html">百度</a>)、

(2)相对路径(一般指向本站点内的文件,如<a href="login/login.html">登陆<a>)

(3)相对路径中"../"表示当前目录的上级目录，"../../"表示上上级目录

15.超链接的应用场合：

(1)页面间链接：如<a href="login.html" target="_blank">为您跳转到登录页</a>

(2)锚链接：

先在目标位置B设置标记如：<a name="new">这里是目标位置</a>，

然后在A位置设置链接路径href属性值为"#标记名"如：<a href="#marker"当前位置A</a>

(3)功能性链接：单击时启动本机自带的应用程序如QQ、电子邮箱等

如电子邮件链接："mailto:电子邮件地址"

16.元素分类

(1)块元素：如<p><h1><div>无论内容有多少，该元素都独占一行(一块)。

块状元素特点：如果没有设置自身宽度，则显示为父容器的100%。

(2)行内元素：如<strong><a> 显示宽度由自己的内容决定，其他元素可以排在它后面。

16.元素类型转换：

(1)块状元素转为内联元素：display: inline;

(2)内联元素转为块状元素：display: block;

(3)把元素设为内联块状元素： display: inline-block;

(就是同时具备内联元素、块状元素特点，如img、input)

——2 列表、表格、媒体元素——

17.三种列表：

(1)有序列表<ol><li>

(2)无序列表<ul><li> ul中只能嵌套li，而li可以嵌套任意标签。

(3)定义列表<dl><dt><dd> 是标题及列表项的结合。

18.表格基本结构：单元格、行、列

(1)<table><tr><th><td>

(2)HTML5中已废除table的border属性，用css控制边框宽度。

(3)跨列(横跨)：<td colspan="所跨的列数">内容</td>

跨行(竖跨)：<td rowspan="所跨行数">内容</td>，两者都要删除被合并的其他单元格。

(4)表格特点：同行单元格高度一致且水平对齐，同列单元格宽度一致且垂直对齐。

19.视频元素：

(1)controls属性提供播放暂停和音量控件、autoplay属性自动播放、loop属性循环播放

<video src="视频路径" controls="controls">你的浏览器不支持video标签</video>
(2)source元素链接不同的视频文件，浏览器会自动选择第一个可识别的格式：

<video controls>

<source src="video/video.webm" />

<source src="video/video.mp4" />你的浏览器不支持video标签

<video>

20.音频元素：使用语法和视频元素一样，只要把video换成audio即可。

21.HTML5的结构元素(先划分结构再写内容)：

header(头部)、footer(脚部)、

section(独立区域)、article(独立文章内容)、

aside(相关内容或应用,常用于侧边栏)、nav(导航类辅助内容)

22.<iframe>框架:方便在页面中引用站外的页面内容。

<iframe name="此框架的标识名" src="引用的页面地址"></iframe>
23.<iframe>和锚链接的结合：使锚链接的内容在iframe框架中打开

<iframe name="mainFrame" src="框架引用的页面地址" />
<a href="链接路径" target="mainFrame">点击在框架中打开</a>

——3 表单——

24.表单标签form：

<form method="post" action="login.html" enctype="text/plain">

表单内容

</form>

(1)action="url"属性意为把表单提交到某个页面,method=get|post意为向服务器发送数据的方式。

(2)提交方法：get提交,表单数据会在地址栏url中显示；而post提交不会显示，所以post提交更安全。

(3)enctype="text/plain"指enctype 属性规定在发送到服务器之前应该如何对表单数据进行编码。text/plain 空格转换为加号+，但不对特殊字符编码。

24.表单元素：

(1)表单元素<input>标签的属性：

type(默认text,其他password,email,checkbox,radio,button,submit,reset,file,image,url,hidden,number,range,search等)、name、value(可选,该元素的初始值)、size(该元素的初始宽度)、maxlength(可输入的最大字符数)、checked(按钮被选中)

(2)列表框<select><option>标签：

<select>中至少包含一个<option>。在<option>有多行选项需滚动查看时，size属性设置可提示看到的行数，selected属性默认选中该列表项。

(3)按钮：button普通(要和事件如onclick关联使用),submit(提交表单到action指定的url并传递表单数据),reset重置。要求美观可使用图片按钮如<input type="image" src="图片路径"/>

(4)多行文本域：不能用value属性赋初始值

<textarea name="标识名" cols="显示的列数" rows="显示的行数">

自我评价

</textarea>

(5)数字number：限制输入的数据为数字，设定最大值最小值、合法的数据间隔step或默认值等

<input type="number" name="num" min="0" max="100" step="10"/>

(6)滑块range：作用和数字number一样，只是外观显示为用滑动条选择数值

<input type="range" name="range" min="0" max="100" step="10"/>

(7)search搜索框:在任意页面中用于输入搜索关键词的文本框

<input type="search" name="sousuo" />

(8)文件域：多用于文件上传

<input type="file" name="files"/>

<input type="submit" name="upfiles" value="上传"/>

(9)当表单数据包含普通数据、文件数据等多部分内容时，要设置表单的enctype编码属性为 multipart/form-data,表示把表单数据分为多部分提交。

(10)表单隐藏域hidden：数据不会页面中显示，但会随表单一同提交。

<input type="hidden" name="userid" value="20"/>

(11)表单元素 只读属性readonly、禁用disabled

(12)W3CHTML5标准中，规定对布尔类型的属性，属性值可以省略。

(13)表单元素的标注label：当鼠标单击标注的文本时，浏览器会自动对焦关联的表单元素，for属性规定label与哪个表单元素绑定。name和id属性必需。

<label for="female">女</label>

<input type="radio" name="sex" id="female" />

24.HTML5表单新标签

<form>供用户输入的表单

<input>输入域

<textarea>文本域 (多行输入)

<label>定义 <input> 元素的标签，一般为输入标题

<fieldset>定义一组相关的表单元素，并使用外框包含起来

<legend>定义 <fieldset> 元素的标题

<select>下拉选项列表

<optgroup>选项组

<option>下拉列表中的选项

<button>点击按钮

<datalist>指定一个预先定义的输入控件选项列表

<keygen>定义了表单的密钥对生成器字段

<output>计算结果

25.表单验证

(1)好处：减轻服务器的压力；保证数据的可行性和安全性。

(2)placeholder:为文本框提示用户输入

<input type="text" name="name" placeholder="请输入你的姓名"/>

(3)required：规定文本框不能为空

<input type="email" name="email" required/>

(4)pattern：输入的内容必须符合正则表达式自定义的规则

<input type="text" name="tel" required pattern="^1[35]\d{9}"/>规定以13、15开头的11位数字

——4 CSS3基础——

26.CSS：Cascading Style Sheet层叠样式表,又称风格样式表Style Sheet，用于设计网页风格。

27.CSS3的基本语法结构：

(1)CSS中注释为 /*内容*/

(2)CSS规则由选择器、声明组成。

(3)声明必须放在大括号{}中，声明可以是一或多条。

(4)每条声明由一个属性和值组成，属性和值用冒号:分开，每条语句以分号;结尾如

<style type="text/css">

h1{

font-size : 12px;

color : red;

}

</style>

28.在HTML中引入CSS样式的方式：

(1)行内样式：直接在标签中用style属性设置CSS样式。

<p style="font-size:10px;">字体大小</p>
(2)内部样式表：把css代码写在<head>的<style>中，规范化应写为<style type="text/css">

(3)外部样式表：文件扩展名为.css 在外部样式表中可直接写样式代码，不需要<style>标签。

a.链接式引用外部样式表(常用)：

<head>

<link href="外部样式表路径" rel="stylesheet" type="text/css"/>

</head>

b.导入式引用外部样式表：

<head>

<style>

@import url("外部样式表路径");

</style>

</head>

29.样式优先级："就近原则"，行内样式>内部样式表>外部样式表

当有很多样式时，用 !important 可以为某一个样式覆盖掉其他所有样式。

如 #textcolor{ clor:pink !important;}

30.CSS选择器命名规范：驼峰命名法，用语义化单词命名且不能和ID选择器同名.

31.CSS3的基本选择器

(1)标签选择器：以标签名作选择器的名称如 h1{color:red;}

(2)类选择器：选择器名可自定义如 .red{color:red;}同时要设置<标签 class="red">内容</标签>

(3)ID选择器：选择器名可自定义如 #red{color:red;}同时要设置<标签 id="red">内容</标签>，但同一个id属性的选择器在页面中只能用一次。

32.基本选择器的优先级：ID选择器>类选择器>标签选择器

31.CSS3的高级选择器

1.层次选择器：

(1)后代选择器A B{ }：中间用空格隔开，只要是A的后代元素都会被选中。

(2)子选择器A>B{ }：只能选择A的子元素。

(3)相邻兄弟选择器A+B{ }：只用于A后面一个同级元素

(4)通用兄弟选择器A~B{ }：用于A后面所有的同级元素

2.结构伪类选择器：根据文档对象模型DOM的节点(元素级别)来操作。

(1)B:first-child作为父元素的第一个子元素B，作用和(3)相似

(2)B:last-child作为父元素的最后一个子元素B

(3)A B:nth-child(n)在父级中查第n个子元素是不是B，不分类型

(4)B:first-of-type 选择父元素内B类型的第一个元素B

(5)B:last-of-type选择父元素内B类型的最后一个元素B

(6)A B:nth-of-type(n) 在父级里先是不是B类型，再看位置n

3.属性选择器：

(1)A[arrt] 选择包含属性arrt的A标签(也可写多个属性，但要同时存在)

(2)A[arrt = val] 选择包含属性arrt,且属性值=val(区分大小写)的A标签

(3)A[arrt ^= val] 选择包含属性arrt,且属性值以val开头的任意字符串

(4)A[arrt $= val] 选择包含属性arrt,且属性值以val结尾的任意字符串

(5)A[arrt *= val] 选择包含属性arrt,且属性值包含val字符串的A标签

——5 CSS3美化网页——

32.CSS3设置文本样式：

(1)<span>标签：用来设置行内元素(或几个文字)的样式。

(2)字体样式：

font-size：常用单位px

font-family：若同时设中英文字体，英文字体要设置在中文字体前面

font-style：normal标准、italic斜体、oblique倾斜

font-variant：small-caps; 字体设置为新型的大写字母，所有小写字母都转换为大写。

font-weight：normal标准、bold粗、bolder更粗、lighter细、100-900数字越大越粗

font：一次设置字体所有属性，顺序为"字体风格-粗细-大小-类型"

如 font:italic bold 36px "宋体";

(3)用font简写方式至少要指定 font-size和 font-family 属性，其他的属性(如font-weight、font-style、font-variant、line-height)如未指定将使用默认值。缩写时 font-size 与 line-height中间要加"/"斜扛如 "12px/1.5em"



32.Text-transform：控制文本的大小写：

none 默认，定义既有小写字母也有大写字母的标准文本(原文)

capitalize 每个单词以大写字母开头

uppercase 全部为大写字母

lowercase 全部小写字母

inherit 从父元素继承text-transform属性的值。

32.direction属性：规定文本的方向/书写方向。

ltr文本方向从左到右

rtl方向从右到左

inherit 继承父元素direction属性的值。

32.文字排版

(1)适用大多数浏览器：

从左向右 writing-mode: vertical-lr;

从右向左 writing-mode: vertical-rl;

(2)只适用IE浏览器:

从左向右 writing-mode: tb-lr;

从右向左 writing-mode: tb-rl

33.排版网页文本

(1)color文本颜色:

RGB：如color:#FF0000; 另一种方法rgb(r,g,b)其中三个参数取整0~255

RGBA：在RGB基础上加控制alpha透明度的参数，取值0~1，0表示完全透明

(2)text-align水平对齐:

left左(默认)、center中间、right右、justify两端对齐

(3)text-indent首行缩进：2em或2px 缩进两个字符

(4)text-height文本行高: 单位px或 按倍数(行高是字体大小的倍数)

(5)text-decoration文本装饰：

none默认无、underline下划线、overline上划线、line-through删除线



(2)vertical-align垂直对齐：只能作用于<table>表格单元格的对象:

top顶、middle居中、bottom底



(4)text-shadow文本阴影：

语法"text-shadow:阴影颜色 x轴位移(x-offset) y轴位移(y-offset) 模糊半径(blur-radius);"

如text-shadow: blue 10px 10ox 2px;

(5)查询浏览器是否支持HTML5及CSS3属性的网址www.caniuse.com

33.CSS3设置超链接样式：

伪类:根据标签处于某种行为或状态来修饰超链接样式。其他标签如p可以使用hover

和active。

语法"标签名:伪类名{声明;}"

(1)a:link未访问前的超链接

(2)a:visited 访问过后

(3)a:hover鼠标移到链接上

(4)a:link鼠标点击未释放

(5)设置伪类的顺序：a:link - a:visited - a:hover - a:active

(6)虽有四种样式,但实际开发中只设置<a>标签选择器样式、鼠标悬浮链接样式

34.CSS3设置列表样式

(1)list-style-type：列表项标记类型

none无符号、decimal数字、disc实心圆(默认)、circle空心圆、square实心正方形

(2)list-style-image：用图像做列表项标记

(3)list-style-position：设置列表项标记的位置

(4)list-style：一次设置列表的所有属性 (属性值为none时说明列表无样式)

顺序为 list-style-type + list-style-position + list-style-image

35.<div>标签：用于网页布局，把HTML文档分成独立不同的部分。

36.CSS3设置背景样式：

(1)background-color：背景色不能继承，其默认值是透明transparent

(2)background-image：url(图片路径)、none(不显示背景图像)

(3)background-repeat：背景图像重复平铺

repeat(沿水平和垂直方向)、no-repeat(不平铺,只显示一次)、

repeat-x(只沿水平方向)、repeat-y(只沿垂直方向)、



(4)background-position：背景图的位置(X水平Y垂直方向的偏移量，如果只有一个方向关键字，则默认另一个关键字为center)

1.Xpos Ypos:如 0px 0px：默认无偏移,从左上角出现

30px 40px：正向偏移,图像向右和向下出现

-50px -60px：反向偏移,图像向左和向上出现

2.X% Y%：如30% 50%(水平方向偏移30%，垂直方向居中)

3.X水平关键词(left,center,right)、Y垂直关键词(top,center,bottom)



(5)background：一次设置背景的所有属性

(6)background-size背景图片尺寸：

auto(保持图片原尺寸,不易失真)、cover(放大填满容器标签)、

百分比percentage、contain(按照图片本身的宽高比例适应定义背景的区域)

37.gradient线性渐变：颜色沿着一条直线方向过渡

(1)常规语法：" linear-gradient(position, color1, color2,...)"

(2)浏览器兼容语法：" -兼容前缀-linear-gradient(position,color1,color2,...)"

(3)渐变的直线方向：

to left 从右向左、to top left 向左上方、to bottom left 向左下方、

to right 从左向右、to top right向右上方、to bottomo right向右下方、

to bottom从上向下、to top 从下向上、

38.CSS3径向渐变radial-gradient：圆形渐变，颜色从一个起点朝所有方向混合，语法和线性渐变相似。

———6 盒子模型———

39.盒子模型的组成：

content网页内容、border边框、padding内边距、margin外边距

(1)边框border：

border-color 边框颜色：如border-color:#369 #000 #111 #F00;按“上右下左顺时针”设置

border-width 边框粗细：如细thin、中等medium、粗的thick

border-style 边框样式：常用none无边框、dotted点线边框、dashed虚线边框、solid实线边框

border 简写:如下边框border-bottom:9px red dashed;四条边框border:9px blue solid;

(2)margin外边距：盒子边框以外和其他盒子间的距离

margin-top:上外边距、margin-bottom：下外边距

margin-left：左外边距、margin-right：右外边距

margin：简写"上右下左"

auto：设置盒子在它的父容器里居中显示。如margin:0px auto;让整个盒子居中。

如果将元素的 margin设为负值，则元素会变大。

(块元素可以把左右页边距设置为"自动"中心对齐。margin:auto;但前提宽度不能是100%)

注意：很多标签都有自身默认的外边距，所以一般用并集选择器统一设置这些标签的外边距为0px,这样不会产生不必要的空隙。

如清除body和h2自带的外边距 body,h2{margin:0px;}

(3)padding内边距：

padding-left、padding-right、padding-top、padding-bottom、

padding"上右下左"

40.盒子模型的尺寸：

增加边框、内边距、外边距后不会影响内容区域的尺寸，但会增加盒子模型的总尺寸。

(1)内盒总尺寸 = border(上下/左右)+padding(上下/左右)+内容宽/高度

(2)整个盒子的宽度 = 内容宽度+左右padding+左右边框border+左右margin

41.box-sizing拯救布局

(语法)box-sizing:content-box、border-box、inherit

(1)content-box：盒子的宽度或高度=border+padding+(margin)+width/height

(2)border-box：盒子的宽或高度等于元素内容的宽或高度

(即 该内容宽/高度=盒子宽/高度-border-padding )

(3)inherit：使元素继承父元素的盒子模型模式。

42.border-radius圆角边框：语法和边框相似，只是四个边框带圆角

(语法)border-radius:length{1~4个数字};

(1)用border-radius制作特殊图形

圆形：元素的宽度和高度必须相同。圆角半径为元素宽度的一半，或直接设圆角半径为50%

半圆形：元素的高度是宽度的2倍，且圆角半径为元素的宽度值。

扇形：即制作四分之一圆形。"三同"元素宽度、高度、圆角半径 "一不同"

43.盒子阴影：和文本阴影相似

(语法)box-shadow:inset x-offset y-offset blur-radius color;

inset：内部阴影，可选。

x-offset：X轴水平位移，正值在右，负值在左。

y-offset：Y轴垂直位移，正值在下，负值在上。

blur-radius：模糊半径可选，只能>=0 值越大阴影向外面积越大,边缘越模糊。

——7 浮动——

44.标准文档流：元素根据块元素或行内元素的特性从上到下，从左到右的方式自然排列。

45.display属性：用于指定标签的显示方式

block：块元素的默认值，该元素前后自带换行符

inline：行内元素的默认值，元素会显示为行内元素

inline-block：行内块元素，兼具行内元素和块元素的特性

none：元素不会显示

46.Float：指定网页元素向哪个方向浮动

left左、right右、none默认无(元素不浮动 显示在其文本出现的位置)

元素的水平方向浮动，意味着元素只能左右移动而不能上下移动。

一个浮动元素会尽量向左或向右移动，直到它的外边缘碰到包含框或另一个浮动框的边框为止。

浮动元素之后的元素将围绕它。

浮动元素之前的元素将不会受到影响。

如果是右浮动，后面的文本流将环绕在它左边：

47.clear清除浮动：当子元素全部浮动了，父级将包不住子元素会造成边框塌陷，所以要清除浮动元素对其他元素的影响。

48.clear属性：规定元素的哪一侧不允许其他浮动元素。

left(左侧不允许浮动元素)、right(右侧不允许)、

both(左右都不允许，常用于文本在图片下方显示)、

none(允许浮动元素出现在两侧)

49.解决父级边框塌陷

(1)浮动元素后加空的div，该div样式要设置clear:both;margin:0px;padding:0px;

(2)设置父元素固定高度把边框撑开。

(3)父级添加overflow属性:设置外层盒子的overflow:hidden。但此方法不能用于有下拉列表框的场景。

(4)父级添加伪类after，推荐。

50.Overflow属性：溢出处理，也可用于扩展盒子高度。

1) visible 默认溢出内容可见，显示在盒子外面

2) hidden 多出来的内容被隐藏且没有滚动条

3) scroll 有垂直水平2条滚动条，可查看多余内容

4) auto 如果内容溢出，自动显示滚动条(只有垂直条)查看

5) inherit 继承父特性

————8 定位网页元素————

51.Position属性：指定盒子的位置，相对它父级的位置或它自身应该在的位置。

(1)static 默认无定位，元素按照标准文档布局。

(2)relative相对定位

a.特性:

1.以标准文档流排版为基础，相当于它在原来位置偏移指定的距离。

2.元素位置偏移后，仍会保留原位置。

3.层级提高，可以遮盖标准文档流元素和浮动元素。

b.使用场景：

相对定位可以不设偏移量，让后面的绝对定位以它为祖先元素作基准定位。

c.语法 position:relative,指定偏移量时：水平left(正值向右移)、right(正值向左),垂直top(正值向下)、bottom(正值向上)。如

div{

position: relative;

top:-20px;

left:20px;

}

(3)absolute绝对定位

a.特性：

1.以已定位的祖先元素作基准定位，如果没有定位的祖先元素，则以浏览器窗口为基准定位。

2.元素位置偏移后，不保留原位置(其他元素可以用它原来的空位)

3.层级提高，可以遮盖标准文档流元素和浮动元素。

4.设置绝对定位的元素脱离文档流，对其他盒子的定位无影响

b.使用场景：下拉菜单、焦点图轮播、弹出数字气泡、特别花边等。

(4)fixed固定定位

a.特性：直接以浏览器窗口为基准定位，偏移位置不受窗口滚动条滚动的影响。

b.使用场景：窗口边缘的固定广告、返回顶部图标、边缘固定导航栏等。

52.z-index属性：设置定位元素的堆叠顺序。默认值0，值大的层位于值小层的上方。

(1)网页中的元素都含有两个堆叠层级，一个是未设置绝对定位时所处的环境，此时z-index是0；另一个是设置绝对定位时所处的堆叠环境，此时层的位置由z-index的值确定。

53.设置元素透明度的方法(通常两种方法搜设置以适应所有浏览器兼容)

(1)opacity:x x值为0~1，值越小越透明

(2)filter:alpha(opacity=x) x值为0~100，值越小越透明

——9 CSS3做网页动画——

54.transform变形：

指效果的集合，如平移、旋转、缩放、倾斜效果。

语法 transform:[transform-function]*;

其中transform-function是变形函数，如要设置多个，则中间以空格分开。在用2D变形时要加浏览器兼容性前缀。



常用2D变形函数如下：

(1)translate(tx,ty)：

平移函数，将元素从原位置(基于X,Y坐标)移动到指定位置上。

tx表示X轴(横坐标)上移动的向量长度，正值向右，负值向左。

ty表示Y轴(纵坐标)上移动的向量长度，正值向下，负值向上。

(2)scale(sx,sy)：

缩放函数，定义宽高度(元素尺寸)的缩放比例，默认值1。0~0.99缩小，大于1放大。

sx表示宽度即横坐标方向的缩放量。

sy表示高度即纵坐标方向的缩放量。

(3)rotate(a)；

旋转函数，只取一个值为度数值,单位deg表示角度°

正值顺时针转，负值逆时针转。

rotate函数只旋转，不改变元素形状。

(4)skew(ax,ay)：

倾斜函数，取值为度数值，单位deg

ax表示水平方向即X轴的倾斜角度。

ay表示垂直方向即Y轴的倾斜角度。

55.3D变形函数：translate3d()平移函数、scale3d()缩放函数、rotate3d()旋转函数

56.transition过渡：

指动画转换的过程，如渐现、渐弱、动画快慢等。

通过指定属性的初始状态、结束状态，在两个状态间通过平滑过渡的方式实现动画。

语法:[transition-property transition-duration

transition-timing-function transition-delay]*

(速记法)transition: 过渡属性 过渡用时 过渡的动画函数 过渡的延迟时间

主要包括四个属性值：

(1)transition-property：

过渡属性，设置过渡或动态模拟的CSS属性

(2)transition-duration：

过渡用时，从旧属性到新属性的用时，单位为s

(3)transition-timing-function：

指定过渡函数、过渡速度,有以下方式：

ease速度由快到慢，逐渐变慢(默认)

liner匀速

ease-in 越来越快(渐显)

ease-out 越来越慢(渐隐)

ease-in-out 先加速再减速(渐显渐隐)

(4)transition-delay：设置过渡是否延迟时间执行。

注意：transition-duration指完成过渡需要的时间；transition-delay指过渡在什么时间之后触发。

57.总结如何用transition实现过渡动画：

(1)在默认样式中声明元素的初始状态。

(2)声明过渡元素之中状态样式，如悬浮状态

(3)在默认样式中通过添加过渡函数，添加不同的样式。

58.过渡的触发机制：

(1)伪类触发: :hover、 :active、 :focus、 :checked等

(2)媒体查询：通过@media属性判断设备的尺寸、方向等。

(3)JavaScript触发：用JavaScript脚本触发。

59.animation动画

animation制作动画的步骤:

(1)通过类似Flash动画的关键帧(@keyframes)声明一个动画；

其中@keyframes称为关键帧，可以设置多段属性。语法

@keyframes 动画名称{

from{ //css样式代码 }

百分比1{ //css样式 }

百分比2{ //css样式 }

100%{ //css样式 }

}

(2)找到要设置动画的元素，调用关键帧已声明的动画。

如 animation: spread(动画名) 2s linear(匀速);

60.animation动画的语法和属性：

" animation: 动画名称 播放时间 播放方式 开始播放的时间 播放次数 播放方向 播放状态 动画时间之外的状态 "

其中属性分别为：

animation-name 动画名称、

animation-duration 播放时间、

animation-timing-function 播放方式、

animation-delay 开始播放的时间、

animation-iteration-count 播放次数(无限次用infinite)、

animation-diriection 播放方向、

animation-play-state 播放状态、

animation-fill-mode 动画时间之外的状态、

——HTML部分——

utf-8 和 utf8的使用

只有MySQL可以用"utf8"，但其他地方一律使用大写"UTF-8"。

网页推荐使用长后缀名.html

有的浏览器中直接输出中文会出现中文乱码，可加声明<meta charset="UTF-8">

<!--HTML注释-->

//空格

\> //大于号

< //小于号

" //双引号

© //版本符号

<em></em>斜体

<img src="地址" alt="图片代替文字" title="鼠标悬停提示" width="" height=""/>

<a href="链接网址" target="目标">页面间链接</a>

<!--1.页面间链接：A页到B页 主要运用于网页导航-->

<a name=wo></a>

<a href=#wo>锚链接</a>

<!--2.锚链接：A页甲位置到A页的乙位置或A页甲位置到B页的乙位置 # 跳其他页面要为href="页面名.html#锚链接"-->

<a href="mailto:bdqnWebmaster@bdqn.cn" target="_blank">功能性链接</a>

<!--3.功能性链接：在页面中调用其他软件功能，电子邮件"mailto: @bdqn.cn" qq msn-->

————

创建表格：

1、<table>：整个表格以<table>标记开始、</table>标记结束，table在没有添加css样式之前，在浏览器中显示是没有表格线的。

2、<tbody>：如果不加<thead><tbody><tfooter> , table表格加载完后才显示。加上这些表格结构， tbody包含行的内容下载完优先显示，不必等待表格结束后在显示，同时如果表格很长，用tbody分段，可以一部分一部分地显示。（通俗理解table 可以按结构一块块的显示，不用等整个表格加载完后显示。）

3、<tr>：表格的一行，所以有几对tr 表格就有几行。

4、<th>：表格的头部的一个单元格，表格表头，文本默认粗体且居中显示。

5、<td>：表格的一个单元格，一行中包含几对<td>这行中就有几个单元格。

6、表格中列的个数，取决于一行中数据单元格的个数。

7.设置样式border-collapse:collapse;可以把双线边框线合并为一条线边框。

<table border="边距宽度">

<tr>

<td rowspan="跨行数量" colspan="跨列数量" align="文本状态"></td>

</tr>

</table>

————

表格可以添加标题和摘要标签进行优化。

(1)摘要：

摘要的内容不会在浏览器中显示。作用是增加表格的可读性(语义化)，使搜索引擎更好的读懂表格内容，还可以使屏幕阅读器更好的帮助特殊用户读取表格内容。

语法：<table summary="表格简介文本">

(2)标题：

描述表格内容，标题的显示位置：表格上方。

语法：

<table summary="表格简介">

<caption>标题文本</caption>

<tr>

<td>…</td>

</tr>

</table>

————

内联框架iframe

<iframe></iframe>
相关属性 src="引用页面地址" name="框架标识名" frameborder="边框"

scrolling="是否出现滚动条" noresize="noresize"更改页面大小

<a targer="">配合<iframe name="">可实现窗口间的关联

表单

<form method="提交方式" action="提交地址">

如果是文件域要在表单中加 enctype="multipart/form=data" 属性

隐藏域：type="hidden"

只读：readonly="readonly"

禁用：disabled="disabled"

普通输入框

<input type="text" name="名称" size="长度" maxlength="最大长度"/>

radio单选按钮

<input type="radio" name="sex" value="男" id="nan"/>

<label for="nan">男</label>

<input type="radio" name="sex" value="女" id="nv"/>

<label for="nv">女</label>

checkbox多选按钮

<input type="checkbox" name="" value="1"/>

下拉列表

<select name="名称">

<optoin value="值">1</option>

</select>

文本域textarea

<textarea rows="行数" cols="列数">文本</textarea>
导入外部cs样式

1.<link type="text/css" rel="stylesheet" href="css/style.css"/>链接式

2.<style type="text/css">

@import url("css/stype.css");导入式

</typle>

————

超链接

<a href="目标网址" title="鼠标滑过显示的文本">链接显示的文本</a>

title属性：鼠标滑过链接时显示该属性的内容。方便搜索引擎了解链接地址的内容(语义化)

<a>标签链接Email地址，使用mailto能发送电子邮件。

如果mailto后面同时有多个参数，第一个参数必须以“?”开头，后面的参数每一个都以“&”分隔。

<a href="mailto:xxx@qq.com?subject=主题名称 &body=邮件内容">

超链接伪类：

a:link 访问前

a:visited 访问后

a:hover 鼠标悬停

a:active 鼠标选中未释放

————

透明度

opacity:(范围0~1)

filter:aplha(opcitive=透明度<(100)>);

————

Location 对象的方法：

.assign() //加载新文档

.reload() //刷新当前文档

.replace() //用新文档替换当前文档

————

在网页中显示代码，当代码为一行时可用<code>包裹，多行代码用<pre>。

<pre>：预格式化，它包围的文本会保留空格和换行符

下拉列表进行多选操作：在<select>标签中设置multiple="multiple"属性，就可以实现多选功能，在windows 操作系统下，进行多选时按下Ctrl键同时进行单击（在 Mac下使用 Command +单击），可以选择多个选项。

通用选择器匹配所有标签 *{ }

浏览器根据选择器权值来使用权值最高的css样式

规则：

标签的权值为1，类选择器的权值为10，ID选择器的权值为100。

!important有最高权值

!important要写在分号的前面，但注意当网页制作者不设置css样式时，浏览器会按照自己的样式来显示网页。并且用户也可以在浏览器中设置自己习惯的样式，比如有的用户习惯把字号设置为大一些，使其查看网页的文本更加清楚。这时注意样式优先级为：浏览器默认的样式 < 网页制作者样式 < 用户自己设置的样式，但 !important优先级例外，权值高于用户自设置的样式。

什么是“置换元素”？

置换元素会根据标签属性来显示的元素。反之就是非置换元素了。

如img根据src属性来显示，input根据value属性显示，因此可知img和input是置换元素，同理textarea、 select也是置换元素。

————

段落排版：

(1)letter-spacing：单个汉字间隔或单个字母间隔。

(2)word-spacing：按单词来设置间隔。

——

1、border-style 边框样式：

dashed（虚线）| dotted（点线）| solid（实线）

2、border-color 边框颜色

3、border-width 边框宽度：

thin | medium | thick。常用像素(px)。

4、当margin(或padding或border)的left和right的值相同，如：

margin:10px 20px 30px 20px;

缩写：

margin:10px 20px 30px;

——

布局模型与盒模型都是 CSS概念。布局模型建立在盒模型基础上。

在网页中，元素有三种布局模型：

1、流动模型（Flow）

流动（Flow）是默认的网页布局模式。特征：块状元素都会在所处的包含元素内自上而下按顺序垂直延伸分布，因为在默认状态下，块状元素的宽度都为100%。实际上，块状元素都会以行的形式占据位置。

流动模型下，内联元素会在所处的包含元素内从左到右水平分布显示。

2、浮动模型 (Float)

任何元素默认是不能浮动的，可用CSS定义为浮动。

3、层模型（Layer）

让html元素在网页中精确定位，就像PhotoShop中的图层一样可以对每个图层能够精确定位操作。CSS定义了一组定位（positioning）属性来支持层布局模型。

层模型有三种形式：

(1)绝对定位(position: absolute)

将元素从文档流中拖出来，然后用left、right、top、bottom属性相对最靠近它的一个带有定位属性的父包含块进行绝对定位。如果不存在这样的父包含块，则相对于body元素即相对于浏览器窗口。

(2)相对定位(position: relative)

元素在正常文档流中的偏移位置。首先按static(float)方式生成一个元素(元素像层一样浮动了起来)，然后相对于以前的位置移动，移动的方向和幅度由left、right、top、bottom属性确定，偏移前的位置保留。

(3)固定定位(position: fixed)

始终位于浏览器窗口内视图的设置位置，不受文档流动影响，

另外属性background-attachment:fixed;的作用也是设置背景图片固定。

relative与absolute组合：

1、参照定位的元素必须是相对定位元素的前辈元素。

2、参照定位的元素必须加入position:relative。

3、定位元素加入position:absolute，便可以使用top、bottom、left、right来进行偏移定位了。

设置颜色的方法：

1、单词：p{color:red}

2、RGB

由 R(red)、G(green)、B(blue)三种颜色比例来配色。

p{color:rgb(133,45,200)}

每一项的值可以是 0~255 的整数，也可以是0%~100% 的百分数。如：

p{color:rgb(20%,33%,25%)}

3、十六进制颜色

其原理也是 RGB 设置，每一项的值由 0-255 变成了十六进制 00-ff。p{color:#00ffff;}

——

相对单位长度值：

1、px像素

像素指的是显示器上的小点(CSS规范中假设“90像素=1英寸”)。实际情况是浏览器和使用显示器的实际像素值有关。

2、em

(1)元素给定字体的 font-size 值，如果元素的 font-size 为 14px，那么 1em = 14px；如果font-size 为18px，那么 1em = 18px。

如 p{font-size:12px; text-indent:2em;}意思首行缩进 24px(即两个字体大小的距离)

(2)当 font-size 设置为 em时，计算标准以它父元素的 font-size 为基础。

如：<p>以这个<span>例子</span>为例</p>

p{font-size:14px} span{font-size:0.8em;}

这里 span 字体大小就为11.2px(14 * 0.8 = 11.2px)

3、%百分比

p{font-size:12px; line-height:130%}

设置行高(行间距)为字体的130%（12 * 1.3 = 15.6px）

块状元素没有设置宽度时怎么居中?

1.加入 table 标签

2.设置 display: inline方法：显示类型设为行内元素，进行不定宽元素的属性设置

3.设置 position: relative 和 left:50%。利用相对定位，将元素从左偏移50%实现居中。

——

隐性改变display类型：

\1. position : absolute;

\2. float:left 或 float:right;

不论什么元素(display:none除外)，设置以上属性之一，该元素的display显示类型就会自动变为 以display:inline-block(行内块状元素)方式显示，此时可设置元素的 width 和 height，且默认宽度不占满父元素。

(如 a是行内元素，直接设置它的 width 无效，但设置 position:absolute 绝对定位后就可以设置宽度)

文本格式化标签：

<b> 文本加粗

<strong>文本加粗(加重语气)

<i> 斜体字

<em> 斜体(强调文字)

<big> 字体放大

<small> 字体缩小

<sub> 定义下标字

<sup> 定义上标字

<ins> 插入字(字体下划线)

<del> 字体删除线

"计算机输出" 标签：

<code> 定义计算机代码

<kbd> 键盘输入

<samp> 定义计算机代码样本

<var> 定义变量

<pre> 预格式化文本(会保留文本的多个空格)

引文、引用、及标签定义：

<abbr>缩写

<address>地址联系信息

<bdo>文字方向(设置dir="rtl"为从右到左显示)

<blockquote>长文本引用(不会自带双引号，但会两边自动缩进)

<q>短句引用语(自带双引号)

<cite>定义引用、引证

<dfn>定义一个定义项目。

title=""属性规定关于元素的额外信息。标签中加上title属性可实现鼠标移过时出现提示文字，如<p title="提示">

——

<base>元素:

描述了基本的链接地址/链接目标，该标签作为HTML文档中所有的链接标签的默认链接:

<head>

<base href="//www.baidu.cn" target="_blank"/>
</head>

提示：在HTML中，<base>标签没有结束标签。

——

HTML 颜色值RGB

由红(R)、绿(G)、蓝(B)组成。

每个颜色的最低值为0(十六进制为00)，最高值为255(十六进制为FF)。

十六进制值写法：#号后加3个或6个十六进制字符。

三位数表示法为：#RGB，转换为六位数表示为：#RRGGBB

——

常见的 URL Schemes

http超文本传输协议以http:// 开头的普通网页，不加密。

https安全超文本传输协议安全网页，加密所有信息交换。

ftp文件传输协议用于将文件下载或上传至网站。

file 您计算机上的文件。

——

HTML5 多媒体标签

<embed>定义内嵌对象。HTML4不赞成，HTML5允许。

<object>定义内嵌对象。

<param>定义对象的参数。

<audio>定义声音内容

<video>定义视频或者影片

<source>定义media元素的多媒体资源(<video>、<audio>)

<track>规定media元素的字幕文件或其他包含文本的文件 (<video>、<audio>)

——

audio音频设置

1.最好的解决方法：

下例使用两个不同的音频格式。HTML5 <audio> 元素会尝试以 mp3 或 ogg来播放音频。如果失败，代码将回退尝试 <embed> 元素。

<audio controls height="100" width="100">

<source src="horse.mp3" type="audio/mpeg">

<source src="horse.ogg" type="audio/ogg">

<embed height="50" width="100" src="horse.mp3">

</audio>

2.雅虎播放器使用免费，提供一个小型的播放按钮。

（1）如需使用它，需要把这段 JavaScript 插入网页底部：

<script src="http://mediaplayer.yahoo.com/latest"></script>
（2）然后把MP3文件链接到页面中，JS会自动为每首歌创建播放按钮如：

<a href="音频路径" >音乐1</a>

<a href="horse.mp3">音乐2</a>

<script src="http://mediaplayer.yahoo.com/latest"></script>
3.用超链接

以下代码指向 mp3 文件链接。如果用户点击该链接，浏览器会启动"辅助应用程序"来播放该文件：

<a href="horse.mp3">音乐1</a>

——

video视频播放设置

1.最好的解决方法

下例中使用了4种不同的视频格式。HTML 5 <video> 元素会尝试以 mp4、ogg、webm其中一种格式来播放视频。如果都失败，则回退到 <embed> 元素。

HTML5的source + object + embed。

<video width="320" height="240" controls>

<source src="movie.mp4" type="video/mp4">

<source src="movie.ogg" type="video/ogg">

<source src="movie.webm" type="video/webm">

<object data="movie.mp4" width="320" height="240">

<embed src="movie.swf" width="320" height="240">

</object>

</video>

2.优酷解决方案

如果要在网页中播放视频，可把视频上传到优酷等视频网站，然后在你的网页中插入 HTML代码即可播放视频：

<embed src="http://player.youku.com/player.php/sid/XMzI2NTc4NTMy/v.swf"

width="480" height="400"

type="application/x-shockwave-flash">

</embed>

3.使用超链接

如果网页包含指向媒体文件的超链接，大多数浏览器会使用"辅助应用程序"来播放文件。

以下代码指向 AVI文件的链接。如果用户点击该链接，浏览器会启动"辅助应用程序"如 Windows Media Player 来播放该 AVI 文件：

<a href="https://www.w3cschool.cn/html/intro.swf">播放该视频</a>

——

HTML中如何键入空格？

1.用空格占位符

但有不间断的特性。即连续的会在同一行内显示。即使有100个连续的，浏览器也不会把它们回车拆行。

2.段落间距<p>、换行<br/>

3.用JS动态给HTML添加空格：

例为照顾CSS样式或照顾特殊效果的实现。如果你不单单想让div之间是null，而是想动态添加空格的话，这样(jquery):

$("#id").innerHTML += " ";

——

display: none; 元素不显示也不会占位

visibility: hidden; 元素只是隐藏但仍然占位置

visibility: collapse; 隐藏但不占空间，类似display:none;

——

clip 剪辑一个绝对定位的元素。

clip : rect(top, right, bottom, left);

CSS 伪类：

1.Anchor伪类 (伪类如:link冒号和伪类名之间不能有空格)

在支持 CSS 的浏览器中，链接的不同状态可用不同的方式显示：

a:link {color:#FF0000;} /* 未访问的链接 */

a:visited {color:#00FF00;} /* 已访问的链接 */

a:hover {color:#FF00FF;} /* 鼠标划过链接 */

a:active {color:#0000FF;} /* 已选中的链接 */

2.CSS类和伪类配合使用：

p : first-child{ } 匹配父级中第一个<p>子元素

p > i:first-child{ } 匹配所有<p>元素的第一个 <i> 子元素

p:first-child i{ } 匹配第一个<p>元素中的所有 <i> 元素

——

White-space属性：设置如何处理元素内的空白。

normal 默认。空白会被浏览器忽略。

pre 空白会被浏览器保留。其行为方式类似 <pre>标签。

nowrap 文本不会换行，文本在同一行上继续，直到遇到<br/>为止。

pre-wrap 保留空白符序列，但是正常地进行换行。

pre-line 合并空白符序列，但是保留换行符。

inherit 从父元素继承 white-space 属性的值。

——

浏览器兼容前缀：

-moz- 火狐等使用 Mozilla内核的浏览器

-webkit- 谷歌、Safari等使用 Webkit内核的浏览器

-o- Opera浏览器，使用Blink内核

-ms- IE，使用 Trident内核

——

viewport 是用户网页的可视区域。