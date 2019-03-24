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