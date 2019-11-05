# Json.net

Inheritance Hierarchy 

```F#
type JToken() = inherit System.Object()
type JContainer() = inherit JToken()
type JValue() = inherit JToken()
type JObject() = inherit JContainer()
type JArray() = inherit JContainer()
```

#### ISerializable

Types that implement `ISerializable` are serialized as JSON objects. When serializing, only the values returned from `ISerializable.GetObjectData` are used; members on the type are ignored. When deserializing, the constructor with a `SerializationInfo` and `StreamingContext` is called, passing the JSON object's values.

In situations where this behavior is not wanted, the `JsonObjectAttribute` can be placed on a .NET type that implements `ISerializable` to force it to be serialized as a normal JSON object.

查看自己的本地 SDK 是什么版本的，打开命令行程序，执行下面命令：

```
Microsoft Windows [版本 10.0.18362.295]

cuishengli@NANHU C:\Users\cuishengli
# dotnet --list-sdks
2.1.604 [C:\Program Files\dotnet\sdk]
2.1.801 [C:\Program Files\dotnet\sdk]
2.2.204 [C:\Program Files\dotnet\sdk]
2.2.300 [C:\Program Files\dotnet\sdk]
2.2.401 [C:\Program Files\dotnet\sdk]

cuishengli@NANHU C:\Users\cuishengli
#
```

