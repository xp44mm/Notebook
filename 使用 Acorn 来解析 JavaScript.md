# [使用 Acorn 来解析 JavaScript](https://segmentfault.com/a/1190000007473065)


## Talk

因为最近工作上有需要使用解析 JavaScript 的代码，大部分情况使用正则表达式匹配就可以处理，但是一旦依赖于代码上下文的内容时，正则或者简单的字符解析就很力不从心了，这个时候需要一个语言解析器来获取整一个 AST（abstract syntax tree）。

然后我找到了多个使用 JavaScript 编写的 JavaScript 解析器：

- [Esprima](https://link.segmentfault.com/?enc=t01XnuPkdtnaPBWG8p2amw%3D%3D.m%2FhCbQlvysvNEPLbORAgJYUvogV6jBOPUW28%2FVM7AGFVqAS8pp%2BVkLWWzuVLoBOD)
- [Acorn](https://link.segmentfault.com/?enc=gtHR0V5SY%2FH5DD7%2F%2BphJnQ%3D%3D.i1MK7fuEDbF9jPytQaEFrAQ%2BQs%2Fnb%2BvjzphlwXQq8kA%3D)
- [UglifyJS 2](https://link.segmentfault.com/?enc=U7sZWKFmyBChZKHBiF4tTA%3D%3D.TRxN%2Flu8XuwU9ymMZeutOx8lAJpI1lDzBqjtfbl8pIp3%2FV2xAosR7LHiT2EOfXnP)
- [Shift](https://link.segmentfault.com/?enc=tvpH6mzjqqI1w9fA%2B8dZww%3D%3D.WWhLb3mNYV9RH%2BhvBnBkNQs4oxwviGK%2BKvsHjJN6k43P6UOOegTJOlfCBb3k4wz17Q302kgp8t9bwnxCu%2FbC4A%3D%3D)

从提交记录来看，维护情况都蛮好的，ES 各种发展的特性都跟得上，我分别都简单了解了一下，聊聊他们的一些情况。

Esprima 是很经典的一个解析器，Acorn 在它之后诞生，都是几年前的事情了。按照 Acorn 作者的说法，当时造这个轮子更多只是好玩，速度可以和 Esprima 媲美，但是实现代码更少。其中比较关键的点是这两个解析器出来的 AST 结果（对，只是 AST，tokens 不一样）都是符合 [The Estree Spec](https://link.segmentfault.com/?enc=HBL8TbUOHDubduKq%2BD%2FPPA%3D%3D.wlS3f36JaF3VTsUKnhk4xWQ7X2XdexjrXeQ5JJ%2BieS3w%2BZnKMFlIozbzH1kVsQmC) 规范（这是 Mozilla 的工程师给出的 SpiderMonkey 引擎输出的 JavaScript AST 的规范文档，也可以参考：[SpiderMonkey in MDN](https://link.segmentfault.com/?enc=ABu7TzOjcJFV5%2FOrOrDYhw%3D%3D.sDXdU0LtzPxEHlVndRw%2FXMZh9wfbGGhUhUGQrDlbdMQmHF0QlaRJXikgmVRCjjyU%2FqtW0r8jEO7Ibwo49wmW2bt1uaPWQPjkDnE4Utkz9xhNDUaRD6NEveF32Wg5qDW2)）的，也就是得到的结果在很大部分上是兼容的。

现在很出名的 Webpack 解析代码时用的也是 Acorn。

至于 Uglify，很出名的一个 JavaScript 代码压缩器，其实它自带了一个代码解析器，也可以输出 AST，但是它的功能更多还是用于压缩代码，如果拿来解析代码感觉不够纯粹。

Shift 这个没做多少了解，只知道他定义了自己的一套 AST 规范。

Esprima 官网上有一个[性能测试](https://link.segmentfault.com/?enc=04WWNmw6T5Jh7%2BkkY%2Bk%2Fgw%3D%3D.38%2FvYU2InxXqWwfRRSIPy0f8glMPJ7Otk4NhzQ20W6EbIIcwulck0ueZty0JT%2F9X)，我在 chrome 上跑的结果如下：

<img src="http://ww1.sinaimg.cn/large/0... alt="性能测试" style="width:100%;">

可见，Acorn 的性能很不错，而且还有一个 Estree 的规范呢（规范很重要，我个人觉得遵循通用的规范是代码复用的重要基础），所以我就直接选用 Acorn 来做代码解析了。

> 图中做性能对比的还有 Google 的 Traceur，它更多是一个 ES6 to ES5 的 compiler，于我们想要找的解析器定位不符。

下面进入正题，如何使用 Acorn 来解析 JavaScript。

## API

解析器的 API 都是很简单的：

```js
const ast = acorn.parse(code, options)
```

Acorn 的配置项蛮多的，里边还包括了一些事件可以设置回调函数。我们挑几个比较重要的讲下：

- **ecmaVersion**
  字面意义，很好理解，就是设置你要解析的 JavaScript 的 ECMA 版本。默认是 ES7。
- **sourceType**
  这个配置项有两个值：`module` 和 `script`，默认是 `script`。

主要是严格模式和 `import/export` 的区别。ES6 中的模块是严格模式，也就是你无须添加 `use strict`。我们通常浏览器中使用的 script 是没有 `import/export` 语法的。
所以，选择了 `script` 则出现 `import/export` 会报错，可以使用严格模式声明，选择了 `module`，则不用严格模式声明，可以使用 `import/export` 语法。

- **locations**
  默认值是 `false`，设置为 `true` 之后会在 AST 的节点中携带多一个 `loc` 对象来表示当前的开始和结束的行数和列数。
- **onComment**
  传入一个回调函数，每当解析到代码中的注释时会触发，可以获取当年注释内容，参数列表是：`[block, text, start, end]`。

`block` 表示是否是块注释，`text` 是注释内容，`start` 和 `end` 是注释开始和结束的位置。

> 上边提及的 Espree 需要 Esprima 的 `attachComment` 的配置项，设置为 true 后，Esprima 会在代码解析结果的节点中携带注释相关信息（`trailingComments` 和 `leadingComments`）。Espree 则是利用 Acorn 的 `onComment` 配置来实现这个 Esprima 特性的兼容。

解析器通常还会有一个获取词法分析结果的接口：

```js
const tokens = [...acorn.tokenizer(code, options)]
```

`tokenizer` 方法的第二个参数也能够配置 `locations`。

词法结果 token 和 Esprima 的结果数据结构上有一定的区别（Espree 又是做了这一层的兼容），有兴趣了解的可以看下 Esprima 的解析结果：[http://esprima.org/demo/parse...](https://link.segmentfault.com/?enc=aymM5Wb1etwL00KLQBNiYA%3D%3D.d%2FsnGMw%2BHh2Q%2B1nBQeLPzw41nrgebPfABTfCqK3E1NeYi6HJ3wTmjvUhm%2BxG9g6t) 。

至于 Acorn 解析的 AST 和 token 的内容我们接下来详述。

## Token

我找了半天，没找到关于 token 数据结构的详细介绍，只能自己动手来看一下了。

我用来测试解析的代码是：

```
import "hello.js"

var a = 2;

// test
function name() { console.log(arguments); }
```

解析出来的 token 数组是一个个类似这样的对象：

```
Token {
    type:
     TokenType {
       label: 'import',
       keyword: 'import',
       beforeExpr: false,
       startsExpr: false,
       isLoop: false,
       isAssign: false,
       prefix: false,
       postfix: false,
       binop: null,
       updateContext: null },
    value: 'import',
    start: 5,
    end: 11 },
```

看上去其实很好理解对不对，在 `type` 对应的对象中，`label` 表示当前标识的一个类型，`keyword` 就是关键词，像例子中的 `import`，或者 `function` 之类的。

`value` 则是当前标识的值，`start/end` 分别是开始和结束的位置。

通常我们需要关注的就是 `label/keyword/value` 这些了。其他的详细可以参考源码：[tokentype.js](https://link.segmentfault.com/?enc=fYy1KRMe4m5PbA0kkPYAXg%3D%3D.IagOzUMcygjyvxiSsx8KDIPM1l24%2BiQiy7fzjYAvi5TOlVa5otSSGV1x2%2FFlabD0O6zLd3kAkk87AJ4AyDh%2FAA%3D%3D)。

## The Estree Spec

这一部分是重头戏，因为实际上我需要的还是解析出来的 AST。最原滋原味的内容来自于：[The Estree Spec](https://link.segmentfault.com/?enc=N0x3vqK%2BlXYsMZc9gFVZeQ%3D%3D.bUqSncSqRzYmYAcbxbTi%2BJUZqICQN1E5%2FAJcXPvB84h0Sq5GheAAJiAOFPtXuzTL)，我只是阅读了之后的搬运工。

提供了标准文档的好处是，很多东西有迹可循，这里还有一个工具，用于把满足 Estree 标准的 AST 转换为 ESMAScript 代码：[escodegen](https://link.segmentfault.com/?enc=6Jd54x4C3%2BPYPx3agQnfZw%3D%3D.f9IgKS%2Bh9XAEq622iIm38Ch%2FiaYOwyDlTl2dUbyDctacowtXgl8HUCvn3HNYoUDk)。

好吧，回到正题，我们先来看一下 ES5 的部分，可以在 [Esprima: Parser](https://link.segmentfault.com/?enc=Tfu4gHldB5EHDIHsFMjSXA%3D%3D.f7kv%2BmBTtZ1RqA3ITiGWeWWXM66we83qu5O4559l%2FH4gC7%2FBkxmKuuf%2FQm%2FOwnA3) 这个页面测试各种代码的解析结果。

符合这个规范的解析出来的 AST 节点用 `Node` 对象来标识，`Node` 对象应该符合这样的接口：

```
interface Node {
    type: string;
    loc: SourceLocation | null;
}
```

`type` 字段表示不同的节点类型，下边会再讲一下各个类型的情况，分别对应了 JavaScript 中的什么语法。
`loc` 字段表示源码的位置信息，如果没有相关信息的话为 `null`，否则是一个对象，包含了开始和结束的位置。接口如下：

```
interface SourceLocation {
    source: string | null;
    start: Position;
    end: Position;
}
```

这里的 `Position` 对象包含了行和列的信息，行从 1 开始，列从 0 开始：

```
interface Position {
    line: number; // >= 1
    column: number; // >= 0
}
```

好了，基础部分就是这样，接下来看各种类型的节点，顺带温习一下 JavaScript 语法的一些东西吧。对于这里每一部分的内容，会简单谈一下，但不会展开（内容不少），对 JavaScript 了解的人很容易就明白的。

> 我觉得看完就像把 JavaScript 的基础语法整理了一遍。

### Identifier

标识符，我觉得应该是这么叫的，就是我们写 JS 时自定义的名称，如变量名，函数名，属性名，都归为标识符。相应的接口是这样的：

```
interface Identifier <: Expression, Pattern {
    type: "Identifier";
    name: string;
}
```

一个标识符可能是一个表达式，或者是解构的模式（ES6 中的解构语法）。我们等会会看到 `Expression` 和 `Pattern` 相关的内容的。

### Literal

字面量，这里不是指 `[]` 或者 `{}` 这些，而是本身语义就代表了一个值的字面量，如 `1`，`“hello”`, `true` 这些，还有正则表达式（有一个扩展的 `Node` 来表示正则表达式），如 `/d?/`。我们看一下文档的定义：

```
interface Literal <: Expression {
    type: "Literal";
    value: string | boolean | null | number | RegExp;
}
```

`value` 这里即对应了字面量的值，我们可以看出字面量值的类型，字符串，布尔，数值，`null` 和正则。

#### RegExpLiteral

这个针对正则字面量的，为了更好地来解析正则表达式的内容，添加多一个 `regex` 字段，里边会包括正则本身，以及正则的 `flags`。

```
interface RegExpLiteral <: Literal {
  regex: {
    pattern: string;
    flags: string;
  };
}
```

### Programs

一般这个是作为根节点的，即代表了一棵完整的程序代码树。

```
interface Program <: Node {
    type: "Program";
    body: [ Statement ];
}
```

`body` 属性是一个数组，包含了多个 `Statement`（即语句）节点。

### Functions

函数声明或者函数表达式节点。

```
interface Function <: Node {
    id: Identifier | null;
    params: [ Pattern ];
    body: BlockStatement;
}
```

`id` 是函数名，`params` 属性是一个数组，表示函数的参数。`body` 是一个块语句。

有一个值得留意的点是，你在测试过程中，是不会找到 `type: "Function"` 的节点的，但是你可以找到 `type: "FunctionDeclaration"` 和 `type: "FunctionExpression"`，因为函数要么以声明语句出现，要么以函数表达式出现，都是节点类型的组合类型，后边会再提及 `FunctionDeclaration` 和 `FunctionExpression` 的相关内容。

这让人感觉这个文档规划得蛮细致的，函数名，参数和函数块是属于函数部分的内容，而声明或者表达式则有它自己需要的东西。

### Statement

语句节点没什么特别的，它只是一个节点，一种区分，但是语句有很多种，下边会详述。

```
interface Statement <: Node { }
```

#### ExpressionStatement

表达式语句节点，`a = a + 1` 或者 `a++` 里边会有一个 `expression` 属性指向一个表达式节点对象（后边会提及表达式）。

```
interface ExpressionStatement <: Statement {
    type: "ExpressionStatement";
    expression: Expression;
}
```

#### BlockStatement

块语句节点，举个例子：`if (...) { // 这里是块语句的内容 }`，块里边可以包含多个其他的语句，所以有一个 `body` 属性，是一个数组，表示了块里边的多个语句。

```
interface BlockStatement <: Statement {
    type: "BlockStatement";
    body: [ Statement ];
}
```

#### EmptyStatement

一个空的语句节点，没有执行任何有用的代码，例如一个单独的分号 `;`

```
interface EmptyStatement <: Statement {
    type: "EmptyStatement";
}
```

#### DebuggerStatement

`debugger`，就是表示这个，没有其他了。

```
interface DebuggerStatement <: Statement {
    type: "DebuggerStatement";
}
```

#### WithStatement

`with` 语句节点，里边有两个特别的属性，`object` 表示 `with` 要使用的那个对象（可以是一个表达式），`body` 则是对应 `with` 后边要执行的语句，一般会是一个块语句。

```
interface WithStatement <: Statement {
    type: "WithStatement";
    object: Expression;
    body: Statement;
}
```

------

下边是控制流的语句：

#### ReturnStatement

返回语句节点，`argument` 属性是一个表达式，代表返回的内容。

```
interface ReturnStatement <: Statement {
    type: "ReturnStatement";
    argument: Expression | null;
}
```

#### LabeledStatement

`label` 语句，平时可能会比较少接触到，举个例子：

```
loop: for(let i = 0; i < len; i++) {
    // ...
    for (let j = 0; j < min; j++) {
        // ...
        break loop;
    }
}
```

这里的 `loop` 就是一个 `label` 了，我们可以在循环嵌套中使用 `break loop` 来指定跳出哪个循环。所以这里的 `label` 语句指的就是 `loop: ...` 这个。

一个 `label` 语句节点会有两个属性，一个 `label` 属性表示 `label` 的名称，另外一个 `body` 属性指向对应的语句，通常是一个循环语句或者 `switch` 语句。

```
interface LabeledStatement <: Statement {
    type: "LabeledStatement";
    label: Identifier;
    body: Statement;
}
```

#### BreakStatement

`break` 语句节点，会有一个 `label` 属性表示需要的 `label` 名称，当不需要 `label` 的时候（通常都不需要），便是 `null`。

```
interface BreakStatement <: Statement {
    type: "BreakStatement";
    label: Identifier | null;
}
```

#### ContinueStatement

`continue` 语句节点，和 `break` 类似。

```
interface ContinueStatement <: Statement {
    type: "ContinueStatement";
    label: Identifier | null;
}
```

------

下边是条件语句：

#### IfStatement

`if` 语句节点，很常见，会带有三个属性，`test` 属性表示 `if (...)` 括号中的表达式。

`consequent` 属性是表示条件为 `true` 时的执行语句，通常会是一个块语句。

`alternate` 属性则是用来表示 `else` 后跟随的语句节点，通常也会是块语句，但也可以又是一个 `if` 语句节点，即类似这样的结构：
`if (a) { //... } else if (b) { // ... }`。
`alternate` 当然也可以为 `null`。

```
interface IfStatement <: Statement {
    type: "IfStatement";
    test: Expression;
    consequent: Statement;
    alternate: Statement | null;
}
```

#### SwitchStatement

`switch` 语句节点，有两个属性，`discriminant` 属性表示 `switch` 语句后紧随的表达式，通常会是一个变量，`cases` 属性是一个 `case` 节点的数组，用来表示各个 `case` 语句。

```
interface SwitchStatement <: Statement {
    type: "SwitchStatement";
    discriminant: Expression;
    cases: [ SwitchCase ];
}
```

##### SwitchCase

`switch` 的 `case` 节点。`test` 属性代表这个 `case` 的判断表达式，`consequent` 则是这个 `case` 的执行语句。

当 `test` 属性是 `null` 时，则是表示 `default` 这个 `case` 节点。

```
interface SwitchCase <: Node {
    type: "SwitchCase";
    test: Expression | null;
    consequent: [ Statement ];
}
```

------

下边是异常相关的语句：

#### ThrowStatement

`throw` 语句节点，`argument` 属性用以表示 `throw` 后边紧跟的表达式。

```
interface ThrowStatement <: Statement {
    type: "ThrowStatement";
    argument: Expression;
}
```

#### TryStatement

`try` 语句节点，`block` 属性表示 `try` 的执行语句，通常是一个块语句。

`hanlder` 属性是指 `catch` 节点，`finalizer` 是指 `finally` 语句节点，当 `hanlder` 为 `null` 时，`finalizer` 必须是一个块语句节点。

```
interface TryStatement <: Statement {
    type: "TryStatement";
    block: BlockStatement;
    handler: CatchClause | null;
    finalizer: BlockStatement | null;
}
```

##### CatchClause

`catch` 节点，`param` 用以表示 `catch` 后的参数，`body` 则表示 `catch` 后的执行语句，通常是一个块语句。

```
interface CatchClause <: Node {
    type: "CatchClause";
    param: Pattern;
    body: BlockStatement;
}
```

------

下边是循环语句：

#### WhileStatement

`while` 语句节点，`test` 表示括号中的表达式，`body` 是表示要循环执行的语句。

```
interface WhileStatement <: Statement {
    type: "WhileStatement";
    test: Expression;
    body: Statement;
}
```

#### DoWhileStatement

`do/while` 语句节点，和 `while` 语句类似。

```
interface DoWhileStatement <: Statement {
    type: "DoWhileStatement";
    body: Statement;
    test: Expression;
}
```

#### ForStatement

`for` 循环语句节点，属性 `init/test/update` 分别表示了 `for` 语句括号中的三个表达式，初始化值，循环判断条件，每次循环执行的变量更新语句（`init` 可以是变量声明或者表达式）。这三个属性都可以为 `null`，即 `for(;;){}`。
`body` 属性用以表示要循环执行的语句。

```
interface ForStatement <: Statement {
    type: "ForStatement";
    init: VariableDeclaration | Expression | null;
    test: Expression | null;
    update: Expression | null;
    body: Statement;
}
```

#### ForInStatement

`for/in` 语句节点，`left` 和 `right` 属性分别表示在 `in` 关键词左右的语句（左侧可以是一个变量声明或者表达式）。`body` 依旧是表示要循环执行的语句。

```
interface ForInStatement <: Statement {
    type: "ForInStatement";
    left: VariableDeclaration |  Pattern;
    right: Expression;
    body: Statement;
}
```

### Declarations

声明语句节点，同样也是语句，只是一个类型的细化。下边会介绍各种声明语句类型。

```
interface Declaration <: Statement { }
```

#### FunctionDeclaration

函数声明，和之前提到的 Function 不同的是，`id` 不能为 `null`。

```
interface FunctionDeclaration <: Function, Declaration {
    type: "FunctionDeclaration";
    id: Identifier;
}
```

#### VariableDeclaration

变量声明，`kind` 属性表示是什么类型的声明，因为 ES6 引入了 `const/let`。
`declarations` 表示声明的多个描述，因为我们可以这样：`let a = 1, b = 2;`。

```
interface VariableDeclaration <: Declaration {
    type: "VariableDeclaration";
    declarations: [ VariableDeclarator ];
    kind: "var";
}
```

##### VariableDeclarator

变量声明的描述，`id` 表示变量名称节点，`init` 表示初始值的表达式，可以为 `null`。

```
interface VariableDeclarator <: Node {
    type: "VariableDeclarator";
    id: Pattern;
    init: Expression | null;
}
```

### Expressions

表达式节点。

```
interface Expression <: Node { }
```

#### ThisExpression

表示 `this`。

```
interface ThisExpression <: Expression {
    type: "ThisExpression";
}
```

#### ArrayExpression

数组表达式节点，`elements` 属性是一个数组，表示数组的多个元素，每一个元素都是一个表达式节点。

```
interface ArrayExpression <: Expression {
    type: "ArrayExpression";
    elements: [ Expression | null ];
}
```

#### ObjectExpression

对象表达式节点，`property` 属性是一个数组，表示对象的每一个键值对，每一个元素都是一个属性节点。

```
interface ObjectExpression <: Expression {
    type: "ObjectExpression";
    properties: [ Property ];
}
```

##### Property

对象表达式中的属性节点。`key` 表示键，`value` 表示值，由于 ES5 语法中有 `get/set` 的存在，所以有一个 `kind` 属性，用来表示是普通的初始化，或者是 `get/set`。

```
interface Property <: Node {
    type: "Property";
    key: Literal | Identifier;
    value: Expression;
    kind: "init" | "get" | "set";
}
```

#### FunctionExpression

函数表达式节点。

```
interface FunctionExpression <: Function, Expression {
    type: "FunctionExpression";
}
```

------

下边是一元运算符相关的表达式部分：

#### UnaryExpression

一元运算表达式节点（`++/--` 是 update 运算符，不在这个范畴内），`operator` 表示运算符，`prefix` 表示是否为前缀运算符。`argument` 是要执行运算的表达式。

```
interface UnaryExpression <: Expression {
    type: "UnaryExpression";
    operator: UnaryOperator;
    prefix: boolean;
    argument: Expression;
}
```

##### UnaryOperator

一元运算符，枚举类型，所有值如下：

```
enum UnaryOperator {
    "-" | "+" | "!" | "~" | "typeof" | "void" | "delete"
}
```

#### UpdateExpression

update 运算表达式节点，即 `++/--`，和一元运算符类似，只是 `operator` 指向的节点对象类型不同，这里是 update 运算符。

```
interface UpdateExpression <: Expression {
    type: "UpdateExpression";
    operator: UpdateOperator;
    argument: Expression;
    prefix: boolean;
}
```

##### UpdateOperator

update 运算符，值为 `++` 或 `--`，配合 update 表达式节点的 `prefix` 属性来表示前后。

```
enum UpdateOperator {
    "++" | "--"
}
```

------

下边是二元运算符相关的表达式部分：

#### BinaryExpression

二元运算表达式节点，`left` 和 `right` 表示运算符左右的两个表达式，`operator` 表示一个二元运算符。

```
interface BinaryExpression <: Expression {
    type: "BinaryExpression";
    operator: BinaryOperator;
    left: Expression;
    right: Expression;
}
```

##### BinaryOperator

二元运算符，所有值如下：

```
enum BinaryOperator {
    "==" | "!=" | "===" | "!=="
         | "<" | "<=" | ">" | ">="
         | "<<" | ">>" | ">>>"
         | "+" | "-" | "*" | "/" | "%"
         | "|" | "^" | "&" | "in"
         | "instanceof"
}
```

#### AssignmentExpression

赋值表达式节点，`operator` 属性表示一个赋值运算符，`left` 和 `right` 是赋值运算符左右的表达式。

```
interface AssignmentExpression <: Expression {
    type: "AssignmentExpression";
    operator: AssignmentOperator;
    left: Pattern | Expression;
    right: Expression;
}
```

##### AssignmentOperator

赋值运算符，所有值如下：（常用的并不多）

```
enum AssignmentOperator {
    "=" | "+=" | "-=" | "*=" | "/=" | "%="
        | "<<=" | ">>=" | ">>>="
        | "|=" | "^=" | "&="
}
```

#### LogicalExpression

逻辑运算表达式节点，和赋值或者二元运算类型，只不过 `operator` 是逻辑运算符类型。

```
interface LogicalExpression <: Expression {
    type: "LogicalExpression";
    operator: LogicalOperator;
    left: Expression;
    right: Expression;
}
```

##### LogicalOperator

逻辑运算符，两种值，即与或。

```
enum LogicalOperator {
    "||" | "&&"
}
```

#### MemberExpression

成员表达式节点，即表示引用对象成员的语句，`object` 是引用对象的表达式节点，`property` 是表示属性名称，`computed` 如果为 `false`，是表示 `.` 来引用成员，`property` 应该为一个 `Identifier` 节点，如果 `computed` 属性为 `true`，则是 `[]` 来进行引用，即 `property` 是一个 `Expression` 节点，名称是表达式的结果值。

```
interface MemberExpression <: Expression, Pattern {
    type: "MemberExpression";
    object: Expression;
    property: Expression;
    computed: boolean;
}
```

------

下边是其他的一些表达式：

#### ConditionalExpression

条件表达式，通常我们称之为三元运算表达式，即 `boolean ? true : false`。属性参考条件语句。

```
interface ConditionalExpression <: Expression {
    type: "ConditionalExpression";
    test: Expression;
    alternate: Expression;
    consequent: Expression;
}
```

#### CallExpression

函数调用表达式，即表示了 `func(1, 2)` 这一类型的语句。`callee` 属性是一个表达式节点，表示函数，`arguments` 是一个数组，元素是表达式节点，表示函数参数列表。

```ts
interface CallExpression <: Expression {
    type: "CallExpression";
    callee: Expression;
    arguments: [ Expression ];
}
```

#### NewExpression

`new` 表达式。

```ts
interface NewExpression <: CallExpression {
    type: "NewExpression";
}
```

#### SequenceExpression

这个就是逗号运算符构建的表达式（不知道确切的名称），`expressions` 属性为一个数组，即表示构成整个表达式，被逗号分割的多个表达式。

```
interface SequenceExpression <: Expression {
    type: "SequenceExpression";
    expressions: [ Expression ];
}
```

### Patterns

模式，主要在 ES6 的解构赋值中有意义，在 ES5 中，可以理解为和 `Identifier` 差不多的东西。

```
interface Pattern <: Node { }
```

这一部分的内容比较多，但都可以举一反三，写这个的时候我就当把 JavaScript 语法再复习一遍。这个文档还有 ES2015，ES2016，ES2017 相关的内容，涉及的东西也蛮多，但是理解了上边的这一些，然后从语法层面去思考这个文档，其他的内容也就很好理解了，这里略去，有需要请参阅：[The Estree Spec](https://link.segmentfault.com/?enc=FarLPsqaIgUVILl6nETv8A%3D%3D.lMAKEx%2F0m4ssIwyoQX2NK%2Fn%2BsjNy0%2FlTOUgds0AsBZgLifkLff5WNRiMsBhwmk2c)。

## Plugins

回到我们的主角，Acorn，提供了一种扩展的方式来编写相关的插件：[Acorn Plugins](https://link.segmentfault.com/?enc=D8I%2F%2FyYiGZtyPzOgc%2FY5AQ%3D%3D.A6x08fr0rTTqvynfcDCCOUkocRz2979heDFgzzNap0leV0A9ZvCIZL9JElqwCZ%2F9)。

我们可以使用插件来扩展解析器，来解析更多的一些语法，如 `.jsx` 语法，有兴趣的看看这个插件：[acorn-jsx](https://link.segmentfault.com/?enc=b4jnAJtEDxOl55f%2FLSZWbQ%3D%3D.nghBjmO0%2BW%2B1Xd7EiJ1dijghBMdmJ5NC7PXTVpdUNiE9FW6YSX4OZmAOhuAEQDlS)。

官方表示 Acorn 的插件是用于方便扩展解析器，但是需要对 Acorn 内部的运行极致比较了解，扩展的方式会在原本的基础上重新定义一些方法。这里不展开讲了，如果我需要插件的话，会再写文章聊聊这个东西。

## Examples

现在我们来看一下如何应用这个解析器，例如我们需要用来解析出一个符合 CommonJS 规范的模块依赖了哪些模块，我们可以用 Acorn 来解析 `require` 这个函数的调用，然后取出调用时的传入参数，便可以获取依赖的模块。

下边是示例代码：

```js
// 遍历所有节点的函数
function walkNode(node, callback) {
  callback(node)

  // 有 type 字段的我们认为是一个节点
  Object.keys(node).forEach((key) => {
    const item = node[key]
    if (Array.isArray(item)) {
      item.forEach((sub) => {
        sub.type && walkNode(sub, callback)
      })
    }

    item && item.type && walkNode(item, callback)
  })
}

function parseDependencies(str) {
  const ast = acorn.parse(str, { ranges: true })
  const resource = [] // 依赖列表

  // 从根节点开始
  walkNode(ast, (node) => {
    const callee = node.callee
    const args = node.arguments

    // require 我们认为是一个函数调用，并且函数名为 require，参数只有一个，且必须是字面量
    if (
      node.type === 'CallExpression' &&
      callee.type === 'Identifier' &&
      callee.name === 'require' &&
      args.length === 1 &&
      args[0].type === 'Literal'
    ) {
      const args = node.arguments

      // 获取依赖的相关信息
      resource.push({
        string: str.substring(node.range[0], node.range[1]),
        path: args[0].value,
        start: node.range[0],
        end: node.range[1]
      })
    }
  })

  return resource
}
```

这只是简单的一个情况的处理，但是已经给我们呈现了如何使用解析器，Webpack 则在这个的基础上做了更多的东西，包括 `var r = require; r('a')` 或者 `require.async('a')` 等的处理。

AST 这个东西对于前端来说，我们无时无刻不在享受着它带来的成果（模块构建，代码压缩，代码混淆），所以了解一下总归有好处。

有问题欢迎讨论。