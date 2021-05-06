# Json.net食谱

主页：https://www.newtonsoft.com/json
源代码：https://github.com/JamesNK/Newtonsoft.Json

读取文件，返回一个`JToken`对象实例。
```F#
let readfile path =
    use reader = File.OpenText(path)
    JToken.ReadFrom(new JsonTextReader(reader))
```
判断一个符记的类型，将其转化具体的类型。如果是兼容类型，用对象转换，如果是基元类型用显式强制转换。布尔类型没有定义快捷强制转换操作符，用静态方法`JToken.op_Explicit`进行强制转换。这个静态方法可以根据返回值进行重载。`JToken`定义了许多显式或隐式的强制转化方法确定类型后，使用强制转化成具体的类型。
```F#
let jtokentype (jtoken:JToken) =
    match jtoken.Type with
    | JTokenType.None -> "No token type has been set."
    | JTokenType.Object ->
        let jo = jtoken :?> JObject
        jo.ToString()
    | JTokenType.Array ->
        let ja = jtoken :?> JArray
        ja.ToString()
    | JTokenType.Constructor -> "A JSON constructor."
    | JTokenType.Property ->
        let prop = jtoken :?> JProperty
        prop.ToString()
    | JTokenType.Comment -> "A comment."
    | JTokenType.Integer ->
        let i = int jtoken
        sprintf "%d" i
    | JTokenType.Float ->
        let f = float jtoken
        sprintf "%f" f
    | JTokenType.String ->
        string jtoken
    | JTokenType.Boolean ->
        let f:bool = JToken.op_Explicit(jtoken)
        sprintf "%b" f

    | JTokenType.Null -> "A null value."
    | JTokenType.Undefined -> "An undefined value."
    | JTokenType.Date -> "A date value."
    | JTokenType.Raw -> "A raw JSON value."
    | JTokenType.Bytes -> "A collection of bytes value."
    | JTokenType.Guid -> "A Guid value."
    | JTokenType.Uri -> "A Uri value."
    | JTokenType.TimeSpan -> "A TimeSpan value."
    | t -> failwithf "%A" t

```
枚举或遍历`JObject`对象的成员：

```F#
let entries (jobj:JObject) =
    [
        for e in jobj do
            let prop = e :?> JProperty
            yield (prop.Name, prop.Value)
    ]
```

注意，`jObject`对象可以直接被枚举，每个元素是`JToken`类型，需要强制转换成`JProperty`类型。

枚举或遍历`JArray`对象的成员：

```F#
let tolist (jarr:JArray) =
    [
        for jt in jarr do
            yield jt.ToString()
    ]
```

注意，`jArray`对象也可以直接被枚举，每个元素是`JToken`类型，代表数组中的元素。相当于对象例子里面的`prop.Value`。

# Serializing and Deserializing JSON

JsonConvert

`JsonConvert` 包裹 JsonSerializer，用于转化JSON字符串，包含两个方法`SerializeObject()` 和`DeserializeObject()`。

`JsonSerializerSettings` 通过设置 `JsonSerializer` 来使用简单序列化方法。

JsonSerializer

更好控制序列化，

读写文本通过： [JsonTextReader](https://www.newtonsoft.com/json/help/html/T_Newtonsoft_Json_JsonTextReader.htm) and [JsonTextWriter](https://www.newtonsoft.com/json/help/html/T_Newtonsoft_Json_JsonTextWriter.htm).

[JTokenReader](https://www.newtonsoft.com/json/help/html/T_Newtonsoft_Json_Linq_JTokenReader.htm)/[JTokenWriter](https://www.newtonsoft.com/json/help/html/T_Newtonsoft_Json_Linq_JTokenWriter.htm),  LINQ to JSON objects

JsonSerializer 有许多属性用于定制序列化 JSON的方式. These can also be used with the methods on JsonConvert via the JsonSerializerSettings overloads.

Converters

https://www.newtonsoft.com/json/help/html/SerializationSettings.htm

### 安装设置

单独使用Json.net的配置方法是设置`JsonConvert.DefaultSettings`。

获取或设置一个函数，创造默认`JsonSerializerSettings`。默认设置被自动调用， `JsonConvert`的序列化方法，和`JToken`的`ToObject ()`和`FromObject(Object)`。

欲序列化而不使用任何默认设置创造一个`JsonSerializer`，请使用`JsonSerializer.Create()`。

默认配置只需要在全局执行一次。然后可以持续使用。

```F#
type Test() =
    do
        // Any user supplied settings to these calls will override the default settings.
        JsonConvert.DefaultSettings <-
            new System.Func<_>(fun () ->
                let s = new JsonSerializerSettings()
                Cuisl.Newtonsoft.MySettings.mutate(s)
                s
            )
```

默认配置的具体配置方法，将在下节介绍。

Asp.net本身也使用了JSON.net，所以要对你偏好的配置对象应用两次，配置位置在`Startup.cs`文件中。

```c#
services
.AddJsonOptions(options => {
	var settings = options.SerializerSettings;
	settings.Converters.Add(desulfurization.fluids.ShapeConverter.jsonConverter());
	Cuisl.Newtonsoft.MySettings.mutate(settings);
	
	JsonConvert.DefaultSettings = () => settings;
})
```

这个配置放在`AddJsonOptions`中，首先修改`options`中的序列化设置，然后再把这个设置传给`JsonConvert`。

asp.net用到序列化的地方包括，响应的返回结果是json格式的。用来将.net对象序列化为json格式。

### 配置设置

JSON.net有许多配置，可以参考https://www.newtonsoft.com/json/help/html/SerializationSettings.htm

下面是上节未给出的`mutate`方法：

```F#
let mutate (settings:JsonSerializerSettings) =
    //忽略循环引用
    settings.ReferenceLoopHandling <- ReferenceLoopHandling.Ignore
    
	//JSON中Key首字母小写，映射到.net类型首字母大写
    settings.ContractResolver <-
        new CamelCasePropertyNamesContractResolver()
        //new DefaultContractResolver()//成员名称保持不变
        
    settings.Converters.Add(new TupleConverter())
    settings.Converters.Add(new DiscriminatedUnionConverter())

```

`Converters`是`JsonConverter`的集合，在序列化和解序列化时使用。转换器的添加顺序应该是从具体到一般，先来赢。

`JsonConverter`允许JSON手动序列化读，解序列化写。用途包括特别复杂的JSON结构，改变某个类型序列化的方式。

当`JsonConverter`起作用时，它将完全接管控制权去序列化或解序列化。许多特征将不再起作用。比如类型名称，循环引用处理。

### 自定义转换器

通用型转换器从`JsonConverter`继承，实例代码见`TupleConverter`，或者`DiscriminatedUnionConverter`，以及源代码中的默认转换器。

具体类型的转换器从`JsonConverter<>`继承，泛型版是从`JsonConverter`继承，实例代码见`ShapeConverter`，可以使用F#的对象表达式创建。

删除不适用于当前系统的条件编译：

1. 删除条件编译语句中灰色的代码。灰色代码就是不适用于当前系统的代码。
2. 使用替换`^#(if|else|endif).*`，删除条件编译语句。



VS使用正则表达式的常用法

C#参数转F#参数：

```pseudocode
regex:
(String|Type|Type\[\]|ILGenerator|MethodBase|MethodInfo|Object|Int32|PropertyInfo|FieldInfo) (\w+)
with
$2:$1
```

替换数组常量：

```
new\[\] \{(.*?)\}
[|$1|]
```

VS中匹配多行的方法是，用`\n`匹配一个换行。例如：

```perl
\}\s*\n\s*else \{
```

匹配

```C#
}
else {
```



