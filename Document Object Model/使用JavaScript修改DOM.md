## 使用JavaScript修改DOM

原文链接： [zellwk.com](https://zellwk.com/blog/js-in-dom/)

如果你正在学习Javascript, 你要学习的第一件事情就是去修改DOM （在了解的一些像变量函数等基础之后），作为一个前端开发者，这是每天你要做的事情之一。

过去修改DOM是不容易的。 我们需要jQuery来让事情变得更简单幸运的是，这里再不需要jQuery了。

在这篇文章里，我会向你展示一些作为前端开发者你需要熟练掌握的一些事情。

## 你会用DOM做什么呢?

当你用DOM工作的时候, 你会发现自己正在做下面一件或者更多件事情:

1. 选择HTML元素
2. 添加或者移除事件监听
3. 添加或者删除类
4. 添加，改变或者删除属性
5. 添加或者移除HTML元素

在接下来的部分，我将会解释每个事情都是什么，你为什么要用它们和怎么用它们。让我们进入第一个——选择HTML元素

## 选择HTML元素

在你用DOM做其它任何事情之前，知道怎么去选择HTML元素是第一步，一旦你选择了一个元素，你就能够添加事件监听，修改类，做一些其它可以想到的事情。

你只需要知道两个方法就可以选择你想选择的任何事情——`querySelector`和`querySelectorAll`

### querySelector

`querySelector` 帮助你 **选择一个HTML元素**。如果在你的内容里有多个HTML元素被查找，`querySelector` 总是会返回第一个元素，像这样：

```
`document.querySelector(selector)`
```

你可以用`querySelector`通过元素的id,类，甚至是标签来选择一个元素。

让我们快速写一个的例子，用下面这个HTML告诉你：

```
<div id="the-one">ID</div>
<div class="an-awesome-class">Class</div>
<p>A tag</p>
```

- **使用id选择一个元素**, 你要在id前面加一个`#`
- **使用一个类选择元素**,你要在类前边加上一个`.`
- **使用一个标签选择元素**,你只要简单地写下和你选择器一样标签即可

```
document.querySelector('#the-one')
// => <div id="id">ID</div>

document.querySelector('.an-awesome-class')
// => <div class="an-awesome-class">Class</div>

document.querySelector('p')
// => <p>A tag</p>
```

### 复杂的选择

`querySelector` 强大的令人难以置信。你可以通过绑定id,类，和标签来执行复杂的选项。 像下面这样:

```
`document.querySelector('div#the-one')`
```

即使绑定选择器是可以的，但我建议你不要做这件事情，因为大多数情况下，它是不必要的。

### 在元素内选择元素

这里有一件关于 `querySelector`很棒的事，你可以命令它在其它元素里查找一个元素，这样会减少查找一个较深选择器所需的时间。

这样做的时候，在你的多个类，id，标签之间要加一个空格。这有一个例子：

```
<div class="container">
  <div class="inner-item">Inner item!</div>
</div>
let innerItem = document.querySelector('.container .inner-item')
// => <div class="inner-item">Inner item!</div>
```

在二选一的情况下，如果你已经使用`querySelector`选择了一个元素，你也可以使用这个元素去执行另外的 `querySelector`调用：

```
let container = document.querySelector('.container')
let innerItem = container.querySelector('.inner-item')
// => innerItem is <div class="inner-item">Inner item!</div>
```

这就是你需要了解的关于`querySelector`的全部。

现在，如果你需要选择超过一个的元素该怎么办呢？这就是`querySelectorAll` 起作用的地方。

### querySelectorAll

`querySelectorAll` 是一个帮助你选择多个元素的方法。

```
`let allELements = document.querySelectorAll(selectors);`
```

在这个例子中，`selectors`拥有和`querySelector`相同的语法 ,唯一不同之处就是你可以运行多个选择的对象。

```
<div class="thing">A thing</div>
<div class="thing">A thing</div>
<div class="another-thing">Another thing</div>
let allThings = document.querySelectorAll('.thing, .another-thing')
// => [
//   <div class="thing">A thing</div>,
//   <div class="thing">A thing</div>,
//   <div class="thing">Another thing</div>
// ]
```

这就是重要的部分。

`querySelectorAll`返回一个集合。（即使它看起来像一个数组）

如果你目前工作只是用现代浏览器，你可以调用`NodeList.forEach`在这个集合内获取每个单独的元素。

如果你正在使用老的浏览器工作，在用forEach call遍历它的之前，你需要把这个集合列表转化成数组。最简单的实现这个的办法就是使用`Array.from()`

```
// Modern browsers
let allThings = document.querySelectorAll('.thing, .another-thing')
allThings.forEach(el => {/* do something with element */})

// Older browsers
let allThings = document.querySelectorAll('.thing, .another-thing')
// You might need a polyfill for Array.from.
// Alternatively, use Array.prototype.slice.call(allThings);
let allThingsArray = Array.from(allThings)
allThingsArray.forEach(el => {/* do something with element */})
```

接下来，让我们继续添加和移除事件监听。

## 添加或者移除事件监听器

无论什么时候事件被触发，事件监听器允许你的JavaScript执行一个操做。这就是你怎么知道用户和DOM在什么时候交互。举个例子就是当他们点击一个按钮的时候：

看看Zell Liew在[CodePen](http://codepen.io/)上的demo[JS改变DOM](https://codepen.io/zellwk/pen/eWBLdZ/)

这里，你只需要知道两个方法——`addEventListener` 和 `removeEventListener`

### 添加事件监听器

要添加你的事件监听器，你首先要选择你的HTML元素，然后调用`addEventListener`方法。他接受两个参数，像这样：

```
let thing = document.querySelector('.thing')
thing.addEventListener(event, callback)
```

`event` 是你想要监听的事件的名字。这些事件在参数里已经被预先确定了。这里有一个你想要的关于常规事件类型的[便利清单](https://developer.mozilla.org/en-US/docs/Web/Events)

`callback`是一个无论在任何时候事件被触发你想要执行的函数。它包括一个参数——事件对象。这里有一个典型的callback像下边这样：

```js
thing.addEventListener('click', callback)

function callback (e) {
  e.preventDefault() // Prevents default behavior. Only use this when necessary
  console.log('thing is clicked!')
}
```

我们可以用事件[对象（e）](https://developer.mozilla.org/en/docs/Web/API/Event)做很多事情。但那是另外的话题。让我们继续移除事件监听。

### 移除事件监听器

移除事件监听器和添加事件监听器类似。这里，你可以调用`removeEventListener`方法，然后传进两个参数——`event`类型和你的回调函数。

```js
thing.removeEventListener('click', callback)
```

通常，当一个任务完成后你将移除一个事件监听器。所以，常常会发现在一个`addEventListener`内调用`removeEventListener`。

```
thing.addEventListener('click', callback)

function callback () {
  console.log('thing is clicked!')
  // removes event listener
  thing.removeEventListener('click', callback)
}
```

当你要移除一个事件监听，你将不再监听到这个事件。所以，在上面的代码里，回调函数只会在`.thing`得到第一次点击时触发。更进一步的点击不会再触发这个回调函数。

注释：当你不再需要事件监听的时候应该移除它， 通过这么做，你会释放一些资源给其它的任务。

让我们继续。

## 添加或者移除类

还记得上面的按钮demo么？

看看Zell Liew在[CodePen](http://codepen.io/)上的demo[JS改变DOM](https://codepen.io/zellwk/pen/eWBLdZ/)

这里是我让这个demo跑起来我要做的事情：

1. 当用户点击一个按钮的时候添加 `.is-open` 到``
2. 当用户点击按钮的时候从已经公开的if及移除 `.is-open`
3. ``的过渡已经用css做了。

这个demo向你展示了css与Javascript协同工作的威力。你可以仅仅通过添加（或移除）一个类来创建各种交互。

这里是你怎样可以添加一个类，移除一个类或者检查一个类是否存在：

1. 使用`element.classList.add('classname')` **添加一个类**
2. 使用`element.classList.remove('classname')` **移除一个类**
3. 使用 `element.classList.contains('classname')` **检查一个类是否存在**

这里是当你点击按钮时候从'button'里添加或者移除`.is-open`的代码：

```
let button = document.querySelector('button')
let nav = document.querySelector('nav')

button.addEventListener('click', toggleNav)

function toggleNav() {
  // Checks if nav has is-open class
  if (nav.classList.contains('is-open')) {
    // removes is-open class
    nav.classList.remove('is-open')
  } else {
    // adds is-open class
    nav.classList.add('is-open')
  }
}
```

让我们继续来添加，修改和移除属性。

## 添加，修改和移除属性

属性是HTML元素中重要的一部分。有时候，你需要从这些属性提取信息来提供环境给你的javascript.还有一些时候，你可能用这些属性去帮助 你写一些容易理解的接口。

这里有上面导航的demo,用一个容易理解的方法写出来：

看看Zell Liew在[CodePen](http://codepen.io/)上的demo[JS改变DOM（容易理解的方法）](https://codepen.io/zellwk/pen/aWBaME/)

在这个demo里，两件事情发生了改变：

1. 我添加了`aria-expanded`到`button`里来告诉浏览器的阅读者菜单列表是否被展开。
2. 我添加了`aria-hidden` 到 `nav`里来阻止屏幕读者阅读被隐藏的菜单。

这里是你可以从属性中提取信息，设置属性和移除属性：

1. 使用`getAttribute('attribute-name')` **获取属性**
2. 使用`setAttribute('attribute-name', 'attribute-value')` **改变/设置属性**
3. 使用`removeAttribute('attribute-name')` **移除属性**

```
// Get attribute
button.getAttribute('aria-expanded')

// Set attribute
button.setAttribute('aria-expanded', true)

// Remove attribute
button.removeAttribute('aria-expanded')
```

最后，让我们继续添加或移除元素。

## 添加或者移除元素

让我们用一个demo开始这部分：

看看Zell Liew在[CodePen](http://codepen.io/)上的demo[JS改变DOM（添加或者移除元素） ](https://codepen.io/zellwk/pen/EmNdWp/)在上面如果你多次点击这个按钮，你会看到我像添加其它列表项一样添加了文本`Hello again, world!`到这个DOM里。

### 添加元素到DOM里

这里有三个步骤来添加文本到DOM里。他们是：

1. 用`document.createElement`创建一个HTML元素。
2. 通过设置`innerHTML`来添加内容到HTML元素里。
3. 用 `parentNode.insertBefore` 或者 `parentNode.appendChild` 添加它到DOM里。

```
let ul = document.querySelector('ul')

// Creating a <li> element
let li = document.createElement('li')

// Adding content to the <li> element
li.innerHTML = 'Hello again, world!'

// Adding it to the DOM
ul.appendChild(li)
```

### 从DOM里移除元素

从DOM里移除一个元素，你需要调用`parentNode.removeChild`，这个方法接受一个参数——要移除的元素。

```
`ul.removeChild(li)`
```

我们不能简单地说移除`li` 并且期望Javascript知道移除哪个列表。我们需要明确地告诉我们的Javascript哪个要移除。

如果你使用`querySelector`来选择移除的元素，这会是最简单的方法：

```
let parent = document.querySelector('.parent')
let elToRemove = document.querySelector('.element-to-remove')
parent.removeChild(elToRemove)

// Of if you don't want to write a separate querySelector
elToRemove.parentNode.removeChild(elToRemove)
```

在上面的demo里，我们不能这么做，因为那儿没有办法用类或者id来告诉哪个是第一个哪个是最后一个列表项。

反而，我们可以使用`parentNode.children`来得到一个ul内的元素列表集合，然后，使用数组的方法获取明确的元素来移除。

这是移除第一个子元素的代码：

```
let list = document.querySelector('ul')
removeFirst.addEventListener('click', e => {
  if (list.children.length) {
    let firstNode = list.children[0]
    list.removeChild(firstNode)
  }
})
```

replaceChild

## 小结

作为一个前端开发者，修改DOM是你需要知道的最重要的事情之一，你可以用DOM做各种你想象到的事情。 在这篇文章中，我向你展示了五个你需要改变DOM常用的方法，附加你需要了解的关键代码。现在，来吧和DOM玩耍，来创造奇迹吧? 你对这篇文章有什么看法？我很乐意在下面的评论区里看到你的想法:)