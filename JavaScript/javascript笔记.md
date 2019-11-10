ClearScript is a library that makes it easy to add scripting to your .NET applications. It currently supports JavaScript (via V8 and JScript) and VBScript.

Chrome控制台,查看JavaScript类型:

```javascript
console.dir(Promise)
console.dir(Array)
console.log(Array)
```

# 对象(Object)

### 原型(prototype)

每个JavaScript对象都有第二个对象（或null，但罕见）与其相关.这第二个对象被称为**原型**，第一个对象自原型**继承(inherits)**属性。

对象字面量的原型都为`Object.prototype`

由关键字`new`调用构造函数创造的对象,其原型为构造函数的原型属性:

```javascript
let o = new Object();
o.prototype === Object.prototype //Object是构造函数
let a = new Array();
a.prototype === Array.prototype//Array是构造函数
let d = new Date();
d.prototype === Date.prototype//Date是构造函数
```

大部分内建构造函数的原型继承自`Object.prototype`,如:

```javascript
Date.prototype.prototype === Object.prototype
```

引出定义原型链(prototype chain)

### Object.create()
是es5的方法
以其第一个参数为原型创建一个新对象
```javascript
let o3 = Object.create(Object.prototype); // same {} or new Object().
```
## 查询和设置属性
查询属性通过`.`或者`[]`来,运算符左手边是一个返回对象的表达式,点号右手边是简单标识符表示属性名称.方括号内是一个返回字符串的表达式.
```javascript
let author = book.author
let name = author.surname
let title = book["main title"]
```
创建或修改属性值:
```javascript
book.edition = 6
book["main title"] = "ECMAScript"
```
### 对象作为关联数组
下面两个表达式等价:
```javascript
object.property === object["property"]
```
像第二个表达式用字符串作为索引的数组叫**关联数组**
在编程时并不知道属性名称,这时就要用关联数组写法.
标识符是静态的,字符串是动态的.
### 继承

自有属性

从他们的原型对象继承属性

查询属性时:

```javascript
function read(o.x) {
    if (o.hasOwnProperty('x')) {//存在自有属性,就读取这个自有属性
        return o.x
    }
    else if (o.prototype) {//不存在自有属性,但存在原型
        read o.prototype.x
    }
    else {
        throw error
    }
}
```

这就是属性继承链. 写属性时,伪代码过程:

```javascript
function write(o.x = v) {
    if (o.hasOwnProperty('x')) {//存在自有属性,就修改这个自有属性
        return (o.x = v)
    } else {//不存在自有属性,新增一个自有属性
        o.add(x) 
        return (o.x = v)
    }
}
```

 If `o` previously inherited the property `x`, that inherited property is now hidden by the newly created own property with the same name.

The fact that inheritance occurs when querying properties but not when setting them is a key feature of JavaScript because it allows us to selectively override inherited properties:

## 测试属性

in

```javascript
var o = { x: 1 }
"x" in o;         // true
"y" in o;         // false
"toString" in o;  // true
```

hasOwnProperty()
```javascript
var o = { x: 1 }
o.hasOwnProperty("x");        // true
o.hasOwnProperty("y");        // false
o.hasOwnProperty("toString"); // false
```
propertyIsEnumerable() 
```javascript
o.propertyIsEnumerable("x") === o.hasOwnProperty("x") &&  its enumerable attribute is true
```
## 枚举属性

for/in循环
```javascript
for(p in o) {
    if (!o.hasOwnProperty(p)) continue;
}
for(p in o) {
    if (typeof o[p] === "function") continue;
}
```
for/of循环

Object.keys()

Object.getOwnPropertyNames()

## Getter和Setter

访问属性(accessor properties) 区分于数据属性
```javascript
let p = {

   x: 1.0,
   y: 1.0,

   get r() { return Math.sqrt(this.x*this.x + this.y*this.y); },
   set r(newvalue) {
     let oldvalue = this.r
     let ratio = newvalue/oldvalue;
     this.x *= ratio;
     this.y *= ratio;
   },

   get theta() { return Math.atan2(this.y, this.x); }
};
```
注意`this`是指`p`,定义他们的对象.

## prototype属性
Object.getPrototypeOf()
constructor属性保存用于创建对象的构造函数
isPrototypeOf()
instanceof操作符
`__proto__`

## class属性
对象的类属性是一个字符串,表示对象的类型信息.此属性不能设置只能查询.且是间接查询.
## 序列化对象
JSON.stringify
JSON.parse
## Object.assign()

# 数组

# 函数

### 定义自己的函数属性

```javascript
// Compute factorials and cache results as properties of the function itself.
function factorial(n) {
    if (isFinite(n) && n>0 && n==Math.round(n)) { // Finite, positive ints only
        if (!(n in factorial))                    // If no cached result
            factorial[n] = n * factorial(n-1);    // Compute and cache it
        return factorial[n];                      // Return the cached result
    }
    else return NaN;                              // If input was bad
}
factorial[1] = 1;  // Initialize the cache to hold this base case.
```

## 闭包

### call()和apply()

``` javascript
f.call(o) === o.f()
f.apply(o) === o.f()
```
他们的区别在于参数表:
``` javascript
f.call(o,1,2) === o.f(1,2)
f.apply(o,[1,2]) === o.f(1,2)
```
另外一个典型用法是`apply()`结合`arguments`:
``` javascript
function trace(o, m) {
    var original = o[m];
    o[m] = function() {
        console.log(new Date(), "Entering:", m)
        var result = original.apply(this, arguments)
        console.log(new Date(), "Exiting:", m)
        return result
    };
}        
```
### bind()

# 类

In JavaScript, classes are based on JavaScript's prototype-based inheritance mechanism. If two objects inherit properties from the same prototype object, then we say that they are instances of the same class.

## 构造函数

The critical feature of constructor invocations is that the prototype property of the constructor is used as the prototype of the new object. This means that all objects created with the same constructor inherit from the same object and are therefore members of the same class.

```javascript
function Range(from, to) {
    this.from = from;
    this.to = to;
}

Object.assign(Range.prototype, {
    includes: function(x) { return this.from <= x && x <= this.to; },
    foreach: function(f) {
        for (var x = Math.ceil(this.from); x <= this.to; x++) f(x);
    },
    toString: function() { return "(" + this.from + "..." + this.to + ")"; }
});  
```

This is a very common coding convention: constructor functions define, in a sense, classes, and classes have names that begin with capital letters. Regular functions and methods have names that begin with lowercase letters.

An invocation of the `Range()` constructor automatically uses Range.prototype as the prototype of the new Range object.

### 构造函数是类的标识

Constructors are used with the `instanceof` operator when testing objects for membership in a class. If we have an object r and want to know if it is a Range object, we can write:

```javascript
let r = new Range(1,100)
r instanceof Range
```

The `instanceof` operator does not actually check whether `r` was initialized by the `Range` constructor. It checks whether it inherits from `Range.prototype`. Nevertheless, the instanceof syntax reinforces **the use of constructors as the public identity of a class**.

### constructor属性

constructor属性是原型对象对构造函数的逆向指针.

```javascript
Range.prototype.constructor === Range
```

直接使用`prototype`并不可靠,有时可能要使用`__proto__`属性.应该逐个测试.

## 类继承

**Constructor object**

A constructor function (an object) defines a name for a JavaScript class. Properties you add to this constructor object serve as **class fields** and **class methods** (depending on whether the property values are functions or not). 

**Prototype object** 

The properties of this object are inherited by all instances of the class, and properties whose values are functions behave like **instance methods** of the class. 

**Instance object** 

Each instance of a class is an object in its own right, and properties defined directly on an instance are not shared by any other instances. Nonfunction properties defined on instances behave as the **instance fields** of the class.

```javascript
//constructor
function Complex(real, imaginary) {
    if (isNaN(real) || isNaN(imaginary))
        throw new TypeError();
    this.r = real;
    this.i = imaginary;
}

//instance methods
Object.assign(Complex.prototype, {
    add: function (that) {
        return new Complex(this.r + that.r, this.i + that.i);
    },
    mul: function (that) {
        return new Complex(this.r * that.r - this.i * that.i,
            this.r * that.i + this.i * that.r);
    },
    mag: function () {
        return Math.sqrt(this.r * this.r + this.i * this.i);
    },  
    neg: function () { return new Complex(-this.r, -this.i); },
  
    toString: function () {
        return "{" + this.r + "," + this.i + "}";
    },
    equals: function (that) {
        return that != null &&
            that.constructor === Complex &&
            this.r === that.r && this.i === that.i;
    },
})

//class member(static member)
Object.assign(Complex, {
    ZERO: new Complex(0, 0),
    ONE: new Complex(1, 0),
    I: new Complex(0, 1),

    parse: function (s) {
        try {
            var m = Complex._format.exec(s);
            return new Complex(parseFloat(m[1]), parseFloat(m[2]));
        } catch (x) {
            throw new TypeError("Can't parse '" + s + "' as a complex number.");
        }
    },

    _format: /^\{([^,]+),([^}]+)\}$/,
})
```

## 类型

The built-in objects of core JavaScript (and often the host objects of client-side JavaScript) can be distinguished on the basis of their `class` attribute

### instanceof

The left-hand operand should be the object whose class is being tested, and the right-hand operand should be a constructor function that names a class. The expression `o instanceof c` evaluates to true if `o` inherits from `c.prototype`. The inheritance need not be direct. If `o` inherits from an object that inherits from an object that inherits from `c.prototype`, the expression will still evaluate to true.

Constructors act as the public identity of classes, but prototypes are the fundamental identity. Despite the use of a constructor function with `instanceof`, this operator is really testing what an object inherits from, not what constructor was used to create it.

If you want to test the prototype chain of an object for a specific prototype object and do not want to use the constructor function as an intermediary, you can use the `isPrototypeOf()` method.
```javascript
Range.prototype.isPrototypeOf(r) //true
```
### constructor

JavaScript does not require that every object have a `constructor` property: this is a convention based on the default `prototype` object created for each function. 

不要替换默认`prototype`,而向其添加新成员扩展它.

### 9.6.2  Example: Enumerated Types
枚举类型定义:
```javascript
function enumeration(namesToValues) {
    var enu = function () { throw "Can't Instantiate Enumerations"; };

    Object.assign(enu.prototype, {
        toString: function () { return this.name; },
        valueOf: function () { return this.value; },
        toJSON: function () { return this.name; },
    });

    enu.values = Object.keys(namesToValues).map((name) => {
        const e = Object.create(enu.prototype);
        e.name = name;
        e.value = namesToValues[name];
        return e
    })

    enu.values.forEach((e) => enu[name] = e)

    enu.foreach = function (callback, c) {
        this.values.forEach((value) => callback.call(c, value))
    };

    return enu;
}

```
使用:
```javascript
function Card(suit, rank) {
    this.suit = suit;
    this.rank = rank;
}

Card.Suit = enumeration({ Clubs: 1, Diamonds: 2, Hearts: 3, Spades: 4 });

Card.Rank = enumeration({
    Two: 2, Three: 3, Four: 4, Five: 5, Six: 6,
    Seven: 7, Eight: 8, Nine: 9, Ten: 10,
    Jack: 11, Queen: 12, King: 13, Ace: 14
});

let cards =
        Array.prototype.concat.apply(
            Card.Suit.values.map(
                (suit) => Card.Rank.values.map(
                    (rank) => new Card(suit, rank)
                )
            )
        )
```
`.valueOf()`Its job is to convert an object to a **primitive value**.

`toJSON()` is invoked automatically by `JSON.stringify()`. JSON会简化掉javascript数据对象的原型继承层次结构.`toJSON()`返回另一个对象,

If an object has a `toJSON()` method, `JSON.stringify()` does not serialize the object but instead calls `toJSON()` and serializes the value (either primitive or object) that it returns.

```javascript
function Set() {          // This is the constructor
    this.values = {};
    this.n = 0;
    this.add.apply(this, arguments);
}

// Add each of the arguments to the set.
Set.prototype.add = function () {
    Array.from(arguments).forEach((val) => {
        var str = Set._v2s(val);
        if (!this.values.hasOwnProperty(str)) {
            this.values[str] = val;
            this.n++;
        }
    })
    return this;
};

// Remove each of the arguments from the set.
Set.prototype.remove = function () {
    Array.from(arguments).forEach((val, i) => {
        var str = Set._v2s(val);
        if (this.values.hasOwnProperty(str)) {
            delete this.values[str];
            this.n--;
        }
    })
    return this;
};

// Return true if the set contains value; false otherwise.
Set.prototype.contains = function (value) {
    return this.values.hasOwnProperty(Set._v2s(value));
};

// Return the size of the set.
Set.prototype.size = function () { return this.n; };

// Call function f on the specified context for each element of the set.
Set.prototype.foreach = function (f, context) {
    this.values.keys().forEach((s) => f.call(context, this.values[s]))
};

// This internal function maps any JavaScript value to a unique string.
Set._v2s = function (val) {
    switch (val) {
        case undefined: return 'u';
        case null: return 'n';
        case true: return 't';
        case false: return 'f';
        default: switch (typeof val) {
            case 'number': return '#' + val;
            case 'string': return '"' + val;
            default: return '@' + objectId(val);
        }
    }

    function objectId(o) {
        var prop = "|**objectid**|";
        if (!o.hasOwnProperty(prop))
            o[prop] = Set._v2s.next++;
        return o[prop];
    }
};

Set._v2s.next = 100;

// Add these methods to the Set prototype object.
Object.assign(Set.prototype, {
    // Convert a set to a string
    toString: function () {
        return '{' + Array.prototype.join(this.toArray(), ",") + '}'
    },

    // Like toString, but call toLocaleString on all values
    toLocaleString: function () {
        let arr = Array.prototype.map.call(this.toArray(), (v) => v.toLocaleString())
        return '{' + Array.prototype.join(arr, ",") + '}'
    },

    // Convert a set to an array of values
    toArray: function () {
        return Object.getOwnPropertyNames(this.values).map((nm) => this.values[nm])
    }
});

// Treat sets like arrays for the purposes of JSON stringification.
Set.prototype.toJSON = Set.prototype.toArray;

Set.prototype.equals = function (that) {
    if (this === that) return true;

    if (!(that instanceof Set)) return false;
    if (this.size() != that.size()) return false;

    // Now check whether every element in this is also in that.
    // Use an exception to break out of the foreach if the sets are not equal.

    try {
        this.foreach(function (v) { if (!that.contains(v)) throw false; });
        return true;                    // All elements matched: sets are equal.
    } catch (x) {
        if (x === false) return false;  // An element in this is not in that.
        throw x;                        // Some other exception: rethrow it.
    }
};
```

# 16. 模块

- 导出`const`，尽量不可变。不要导出可变量`let`、`var`。
- 不要将循环依赖分离到两个文件（模块）中。
- 不要编写只在被空导入时执行一次的模块，应该导出一个执行函数供其他部分显式调用。
- 不要使用默认导出，即使只导出一个成员（函数、类、对象）。使用命名导出。
- 再导出仅用于文件夹下的`index.js`文件。`index.js`把文件夹下的其他的js文件导入进来，作为了唯一的对外的接口。


导入一个文件目录作为 Node 模块的情况。如果文件目录下有 `package.json`，就根据它的 main 字段找到 js 文件。如果没有 package.json，那就取文件夹下的`index.js`。

## 16.4 Importing and exporting in detail

### 16.4.1 Importing styles

ECMAScript 6 provides several styles of importing:

Default import:

```javascript
import localName from 'src/my_lib';
```

Namespace import: imports the module as an object (with one property per named export).

```javascript
import * as my_lib from 'src/my_lib';
```

Named imports:

```javascript
import { name1, name2 } from 'src/my_lib';
```

You can rename named imports:

```javascript
// Renaming: import `name1` as `localName1`
import { name1 as localName1, name2 } from 'src/my_lib';

// Renaming: import the default export as `foo`
import { default as foo } from 'src/my_lib';
```

Empty import: only loads the module, doesn't import anything. The first such import in a program executes the body of the module.

```javascript
import 'src/my_lib';
```

There are only two ways to combine these styles and the order in which they appear is fixed; the default export always comes first.

Combining a default import with a namespace import:

```javascript
  import theDefault, * as my_lib from 'src/my_lib';
```

Combining a default import with named imports

```javascript
import theDefault, { name1, name2 } from 'src/my_lib';
```

### 16.4.2 Named exporting styles: inline versus clause

There are two ways in which you can export named things inside modules.

On one hand, you can mark declarations with the keyword `export`.

```javascript
export var myVar1 = ···;
export let myVar2 = ···;
export const MY_CONST = ···;

export function myFunc() {
    ···
}
export function* myGeneratorFunc() {
    ···
}
export class MyClass {
    ···
}
```

On the other hand, you can list everything you want to export at the end of the module (which is similar in style to the revealing module pattern).

```javascript
const MY_CONST = ···;
function myFunc() {
    ···
}

export { MY_CONST, myFunc };
```

You can also export things under different names:

```javascript
export { MY_CONST as FOO, myFunc };
```

### 16.4.3 Re-exporting

Re-exporting means adding another module's exports to those of the current module. You can either add all of the other module's exports:

```javascript
export * from 'src/other_module';
```

Default exports are ignored by `export *`.

Or you can be more selective (optionally while renaming):

```javascript
export { foo, bar } from 'src/other_module';

// Renaming: export other_module's foo as myFoo
export { foo as myFoo, bar } from 'src/other_module';
```

#### 16.4.3.1 Making a re-export the default export

The following statement makes the default export of another module foo the default export of the current module:

```javascript
export { default } from 'foo';
```

The following statement makes the named export `myFunc` of module `foo` the default export of the current module:

```javascript
export { myFunc as default } from 'foo';
```

### 16.4.4 All exporting styles

ECMAScript 6 provides several styles of exporting:

Re-exporting:

Re-export everything (except for the default export):

```javascript
export * from 'src/other_module';
```

Re-export via a clause:

```javascript
export { foo as myFoo, bar } from 'src/other_module';

export { default } from 'src/other_module';
export { default as foo } from 'src/other_module';
export { foo as default } from 'src/other_module';
```

Named exporting via a clause:

```javascript
export { MY_CONST as FOO, myFunc };
export { foo as default };
```

Inline named exports:

Variable declarations:

```javascript
export var foo;
export let foo;
export const foo;
```

Function declarations:

```javascript
export function myFunc() {}
export function* myGenFunc() {}
```

Class declarations:

```javascript
export class MyClass {}
```

Default export:

Function declarations (can be anonymous here):

```javascript
export default function myFunc() {}
export default function () {}

export default function* myGenFunc() {}
export default function* () {}
```

Class declarations (can be anonymous here):

```javascript
  export default class MyClass {}
  export default class {}
```

Expressions: export values. Note the semicolons at the end.

```javascript
  export default foo;
  export default 'Hello world!';
  export default 3 * 7;
  export default (function () {});
```

### 16.4.5 Having both named exports and a default export in a module

The following pattern is surprisingly common in JavaScript: A library is a single function, but additional services are provided via properties of that function. Examples include jQuery and Underscore.js. The following is a sketch of Underscore as a CommonJS module:

```javascript
//------ underscore.js ------
var _ = function (obj) {
    ···
};
var each = _.each = _.forEach =
    function (obj, iterator, context) {
        ···
    };
module.exports = _;

//------ main.js ------
var _ = require('underscore');
var each = _.each;
···
```

With ES6 glasses, the function `_` is the default export, while `each` and `forEach` are named exports. As it turns out, you can actually have named exports and a default export at the same time. As an example, the previous CommonJS module, rewritten as an ES6 module, looks like this:

```javascript
//------ underscore.js ------
export default function (obj) {
    ···
}
export function each(obj, iterator, context) {
    ···
}
export { each as forEach };

//------ main.js ------
import _, { each } from 'underscore';
···
```

Note that the CommonJS version and the ECMAScript 6 version are only roughly similar. The latter has a flat structure, whereas the former is nested.

#### 16.4.5.1 Recommendation: avoid mixing default exports and named exports

I generally recommend to keep the two kinds of exporting separate: per module, either only have a default export or only have named exports.

However, that is not a very strong recommendation; it occasionally may make sense to mix the two kinds. One example is a module that default-exports an entity. For unit tests, one could additionally make some of the internals available via named exports.

#### 16.4.5.2 The default export is just another named export

The default export is actually just a named export with the special name default. That is, the following two statements are equivalent:

```javascript
import { default as foo } from 'lib';
import foo from 'lib';
```

Similarly, the following two modules have the same default export:

```javascript
//------ module1.js ------
export default function foo() {} // function declaration!

//------ module2.js ------
function foo() {}
export { foo as default };
```

#### 16.4.5.3 default: OK as export name, but not as variable name

You can't use reserved words (such as `default` and `new`) as variable names, but you can use them as names for exports (you can also use them as property names in ECMAScript 5). If you want to directly import such named exports, you have to rename them to proper variables names.

That means that default can only appear on the left-hand side of a renaming import:

```javascript
import { default as foo } from 'some_module';
```

And it can only appear on the right-hand side of a renaming export:

```javascript
export { foo as default };
```

In re-exporting, both sides of the as are export names:

```javascript
export { myFunc as default } from 'foo';
export { default as otherFunc } from 'foo';

// The following two statements are equivalent:
export { default } from 'foo';
export { default as default } from 'foo';
```