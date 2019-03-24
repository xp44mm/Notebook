在.NET里面，如何将`string`解析为其它的基元类型（Primitive Type）？

在.NET里面，我们为这些基元类型提供了
```C#
Convert.To{PrimitiveType}()
{PrimitiveType}.Parse()
{PrimitiveType}.TryParse()
```
方法，用于把string解析成对应的基元类型。

全名`Systme.Convert`

当然，你得首先保证待解析的`string`字面值与对应的基元类型兼容。这些基元类型是（括号中是对应的C#关键字）：
```C#
SByte (sbyte)
Byte (byte)
Int16 (short)
UInt16 (ushort)
Int32 (int)
UInt32 (uint)
Int64 (long)
UInt64 (ulong)
Char (char)
Single (float)
Double (double)
Boolean (bool)
Decimal (decimal)
```

另外，.NET也提供了把string解析为`DateTime`结构的方法，除了有上面提到的三种形式（`Convert.ToDateTime`、`DateTime.Parse`、`DateTime.TryParse`）之外，我们还可以使用`DateTime.TryParseExact`。

所有的这些方法都是大同小异，差别在于对字符串字面值格式（包括其所包含符号）的规定，有兴趣的话，你可以参考MSDN的文档或者其他专题资料。