# Nullable\<'T> Struct

## Remarks

A type is said to be nullable if it can be assigned a value or can be assigned `null`, which means the type has no value whatsoever. By default, all reference types, such as String, are nullable, but all value types, such as Int32, are not.

In C# and Visual Basic, you mark a value type as nullable by using the `?` notation after the value type. For example, `int?` in C# or `Integer?` in Visual Basic declares an integer value type that can be assigned `null`.

The Nullable structure supports using only a value type as a nullable type because reference types are nullable by design.

The Nullable class provides complementary support for the Nullable structure. The Nullable class supports obtaining the underlying type of a nullable type, and comparison and equality operations on pairs of nullable types whose underlying value type does not support generic comparison and equality operations.

### Fundamental Properties

The two fundamental members of the Nullable structure are the HasValue and Value properties. If the HasValue property for a Nullable object is `true`, the value of the object can be accessed with the Value property. If the HasValue property is `false`, the value of the object is undefined and an attempt to access the Value property throws an InvalidOperationException.

### Boxing and Unboxing

When a nullable type is boxed, the common language runtime automatically boxes the underlying value of the Nullable object, not the Nullable object itself. That is, if the HasValue property is `true`, the contents of the Value property is boxed. When the underlying value of a nullable type is unboxed, the common language runtime creates a new Nullable structure initialized to the underlying value.

有值时，装箱值本身

```F#
let a = box 1
let b = box (Nullable 1) // the contents of the Value property is boxed
Assert.Equal(unbox<int> a, unbox<int> b)
```

If the `HasValue` property of a nullable type is `false`, the result of a boxing operation is `null`. Consequently, if a boxed nullable type is passed to a method that expects an object argument, that method must be prepared to handle the case where the argument is `null`. When `null` is unboxed into a nullable type, the common language runtime creates a new Nullable structure and initializes its `HasValue` property to `false`.

无值时，装箱的是空

```F#
let a = box (Nullable<int>())
Assert.True(isNull a)
```

可空类型装箱后，和引用类型的拆箱方法相同了？