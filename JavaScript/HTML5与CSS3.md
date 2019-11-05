# HTML5与CSS3

## 网络的构造块

### 文件名规则:

Remember to use all lowercase letters for your file names, separate words with a dash, and add the `.html`
extension.

Use all lowercase letters and dashes for your directories and folders as well. The key is consistency.

### 相对URL

A relative URL describes the location of the desired file with reference to the location of the file that contains the URL reference itself.

The relative URL for a file that is in the same directory as the current page (that is, the one containing the URL in question) is simply the file name and extension.

You create the URL for a file in a subdirectory of the current directory by typing the name of the subdirectory followed by a forward slash and then the name and extension of the desired file.

To reference a file in a directory at a higher level of the file hierarchy, use two periods and a forward slash. You can combine and repeat the two periods and forward slash to reference any file on the same server or drive as the current file.

by first jumping straight to your site's root and then drilling down from there to the targeted file. A single forward slash at the beginning achieves this.

网页保存格式:UTF-8, no BOM,UTF-8无签名

## 基本HTML结构

### Starting Your Web Page

### Creating a Title

```html
<head><title></title></head>
```

### Creating Headings

```html
<!DOCTYPE html>

<html lang="en" xmlns="http://www.w3.org/1999/xhtml">
<head>
    <meta charset="utf-8" />
    <title>Antoni Gaudí - Introduction</title>
</head>
<body>
    <h1>Antoni Gaudí</h1>
    <h2 lang="es">La Casa Milà</h2>
    <h2 lang="es">La Sagrada Família</h2>
</body>
</html>

```

### Understanding HTML5’s Document Outline

[HTML elements reference](https://developer.mozilla.org/en-US/docs/Web/HTML/Element)

HTML5 includes four sectioning content elements—`article`, `aside`, `nav`, and `section`—that demarcate distinct sections within a document and define the scope of the `h1`-`h6` (as well as `header` and `footer`) elements within them.

Use headings that indicate the hierarchy explicitly with `h1`-`h6`, just as **sectioning elements** weren't present. 

```html
<article>
   <h1>Product User Guide</h1>
   <section>
      <h2>Setting it Up</h2>
   </section>
   <section>
      <h2>Basic Features</h2>
      <section>
         <h3>Video Playback</h3>
      </section>
   </section>
   <section>
      <h2>Advanced Features</h2>
   </section>
</article>
```

### Grouping Headings

```html
<hgroup>
   <h1>Giraffe Escapes from Zoo</h1>
   <h2>Animals Worldwide Rejoice</h2>
</hgroup>
```

### Common Page Constructs

However, the semantics that apply to these common page constructs are pretty similar no matter the layout. You'll explore them for most of the remaining pages of this chapter. Working from the top of the page down, you'll see how to use the `header`, `nav`, `article`, `section`, `aside`, and `footer` elements to define the structure of your pages, and then how to use `div` as a generic container for additional styling and other purposes.

### 创建页眉(Header)

```html
<header>
   <nav>
      <ul>
         <li>
            <a href="#gaudi">Barcelona's Architect</a>
         </li>
         <li lang="es">
            <a href="#sagrada- familia">La Sagrada Família</a>
         </li>
         <li>
            <a href="#park-guell">Park Guell</a>
         </li>
      </ul>
   </nav>
</header>
```

You may not nest a `footer` element or another `header` within a `header`, nor may you nest a `header` within a `footer` or `address` element.

### 标记导航(Navigation)

```html
<header role="banner">
   <nav role="navigation">
      <ul>
         <li>
            <a href="#gaudi">Barcelona's Architect</a>
         </li>
         <li lang="es">
            <a href="#sagrada- familia">La Sagrada Família</a>
         </li>
         <li>
            <a href="#park-guell">Park Guell</a>
         </li>
      </ul>
   </nav>
</header>
```

HTML5 doesn't allow nesting a `nav` within an `address` element.

### 创建文章(Article)

[&lt;article&gt;](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/article)
[&lt;address&gt;: The Contact Address element](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/address)
You can nest an `article` inside another one. You can't nest an `article` inside an `address` element, though.

### 定义节(Section)

[&lt;section&gt;](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/section)

### 指定侧边栏(Aside)

[&lt;aside&gt;](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/aside)

### 创建页脚(Footer)

[&lt;footer&gt;](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/footer)

The `footer` element represents a footer for the nearest `article`, `aside`, `blockquote`, `body`, `details`, `fieldset`, `figure`, `nav`, `section`, or `td` element in which it is nested. It's the footer for the whole page only when its nearest ancestor is the `body`. And if a `footer` wraps all the content in its section (an article, for example), it represents the likes of an appendix, index, long colophon, or long license agreement, depending on its content.

### 创建通用容器

That container is the `div` element (think of a **"division"**). With a `div` in place, you can apply the desired style or JavaScript effect to it.

[&lt;div&gt;](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/div)

### ARIA提升可访问性

WAI-ARIA

[Definition of Roles](https://www.w3.org/TR/wai-aria-1.1/#role_definitions)

* role="banner"
* role="navigation"
* role="main"
* role="complementary"
* role="contentinfo"

### 以类或ID命名元素
[id](https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes/id)
The `id` attribute automatically turns the element into a named anchor, to which you can direct a link.
[class](https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes/class)

### 添加标题属性(Title Attribute)到元素
[title](https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes/title)

### 添加注释
```html
<!-- 这是注释 -->
```

## 文本

This chapter explains which HTML semantics are appropriate for different types of text, especially (but not solely) for text within a sentence or phrase.

### 段落(Paragraph)

[&lt;p&gt;](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/p)

### 作者联系信息

[&lt;address&gt;](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/address)

### 图片(Figure)

[&lt;figure&gt;](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/figure)

这个标签经常是在主文中引用的图片，插图，表格，代码段等等

[&lt;figcaption&gt;](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/figcaption)

```html
<figure>
   <figcaption>
      Figure 3: 2011 Revenue by Industry
   </figcaption>
   <img src="chart-revenue.png" />
</figure>
```

The `figure` element is known as a **sectioning root** in HTML5, which means it can have `h1`-`h6` headings (and thus, its own outline), but they don't contribute to the document outline.

### Specifying Time

[&lt;time&gt;](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/time)

```html
<time datetime="2011-10-15" pubdate="pubdate">
   October 15, 2011
</time>
```

### Marking Important and Emphasized Text

The [`strong`]() element denotes important text, while [`em`]() conveys **emphasis**. You can use them individually or together as your content requires.

```html
<strong>
   Warning: Do not approach the zombies <em>under any circumstances</em>.
</strong> They may <em>look</em> friendly, but that's just because they want to eat your arm.
```

HTML5 emphasizes that you use [`b`](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/b) and [`i`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/i) only as a last resort when another element (such as `strong`, `em`, `cite`, and others) won't do.

### Indicating a Citation or Reference

Use the [`cite`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/cite) element for a citation or reference to a source. Examples include the title of a play, script, or book; the name of a song, movie, photo, or sculpture; a concert or musical tour; a specification; a newspaper or legal paper; and more.

```html
<p>
   He listened to
   <cite>Abbey Road</cite> while watching
   <cite>Hard Day's Night</cite> and reading
   <cite>The Beatles Anthology</cite>.
</p>

<p>
   When he went to The Louvre, he learned that
   <cite>Mona Lisa</cite> is also known as
   <cite lang="it">La Gioconda</cite>.
</p>
```

`cite`元素引用标题

For instances in which you are quoting from the cited source, use the `blockquote` or `q` elements, as appropriate, to mark up the quoted text. To be clear, `cite` is only for the source, not what you are quoting from it.

### Quoting Text

There are two special elements for marking text quoted from a source. The [`blockquote`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/blockquote) element represents a quote (generally a longer one, but not necessarily) that stands alone and renders on its own line by default. Meanwhile, the [`q`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/q) element is for short quotes, like those within a sentence.

`blockquote`元素是块级的, 是区块根(sectioning root).

`q`元素是短语内容, 不能跨越段落.

### Highlighting Text

[&lt;mark&gt;](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/mark)

### Explaining Abbreviations

[&lt;abbr&gt;](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/abbr)

~~&lt;acronym&gt;: 首字母缩写词,已被删除.~~

### Defining a Term

[&lt;dfn&gt;: The Definition element](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/dfn)

### Creating Superscripts and Subscripts

[&lt;sup&gt;](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/sup)

[&lt;sub&gt;](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/sub)

### Noting Edits and Inaccurate Text

[&lt;del&gt;: The Deleted Text element](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/del)

[&lt;ins&gt;](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/ins)

[&lt;s&gt;](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/s)

Both `del` and `ins` support two attributes: `cite` and `datetime`. The `cite` attribute is for providing a URL to a source that explains why an edit was made. Use the `datetime` attribute to indicate the time of the edit. 

### Marking Up Code

[<code>: The Inline Code element](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/code)

The `kbd` Element(**keyboard**)
Use kbd to mark up user input instructions.

The `samp` Element
The samp element indicates **sample** output from a program or system.

The `var` Element
The var element represents a **variable** or placeholder value.

### Using Preformatted Text

[&lt;pre&gt;](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/pre)

### Specifying Fine Print

[&lt;small&gt;](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/small)

### Creating a Line Break

[&lt;br&gt;](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/br)

[&lt;hr&gt;](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/hr)

### Creating Spans

[&lt;span&gt;](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/span)

### Other Elements

* `u`元素:无语义的下划线.

* `wbr`元素: 可换行处(a word break opportunity)

* `ruby`, `rp`, `rt`, `rtc`, `rb`:注音

  ```html
  <ruby>
     旧<rt>jiù</rt>
     金<rt>jīn</rt>
     山<rt>shān</rt>
     <rtc>San Francisco</rtc>
  </ruby>
  ```
  HTML Ruby Fallback Parenthesis (&lt;rp&gt;) element

  HTML Ruby Text (&lt;rt&gt;) element

  HTML Ruby Text Container (&lt;rtc&gt;) element

  ​

* `bdi`, `bdo`:控制书写顺序, 是从左至右,或是相反

* `meter`:仪表

  ```html
  <meter min="200" max="500" value="350">350 degrees</meter>
  ```

* `progress`:进度条

  ```html
  <progress value="70" max="100">70 %</progress>
  ```

## 图像

```html
<img src="stupa.jpg" alt="Two Stupas" width="220" height="170" />
```

[&lt;img&gt;](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/img)

You can also use a percentage value in step 2, with respect to the **browser window** (not to the original image size).

You can set just the `width` or just the `height` and have the browser adjust the other value proportionally.

## 链接

```html
<a href="https://developer.mozilla.org/" target="_blank">Mozilla Developer Network</a>
```

[&lt;a&gt;](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/a)

Some users typically tab to links (using the **Tab** key to move forward, and **Shift-Tab** to move backward) and then trigger them with the **Enter** or **Return** key.

样式设置:

```css
a {
   color: blue;
   font-weight: bold;
}

   a:visited {
      color: purple;
   }

   a:hover,
   a:active,
   a:focus {
      color: darkred;
      text-decoration: none;
   }
```

## 列表

All lists are formed by a *principal* element to specify what sort of list you want to create (`ul` for unordered list, `ol` for ordered list, and `dl` for description list, known as a definition list before HTML5) and *secondary* elements to specify what sort of list items you want to create (`li` for a list item in an `ol` or `ul`, and `dt` for the term with `dd` for the description in a `dl`).

### Creating Ordered and Unordered Lists

有序列表

```html
<ol>
  <li>Screw in the new bulb.</li>
  <li>
    Plug in the lamp and turn it on!
  </li>
</ol>
```

无序列表

```html
<ul>
  <li>Image retouching plug-in</li>
  <li>Special HTML filters</li>
  <li>Unlimited Undo's and Redo's</li>
  <li>Automatic book writing</li>
</ul>
```

### 选择标识

[CSS list-style-type 属性](http://www.w3school.com.cn/cssref/pr_list-style-type.asp)

```css
ol li {
  list-style-type: upper-roman;
}
```

显示无标识列表

```css
{
  list-style-type: none;
}
```

You can also specify an ordered list's marker type with the `type` attribute. The acceptable values for type are `A`, `a`, `I`, `i`, and `1`, which indicate the kind of numeration to be used (`1` is the default). For example, `<ol type="I">` specifies uppercase Roman numerals.

`type`属性的优先级低于css`list-style-type:`, 如果定义了`list-style-type`, 则`type`属性不起作用.

### 选择列表的起始编号

```html
<ol start="2">
  <li>Unscrew the old bulb.</li>
  <li value="5">
    Screw in the new bulb.
  </li>
  <li>
    Plug in the lamp and turn it on!
  </li>
</ol>
```

### Using Custom Markers

[list-style-image](https://developer.mozilla.org/en-US/docs/Web/CSS/list-style-image)
html

```html
<ul>
  <li>Item 1</li>
  <li>Item 2</li>
</ul>
```

css

```css
ul {
  list-style-image: url("https://mdn.mozillademos.org/files/11981/starsolid.gif")
}
```

### Controlling Where Markers Hang

[list-style-position](https://developer.mozilla.org/en-US/docs/Web/CSS/list-style-position)

Type `inside` to display the markers flush with the list item text, or `outside` to display the markers to the left of the list item text (the default).

### Setting All List-Style Properties at Once

[list-style](https://developer.mozilla.org/en-US/docs/Web/CSS/list-style)

The **list-style** CSS property is a [shorthand](https://developer.mozilla.org/en-US/docs/Web/CSS/Shorthand_properties) for setting the individual values that define how a list is displayed: [`list-style-type`](https://developer.mozilla.org/en-US/docs/Web/CSS/list-style-type), [`list-style-image`](https://developer.mozilla.org/en-US/docs/Web/CSS/list-style-image), and [`list-style-position`](https://developer.mozilla.org/en-US/docs/Web/CSS/list-style-position).

### Creating Description Lists

一个基本的描述列表

```html
<dl>
   <dt>Boris Karloff</dt>
   <dd>
      Best known for his role in <cite>Frankenstein</cite> and related horror films, this scaremaster's real name was William Henry Pratt.
   </dd>
   <dt>Christopher Lee</dt>
   <dd>
      Lee took a bite out of audiences as Dracula in multiple Hammer horror classics.
   </dd>
</dl>
```

`dl` description list

`dt` description term

`dd` description details

`dt`与`dd`可以是: 一对一, 一对多, 多对一, 多对对, 可以使用`dfn`元素包围`dt`中的名称

```html
<dl>
   <dt><dfn>bogeyman</dfn>, n.</dt>
   <dt><dfn>boogeyman</dfn>, n.</dt>
   <dd>
      A mythical creature that lurks under the beds of small children.
   </dd>
   <dt>
      <dfn lang="en-gb">aluminium</dfn>, n.
   </dt>
   <dt><dfn>aluminum</dfn>, n.</dt>
   <dd>...</dd>
</dl>
```

嵌套:

```html
<dl>
   <dt>Director</dt>
   <dd>Jean-Pierre Jeunet</dd>
   <dt>Writers</dt>
   <dd>
      Guillaume Laurant (story, screenplay)
   </dd>
   <dd>Jean-Pierre Jeunet (story)</dd>
   <dt>Cast</dt>
   <dd>
      <!-- Start nested list -->
      <dl>
         <dt>Audrey Tautou</dt>
         <!-- Actor/ Actress -->
         <dd>Am&eacute;lie Poulain</dd> <!-- Character -->
         <dt>Mathieu Kassovitz</dt>
         <dd>Nino Quincampoix</dd>
         ...
      </dl>
      <!-- end nested list -->
   </dd>
   ...
</dl>
```

使用的样式表

```css
dt {
   font-weight: bold;
   text-transform: uppercase;
}
/* style the dt of any dl within another dl */
dl dl dt {
   text-transform: none;
}

dd + dt {
   margin-top: 1em;
}
```

## 表单
[HTML 中的表单](https://developer.mozilla.org/zh-CN/docs/Web/Guide/HTML/Forms_in_HTML)

### 创建表单

[HTML &lt;form&gt; 标签](http://www.w3school.com.cn/tags/tag_form.asp)

### 对表单元素进行组织

[HTML &lt;fieldset&gt; 标签](http://www.w3school.com.cn/tags/tag_fieldset.asp)
[HTML &lt;legend&gt; 标签](http://www.w3school.com.cn/tags/tag_legend.asp)

### 创建文本框

```html
<label for="first_name">First Name:</label>
<input type="text" id="first_ name" name="first_name" class="large" required placeholder="Enter your first name" />

<label for="last_name">Last Name:</label>
<input type="text" id="last_ name" name="last_name" size="15" />
```

[HTML &lt;input&gt; 标签的 size 属性](http://www.w3school.com.cn/tags/att_input_size.asp)
对于 `<input type="text">` 和` <input type="password">`，`size` 属性定义的是可见的字符数。而对于其他类型，`size` 属性定义的是以像素为单位的输入字段宽度。
由于`size`属性是一个可视化的设计属性，我们推荐您使用CSS来代替它。
CSS 语法：`<input style="width:100px" />`

### 创建密码框

```html
<label for="password">Password:</label>
<input type="password" id="password" name="password" />

<label for="password">Re-enter Password:</label>
<input type="password" id="password2" name="password2" />
```

### 创建电子邮件, 电话, URL框

```html
<label for="email">Email:</label>
<input type="email" id="email" name="email" class="large" />
<label for="phone">Phone:</label>
<input type="tel" id="phone" name="phone" placeholder= "xxx-xxx-xxxx" class="large" pattern="\d{3}-\d{3}-\d{4}" />
<label for="web_site">Web:</label>
<input type="url" id="web_site" name="web_site" class="large" />
```

比文本框多了帮助验证和输入内容的功能.

`<input>`元素的类型:`button|checkbox|file|hidden|image|password|radio|reset|submit|text`

html5中的`input`的`type`属性总共是新增了13个，分别是`type`的这些属性：

* `color`定义拾色器
* `date`定义日期字段、`datetime`定义日期字段、`datetime-local`定义日期字段、`month`定义日期字段的月、`week`定义日期字段的周、`time`定义日期字段的时、分、秒
* `email`定义用于 e-mail 地址的文本字段
* `number`定义带有 spinner 控件的数字字段、`range`定义带有 slider 控件的数字字段
* `search`定义用于搜索的文本字段
* `tel`定义用于电话号码的文本字段
* `url`定义用于 URL 的文本字段

常用的正则表达式[见这里](http://html5pattern.com/)

### 标签

[HTML &lt;label&gt; 标签](http://www.w3school.com.cn/tags/tag_label.asp)

`<label>` 元素为 `<input>` 元素定义标注（标记）。

label 元素不会向用户呈现任何特殊效果。不过，它为鼠标用户改进了可用性。如果您在 label 元素内点击文本，就会触发此控件。就是说，当用户选择该标签时，浏览器就会自动将焦点转到和标签相关的表单控件上。

`<label>`  标签的 `for` 属性应当与相关元素的 `id` 属性相同。

> If you omit the `for` attribute, no `id` attribute is required in the element being labeled. The label and the element, in that case, are then associated by proximity or perhaps by being placed in a common li element.

为`<label>`添加css样式:

```css
label {
	display: inline-block;
	padding: 3px 6px;
	text-align: right;
	width: 150px;
	vertical-align: top;
}
```

### 创建单选按钮

```html
<input type="radio" id="gender_male" name="gender" value="male" />
<label for="gender_male">Male</label>
<input type="radio" id="gender_female" name="gender" value="female" />
<label for="gender_female">Female</label>
```

还可以这样`<lable>`嵌套包围`<input>`实现关联:

```html
<label><input type="radio" name="gender" value="male" checked>Male</label>
<label ><input type="radio" name="gender" value="female">Female</label>
```

`name`属性将相关单选按钮关联在一给定集合.

`value`属性指按钮被选中时提供的文本.

同名的`radio`只有一个被选中, 只有一个`radio`的值会被发送.

### 创建选择框

Select boxes are made up of two HTML elements: `select` and `option`. You set the common `name` attribute in the `select` element, and you set the `value` attribute in each of the `option` elements.

```html
<label for="state">State:</label>
<select id="state" name="state">
  <option value="AL">Alabama</option>
  <option value="AK">Alaska</option>
  <option value="CA">California</option>
</select>

```

`size`属性指定选择框的高度, 有几行.

`multiple`属性可以多选

`<option>`元素, [HTML &lt;option&gt; 标签](http://www.w3school.com.cn/tags/tag_option.asp)

* `value`属性该选项被选中时发送的数据.
* `selected`默认选中, 规定选项（在首次显示在列表中时）表现为选中状态。
* `disabled`

#### 对选择框选项进行分组

[HTML &lt;optgroup&gt; 标签](http://www.w3school.com.cn/tags/tag_optgroup.asp)

```html
<label>
  Where did you find out about us?
  <select name="referral">
    <optgroup label="On-line">
      <option value="social_network">Social Network</option>
      <option value="search_engine">Search Engine</option>
    </optgroup>
    <optgroup label="Off-line">
      <option value="postcard">Postcard</option>
      <option value="word_of_mouth">Word of Mouth</option>
    </optgroup>
  </select>
</label>
```

### 创建复选框

* `value`属性该选项被选中时发送的数据.
* `checked`默认选中。

```html
<label>
  <input type="checkbox" name="email_signup" value="user_emails">
  It is okay to email me with messages from other users.
</label>
<label>
  <input type="checkbox" name="email_signup" value="occasional_updates">
  It is okay to email me with occasional promotions about our other products.
</label>
```

### 创建文本区域

```html
<label for="bio">Bio:</label>
<textarea id="bio" name="bio" rows="8" cols="50" class="large"></textarea>
```

The `value` attribute is not used with the `textarea` element. Default values are set by adding text between the start and end tags.

### Allowing Visitors to Upload Files

```html
<form enctype="multipart/form-data">
  <label>
    Picture:
    <input type="file" name="picture">
  </label>
</form>
```

`enctype`属性规定在发送到服务器之前应该如何对表单数据进行编码，默认的编码是：”`application/x-www-form-urlencoded`“。对于文件只能使用`multipart/form-data`作为`enctype`属性值。

### Creating Hidden Fields

```html
<input type="hidden" name="id" value="78" >
```

To create an element that will be submitted with the rest of the data when the visitor clicks the submit button but that is also visible to the visitor, create a regular form element and use the `readonly` attribute.

### Creating a Submit Button

```html
<input type="submit" value="Create Account">
```

#### 创建提交按钮

```html
  <button type="submit">Create Account</button>
```

### Using an Image to Submit a Form

```html
<input type="image" src="/favicon.ico" alt="Create Account">
```

参考内容: [HTML&lt;input&gt; 标签的 type 属性](http://www.w3school.com.cn/tags/att_input_type.asp)

### Disabling Form Elements

```html
<input type="text" disabled>
```

## 表格

### 结构化表格

`<table>` 标签定义 HTML 表格。

简单的 HTML 表格由 `table` 元素以及一个或多个 `tr`、`th` 或 `td` 元素组成。

`tr` 元素定义表格行，`th` 元素定义表头，`td` 元素定义表格单元。

更复杂的 HTML 表格也可能包括 `caption`、`col`、`colgroup`、`thead`、`tfoot` 以及 `tbody` 元素。

最基本表格:

```html
<table>
  <tr>
    <th>Month</th>
    <th>Savings</th>
  </tr>
  <tr>
    <td>January</td>
    <td>$100</td>
  </tr>
</table>
```

更全表格:

```html
<table>
  <caption>
    Quarterly Financials for 1962-1964 (in Thousands)
  </caption>
  <thead>
    <!-- table head -->
    <tr>
      <th scope="col">Quarter</th>
      <th scope="col">1962</th>
      <th scope="col">1963</th>
      <th scope="col">1964</th>
    </tr>
  </thead>
  <tbody>
    <!-- table body -->
    <tr>
      <th scope="row">Q1</th>
      <td>$145</td>
      <td>$167</td>
      <td>$161</td>
    </tr>
    <tr>
      <th scope="row">Q2</th>
      <td>$140</td>
      <td>$159</td>
      <td>$164</td>
    </tr>

  </tbody>
  <tfoot>
    <!-- table foot -->
    <tr>
      <th scope="row">TOTAL</th>
      <td>$595</td>
      <td>$648</td>
      <td>$664</td>
    </tr>
  </tfoot>
</table>
```

Type `<th scope="scopetype">` to begin a header cell (where _scopetype_ is `col`, `row`, `colgroup`, or `rowgroup`).

The `thead`, `tfoot`, and `tbody` elements don't affect the layout and are not required (though I recommend using them), except that `tbody` is required whenever you include a `thead` or `tfoot`. Note that a table may have only one `thead` and `tfoot` but may have multiple `tbody` elements. 

表格样式表:

```css
table {
    border-collapse: collapse;
}

caption {
    font-size: .8125em;
    font-weight: bold;
    margin-bottom: .5em;
}

th,
td {
    font-size: .875em;
    padding: .5em .75em;
}

td {
    border: 1px solid #000;
}

tfoot {
    font-style: italic;
    font-weight: bold;
}
```

Without `border-collapse: collapse;` defined on the table, a space would appear between the border of each `td` and the border of its adjacent `td` (the default setting is `bordercollapse: separate;`). 

### 让单元格跨越多列或多行

```html
<table>
  <caption>TV Schedule</caption>
  <thead>
    <!-- table head -->
    <tr>
      <th scope="rowgroup">Time</th>
      <th scope="col">Mon</th>
      <th scope="col">Tue</th>
      <th scope="col">Wed</th>
    </tr>
  </thead>
  <tbody>
    <!-- table body -->
    <tr>
      <th scope="row">8 pm</th>
      <td>Staring Contest</td>
      <td colspan="2">
        Celebrity Hoedown
      </td>
    </tr>
    <tr>
      <th scope="row">9 pm</th>
      <td>Hardy, Har, Har</td>
      <td>What's for Lunch?</td>
      <td rowspan="2">
        Movie of the Week
      </td>
    </tr>
    <tr>
      <th scope="row">10 pm</th>
      <td>
        Healers, Wheelers &amp; Dealers
      </td>
      <td>It's a Crime</td>
    </tr>
  </tbody>
</table>
```

Note, too, that the *Time* th has `scope="rowgroup"`, because it is the header for every header in the group of row headers directly beneath it.

`<th scope=rowgroup`可以理解为`<th scope=row`的标题单元格.


