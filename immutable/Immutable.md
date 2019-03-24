# Immutable

## 继承关系

键值表、列表、集合：

```javascript
class ValueObject{}
class Collection extends ValueObject{}

class Collection.Keyed extends Collection{}
class Collection.Indexed extends Collection{}
class Collection.Set extends Collection{}

class Map extends Collection.Keyed{}
class List extends Collection.Indexed{}
class Set extends Collection.Set{}

class OrderedMap extends Map{}
class OrderedSet extends Set{}
```

序列：

```javascript
class Seq extends Collection{}


```

