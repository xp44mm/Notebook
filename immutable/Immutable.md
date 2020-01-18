# Immutable中的類型

## 继承关系

繼承圖：

```
ValueObject <|- Collection <|- Collection.Keyed   <|- Map  <|- OrderedMap
                            |- Collection.Indexed <|- List
                            |                      |- Stack
                            |- Collection.Set     <|- Set  <|- OrderedSet
                            |- Seq
```


用ES6代碼來表示：

```javascript
class ValueObject{}
class Collection extends ValueObject{}
class Collection.Keyed extends Collection{}
class Collection.Indexed extends Collection{}
class Collection.Set extends Collection{}
class Map extends Collection.Keyed{}
class List extends Collection.Indexed{}
class Stack extends Collection.Indexed{}
class Set extends Collection.Set{}
class Seq extends Collection{}
class OrderedMap extends Map{}
class OrderedSet extends Set{}
```






