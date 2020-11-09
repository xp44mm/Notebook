# F# 聚集函数

Seq模块：

```F#
val average: source:seq<(^T)> -> ^T
val averageBy: projection:('T -> ^U) -> source:seq<'T>  -> ^U
val compareWith: comparer:('T -> 'T -> int) -> source1:seq<'T> -> source2:seq<'T> -> int
val contains: value:'T -> source:seq<'T> -> bool
val exactlyOne: source:seq<'T> -> 'T
val exists2: predicate:('T1 -> 'T2 -> bool) -> source1:seq<'T1> -> source2:seq<'T2> -> bool
val exists: predicate:('T -> bool) -> source:seq<'T> -> bool
val find: predicate:('T -> bool) -> source:seq<'T> -> 'T
val findBack: predicate:('T -> bool) -> source:seq<'T> -> 'T
val findIndex: predicate:('T -> bool) -> source:seq<'T> -> int
val findIndexBack: predicate:('T -> bool) -> source:seq<'T> -> int
val fold2: folder:('State -> 'T1 -> 'T2 -> 'State) -> state:'State -> source1:seq<'T1> -> source2:seq<'T2> -> 'State
val fold: folder:('State -> 'T -> 'State) -> state:'State -> source:seq<'T> -> 'State
val foldBack2: folder:('T1 -> 'T2 -> 'State -> 'State) -> source1:seq<'T1> -> source2:seq<'T2> -> state:'State -> 'State
val foldBack: folder:('T -> 'State -> 'State) -> source:seq<'T> -> state:'State -> 'State
val forall2: predicate:('T1 -> 'T2 -> bool) -> source1:seq<'T1> -> source2:seq<'T2> -> bool
val forall: predicate:('T -> bool) -> source:seq<'T> -> bool
val head: source:seq<'T> -> 'T
val isEmpty: source:seq<'T> -> bool
val item: index:int -> source:seq<'T> -> 'T
val iter2: action:('T1 -> 'T2 -> unit) -> source1:seq<'T1> -> source2:seq<'T2> -> unit
val iter: action:('T -> unit) -> source:seq<'T> -> unit
val iteri2: action:(int -> 'T1 -> 'T2 -> unit) -> source1:seq<'T1> -> source2:seq<'T2> -> unit
val iteri: action:(int -> 'T -> unit) -> source:seq<'T> -> unit
val last: source:seq<'T> -> 'T
val length: source:seq<'T> -> int
val max: source:seq<'T> -> 'T
val maxBy: projection:('T -> 'U) -> source:seq<'T> -> 'T
val min: source:seq<'T> -> 'T
val minBy: projection:('T -> 'U) -> source:seq<'T> -> 'T
val nth: index:int -> source:seq<'T> -> 'T
val pick: chooser:('T -> 'U option) -> source:seq<'T> -> 'U
val reduce: reduction:('T -> 'T -> 'T) -> source:seq<'T> -> 'T
val reduceBack: reduction:('T -> 'T -> 'T) -> source:seq<'T> -> 'T
val sum: source:seq<(^T)> -> ^T
val sumBy: projection:('T -> ^U) -> source:seq<'T>  -> ^U
val tryExactlyOne: source:seq<'T> -> 'T option
val tryFind: predicate:('T -> bool) -> source:seq<'T> -> 'T option
val tryFindBack: predicate:('T -> bool) -> source:seq<'T> -> 'T option
val tryFindIndex: predicate:('T -> bool) -> source:seq<'T> -> int option
val tryFindIndexBack: predicate:('T -> bool) -> source:seq<'T> -> int option
val tryHead: source:seq<'T> -> 'T option
val tryItem: index:int -> source:seq<'T> -> 'T option
val tryLast: source:seq<'T> -> 'T option
val tryPick: chooser:('T -> 'U option) -> source:seq<'T> -> 'U option
```

Array模块：

```F#
val average: array:^T[] -> ^T   
val averageBy: projection:('T -> ^U) -> array:'T[] -> ^U   
val blit: source:'T[] -> sourceIndex:int -> target:'T[] -> targetIndex:int -> count:int -> unit
val compareWith: comparer:('T -> 'T -> int) -> array1:'T[] -> array2:'T[] -> int
val contains: value:'T -> array:'T[] -> bool
val exactlyOne: array:'T[] -> 'T
val exists2: predicate:('T1 -> 'T2 -> bool) -> array1:'T1[] -> array2:'T2[] -> bool
val exists: predicate:('T -> bool) -> array:'T[] -> bool
val fill: target:'T[] -> targetIndex:int -> count:int -> value:'T -> unit
val find: predicate:('T -> bool) -> array:'T[] -> 'T
val findBack: predicate:('T -> bool) -> array:'T[] -> 'T
val findIndex: predicate:('T -> bool) -> array:'T[] -> int
val findIndexBack: predicate:('T -> bool) -> array:'T[] -> int
val fold2: folder:('State -> 'T1 -> 'T2 -> 'State) -> state:'State -> array1:'T1[] -> array2:'T2[] -> 'State
val fold: folder:('State -> 'T -> 'State) -> state:'State -> array: 'T[] -> 'State
val foldBack2: folder:('T1 -> 'T2 -> 'State -> 'State) -> array1:'T1[] -> array2:'T2[] -> state:'State -> 'State
val foldBack: folder:('T -> 'State -> 'State) -> array:'T[] -> state:'State -> 'State
val forall2: predicate:('T1 -> 'T2 -> bool) -> array1:'T1[] -> array2:'T2[] -> bool
val forall: predicate:('T -> bool) -> array:'T[] -> bool
val get: array:'T[] -> index:int -> 'T
val head: array:'T[] -> 'T
val isEmpty: array:'T[] -> bool
val item: index:int -> array:'T[] -> 'T
val iter2: action:('T1 -> 'T2 -> unit) -> array1:'T1[] -> array2:'T2[] -> unit
val iter: action:('T -> unit) -> array:'T[] -> unit
val iter: action:('T -> unit) -> array:'T[] -> unit
val iteri2: action:(int -> 'T1 -> 'T2 -> unit) -> array1:'T1[] -> array2:'T2[] -> unit
val iteri: action:(int -> 'T -> unit) -> array:'T[] -> unit
val iteri: action:(int -> 'T -> unit) -> array:'T[] -> unit
val last: array:'T[] -> 'T
val length: array:'T[] -> int
val max: array:'T[] -> 'T 
val maxBy: projection:('T -> 'U) -> array:'T[] -> 'T
val min: array:'T[] -> 'T 
val minBy: projection:('T -> 'U) -> array:'T[] -> 'T
val pick: chooser:('T -> 'U option) -> array:'T[] -> 'U 
val reduce: reduction:('T -> 'T -> 'T) -> array:'T[] -> 'T
val reduceBack: reduction:('T -> 'T -> 'T) -> array:'T[] -> 'T
val set: array:'T[] -> index:int -> value:'T -> unit
val sortInPlace: array:'T[] -> unit
val sortInPlaceBy: projection:('T -> 'Key) -> array:'T[] -> unit
val sortInPlaceWith: comparer:('T -> 'T -> int) -> array:'T[] -> unit
val sum: array: ^T[] -> ^T 
val sumBy: projection:('T -> ^U) -> array:'T[] -> ^U 
val tryExactlyOne: array:'T[] -> 'T option
val tryFind: predicate:('T -> bool) -> array:'T[] -> 'T option
val tryFindBack: predicate:('T -> bool) -> array:'T[] -> 'T option
val tryFindIndex: predicate:('T -> bool) -> array:'T[] -> int option
val tryFindIndexBack: predicate:('T -> bool) -> array:'T[] -> int option
val tryHead: array:'T[] -> 'T option
val tryItem: index:int -> array:'T[] -> 'T option
val tryLast: array:'T[] -> 'T option
val tryPick: chooser:('T -> 'U option) -> array:'T[] -> 'U option
```

List聚集函数：

```F#
val average: list:^T list -> ^T
val averageBy: projection:('T -> ^U) -> list:'T list  -> ^U
val compareWith: comparer:('T -> 'T -> int) -> list1:'T list -> list2:'T list -> int
val contains: value:'T -> source:'T list -> bool
val exactlyOne: list:'T list -> 'T
val exists2: predicate:('T1 -> 'T2 -> bool) -> list1:'T1 list -> list2:'T2 list -> bool
val exists: predicate:('T -> bool) -> list:'T list -> bool
val find: predicate:('T -> bool) -> list:'T list -> 'T
val findBack: predicate:('T -> bool) -> list:'T list -> 'T
val findIndex: predicate:('T -> bool) -> list:'T list -> int
val findIndexBack: predicate:('T -> bool) -> list:'T list -> int
val fold2: folder:('State -> 'T1 -> 'T2 -> 'State) -> state:'State -> list1:'T1 list -> list2:'T2 list -> 'State
val fold: folder:('State -> 'T -> 'State) -> state:'State -> list:'T list -> 'State
val foldBack2: folder:('T1 -> 'T2 -> 'State -> 'State) -> list1:'T1 list -> list2:'T2 list -> state:'State -> 'State
val foldBack: folder:('T -> 'State -> 'State) -> list:'T list -> state:'State -> 'State
val forall2: predicate:('T1 -> 'T2 -> bool) -> list1:'T1 list -> list2:'T2 list -> bool
val forall: predicate:('T -> bool) -> list:'T list -> bool
val head: list:'T list -> 'T
val isEmpty: list:'T list -> bool
val item: index:int -> list:'T list -> 'T
val iter2: action:('T1 -> 'T2 -> unit) -> list1:'T1 list -> list2:'T2 list -> unit
val iter: action:('T -> unit) -> list:'T list -> unit
val iteri2: action:(int -> 'T1 -> 'T2 -> unit) -> list1:'T1 list -> list2:'T2 list -> unit
val iteri: action:(int -> 'T -> unit) -> list:'T list -> unit
val last: list:'T list -> 'T
val length: list:'T list -> int
val max: list:'T list -> 'T
val maxBy: projection:('T -> 'U) -> list:'T list -> 'T
val min: list:'T list -> 'T
val minBy: projection:('T -> 'U) -> list:'T list -> 'T
val nth: list:'T list -> index:int -> 'T
val pick: chooser:('T -> 'U option) -> list:'T list -> 'U
val reduce: reduction:('T -> 'T -> 'T) -> list:'T list -> 'T
val reduceBack: reduction:('T -> 'T -> 'T) -> list:'T list -> 'T
val sum: list:^T list -> ^T
val sumBy: projection:('T -> ^U) -> list:'T list -> ^U
val tryExactlyOne: list:'T list -> 'T option
val tryFind: predicate:('T -> bool) -> list:'T list -> 'T option
val tryFindBack: predicate:('T -> bool) -> list:'T list -> 'T option
val tryFindIndex: predicate:('T -> bool) -> list:'T list -> int option
val tryFindIndexBack: predicate:('T -> bool) -> list:'T list -> int option
val tryHead: list:'T list -> 'T option
val tryItem: index:int -> list:'T list -> 'T option
val tryLast: list:'T list -> 'T option
val tryPick: chooser:('T -> 'U option) -> list:'T list -> 'U option
```

Map聚合函数：

```F#
val containsKey: key:'Key -> table:Map<'Key,'T> -> bool
val count: table:Map<'Key,'T> -> int
val exists: predicate:('Key -> 'T -> bool) -> table:Map<'Key, 'T> -> bool
val find: key:'Key -> table:Map<'Key,'T> -> 'T
val findKey: predicate:('Key -> 'T -> bool) -> table:Map<'Key,'T> -> 'Key
val fold: folder:('State -> 'Key -> 'T -> 'State) -> state:'State -> table:Map<'Key,'T> -> 'State
val foldBack: folder:('Key -> 'T -> 'State -> 'State) -> table:Map<'Key,'T> -> state:'State -> 'State
val forall: predicate:('Key -> 'T -> bool) -> table:Map<'Key, 'T> -> bool
val isEmpty: table:Map<'Key,'T> -> bool
val iter: action:('Key -> 'T -> unit) -> table:Map<'Key,'T> -> unit
val pick: chooser:('Key -> 'T -> 'U option) -> table:Map<'Key,'T> -> 'U
val tryFind: key:'Key -> table:Map<'Key,'T> -> 'T option
val tryFindKey: predicate:('Key -> 'T -> bool) -> table:Map<'Key,'T> -> 'Key option
val tryPick: chooser:('Key -> 'T -> 'U option) -> table:Map<'Key,'T> -> 'U option
```

Set聚合函数：

```F#
val contains: element:'T -> set:Set<'T> -> bool
val count: set:Set<'T> -> int
val exists: predicate:('T -> bool) -> set:Set<'T> -> bool
val fold: folder:('State -> 'T -> 'State) -> state:'State -> set:Set<'T> -> 'State
val foldBack: folder:('T -> 'State -> 'State) -> set:Set<'T> -> state:'State -> 'State
val forall: predicate:('T -> bool) -> set:Set<'T> -> bool
val isEmpty: set:Set<'T> -> bool
val isProperSubset: set1: Set<'T> -> set2:Set<'T> -> bool
val isProperSuperset: set1: Set<'T> -> set2:Set<'T> -> bool
val isSubset: set1: Set<'T> -> set2:Set<'T> -> bool
val isSuperset: set1: Set<'T> -> set2:Set<'T> -> bool
val iter: action:('T -> unit) -> set:Set<'T> -> unit
val maxElement: set:Set<'T> -> 'T
val minElement: set:Set<'T> -> 'T
```

