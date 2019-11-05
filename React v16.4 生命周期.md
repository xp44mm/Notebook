# React v16.4 生命周期

我们将React的生命周期分为三个阶段，然后详细讲解每个阶段具体调用了什么函数，这三个阶段是：

* 挂载阶段
* 更新阶段
* 卸载阶段

## 挂载阶段

挂载阶段，也可以理解为组件的初始化阶段，就是将我们的组件插入到DOM中，只会发生一次

这个阶段的生命周期函数调用如下：

* constructor
* getDerivedStateFromProps
* render
* componentDidMount

### constructor

组件构造函数，第一个被执行。如果没有显示定义它，我们会拥有一个默认的构造函数。

如果显示定义了构造函数，我们必须在构造函数第一行执行`super(props)`，否则我们无法在构造函数里拿到`this`对象，这些都属于ES6的知识。

在构造函数里面我们一般会做两件事：

* 初始化`state`对象
* 给自定义方法绑定`this`

```js
constructor(props) {
    super(props)
    
    this.state = {
      select,
      height: 'atuo',
      externalClass,
      externalClassText
    }

    this.handleChange1 = this.handleChange1.bind(this)
    this.handleChange2 = this.handleChange2.bind(this)
}
```

> 禁止在构造函数中调用`setState`，可以直接给`state`设置初始值

### getDerivedStateFromProps
```js
    static getDerivedStateFromProps(nextProps, prevState){}
```

一个静态方法，所以不能在这个函数里面使用`this`，这个函数有两个参数`props`和`state`，分别指接收到的新参数和当前的`state`对象，这个函数会返回一个对象用来更新当前的`state`对象，如果不需要更新可以返回`null`。

该函数会在挂载时，接收到新的`props`，调用了`setState`和`forceUpdate`时被调用。

当我们接收到新的属性想去修改我们`state`，可以使用`getDerivedStateFromProps`
```js
class ExampleComponent extends React.Component {
  state = {
    isScrollingDown: false,
    lastRow: null
  }
  static getDerivedStateFromProps(nextProps, prevState) {
    if (nextProps.currentRow !== prevState.lastRow) {
        return {
            isScrollingDown:
            nextProps.currentRow > prevState.lastRow,
            lastRow: nextProps.currentRow
        }
    }
    return null
  }
}
```

### render

React中最核心的方法，一个组件中必须要有这个方法

返回的类型有以下几种：

* 原生的DOM，如div
* React组件
* Fragment（片段）
* Portals（插槽）
* 字符串和数字，被渲染成text节点
* Boolean和null，不会渲染任何东西

`render`函数是纯函数，它只做一件事，就是返回需要渲染的东西，不应该包含其它的业务逻辑，如数据请求，对于这些业务逻辑请移到`componentDidMount`和`componentDidUpdate`中。

### componentDidMount

组件装载之后调用，此时我们可以获取到DOM节点并操作，比如对`canvas`，`svg`的操作，服务器请求，订阅都可以写在这个里面，但是记得在`componentWillUnmount`中取消订阅。

```js
componentDidMount() {
    const { progressCanvas, progressSVG } = this

    const canvas = progressCanvas.current
    const ctx = canvas.getContext('2d')
    canvas.width = canvas.getBoundingClientRect().width
    canvas.height = canvas.getBoundingClientRect().height

    const svg = progressSVG.current
    const rect = document.createElementNS('http://www.w3.org/2000/svg', 'rect')
    rect.setAttribute('x', 0)
    rect.setAttribute('y', 0)
    rect.setAttribute('width', 0)
    rect.setAttribute('height', svg.getBoundingClientRect().height)
    rect.setAttribute('style', 'fill:red')

    const animate = document.createElementNS('http://www.w3.org/2000/svg', 'animate')
    animate.setAttribute('attributeName', 'width')
    animate.setAttribute('from', 0)
    animate.setAttribute('to', svg.getBoundingClientRect().width)
    animate.setAttribute('begin', '0ms')
    animate.setAttribute('dur', '1684ms')
    animate.setAttribute('repeatCount', 'indefinite')
    animate.setAttribute('calcMode', 'linear')
    rect.appendChild(animate)
    svg.appendChild(rect)
    svg.pauseAnimations()

    this.canvas = canvas
    this.svg = svg
    this.ctx = ctx
}
```

在`componentDidMount`中调用`setState`会触发一次额外的渲染，多调用了一次`render`函数，但是用户对此没有感知，因为它是在浏览器刷新屏幕前执行的，但是我们应该在开发中避免它，因为它会带来一定的性能问题，我们应该在`constructor`中初始化我们的`state`对象，而不应该在`componentDidMount`调用`state`方法。

## 更新阶段

更新阶段，当组件的`props`改变了，或组件内部调用了`setState`或者`forceUpdate`发生，会发生多次

这个阶段的生命周期函数调用如下：

* getDerivedStateFromProps
* shouldComponentUpdate
* render
* getSnapshotBeforeUpdate
* componentDidUpdate

### getDerivedStateFromProps

这个方法在装载阶段已经讲过了，这里不再赘述，记住在更新阶段，无论我们接收到新的属性，调用了`setState`还是调用了`forceUpdate`，这个方法都会被调用。

### shouldComponentUpdate

```js
shouldComponentUpdate(nextProps, nextState)
```

有两个参数`nextProps`和`nextState`，表示新的属性和变化之后的`state`，返回一个布尔值，`true`表示会触发重新渲染，`false`表示不会触发重新渲染，默认返回`true`。

注意当我们调用`forceUpdate`并不会触发此方法。

因为默认是返回`true`，也就是只要接收到新的属性和调用了`setState`都会触发重新的渲染，这会带来一定的性能问题，所以我们需要将`this.props`与`nextProps`以及`this.state`与`nextState`进行比较来决定是否返回`false`，来减少重新渲染。

但是官方提倡我们使用`PureComponent`来减少重新渲染的次数而不是手工编写`shouldComponentUpdate`代码，具体该怎么选择，全凭开发者自己选择。

在未来的版本，`shouldComponentUpdate`返回`false`，仍然可能导致组件重新的渲染，这是官方自己说的

> In the future React may treat `shouldComponentUpdate()` as a hint rather than a strict directive, and returning false may still result in a re-rendering of the component.

### render

更新阶段也会触发，装载阶段已经讲过了，不再赘述。

### getSnapshotBeforeUpdate

```js
getSnapshotBeforeUpdate(prevProps, prevState)
```

这个方法在`render`之后，`componentDidUpdate`之前调用，有两个参数`prevProps`和`prevState`，表示之前的属性和之前的`state`，这个函数有一个返回值，会作为第三个参数传给`componentDidUpdate`，如果你不想要返回值，请返回`null`，不写的话控制台会有警告。

还有这个方法一定要和`componentDidUpdate`一起使用，否则控制台也会有警告。 

下面举个例子说明下：

```js
class ScrollingList extends React.Component {
  constructor(props) {
    super(props);
    this.listRef = React.createRef();
  }

  getSnapshotBeforeUpdate(prevProps, prevState) {
    // Are we adding new items to the list?
    // Capture the scroll position so we can adjust scroll later.
    if (prevProps.list.length < this.props.list.length) {
      const list = this.listRef.current;
      return list.scrollHeight - list.scrollTop;
    }
    return null;
  }

  componentDidUpdate(prevProps, prevState, snapshot) {
    // If we have a snapshot value, we've just added new items.
    // Adjust scroll so these new items don't push the old ones out of view.
    // (snapshot here is the value returned from getSnapshotBeforeUpdate)
    if (snapshot !== null) {
      const list = this.listRef.current;
      list.scrollTop = list.scrollHeight - snapshot;
    }
  }

  render() {
    return (
      <div ref={this.listRef}>{/* ...contents... */}</div>
    );
  }
}
```

### componentDidUpdate

```js
componentDidUpdate(prevProps, prevState, snapshot)
```

该方法在`getSnapshotBeforeUpdate`方法之后被调用，有三个参数`prevProps`，`prevState`，`snapshot`，表示之前的`props`，之前的`state`，第三个参数`snapshot`是`getSnapshotBeforeUpdate`的返回值。

在这个函数里我们可以操作DOM，和发起服务器请求，还可以`setState`，但是注意一定要用`if`语句控制，否则会导致无限循环。

## 卸载阶段

卸载阶段，当我们的组件被卸载或者销毁了

这个阶段的生命周期函数只有一个：

* componentWillUnmount

### componentWillUnmount

当我们的组件被卸载或者销毁了就会调用，我们可以在这个函数里去清除一些定时器，取消网络请求，清理无效的DOM元素等垃圾清理工作。

注意不要在这个函数里去调用`setState`，因为组件不会重新渲染了

## 最后

[查看 React v16.4 生命周期](https://link.juejin.im?target=http%3A%2F%2Fprojects.wojtekmaj.pl%2Freact-lifecycle-methods-diagram%2F)

