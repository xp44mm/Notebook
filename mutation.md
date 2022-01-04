# mutation

One pattern many people follow is to be liberal with mutation when constructing data, but conservative with mutation when consuming data.

## let

`let` permits us to rebind variables.

```js
let age = 52;
age = 53;
age
//=> 53
```

## const

Declaring a variable `const` does not prevent us from mutating its value, only prevent us from rebinding its name. This is an important distinction.

```js
const ls = []
ls = [1] // not allowed
ls[0] = 1 // allowed
```

## immutable

js没有immutable变量，F#有，它不仅不允许rebinding, 也不允许修改它的值。

js模块跨文件的`exprot`和`import`都应该是immutable变量。但它不是，我们假设它是的，从来不会修改它，只是会读取它，执行它。
