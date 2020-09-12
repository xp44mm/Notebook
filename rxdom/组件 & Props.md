通常，一个组件代表一个dom元素，也可以代表一个元素中连续的子元素，用返回数组表示无根元素的组件。

元素类型在整个生命周期内都是不可变的，只能用另一个元素代替，伪装成他是可变的。



# 组件 & Props

组件允许你将 UI 拆分为独立可复用的代码片段，并对每个片段进行独立构思。本指南旨在介绍组件的相关理念。你可以[参考详细组件 API](https://react.docschina.org/docs/react-component.html)。

组件，从概念上类似于 JavaScript 函数。它接受任意的输入参数，并返回用于描述页面展示内容的元素。

## 函数组件与 class 组件

定义组件最简单的方式就是编写 JavaScript 函数：

```js
function Welcome(name) {
  return h1(`Hello, ${name}`)
}
```

该函数是一个有效的 React 组件，因为它接收唯一带有数据的 “props”（代表属性）对象与并返回一个 React 元素。这类组件被称为“函数组件”，因为它本质上就是 JavaScript 函数。

你同时还可以使用 [ES6 的 class](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Classes) 来定义组件：

```js
class Welcome extends React.Component {
  render() {
    return <h1>Hello, {this.props.name}</h1>;
  }
}
```

上述两个组件在 React 里是等效的。

我们将在[下一章节](https://react.docschina.org/docs/state-and-lifecycle.html)中讨论关于函数组件和 class 组件的额外特性。

## 渲染组件

之前，我们遇到的 React 元素都只是 DOM 标签（DOM tags）：

```js
const element = div();
```

不过，React 元素也可以是用户自定义的组件：

```js
const element = Welcome("Sara")
```

当 React 元素为用户自定义组件时，它会将 JSX 所接收的属性（attributes）以及子组件（children）转换为单个对象传递给组件，这个对象被称之为 “props”。

例如，这段代码会在页面上渲染 “Hello, Sara”：

```js
function Welcome(name) {
  return h1(`Hello, ${name}`)
}

const element = Welcome("Sara")

document.addEventListener('DOMContentLoaded',function(){
    const root = document.getElementById('root')
    root.appendChild(element)
});
```

让我们来回顾一下这个例子中发生了什么：

1. 我们调用 `ReactDOM.render()` 函数，并传入 `<Welcome name="Sara" />` 作为参数。
2. React 调用 `Welcome` 组件，并将 `{name: 'Sara'}` 作为 props 传入。
3. `Welcome` 组件将 `<h1>Hello, Sara</h1>` 元素作为返回值。
4. React DOM 将 DOM 高效地更新为 `<h1>Hello, Sara</h1>`。

> **注意：** 组件名称必须以大写字母开头。
>
> React 会将以小写字母开头的组件视为原生 DOM 标签。例如，`<div />` 代表 HTML 的 div 标签，而 `<Welcome />` 则代表一个组件，并且需在作用域内使用 `Welcome`。
>
> 你可以在[深入 JSX](https://react.docschina.org/docs/jsx-in-depth.html#user-defined-components-must-be-capitalized)中了解更多关于此规范的原因。

类名用大写，函数名用小写字母开头。



## 组合组件

组件可以在其输出中引用其他组件。这就可以让我们用同一组件来抽象出任意层次的细节。按钮，表单，对话框，甚至整个屏幕的内容：在 React 应用程序中，这些通常都会以组件的形式表示。

例如，我们可以创建一个可以多次渲染 `Welcome` 组件的 `App` 组件：

```js
function Welcome(name) {
  return h1(`Hello, ${name}`)
}

function App() {
  return div(
    Welcome("Sara"),
    Welcome("Cahal"),
    Welcome("Edite"),
  );
}

document.addEventListener('DOMContentLoaded',function(){
    const root = document.getElementById('root')
    root.appendChild(App())
});
```

通常来说，每个新的 React 应用程序的顶层组件都是 `App` 组件。但是，如果你将 React 集成到现有的应用程序中，你可能需要使用像 `Button` 这样的小组件，并自下而上地将这类组件逐步应用到视图层的每一处。

## 提取组件

将组件拆分为更小的组件。

例如，参考如下 `Comment` 组件：

```js
function Comment(props) {
  return (
    <div className="Comment">
      <div className="UserInfo">
        <img className="Avatar" src={props.author.avatarUrl} alt={props.author.name} />
        <div className="UserInfo-name">
          {props.author.name}
        </div>
      </div>
      <div className="Comment-text">
        {props.text}
      </div>
      <div className="Comment-date">
        {formatDate(props.date)}
      </div>
    </div>
  );
}
```



该组件用于描述一个社交媒体网站上的评论功能，它接收 `author`（对象），`text` （字符串）以及 `date`（日期）作为 props。

该组件由于嵌套的关系，变得难以维护，且很难复用它的各个部分。因此，让我们从中提取一些组件出来。

首先，我们将提取 `Avatar` 组件：

```js
function Avatar(props) {
  return img(
    {className:"Avatar",  src:props.user.avatarUrl,  alt:props.user.name});
}
```

`Avatar` 不需知道它在 `Comment` 组件内部是如何渲染的。因此，我们给它的 props 起了一个更通用的名字：`user`，而不是 `author`。

我们建议从组件自身的角度命名 props，而不是依赖于调用组件的上下文命名。

我们现在针对 `Comment` 做些微小调整：

```js
function Comment(props) {
  return (
    <div className="Comment">
      <div className="UserInfo">
        <Avatar user={props.author} />        
<div className="UserInfo-name">
          {props.author.name}
        </div>
      </div>
      <div className="Comment-text">
        {props.text}
      </div>
      <div className="Comment-date">
        {formatDate(props.date)}
      </div>
    </div>
  );
}
```

接下来，我们将提取 `UserInfo` 组件，该组件在用户名旁渲染 `Avatar` 组件：

```js
function UserInfo(props) {
  return (
    <div className="UserInfo">      
    <Avatar user={props.user} />      
    <div className="UserInfo-name">        
    {props.user.name}      
    </div>    
    </div>  );
}
```

进一步简化 `Comment` 组件：

```js
function Comment(props) {
  return (
    <div className="Comment">
      <UserInfo user={props.author} />      
<div className="Comment-text">
        {props.text}
      </div>
      <div className="Comment-date">
        {formatDate(props.date)}
      </div>
    </div>
  );
}
```



最初看上去，提取组件可能是一件繁重的工作，但是，在大型应用中，构建可复用组件库是完全值得的。根据经验来看，如果 UI 中有一部分被多次使用（`Button`，`Panel`，`Avatar`），或者组件本身就足够复杂（`App`，`FeedStory`，`Comment`），那么它就是一个可复用组件的候选项。

## Props 的只读性

组件无论是使用[函数声明还是通过 class 声明](https://react.docschina.org/docs/components-and-props.html#function-and-class-components)，都决不能修改自身的 props。来看下这个 `sum` 函数：

```js
function sum(a, b) {
  return a + b;
}
```

这样的函数被称为[“纯函数”](https://en.wikipedia.org/wiki/Pure_function)，因为该函数不会尝试更改输入参数，且多次调用相同的输入参数始终返回相同的结果。

相反，下面这个函数则不是纯函数，因为它更改了自己的输入参数：

```js
function withdraw(account, amount) {
  account.total -= amount;
}
```

React 非常灵活，但它也有一个严格的规则：

**所有 React 组件都必须像纯函数一样保护它们的 props 不被更改。**

组件就是普通函数，小写字母开头，输入参数不变。输入参数是形式自定义，不需要遵守props内。

当然，应用程序的 UI 是动态的，并会伴随着时间的推移而变化。在[下一章节](https://react.docschina.org/docs/state-and-lifecycle.html)中，我们将介绍一种新的概念，称之为 “state”。在不违反上述规则的情况下，state 允许 React 组件随用户操作、网络响应或者其他变化而动态更改输出内容。


# State & 生命周期

本页面介绍了 React 组件中 state 和生命周期的概念。你可以查阅[详细的组件 API 参考文档](https://react.docschina.org/docs/react-component.html)。

请参考[前一章节](https://react.docschina.org/docs/rendering-elements.html#updating-the-rendered-element)中时钟的例子。在[元素渲染](https://react.docschina.org/docs/rendering-elements.html#rendering-an-element-into-the-dom)章节中，我们只了解了一种更新 UI 界面的方法。通过调用 `ReactDOM.render()` 来修改我们想要渲染的元素：

```js
function tick() {
  const element = (
    <div>
      <h1>Hello, world!</h1>
      <h2>It is {new Date().toLocaleTimeString()}.</h2>
    </div>
  );
  ReactDOM.render(    
      element,    
      document.getElementById('root')  
  );
}

setInterval(tick, 1000);
```



在本章节中，我们将学习如何封装真正可复用的 `Clock` 组件。它将设置自己的计时器并每秒更新一次。

我们可以从封装时钟的外观开始：

```js
function Clock(props) {
  return (
    <div>      
    <h1>Hello, world!</h1>      
    <h2>It is {props.date.toLocaleTimeString()}.</h2>    
    </div>  
);
}

function tick() {
  ReactDOM.render(
    <Clock date={new Date()} />,   
document.getElementById('root')
  );
}

setInterval(tick, 1000);
```

然而，它忽略了一个关键的技术细节：`Clock` 组件需要设置一个计时器，并且需要每秒更新 UI。

理想情况下，我们希望只编写一次代码，便可以让 `Clock` 组件自我更新：

```js
ReactDOM.render(
  <Clock />,  
  document.getElementById('root')
);
```

我们需要在 `Clock` 组件中添加 “state” 来实现这个功能。

State 与 props 类似，但是 state 是私有的，并且完全受控于当前组件。

## 将函数组件转换成 class 组件

通过以下五步将 `Clock` 的函数组件转成 class 组件：

1. 创建一个同名的 [ES6 class](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Classes)，并且继承于 `React.Component`。
2. 添加一个空的 `render()` 方法。
3. 将函数体移动到 `render()` 方法之中。
4. 在 `render()` 方法中使用 `this.props` 替换 `props`。
5. 删除剩余的空函数声明。

```js
class Clock extends React.Component {
  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.props.date.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}
```



现在 `Clock` 组件被定义为 class，而不是函数。

每次组件更新时 `render` 方法都会被调用，但只要在相同的 DOM 节点中渲染 `<Clock />` ，就仅有一个 `Clock` 组件的 class 实例被创建使用。这就使得我们可以使用如 state 或生命周期方法等很多其他特性。

## 向 class 组件中添加局部的 state

我们通过以下三步将 `date` 从 props 移动到 state 中：

1. 把 `render()` 方法中的 `this.props.date` 替换成 `this.state.date` ：

```js
class Clock extends React.Component {
  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.state.date.toLocaleTimeString()}.</h2>      </div>
    );
  }
}
```

1. 添加一个 [class 构造函数](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Classes#Constructor)，然后在该函数中为 `this.state` 赋初值：

```js
class Clock extends React.Component {
  constructor(props) {
    super(props);
    this.state = {date: new Date()};
  }

  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.state.date.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}
```

通过以下方式将 `props` 传递到父类的构造函数中：

```js
  constructor(props) {
    super(props);    
    this.state = {date: new Date()};
  }
```

Class 组件应该始终使用 `props` 参数来调用父类的构造函数。

1. 移除 `<Clock />` 元素中的 `date` 属性：

```js
ReactDOM.render(
  <Clock />,  
  document.getElementById('root')
);
```

我们之后会将计时器相关的代码添加到组件中。

代码如下：

```js
class Clock extends React.Component {
  constructor(props) {    
  super(props);    
  this.state = {date: new Date()};  
  }
  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.state.date.toLocaleTimeString()}.</h2>      
        </div>
    );
  }
}

ReactDOM.render(
  <Clock />,  
  document.getElementById('root')
);
```


接下来，我们会设置 `Clock` 的计时器并每秒更新它。

## 将生命周期方法添加到 Class 中

在具有许多组件的应用程序中，当组件被销毁时释放所占用的资源是非常重要的。

当 `Clock` 组件第一次被渲染到 DOM 中的时候，就为其[设置一个计时器](https://developer.mozilla.org/en-US/docs/Web/API/WindowTimers/setInterval)。这在 React 中被称为“挂载（mount）”。

同时，当 DOM 中 `Clock` 组件被删除的时候，应该[清除计时器](https://developer.mozilla.org/en-US/docs/Web/API/WindowTimers/clearInterval)。这在 React 中被称为“卸载（unmount）”。

我们可以为 class 组件声明一些特殊的方法，当组件挂载或卸载时就会去执行这些方法：

```js
class Clock extends React.Component {
  constructor(props) {
    super(props);
    this.state = {date: new Date()};
  }

  componentDidMount() {  }
  componentWillUnmount() {  }
  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.state.date.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}
```

这些方法叫做“生命周期方法”。

`componentDidMount()` 方法会在组件已经被渲染到 DOM 中后运行，所以，最好在这里设置计时器：

```js
  componentDidMount() {
    this.timerID = setInterval(      () => this.tick(),      1000    );
  }
```

接下来把计时器的 ID 保存在 `this` 之中（`this.timerID`）。

尽管 `this.props` 和 `this.state` 是 React 本身设置的，且都拥有特殊的含义，但是其实你可以向 class 中随意添加不参与数据流（比如计时器 ID）的额外字段。

我们会在 `componentWillUnmount()` 生命周期方法中清除计时器：

```js
  componentWillUnmount() {
    clearInterval(this.timerID);
  }
```

最后，我们会实现一个叫 `tick()` 的方法，`Clock` 组件每秒都会调用它。

使用 `this.setState()` 来时刻更新组件 state：

```js
class Clock extends React.Component {
  constructor(props) {
    super(props);
    this.state = {date: new Date()};
  }

  componentDidMount() {
    this.timerID = setInterval(
      () => this.tick(),
      1000
    );
  }

  componentWillUnmount() {
    clearInterval(this.timerID);
  }

  tick() {    this.setState({      date: new Date()    });  }
  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.state.date.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}

ReactDOM.render(
  <Clock />,
  document.getElementById('root')
);
```



现在时钟每秒都会刷新。

让我们来快速概括一下发生了什么和这些方法的调用顺序：

1. 当 `<Clock />` 被传给 `ReactDOM.render()`的时候，React 会调用 `Clock` 组件的构造函数。因为 `Clock` 需要显示当前的时间，所以它会用一个包含当前时间的对象来初始化 `this.state`。我们会在之后更新 state。
2. 之后 React 会调用组件的 `render()` 方法。这就是 React 确定该在页面上展示什么的方式。然后 React 更新 DOM 来匹配 `Clock` 渲染的输出。
3. 当 `Clock` 的输出被插入到 DOM 中后，React 就会调用 `ComponentDidMount()` 生命周期方法。在这个方法中，`Clock` 组件向浏览器请求设置一个计时器来每秒调用一次组件的 `tick()` 方法。
4. 浏览器每秒都会调用一次 `tick()` 方法。 在这方法之中，`Clock` 组件会通过调用 `setState()` 来计划进行一次 UI 更新。得益于 `setState()` 的调用，React 能够知道 state 已经改变了，然后会重新调用 `render()` 方法来确定页面上该显示什么。这一次，`render()` 方法中的 `this.state.date` 就不一样了，如此以来就会渲染输出更新过的时间。React 也会相应的更新 DOM。
5. 一旦 `Clock` 组件从 DOM 中被移除，React 就会调用 `componentWillUnmount()` 生命周期方法，这样计时器就停止了。

## 正确地使用 State

关于 `setState()` 你应该了解三件事：

### 不要直接修改 State

例如，此代码不会重新渲染组件：

```js
// Wrong
this.state.comment = 'Hello';
```

而是应该使用 `setState()`:

```js
// Correct
this.setState({comment: 'Hello'});

this.comment.next('Hello');
```

构造函数是唯一可以给 `this.state` 赋值的地方。

### State 的更新可能是异步的

出于性能考虑，React 可能会把多个 `setState()` 调用合并成一个调用。

因为 `this.props` 和 `this.state` 可能会异步更新，所以你不要依赖他们的值来更新下一个状态。

例如，此代码可能会无法更新计数器：

```js
// Wrong
this.setState({
  counter: this.state.counter + this.props.increment,
});
```

要解决这个问题，可以让 `setState()` 接收一个函数而不是一个对象。这个函数用上一个 state 作为第一个参数，将此次更新被应用时的 props 做为第二个参数：

```js
// Correct
this.setState((state, props) => ({
  counter: state.counter + props.increment
}));
```

上面使用了[箭头函数](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions)，不过使用普通的函数也同样可以：

```js
// Correct
this.setState(function(state, props) {
  return {
    counter: state.counter + props.increment
  };
});
```

### State 的更新会被合并

当你调用 `setState()` 的时候，React 会把你提供的对象合并到当前的 state。

例如，你的 state 包含几个独立的变量：

```js
  constructor(props) {
    super(props);
    this.state = {
      posts: [],      comments: []
    };
  }
```

然后你可以分别调用 `setState()` 来单独地更新它们：

```js
  componentDidMount() {
    fetchPosts().then(response => {
      this.setState({
        posts: response.posts      
      });
    });

    fetchComments().then(response => {
      this.setState({
        comments: response.comments      });
    });
  }
```

这里的合并是浅合并，所以 `this.setState({comments})` 完整保留了 `this.state.posts`， 但是完全替换了 `this.state.comments`。

## 数据是向下流动的

不管是父组件或是子组件都无法知道某个组件是有状态的还是无状态的，并且它们也并不关心它是函数组件还是 class 组件。

这就是为什么称 state 为局部的或是封装的的原因。除了拥有并设置了它的组件，其他组件都无法访问。

组件可以选择把它的 state 作为 props 向下传递到它的子组件中：

```js
<h2>It is {this.state.date.toLocaleTimeString()}.</h2>
```

这对于自定义组件同样适用：

```js
<FormattedDate date={this.state.date} />
```

`FormattedDate` 组件会在其 props 中接收参数 `date`，但是组件本身无法知道它是来自于 `Clock` 的 state，或是 `Clock` 的 props，还是手动输入的：

```js
function FormattedDate(props) {
  return <h2>It is {props.date.toLocaleTimeString()}.</h2>;
}
```

这通常会被叫做“自上而下”或是“单向”的数据流。任何的 state 总是所属于特定的组件，而且从该 state 派生的任何数据或 UI 只能影响树中“低于”它们的组件。

如果你把一个以组件构成的树想象成一个 props 的数据瀑布的话，那么每一个组件的 state 就像是在任意一点上给瀑布增加额外的水源，但是它只能向下流动。

为了证明每个组件都是真正独立的，我们可以创建一个渲染三个 `Clock` 的 `App` 组件：

```js
function App() {
  return (
    <div>
      <Clock />
      <Clock />
      <Clock />    
      </div>
  );
}

ReactDOM.render(
  <App />,
  document.getElementById('root')
);
```



每个 `Clock` 组件都会单独设置它自己的计时器并且更新它。

在 React 应用中，组件是有状态组件还是无状态组件属于组件实现的细节，它可能会随着时间的推移而改变。你可以在有状态的组件中使用无状态的组件，反之亦然。

