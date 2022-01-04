# IEnumerator vs Iterator

## IEnumerator

不考虑`Reset`方法和原始列表的可变性，`NoveNext`和`Current`的只读向前用法：

Initially, the enumerator is positioned before the first element in the collection. You must call the `MoveNext` method to advance the enumerator to the first element of the collection before reading the value of `Current`; otherwise, `Current` is undefined.

`Current` returns the same object until `MoveNext` is called. `MoveNext` sets `Current` to the next element.

If `MoveNext` passes the end of the collection, the enumerator is positioned after the last element in the collection and `MoveNext` returns false. When the enumerator is at this position, subsequent calls to `MoveNext` also return false. If the last call to `MoveNext` returned false, `Current` is undefined.

输入数据，并初始化枚举器：

```F#
let ls = [0;1;2]
let enumerator = (Seq.ofList ls).GetEnumerator()
let ls = []
```

此时调用`Current`会抛出异常，印证第一段描述：

```F#
enumerator.Current
//InvalidOperationException : 枚举尚未开始。请调用 MoveNext。
```

我们改正代码，连续运行，直到序列的末尾：

```F#
let ls = [0;1;2]
let enumerator = (Seq.ofList ls).GetEnumerator()
show <| enumerator.MoveNext() // true
show enumerator.Current // 0

show <| enumerator.MoveNext() // true
show enumerator.Current // 1

show <| enumerator.MoveNext() // true
show enumerator.Current // 2
```

当再次调用`MoveNext()`返回false，接上段代码：

```F#
show <| enumerator.MoveNext() // false
```

当`MoveNext()`返回false之后，如果此时调用`Current`会抛出异常：

```F#
show enumerator.Current
// InvalidOperationException : 枚举已完成。
```

当`MoveNext()`返回false之后，继续调用`MoveNext()`不会抛出异常，它总是返回false：

```F#
show <| enumerator.MoveNext() // false
show <| enumerator.MoveNext() // false
show <| enumerator.MoveNext() // false
show <| enumerator.MoveNext() // false
```

当输入序列为空时，行为如下：

```F#
let ls = []
let enumerator = (Seq.ofList ls).GetEnumerator()
show <| enumerator.MoveNext() // false
show enumerator.Current
// InvalidOperationException : 枚举已完成。
```

## Iterator

javaScript的枚举器用法如下：

```js
const iterable = ['a', 'b'];
const iterator = iterable[Symbol.iterator]();

iterator.next() //, { value: 'a', done: false }
iterator.next() //, { value: 'b', done: false }
iterator.next() //, { value: undefined, done: true }
```
