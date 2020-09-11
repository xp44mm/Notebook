# Hello World

最简易的rxdom示例如下：

```js
document.addEventListener('DOMContentLoaded',function(){
    const root = document.getElementById('root')
    root.appendChild(h1('Hello, world!'))
});
```

它将在页面上展示一个 “Hello, world!” 的标题。

## 如何阅读本指南

在本指南中，我们将研究rxdom的组成，函数：html中的每一个元素都由函数表示。一旦你掌握了它们，便可以使用这些可复用的小片段组成复杂的应用。

> 提示
>
> 本指南专为喜欢**逐步学习概念**的人士设计。如果你想边学边做，请查阅我们的[实用教程](https://react.docschina.org/tutorial/tutorial.html)。你会发现该指南与教程是相互补充的。



本指南中的每一章节都以其前述章节中介绍的知识点为基础。**你可以按照侧边导航栏中显示的顺序阅读浏览 “核心概念” 的指南章节。以了解 React 的大部分内容。** 例如，[“JSX 简介”](https://react.docschina.org/docs/introducing-jsx.html)就是本文的下一章节。

## 预备知识

React 是一个 JavaScript 库，所以我们假设你对 JavaScript 语言已有基本的了解。**如果你对自己的基础不自信，我们推荐[通过 JavaScript 教程](https://developer.mozilla.org/en-US/docs/Web/JavaScript/A_re-introduction_to_JavaScript)检查你的基础知识储备水平**，使得你能够无压力的阅读本指南。这可能会花费你 30 分钟到 1 个小时的时间，但这样做的好处是你不会觉得同时在学习 React 和 JavaScript。

> 注意
>
> 本指南的示例中偶尔会使用一些 JavaScript 新语法。如果你在过去几年中并没有使用过 JavaScript，大多数情况下[这三点](https://gist.github.com/gaearon/683e676101005de0add59e8bb345340c)应该能帮到你。

## 让我们开始吧！

继续向下滚动，你将在网站页脚右侧找到[下一篇指南](https://react.docschina.org/docs/introducing-jsx.html)的链接。

# JSX 简介

考虑如下变量声明：

```js
const element = h1('Hello, world!');
```

这个有趣的标签语法既不是字符串也不是 HTML。

它被称为 JSX，是一个 JavaScript 的语法扩展。我们建议在 React 中配合使用 JSX，JSX 可以很好地描述 UI 应该呈现出它应有交互的本质形式。JSX 可能会使人联想到模版语言，但它具有 JavaScript 的全部功能。

JSX 可以生成 React “元素”。我们将在[下一章节](https://react.docschina.org/docs/rendering-elements.html)中探讨如何将这些元素渲染为 DOM。下面我们看下学习 JSX 所需的基础知识。

### 为什么使用 JSX？

React 认为渲染逻辑本质上与其他 UI 逻辑内在耦合，比如，在 UI 中需要绑定处理事件、在某些时刻状态发生变化时需要通知到 UI，以及需要在 UI 中展示准备好的数据。

React 并没有采用将*标记与逻辑进行分离到不同文件*这种人为地分离方式，而是通过将二者共同存放在称之为“组件”的松散耦合单元之中，来实现[*关注点分离*](https://en.wikipedia.org/wiki/Separation_of_concerns)。我们将在[后面章节](https://react.docschina.org/docs/components-and-props.html)中深入学习组件。如果你还没有适应在 JS 中使用标记语言，这个[会议讨论](https://www.youtube.com/watch?v=x7cQ3mrcKaY)应该可以说服你。

React [不强制要求](https://react.docschina.org/docs/react-without-jsx.html)使用 JSX，但是大多数人发现，在 JavaScript 代码中将 JSX 和 UI 放在一起时，会在视觉上有辅助作用。它还可以使 React 显示更多有用的错误和警告消息。

搞清楚这个问题后，我们就开始学习 JSX 吧！

### 在 JSX 中嵌入表达式

在下面的例子中，我们声明了一个名为 `name` 的变量，然后在 JSX 中使用它，并将它包裹在大括号中：

```js
const name = 'Josh Perez';
const element = h1(`Hello, ${name}`);

document.addEventListener('DOMContentLoaded', function () {
    const root = document.getElementById('root')
    root.appendChild(element)
});
```

在 JSX 语法中，你可以在大括号内放置任何有效的 [JavaScript 表达式](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Expressions_and_Operators#Expressions)。例如，`2 + 2`，`user.firstName` 或 `formatName(user)` 都是有效的 JavaScript 表达式。

在下面的示例中，我们将调用 JavaScript 函数 `formatName(user)` 的结果，并将结果嵌入到 `<h1>` 元素中。

```js
function formatName(user) {
    return user.firstName + ' ' + user.lastName;
}

const user = {
    firstName: 'Harper',
    lastName: 'Perez'
};

const element = h1(`Hello, ${formatName(user)}!`)

document.addEventListener('DOMContentLoaded', function () {
    const root = document.getElementById('root')
    root.appendChild(element)
});
```

为了便于阅读，我们会将 JSX 拆分为多行。同时，我们建议将内容包裹在括号中，虽然这样做不是强制要求的，但是这可以避免遇到[自动插入分号](http://stackoverflow.com/q/2846283)陷阱。

### JSX 也是一个表达式

在编译之后，JSX 表达式会被转为普通 JavaScript 函数调用，并且对其取值后得到 JavaScript 对象。

也就是说，你可以在 `if` 语句和 `for` 循环的代码块中使用 JSX，将 JSX 赋值给变量，把 JSX 当作参数传入，以及从函数中返回 JSX：

```js
function getGreeting(user) {
    if (user) {
        return h1(`Hello, ${formatName(user)}!`)
    }
    return h1('Hello, Stranger.')
}
```

### JSX 特定属性

你可以通过使用引号，来将属性值指定为字符串字面量：

```js
const element = div({ tabIndex:"0" })
```

也可以使用大括号，来在属性值中插入一个 JavaScript 表达式：

```js
const element = img({ src: user.avatarUrl })
```

在属性中嵌入 JavaScript 表达式时，不要在大括号外面加上引号。你应该仅使用引号（对于字符串值）或大括号（对于表达式）中的一个，对于同一属性不能同时使用这两种符号。

> **警告：**
>
> 属性的名称，完全符合DOM对象的属性成员的名称。
>
> 例如，JSX 里的 `class` 变成了 [`className`](https://developer.mozilla.org/en-US/docs/Web/API/Element/className)，而 `tabindex` 则变为 [`tabIndex`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/tabIndex)。

### 使用 JSX 指定子元素

假如一个标签里面没有内容，你可以使用 `/>` 来闭合标签，就像 XML 语法一样：

```js
const element = img({ src: user.avatarUrl })
```

JSX 标签里能够包含很多子元素:

```js
const element = div(
    h1('Hello!'),
    h2('Good to see you here.')
)
```

### JSX 防止注入攻击

你可以安全地在 JSX 当中插入用户输入内容：

```js
const title = response.potentiallyMaliciousInput;
// 直接使用是安全的：
const element = h1(title)
```

React DOM 在渲染所有输入内容之前，默认会进行[转义](https://stackoverflow.com/questions/7381974/which-characters-need-to-be-escaped-on-html)。它可以确保在你的应用中，永远不会注入那些并非自己明确编写的内容。所有的内容在渲染之前都被转换成了字符串。这样可以有效地防止 [XSS（cross-site-scripting, 跨站脚本）](https://en.wikipedia.org/wiki/Cross-site_scripting)攻击。

### JSX 表示对象

Babel 会把 JSX 转译成一个名为 `React.createElement()` 函数调用。

以下两种示例代码完全等效：

```js
const element = h1({
    className: 'greeting'
}, 'Hello, world!')

const element = document.createElement('h1')
element.className = 'greeting'
element.appendChild(document.createTextNode('Hello, world!'))
```

`React.createElement()` 会预先执行一些检查，以帮助你编写无错代码，但实际上它创建了一个这样的对象：

```js
// 注意：这是简化过的结构
const element = {
  type: 'h1',
  props: {
    className: 'greeting',
    children: 'Hello, world!'
  }
};
```

这些对象被称为 “React 元素”。它们描述了你希望在屏幕上看到的内容。React 通过读取这些对象，然后使用它们来构建 DOM 以及保持随时更新。

我们将在[下一章节](https://react.docschina.org/docs/rendering-elements.html)中探讨如何将 React 元素渲染为 DOM。

> **提示：**
>
> 我们推荐在你使用的编辑器中，使用 [“Babel” 提供的语言定义](https://babeljs.io/docs/editors)，来正确地高亮显示 ES6 和 JSX 代码。本网站使用与其兼容的 [Oceanic Next](https://github.com/voronianski/oceanic-next-color-scheme/) 配色方案。

# 元素渲染

元素是构成rxdom应用的最小砖块。

元素描述了你在屏幕上想看到的内容。

```js
const element = h1('Hello, world!');
```

与浏览器的 DOM 元素不同，React 元素是创建开销极小的普通对象。React DOM 会负责更新 DOM 来与 React 元素保持一致。

> **注意：**
>
> 你可能会将元素与另一个被熟知的概念——“组件”混淆起来。我们会在[下一个章节](https://react.docschina.org/docs/components-and-props.html)介绍组件。组件是由元素构成的。我们强烈建议你不要觉得繁琐而跳过本章节，应当深入阅读这一章节。

## 将一个元素渲染为 DOM

假设你的 HTML 文件某处有一个 `<div>`：

```html
<div id="root"></div>
```

我们将其称为“根” DOM 节点，因为该节点内的所有内容都将由 React DOM 管理。

仅使用 React 构建的应用通常只有单一的根 DOM 节点。如果你在将 React 集成进一个已有应用，那么你可以在应用中包含任意多的独立根 DOM 节点。

想要将一个 React 元素渲染到根 DOM 节点中，只需把它们一起传入 [`ReactDOM.render()`](https://react.docschina.org/docs/react-dom.html#render)：

```js
const element = h1('Hello, world!')

document.addEventListener('DOMContentLoaded', function() {
    const root = document.getElementById('root')
    root.appendChild(element)
})
```

页面上会展示出 “Hello, world”。

## 更新已渲染的元素

React 元素是[不可变对象](https://en.wikipedia.org/wiki/Immutable_object)。一旦被创建，你就无法更改它的子元素或者属性。一个元素就像电影的单帧：它代表了某个特定时刻的 UI。

根据我们已有的知识，更新 UI 唯一的方式是创建一个全新的元素，并将其传入 [`ReactDOM.render()`](https://react.docschina.org/docs/react-dom.html#render)。

考虑一个计时器的例子：

常规计时器：

```js
const element = div(
    h1('Hello, world!'),
    h2()
)

function tick() {
    element.getElementsByTagName('h2')[0].innerText = `It is ${new Date().toLocaleTimeString()}.`
}

document.addEventListener('DOMContentLoaded', function () {
    const root = document.getElementById('root')
    root.appendChild(element)
})
setInterval(tick, 1000);

```

诀窍是知道哪些函数执行一次，哪些函数执行多次。

使用交互式编程的代码实现同样的计时器：

```js
function tick() {
    const x = interval(1000)
        |> map(i => new Date().toLocaleTimeString())
        |> map(t => `It is ${t}.`)
    const element = div(
        h1('Hello, world!'),
        h2(textNode(x))
    )
    return element
}

const element = tick()

document.addEventListener('DOMContentLoaded', function() {
    const root = document.getElementById('root')
    root.appendChild(element)
})
```



这个例子会在 [`setInterval()`](https://developer.mozilla.org/en-US/docs/Web/API/WindowTimers/setInterval) 回调函数，每秒都调用 [`ReactDOM.render()`](https://react.docschina.org/docs/react-dom.html#render)。

> **注意：**
>
> 在实践中，大多数 React 应用只会调用一次 [`ReactDOM.render()`](https://react.docschina.org/docs/react-dom.html#render)。在下一个章节，我们将学习如何将这些代码封装到[有状态组件](https://react.docschina.org/docs/state-and-lifecycle.html)中。
>
> 我们建议你不要跳跃着阅读，因为每个话题都是紧密联系的。

## React 只更新它需要更新的部分

React DOM 会将元素和它的子元素与它们之前的状态进行比较，并只会进行必要的更新来使 DOM 达到预期的状态。

你可以使用浏览器的检查元素工具查看[上一个例子](https://react.docschina.org/redirect-to-codepen/rendering-elements/update-rendered-element)来确认这一点。

![DOM inspector showing granular updates](https://react.docschina.org/c158617ed7cc0eac8f58330e49e48224/granular-dom-updates.gif)

尽管每一秒我们都会新建一个描述整个 UI 树的元素，React DOM 只会更新实际改变了的内容，也就是例子中的文本节点。

根据我们的经验，考虑 UI 在任意给定时刻的状态，而不是随时间变化的过程，能够消灭一整类的 bug。