JavaScript 可以说是交互之王，它作为脚本语言加上许多 **Web Api** 进一步扩展了它的特性集，更加丰富界面交互的可操作性。这类 **API** 的例子包括**WebGL API**、**Canvas API**、**DOM API**，还有一组不太为人所知的 **CSS API**。由于JSX和无数JS框架的出现，使通过**JS API**与**DOM**交互的想法真正流行起来，但是在 CSS 中使用类似技术似乎并没有很多。当然，存在像**CSS-in-JS**这类解决方案，但是最流行的解决方案还是基于**转译**(transpilation)，无需额外运行即可生成 CSS。这肯定对性能有好处，因为**CSS API**的使用可能导致额外的重绘，这与DOM API的使用一样。

但这不是咱们想要的。如果哪天公司要求咱们，既要操纵 DOM 元素的样式和 CSS 类，还要像使用 HTML 一样使用 JS 创建完整的样式表，该怎么办？

**内联样式**

在咱们深入一些复杂的知识之前，先回来顾一下一些基础知识。例如，咱们可以通过修改元素的`.style`属性来编辑给定的`HTMLElement`的内联样式。

```js
const el = document.createElement('div')
el.style.backgroundColor = 'red'
// 或者 
el.style.cssText = 'background-color: red'
// 或者
el.setAttribute('style', 'background-color: red')
```

直接在`.style`对象上设置样式属性将需要使用**驼峰式**命名作为属性键，而不是使用短横线命名。如果咱们需要设置更多的内联样式属性，则可以通过设置`.style.cssText`属性，以更加高效的方式进行设置 。

请记住，给`cssText`设置后原先的css样式被清掉了，因此，要求咱们一次是一堆样式 。

如果这种设置内联样式过于繁琐，咱们还可以考虑将`.style`与`Object.assign()`一起使用，以一次设置多个样式属性。

```js
// ...
Object.assign(el.style, { backgroundColor: "red", margin: "25px"})
```

这些“基本知识”比咱们想象的要多得多。.style对象实现`CSSStyleDeclaration`接口。这说明它带还有一些有趣的属性和方法，这包括刚刚使用的`.cssText`，还包括`.length`（设置属性的数量），以及`.item()`、`.getPropertyValue()`和`.setPropertyValue()`之类的方法：

```js
// ...
const propertiesCount = el.style.length
for(let i = 0; i < propertiesCount; i++) { 
    const name = el.style.item(i) // 'background-color' 
    const value = el.style.getPropertyValue(name) // 're' 
    const priority = el.style.getPropertyPriority(name) // 'important' 
    if(priority === 'important') { 
        el.style.removeProperty() }
}
```

这里有个小窍门—在遍历过程中`.item()`方法具有按索引访问形式的备用语法。

```js
// ...
el.style.item(0) === el.style[0]; // true
```

**CSS 类**

接着，来看看更高级的结构——**CSS类**，它在检索和设置时具有字符串形式是.className。

```js
// ...
el.className = "class-one class-two";
el.setAttribute("class", "class-one class-two");
```

设置类字符串的另一种方法是设置class属性（与检索相同）。但是，就像使用.style.cssText属性一样，设置.className将要求咱们在字符串中包括给定元素的所有类，包括已更改和未更改的类。

当然，可以使用一些简单的字符串操作来完成这项工作，还有一种就是使用较新的.classList属性。

classList属性实现了`DOMTokenList`，有一大堆有用的方法。例如`.add()`、`.remove()`、`.toggle()`和`.replace()`允许咱们更改当前的 CSS 类集合，而其他的，例如`.item()`、`.entries()`或`.foreach()`则可以简化这个索引集合的遍历过程。

```js
// ...
const classNames = ["class-one", "class-two", "class-three"];
classNames.forEach(className => { 
    if(!el.classList.contains(className)) {
        el.classList.add(className); }});
```

**Stylesheets**

一直以来，Web Api 还有一个`StyleSheetList`接口，该接口由`document.styleSheets`属性实现。document.styleSheets 只读属性，返回一个由 StyleSheet 对象组成的 StyleSheetList，每个 StyleSheet 对象都是一个文档中链接或嵌入的样式表。

```js
for(styleSheet of document.styleSheets){ 
    console.log(styleSheet);}
```

通过打印结果咱们可以知道，每次循环打印的是 **CSSStyleSheet** 对象，每个 CSSStyleSheet 对象由下列属性组成：

属性描述

media获取当前样式作用的媒体。

disabled打开或禁用一张样式表。

href返回 CSSStyleSheet 对象连接的样式表地址。

title返回 CSSStyleSheet 对象的title值。

type返回 CSSStyleSheet 对象的type值，通常是text/css。

parentStyleSheet返回包含了当前样式表的那张样式表。

ownerNode返回CSSStyleSheet对象所在的DOM节点，通常是`<link>`或`<style>`。

cssRules返回样式表中所有的规则。

ownerRule如果是通过@import导入的，属性就是指向表示导入的规则的指针，否则值为null。

SSStyleSheet对象方法:

方法描述

insertRule()在当前样式表的 cssRules 对象插入CSS规则。

deleteRule()在当前样式表删除 cssRules 对象的CSS规则。

有了StyleSheetList的全部内容，咱们来CSSStyleSheet本身。在这里就有点意思了， CSSStyleSheet扩展了StyleSheet接口，并且只有这种只读属性，如.ownerNode，.href，.title或.type，它们大多直接从声明给定样式表的地方获取。回想一下加载外部CSS文件的标准HTML代码，咱们就会明白这句话是啥意思：

```html
<head>
    <link rel="stylesheet" type="text/css" href="style.css" title="Styles">
</head>
```

现在，咱们知道HTML文档可以包含多个样式表，所有这些样式表都可以包含不同的规则，甚至可以包含更多的样式表(当使用@import时)。CSSStyleSheet有两个方法:、.insertRule()和.deleteRule() 来增加和删除 Css 规则。

```js
// ...
const ruleIndex = styleSheet.insertRule("div {background-color: red}");
styleSheet.deleteRule(ruleIndex);
```

.insertRule(rule,index):此函数可以在cssRules规则集合中插入一个指定的规则，参数rule是标示规则的字符串，参数index是值规则字符串插入的位置。

deleteRule(index):此函数可以删除指定索引的规规则，参数index规定规则的索引。

CSSStyleSheet也有自己的两个属性：.ownerRule和.cssRule。虽然.ownerRule与@import相关，但比较有趣的是.cssRules。简单地说，它是CSSRuleList的CSSRule，可以使用前面提到的.insertrule()和.deleterule()方法修改它。请记住，有些浏览器可能会阻止咱们从不同的来源(域)访问外部CSSStyleSheet的.cssRules属性。

那么什么是 CSSRuleList？

CSSRuleList是一个数组对象包含着一个有序的CSSRule对象的集合。每一个CSSRule可以通过rules.item(index)的形式访问, 或者rules[index]。这里的rules是一个实现了CSSRuleList接口的对象， index是一个基于0开始的，顺序与CSS样式表中的顺序是一致的。样式对象的个数是通过rules.length表达。

对于**CSSStyleRule**对象:

每一个样式表CSSStyleSheet对象可以包含若干CSSStyleRule对象，也就是css样式规则，如下:

```html
<style type="text/css"> 
    h1{color:red} 
    div{color:green}
</style>
```

在上面的代码中style标签是一个CSSStyleSheet样式表对象，这个样式表对象包含两个CSSStyleRule对象，也就是两个css样式规则。

CSSStyleRule对象具有下列属性:

**1.type:返回0-6的数字，表示规则的类型，类型列表如下:**

0：CSSRule.UNKNOWN_RULE。

1：CSSRule.STYLE_RULE （定义一个CSSStyleRule对象）。

2：CSSRule.CHARSET_RULE （定义一个CSSCharsetRule对象，用于设定当前样式表的字符集，默认与当前网页相同）。

3：CSSRule.IMPORT_RULE （定义一个CSSImportRule对象，就是用@import引入其他的样式表）

4：CSSRule.MEDIA_RULE （定义一个CSSMediaRule对象，用于设定此样式是用于显示器，打印机还是投影机等等）。

5：CSSRule.FONT_FACE_RULE （定义一个CSSFontFaceRule对象，CSS3的@font-face）。

6：CSSRule.PAGE_RULE （定义一个CSSPageRule对象）。

**2.cssText:返回一个字符串，表示的是当前规则的内容，例如:**

```
div{color:green}
```

**3.parentStyleSheet:返回所在的CSSStyleRule对象。**

**4.parentRule:如果规则位于另一规则中，该属性引用另一个CSSRule对象。**

**5.selectorText:返回此规则的选择器，如上面的div就是选择器。**

**6.style:返回一个CSSStyleDeclaration对象。**

```js
// ...
const ruleIndex = styleSheet.insertRule("div {background-color: red}");
const rule = styleSheet.cssRules.item(ruleIndex);
rule.selectorText; // "div"
rule.style.backgroundColor; // "red"
```

**实现**

现在，咱们对 CSS 相关的 **JS Api**有了足够的了解，可以创建咱们自己的、小型的、基于运行时的CSS-in-JS实现。咱们的想法是创建一个函数，它传递一个简单的样式配置对象，生成一个新创建的CSS类的哈希名称供以后使用。

实现流程很简单，咱们需要一个能够访问某种样式表的函数，并且只需使用.insertRule()方法和样式配置就可以运行了。先从样式表部分开始：

```js
function createClassName(style) { 
    // ... 
    let styleSheet; 
    for (let i = 0; i < document.styleSheets.length; i++) { 
        if (document.styleSheets[i].CSSInJS) { 
            styleSheet = document.styleSheets[i];
            break; 
        } 
    } 
    if (!styleSheet) { 
        const style = document.createElement("style"); 
        document.head.appendChild(style); 
        styleSheet = style.sheet; 
        styleSheet.CSSInJS = true; 
    } 
    // ...
}
```

如果你使用的是**ESM**或任何其他类型的JS模块系统，则可以在函数外部安全地创建样式表实例，而不必担心其他人对其进行访问。但是，为了演示例，咱们将stylesheet上的.CSSInJS属性设置为标志的形式，通过标志来判断是否要使用它。

现在，如果如果还需要创建一个新的样式表怎么办？ 最好的选择是创建一个新的`<style/>`标记，并将其附加到HTML文档的`<head/>`上。这会自动将新样式表添加到document.styleSheets列表，并允许咱们通过`<style/>`标记的.sheet属性对其进行访问，是不是很机智？

```js
function createRandomName() { 
    const code = Math.random().toString(36).substring(7); 
    return `css-${code}`;
}

function phraseStyle(style) { 
    const keys = Object.keys(style); 
    const keyValue = keys.map(key => { 
        const kebabCaseKey =  key.replace(/([a-z])([A-Z])/g, "$1-$2").toLowerCase(); 
        const value =  `${style[key]}${typeof style[key] === "number" ? "px" : ""}`;
        return `${kebabCaseKey}:${value};`; 
    }); 
    return `{${keyValue.join("")}}`;
}
```

除了上面的小窍门之外。自然，咱们首先需要一种为CSS类生成新的随机名称的方法。然后，将样式对象正确地表达为可行的CSS字符串的形式。这包括驼峰命名和短横线全名之间的转换，以及可选的像素单位（px）转换的处理。

```js
function createClassName(style) { 
    const className = createRandomName(); 
    let styleSheet; 
    // ... 
    styleSheet.insertRule(`.${className}${phraseStyle(style)}`); 
    return className;
}
```

完整代码如下：

**HTML**

```html
<div id="el"></div>
```

**JS**

```js
function createRandomName() { 
    const code = Math.random().toString(36).substring(7); 
    return `css-${code}`;
}
function phraseStyle(style) { 
    const keys = Object.keys(style); 
    const keyValue = keys.map(key => { 
        const kebabCaseKey = key.replace(/([a-z])([A-Z])/g, "$1-$2").toLowerCase(); 
        const value = `${style[key]}${typeof style[key] === "number" ? "px" : ""}`; 
        return `${kebabCaseKey}:${value};`; 
    }); 
    return `{${keyValue.join("")}}`;
}
function createClassName(style) { 
    const className = createRandomName(); 
    let styleSheet; 
    for (let i = 0; i < document.styleSheets.length; i++) { 
        if (document.styleSheets[i].CSSInJS) { 
            styleSheet = document.styleSheets[i]; break; 
        } 
    } 
    if (!styleSheet) { 
        const style = document.createElement("style"); 
        document.head.appendChild(style); 
        styleSheet = style.sheet; 
        styleSheet.CSSInJS = true; 
    } 
    styleSheet.insertRule(`.${className}${phraseStyle(style)}`); 
    return className;
}
const el = document.getElementById("el");
const redRect = createClassName({ width: 100, height: 100, backgroundColor: "red"});
el.classList.add(redRect);
```

**总结**

正如本文咱们所看到的，使用 JS 操作CSS 是一件非常有趣的事，咱们可以挖掘很多好用的 API,上面的例子只是冰山一角，在**CSS API**(或者更确切地说是API)中还有更多方法，它们正等着被揭开神秘面纱。
