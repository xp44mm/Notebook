## Working with Types

The `System.Type` class is central to all reflection actions. It represents a managed type, a concept that encompasses classes, structures, modules, interfaces, and enums. The `Type` class provides all the means to enumerate a type's fields, properties, methods, and events, as well as set properties and fields and invoke methods dynamically.

An interesting detail: a `Type` object that represents a managed type is unique in a given AppDomain. This means that when you retrieve the `Type` object corresponding to a given type (for example, `System.String`) you always get the same instance, regardless of how you retrieve the `Type` object. This feature allows for the automatic synchronization of multiple static method invocations, among other benefits.

### Retrieving a Type Object

The `Type` class itself doesn't expose any constructors because you never really create a `Type` object; rather, you get a reference to an existing one. You can choose from many ways to retrieve a reference to a `Type` object. In previous sections, you saw that you can enumerate all the types in an `Assembly` or a `Module`:

```F#
for t in asm.GetTypes() do
    Console.WriteLine(t.FullName)
```

More often, you get a `Type` object using the Visual Basic `GetType` operator, which takes the unquoted name of a class:

```F#
let ty: Type = typeof<String>
Console.WriteLine(ty.FullName)            // => System.String
```

If you already have an instance of the class in question, you can use the `GetType` method that all objects inherit from `System.Object`:

```F#
let d: Double = 123.45
let ty = d.GetType()
Console.WriteLine(ty.FullName)            // => System.Double
```

The `Type.GetType` static method takes a quoted class name, so you can build the name of the class dynamically (something you can't do with the `GetType` function):

```F#
// Note that you can't pass Type.GetType a Visual Basic synonym,
// such as Short, int, Long, or Date.
let ty = Type.GetType("System.Int64")
Console.WriteLine(ty.FullName)            // => System.Int64
```

The `GetType` method looks for the specified type in the current assembly and then in the system assembly (mscorlib.dll). Like the `Assembly.GetType` instance method, the `Type.GetType` static method returns null if the specified type doesn't exist, but you can also pass true as its second argument to force a `TypeLoadException` in this case, and you can pass true as its third argument if you want the type name to be compared in a case-insensitive way. If the type you want to reference is neither in the caller's assembly nor in mscorlib.dll, you must append a comma and the name of the assembly in which the type resides. For example, the following code snippet shows how you get a reference to the `System.Data.DataSet` class, which resides in the assembly named `System.Data`. Because the GAC might contain many assemblies with this friendly name, you must pass the complete identity of the assembly after the first comma:

```F#
let typeName = "System.Data.DataSet, System.Data, Version=2.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089"
let ty = Type.GetType(typeName)
```

.NET Framework 2.0 adds a variant of the previous method, which loads a type for inspection purposes only (similar to the `Assembly.ReflectionOnlyLoad` method):

```F#
// Second argument tells whether an exception is thrown if the type isn't found.
// Third argument tells whether case should be ignored in the search.
let ty = Type.ReflectionOnlyGetType(typeName, false, true)
```

#### The TypeResolve Event

When the .NET runtime isn't able to load a type successfully, it fires the `TypeResolve` event of the `AppDomain` object that represents the current application domain. As it happens with the `AssemblyResolve` event, the `TypeResolve` event gives you the ability to override the .NET Framework's default binding policy. The following example shows how you can use this event:

```F#
let AppDomain_TypeResolve(sender: Object)  (e: ResolveEventArgs) : Assembly =
    if e.Name = "System.Windows.Forms.Form" then
        let asmFile = Path.Combine(
                        RuntimeEnvironment.GetRuntimeDirectory(), 
                        "System.Windows.Forms.dll")
        Assembly.LoadFile(asmFile)

    // Return null if unable to provide an alternative.
    else null

let TestTypeResolveEvent() =
    // Subscribe to the AppDomain.TypeResolve event.
    let appDomain: AppDomain = AppDomain.CurrentDomain
    appDomain.add_TypeResolve(ResolveEventHandler(AppDomain_TypeResolve))
    
    // Get a reference to the Form type.
    // (It should fail, but it doesn't because we are handling the TypeResolve event.)
    let ty: Type = Type.GetType("System.Windows.Forms.Form")
    // Create a form and show it.
    let obj: Object = ty.InvokeMember("", BindingFlags.CreateInstance, null,  null, null, null)
    ty.InvokeMember("Show", BindingFlags.InvokeMethod, null, obj, null)
    |> ignore
    
    // Unsubscribe from the event.
    appDomain.remove_TypeResolve <| ResolveEventHandler(AppDomain_TypeResolve)
```

The `TypeResolve` event fires when you fail to load a type through reflection, but it doesn't when the .NET runtime has located the assembly and the assembly doesn't contain the searched type. (If the CLR isn't able to locate the assembly, an `AppDomain.AssemblyResolve` event fires.) For example, the following statement doesn't cause the `TypeResolve` event to be fired because the CLR can locate the mscorlib.dll even though that assembly doesn't contain the definition of the `Form` type:

```F#
let ty: Type = 
    typeof<Object>.Assembly.GetType("System.Windows.Forms.Form")
```

#### Exploring Type Properties

All the properties of the `Type` object are read-only for one obvious reason: you can't change an attribute (such as name or scope) of a type defined in a compiled assembly. The names of most properties are self-explanatory, such as the `Name` (the type's name), `FullName` (the complete name, which includes the namespace), and `Assembly` (the `Assembly` object that contains the type). The `IsClass`, `IsInterface`, `IsEnum`, and `IsValueType` properties let you classify a given `Type` object. For example, the following code lists all the types exported by mscorlib.dll, specifying whether each is a class, an enum, a value type, or an interface:

```F#
let asm: Assembly = Assembly.Load("mscorlib")
for t: Type in asm.GetExportedTypes() do
   if t.IsClass then
      Console.WriteLine(t.Name + " (Class)")
   elif t.IsEnum then
      // An enum is also a value type, so we must test IsEnum before IsValueType.
      Console.WriteLine(t.Name + " (Enum)")
   elif t.IsValueType then
      Console.WriteLine(t.Name + " (Structure)")
   elif t.IsInterface then
      Console.WriteLine(t.Name + " (Interface)")
   else
      // This statement is never reached because a type
      // can't be anything other than one of the above.
      ()
```

The `IsPublic` and `IsNotPublic` properties return information about the type's visibility. You should use these properties only with types that aren't nested in other types: the `IsPublic` property of a nested type is always false.

If the type is nested inside another type, you must use the following IsNestedXxxx properties to deduce the scope used to declare the type: `IsNestedPublic` (public), `IsNestedAssembly` (internal), `IsNestedFamily` (Protected), `IsNestedFamORAssem` (Protected Internal), `IsNestedPrivate` (private), and `IsNestedFamANDAssem` (Protected and visible only from inside the assembly, a scope you can't define with F#). You can also use the `DeclaringType` property to get the enclosing type of a nested type; this property returns null if the type isn't nested.

While we are on this subject, notice that the `FullName` property of a nested type includes a plus sign (+) to separate the name of the class and the name of its enclosing type, as in:

```F#
MyNamespace.MyEnclosingType+MyNestedType
```

A couple of properties are new in .NET Framework 2.0: `IsNested` returns true if the type is nested in another type (regardless of its scope), whereas `IsVisible` lets you determine whether the type can be accessed from outside the assembly; it returns true if the type is a public top-level type or is a public type nested inside a public type.

You can get information about inheritance relationships by means of the `BaseType` (the base class for a type), `IsAbstract` (true for `[<AbstractClass>]` classes), and `IsSealed` (true for `[<Sealed>]` classes) properties:

```F#
// (The asm variable is pointing to mscorlib…)
for t: Type in asm.GetExportedTypes() do
   let mutable text: String = t.FullName + " "
   if t.IsAbstract then text <- text + "[<AbstractClass>] "
   if t.IsSealed then text <- text + "[<Sealed>] "
   
   // We need this test because System.Object has no base class.
   if t.BaseType <> null then
      text <- text + "(base: " + t.BaseType.FullName + ") "

   Console.WriteLine(text)
```

You can get additional information on a given type by querying a few methods, such as `IsSubclassOfType` (returns true if the current type is derived from the type passed as an argument), `IsAssignableFrom` (returns true if the type passed as an argument can be assigned to the current type), and `IsInstanceOfType` (returns true if the object passed as an argument is an instance of the current type). Let's recap a few of the many ways you have to test an object's type:

```F#
//if obj.GetType() = typeof<Person> then
//   // obj can be assigned to a Person variable (the Visual Basic way).
//   ()

if typeof<Person>.IsAssignableFrom(obj.GetType()) then
   // obj can be assigned to a Person variable (the reflection way).
   ()

if typeof<Person>.IsInstanceOfType(obj) then
   // obj is a Person object.
   ()

if typeof<Person> = obj.GetType() then
   // obj is a Person object (but fails if obj is null).
   ()

if obj.GetType().IsSubclassOf(typeof<Person>) then
   // obj is an object that inherits from Person.
   ()
```

#### Enumerating Members

The `Type` class exposes an intimidatingly large number of methods. The following methods let you enumerate type members: `GetMembers`, `GetFields`, `GetProperties`, `GetMethods`, `GetEvents`, `GetConstructors`, `GetInterfaces`, `GetNestedTypes`, and `GetDefaultMembers`. All these methods (note the plural names) return an array of elements that describe the members of the type represented by the current `Type` object. The most generic method in this group is `GetMembers`, which returns an array with all the fields, properties, methods, and events that the type exposes. For example, the following code lists all the members of the `System.String` type:

```F#
let tt = typeof<string>
tt.GetMembers()
|> Array.map(fun m -> m.MemberType, m.Name)
|> Array.map(fun(t,n)-> String.Format("{0} {1}", t, n))
|> String.concat "\n"
|> output.WriteLine
```

The `GetMembers` function returns an array of `MemberInfo` elements, where each `MemberInfo` represents a field, a property, a method, a constructor, an event, or a nested type (including delegates defined inside the class). `MemberInfo` is an abstract type from which more specific types derive—for example, `FieldInfo` for field members and `MethodInfo` for method members. The `MemberInfo.MemberType` enumerated property lets you discern between methods, properties, fields, and so on.

The `GetMembers` method returns two or more `MemberInfo` objects with the same name if the class exposes overloaded properties and methods. So, for example, the output from the preceding code snippet includes multiple occurrences of the `Format` and `Concat` methods. You also find multiple occurrences of the constructor method, which is always named `.ctor`. In the next section, I show how you can explore the argument signature of these overloaded members. Also note that the `GetMembers` method returns public, instance, and static members, as well as methods inherited by other objects, such as the `GetHashCode` method inherited from `System.Object`.

The `GetMembers` method supports an optional `BindingFlags` enumerated argument. This bit-coded value lets you narrow the enumeration—for example, by listing only public or instance members. The `BindingFlags` type is used in many reflection methods and includes many enumerated values, but in this case only a few are useful:

* The `Public` and `NonPublic` enumerated values restrict the enumeration according to the scope of the elements. (You must specify at least one of these flags to get a nonempty result.)
* The `Instance` and `Static` enumerated values restrict the enumeration to instance members and static members, respectively. (You must specify at least one of these flags to get a nonempty result.)
* The `DeclaredOnly` enumerated value restricts the enumeration to members declared in the current type (as opposed to members inherited from its base class).
* The `FlattenHierarchy` enumerated value is used to include static members up the hierarchy.

This code lists only the public, nonstatic, and noninherited members of the `String` class:

```F#
// Get all public, instance, noninherited members of String type.
let minfo: MemberInfo[] =
    typeof<String>.GetMembers(
        BindingFlags.Public ||| BindingFlags.Instance ||| BindingFlags.DeclaredOnly)
```

The preceding code snippet produces an array that includes the `ToString` method, which at first glance shouldn't be in the result because it's inherited from `System.Object`. It's included because the `String` class adds an overloaded version of this method, and this overloaded method is the one that appears in the result array.

To narrow the enumeration to a given member type, you can use a more specific GetXxxxs method. When you're using a GetXxxxs method other than `GetMembers`, you can assign the result to an array of a more specific type, namely, `PropertyInfo`, `MethodInfo`, `ConstructorInfo`, `FieldInfo`, or `EventInfo`. (All these specific types derive from `MemberInfo`.) For example, this code lists only the methods of the `String` type:

```F#
for mi: MethodInfo in typeof<String>.GetMethods() do
    Console.WriteLine(mi.Name)
```

The `GetInterfaces` or `GetNestedTypes` method return an array of `Type` elements, rather than a `MemberInfo` array, so the code in the loop is slightly different:

```F#
for itf: Type in typeof<String>.GetInterfaces() do
    Console.WriteLine(itf.FullName)
```

All the GetXxxxs methods—with the exception of `GetDefaultMembers` and `GetInterfaces`—can take an optional `BindingFlags` argument to restrict the enumeration to public or nonpublic, static or instance, and declared or inherited members. For more sophisticated searches, you can use the `FindMembers` or `FindInterfaces` method, which takes a delegate pointing to a function that filters individual members or interfaces. (See MSDN documentation for additional information.)

In many cases, you don't need to enumerate a type's members because you have other ways to find out the name of the field, property, methods, or event you want to get information about. You can use the `GetMember` or other GetXxxx methods (where Xxxx is a singular word) of the `Type` class—namely, `GetMember`, `GetField`, `GetProperty`, `GetMethod`, `GetEvent`, `GetInterface`, `GetConstructor`, and `GetNestedType`—to get the corresponding `MemberInfo` (or a more specific object):

```F#
// Get information about the String.Chars property.
let pi: PropertyInfo = typeof<String>.GetProperty("Chars")
```

If you're querying for an overloaded property or method, you need to ask for a specific version of the member by using `GetProperty` or `GetMethod` and specifying the exact argument signature by passing an array of `Type` objects as its second argument:

```F#
// Get the MethodInfo object for the IndexOf string method with the
// following signature: IndexOf(char:Char, startIndex:int, endIndex:int).

// Prepare the signature as an array of Type objects.
let argTypes: Type[] = [|typeof<Char>; typeof<int>; typeof<int>|]
// Ask for the method with given name and signature.
let mi2: MethodInfo = typeof<String>.GetMethod("IndexOf", argTypes)
```

**Version 2005 of VB or Version 2.0 of .NET** The method signature you pass to `GetMethod` must include information about whether the argument is passed by reference or is an array. Two new methods of the `Type` class make this task simpler than it is in .NET Framework 1.1:

```F#
// This code shows how you can build a reference to the following method
//      TestMethod(x: byref<int>, arr: String[,]):unit.
let argType1: Type = typeof<int>.MakeByRefType()
let argType2: Type = typeof<String>.MakeArrayType(2)
let arrTypes: Type[] = [|argType1; argType2|]
let mi3: MethodInfo = typeof<TestClass>.GetMethod("TestMethod", arrTypes)
```

Speaking of arrays, notice that the name of array types ends with a pair of brackets:

```F#
let arrTy1: Type = typeof<int[]>
let arrTy2: Type = typeof<int[,]>
Console.WriteLine(arrTy1.FullName) // => System.Int32[]
Console.WriteLine(arrTy2.FullName) // => System.Int32[,]
```

Also, the name of a type that represents a `byref<_> ` argument has a trailing ampersand (`&`) character; therefore, you need to process the value returned by the `FullName` property if you want to display a type name using Visual Basic syntax:

```F#
let vbTypeName: String =
    argType1.FullName
    .Replace("[", "(").Replace("]", ")") //array
    .Replace("&", "") //byref
```

Finally, your code can easily get a reference to the `MethodBase` that describes the method being executed by means of a static member of the `MethodBase` type:

```F#
let currMethod: MethodBase = MethodBase.GetCurrentMethod()
```

### Exploring Type Members

After you get a reference to a `MemberInfo` object—or a more specific object, such as `FieldInfo` or `PropertyInfo`—you can retrieve information about the corresponding member. Because all these specific XxxxInfo objects derive from `MemberInfo`, they have some properties in common, including `Name`, `MemberType`, `ReflectedType` (the type used to retrieve this `MemberInfo` instance), and `DeclaringType` (the type where this member is declared). The values returned by the last two properties differ only if the member has been inherited.

The following loop displays the name of all the members exposed by the `String` type, together with a description of the member type. To make things more interesting, I'm suppressing constructor methods, multiple definitions for overloaded methods, and methods inherited from the base Object class:

```F#
let ty = typeof<string>

ty.GetMembers()
|> Array.filter(fun mi -> mi.DeclaringType = mi.ReflectedType)// Ignore inherited members.
|> Array.filter(fun mi -> mi.MemberType <> MemberTypes.Constructor)// Ignore constructor methods.
|> Array.map(fun mi -> String.Format("{0} {1}", mi.MemberType, mi.Name))
|> Set.ofArray
|> String.concat "\n"
|> output.WriteLine
```

declaring type: 定义或声明成员的所在类型。即`member`语句所处的类型。Reflected Type: `GetMember`等方法所在的实例类型。比如`typeof<Int32>.GetMember("Equals")`的声明类型是`Object`，反射类型是`Int32`.

#### Exploring Fields

Except for the members inherited from `MemberInfo`, a `FieldInfo` object exposes only a few properties, including `FieldType` (the type of the field), `IsLiteral` (true if the field is actually a constant), `IsInitOnly` (true if the field is marked as ReadOnly), `IsStatic` (true if the field is marked as static), and other Boolean properties that reflect the scope of the field, such as `IsPublic`, `IsAssembly` (internal), `IsFamily` (Protected), `IsFamilyOrAssembly` (Protected internal), `IsFamilyAndAssembly` (Protected but visible only from inside the same assembly, a scope not supported by F#), and `IsPrivate`:

```F#
let ty = typeof<Type>

ty.GetFields(BindingFlags.Public ||| BindingFlags.NonPublic ||| BindingFlags.Instance ||| BindingFlags.Static)
|> Array.filter(fun fi -> fi.IsPublic || fi.IsAssembly)
|> Array.filter(fun fi -> not fi.IsLiteral)
|> Array.map(fun fi -> String.Format("{0}: {1}", fi.Name, fi.FieldType.Name))
|> Array.sort
|> String.concat "\n"
|> output.WriteLine
```

A new method in .NET Framework 2.0 allows you to extract the value of a constant:

```F#
// List all the public constants in the Int32 type.
let ty = typeof<int>

ty.GetFields()
|> Array.filter(fun fi -> fi.IsLiteral)
|> Array.map(fun fi -> String.Format("{0} = {1}", fi.Name, fi.GetRawConstantValue()))
|> String.concat "\n"
|> output.WriteLine
```

#### Exploring Methods

Like `FieldInfo`, the `MethodInfo` type exposes the `IsStatic` property and all the other scope-related properties you've just seen, plus a few additional Boolean properties: `IsVirtual` (the method is marked with the Overridable keyword), `IsAbstract` (`abstract`), and `IsFinal` (NotOverridable). The `IsSpecialName` property returns true if the method has been created by the compiler and should be dealt with in a special way, as is the case of the methods generated by properties and operators. If the method returns a value (a Function, in Visual Basic parlance), the `ReturnType` property returns the type of the return value; otherwise, it returns a special type whose name is `System.Void`. This snippet uses these properties to display information on all the methods in a class, exposed in a Visual Basic-like syntax:

```F#
let ty = typeof<Array>
        
ty.GetMethods()
// Ignore special methods, such as property getters and setters.
|> Array.filter(fun mi -> not mi.IsSpecialName)
|> Array.sortBy(fun mi -> mi.Name)
|> Array.map(fun mi ->
    let retTypeName = 
        match mi.ReturnType.FullName with
        | null -> mi.ReturnType.Name
        | "System.Void" -> "unit"
        | retTypeName -> retTypeName
    let flags =
        [
            if mi.IsAbstract then yield "Abstract"
            if mi.IsAssembly then yield "Assembly"
            if mi.IsConstructedGenericMethod then yield "ConstructedGenericMethod"
            if mi.IsConstructor then yield "Constructor"
            if mi.IsFamily then yield "Family"
            if mi.IsFamilyAndAssembly then yield "FamilyAndAssembly"
            if mi.IsFamilyOrAssembly then yield "FamilyOrAssembly"
            if mi.IsFinal then yield "Final"
            if mi.IsGenericMethod then yield "GenericMethod"
            if mi.IsGenericMethodDefinition then yield "GenericMethodDefinition"
            if mi.IsHideBySig then yield "HideBySig"
            if mi.IsPrivate then yield "Private"
            if mi.IsPublic then yield "Public"
            if mi.IsSecurityCritical then yield "SecurityCritical"
            if mi.IsSecuritySafeCritical then yield "SecuritySafeCritical"
            if mi.IsSecurityTransparent then yield "SecurityTransparent"
            if mi.IsSpecialName then yield "SpecialName"
            if mi.IsStatic then yield "Static"
            if mi.IsVirtual then yield "Virtual"
        ] |> String.concat ","
    String.Format("{0}: {1} {{{2}}}", mi.Name, retTypeName, flags)
)
|> String.concat "\r\n"
|> output.WriteLine
```

The `ConstructorInfo` type exposes the same members as the `MethodInfo` type (not surprisingly because both these types inherit from the `MethodBase` abstract class, which in turn derives from `MemberInfo`), with the exception of `ReturnType` (constructors don't have a return type).

#### Exploring Properties

The `PropertyInfo` type exposes only three interesting properties besides those inherited from `MemberInfo`: `PropertyType` (the type returned by the property), `CanRead` (false for write-only properties), and `CanWrite` (false for read-only properties). Oddly, the `PropertyInfo` type doesn't expose members that indicate the scope of the property or whether it's a static property. You can access this information only indirectly by means of one of the following methods: `GetGetMethod` (which returns the `MethodInfo` object corresponding to the Get method), `GetSetMethod` (the `MethodInfo` object corresponding to the Set method), or `GetAccessors` (an array of one or two `MethodInfo` objects, corresponding to the Get and/or Set accessor methods):

```F#
let ty = typeof<Array>

// Display instance and static properties.
ty.GetProperties(BindingFlags.Public ||| BindingFlags.Instance ||| BindingFlags.Static)
|> Array.map(fun pi ->
    // Get either the Get or the Set accessor methods.
    let methods =
        [
            if pi.CanRead then yield pi.GetMethod
            if pi.CanWrite then yield pi.SetMethod
        ]
        |> List.map(fun mi -> "    " + mi.Name) // TODO: call method printer
    let flags =
        [
            if pi.CanRead then yield "CanRead"
            if pi.CanWrite then yield "CanWrite"
        ]|> String.concat ","
    let ps = String.Format("{0}: {1} {{{2}}}", pi.Name, pi.PropertyType.FullName, flags)
    String.concat "\n" (ps::methods)
)
|> String.concat "\n"
|> output.WriteLine
```

If you need to retrieve a property accessor only to determine its scope or whether the property is static, you can use the `GetAccessors` method as follows:

```F#
// Get all the property accessors, even if it's private.
let methods =
    pi.GetAccessors(true)
    |> Array.map(fun mi -> mi.Name)
    |> String.concat ","
String.Format("{0} with {{{1}}}",pi.Name, methods)
```

By default the `GetGetMethod`, `GetSetMethod`, and `GetAccessors` methods return only public accessor methods; if the accessor method doesn't exist or isn't public, the return value is null. However, these methods are overloaded to take a Boolean argument: if you pass true, they return the accessor method even if it doesn't have a public scope.

#### Exploring Events

Getting information about an event is complicated by the fact that the `EventInfo` type has no property that lets you determine the scope of the event or whether it's static. Instead, you must use `GetAddMethod` to return the `MethodInfo` object corresponding to the method that adds a new subscriber to the list of listeners for this event. (This is the method that the `AddHandler` keyword calls for you behind the scenes.) Typically, this method is named `add_Eventname` and is paired with the `remove_Eventname` hidden method (the method called by `RemoveHandler` and whose `MethodInfo` is returned by the `GetRemoveMethod`). The Visual Basic compiler creates these methods for you by default, unless you define a custom event.

You can query the `MethodInfo` object returned by either `GetAddMethod` or `GetRemoveMethod` to discover the event's scope, its arguments, and whether it's static:

```F#
let typ = typeof<obj>
// Get information on the SampleEvent event of the TestClass object.
let ei: EventInfo = typ.GetEvent("SampleEvent")
// Get a reference to the hidden add_SampleEvent method.
let mi2: MethodInfo = ei.GetAddMethod()
// Test the method scope and check whether it's static.
()
```

#### Exploring Parameters

The one thing left to do is enumerate the parameters that a property or a method expects. Both the `GetIndexParameters` (of `PropertyInfo`) and the `GetParameters` (of `MethodInfo`) methods return an array of `ParameterInfo` objects, where each element describes the attributes of the arguments passed to and from the member.

A `ParameterInfo` object has properties with names that are self-explanatory: `Name` (the name of the parameter), `ParameterType` (the type of the parameter), `Member` (the `MemberInfo` the parameter belongs to), `Position` (an integer that describes where the parameter appears in the method signature), `IsOptional` (true for optional parameters), and `DefaultValue` (the default value of an optional parameter). The following code shows how to display the calling syntax for a given method:

```F#
let mi: MethodInfo = typeof<TestClass>.GetMethod("MethodWithOptionalArgs")

mi.GetParameters()
|> Array.sortBy(fun pi -> pi.Position)
|> Array.map(fun pi ->
    let optional  = if pi.IsOptional then "Optional " else "" 

    // Notice how you can discern between ByVal and ByRef parameters.
    let direction: String = 
        if pi.ParameterType.IsByRef then "ByRef" else "ByVal"

    // Process the parameter type.
    let tyName: String = 
        match pi.ParameterType.FullName with
        | null -> sprintf "%A" pi.ParameterType
        | tyName -> 
            tyName
                .Replace("[", "(").Replace("]", ")")// Convert [] into () 
                .Replace("&", "")//and drop the & character (included if parameter is ByRef).

    let dec = String.Format("{0} {1} : {2}", direction, pi.Name, tyName)

    // Append the default value for optional parameters.
    let defaultValue =
        if pi.IsOptional then 
            "=" + GetObjectValue(pi.DefaultValue)
        else ""

    [optional;dec;defaultValue] 
    |> String.concat ""
)
|> String.concat ","
|> output.WriteLine
```

The previous code snippet uses the `GetObjectValue` auxiliary method, which returns the value of an object in Visual Basic syntax:

```F#
let GetObjectValue(obj : Object) : String =
   if obj = null then
      "null"
   elif obj.GetType() = typeof<String> then
      "\"" + obj.ToString() + "\""
   elif obj.GetType() = typeof<DateTime> then
      "#" + obj.ToString() + "#"
   elif obj.GetType().IsEnum then
      // It's an enum type.
      obj.GetType().Name + "." + Enum.GetName(obj.GetType(), obj)
   else
      // It's something else, including a number.
      obj.ToString()
```

Getting the syntax for an event is more complicated because the `EventInfo` object doesn't expose the `GetParameters` method. Instead, you must use the `EventHandlerType` property to retrieve the `Type` object corresponding to the delegate that defines the event. The `Invoke` method of this delegate, in turn, has the same signature as the event:

```F#
let ei: EventInfo = typeof<TestClass>.GetEvent("SampleEvent")
let delegType: Type = ei.EventHandlerType
let mi: MethodInfo = delegType.GetMethod("Invoke")
for pi: ParameterInfo in mi.GetParameters() do
   ()
```

#### Exploring the Method Body

**Version 2005 of VB or Version 2.0 of .NET** Version 2.0 of the .NET Framework introduces a new feature that, although not completely implemented, surely goes in a very promising direction: the ability to peek at the IL code compiled for a given method. The entry point for this capability is the new `MethodBase.GetMethodBody` method, which returns a `MethodBody` object. In turn, a `MethodBody` object exposes properties that let you list the local variables, evaluate the size of the stack that the method uses, and explore the try…Catch exception handlers defined in the inspected method.

```F#
// Get a reference to the method in a type.
let mi: MethodInfo = typeof<TestClass>.GetMethod("TestMethod")
let mb: MethodBody = mi.GetMethodBody()
// Display the number of used stack elements.
Console.WriteLine("Stack Size = {0}", mb.MaxStackSize)

// Display index and type of local variables.
Console.WriteLine("Local variables:")
for lvi: LocalVariableInfo in mb.LocalVariables do
   Console.WriteLine(" var[{0}] As {1}", lvi.LocalIndex, lvi.LocalType.FullName)

// Display information about exception handlers.
Console.WriteLine("Exception handlers:")
for ehc: ExceptionHandlingClause in mb.ExceptionHandlingClauses do
   Console.WriteLine(" Type={0}, ", ehc.Flags.ToString())
   if ehc.Flags = ExceptionHandlingClauseOptions.Clause then
      Console.WriteLine("ex As {0}, ", ehc.CatchType.Name)

   Console.WriteLine("try off/len={0}/{1}, ", ehc.TryOffset, ehc.TryLength)
   Console.WriteLine("Handler off/len={0}/{1}", ehc.HandlerOffset, ehc.HandlerLength)

```

The list of exception handlers doesn't differentiate between Catch and finally clauses belonging to distinct try blocks, but you can group them correctly by looking at elements with identical `TryOffset` properties. The `Flags` property of the `ExceptionHandlingClause` object helps you understand whether the clause is a filter (When block), a clause (Catch block), or a finally block. The type of the `Exception` object is exposed by the `CatchType` property. For example, given the following method:

```F#
let TestMethod(x: byref<int>, arr: String[,]) =
    try
        try
            Console.WriteLine("First try block")
        with
        | :? NullReferenceException as ex ->
            ()
        | ex ->
            ()
    finally
        ()

    try
        Console.WriteLine("Second try block")
    with
    | :? NullReferenceException as ex ->
        ()

    | :? OverflowException as ex ->
        ()

    | ex ->
        ()

```

Here's the information that might be displayed in the console window. (The actual offset and length information varies depending on the actual executable statements in the method.)

```
Stack Size = 2
Local variables:
  var[0] As System.NullReferenceException
  var[1] As System.Exception
  var[2] As System.NullReferenceException
  var[3] As System.OverflowException
  var[4] As System.Exception
Exception handlers:
  Type=Clause, ex As NullReferenceException, Try off/len=2/14, Handler off/len=16/26
  Type=Clause, ex As Exception, Try off/len=2/14, Handler off/len =42/26
  Type=Finally, Try off/len=2/66, Handler off/len=68/12
  Type=Clause, ex As NullReferenceException, Try off/len=82/13, Handler off/len=95/26
  Type=Clause, ex As OverflowException, Try off/len=82/13, Handler off/len=121/26
  Type=Clause, ex As Exception, Try off/len=82/13, Handler off/len=147/27
```

Notice that the list of local variables is likely to include variables that you haven't declared explicitly but that are created for you by the compiler to store intermediate results, such as the `Exception` variables in Catch clauses or the upper limit of a For loop.

The only method of the `MethodBody` object of interest is `GetILAsByteArray`, which returns an array containing the raw IL opcodes. These opcodes are fully documented, so you might use this method to disassemble a .NET executable. As you can guess, this isn't exactly a trivial task, however.

### Reflecting on Generics

**Version 2005 of VB or Version 2.0 of .NET** Reflection techniques in .NET Framework 2.0 fully support generic types, and you must account for them when exploring the types that an assembly exposes.

#### Exploring Generic Types

You can distinguish generic type definitions from regular types when enumerating all the types in an assembly by checking their `IsGenericTypeDefinition` method. The full name of a generic type definition contains a inverse quote character followed by the number of type arguments in the definition. Therefore, given the following code:

```F#
// List all the generic types in mscorlib.
let asm: Assembly = typeof<obj>.Assembly
asm.GetTypes()
|> Array.filter(fun ty -> ty.IsGenericTypeDefinition)
|> Array.map(fun ty -> ty.FullName) //sprintf "%A"
|> String.concat "\r\n"
|> output.WriteLine
```

This is the kind of results you'll see in the console window:

```
System.Collections.Generic.List`1
System.Collections.Generic.Dictionary`2
System.Action`1
...
```

The names of the generic parameters in the type definition don't appear in the type name because they aren't meaningful in the composition of a unique type name: as you might recall from Chapter 11, "Generics," you can have two generic type definitions with the same name only if the number of their generic parameters differs. The syntax based on the inverse quote character becomes important if you want to retrieve a reference to the generic type definition, as in this code:

```F#
let genType: Type = asm.GetType("System.Collections.Generic.Dictionary`2")
Assert.True(genType.IsGenericTypeDefinition)
```

There is no built-in method or property that returns the signature of the generic type as it appears in source code, and thus you have to manually strip the inverse quote character from the name and use the `GetGenericArguments` method to retrieve the name of type parameters:

```F#
// (Continuing previous code example)
let typeName: String = genType.FullName

// Strip the inverse quote character.
let typeName = typeName.Remove(typeName.IndexOf('`'))

let gArgs =
    genType.GetGenericArguments()
    // The GenericParameterPosition property reflects the position where
    // this argument appears in the signature.
    |> Array.sortBy(fun tyArg -> tyArg.GenericParameterPosition)
    |> Array.filter(fun tyArg -> tyArg.IsGenericTypeParameter) // always true
    |> Array.map(fun tyArg -> "'"+tyArg.Name)
    |> String.concat ","

sprintf "%s<%s>" typeName gArgs
|> output.WriteLine // System.Collections.Generic.Dictionary<'TKey,'TValue>
```

#### Exploring Generic Methods

You must adopt a similar approach when exploring the generics methods of a type. (Remember that a method with generic arguments can appear in both a regular and a generic type.) You can check whether a method has a generic definition by means of the `MethodInfo.IsGenericMethodDefinition` method and explore its generic parameters by means of the `MethodInfo.GetGenericArguments` method. For example, the following loop displays the name of all the methods in a type using the `<_>`clause for generic methods:

```F#
let ty = typeof<Array>

ty.GetMethods()
|> Array.filter(fun mi -> mi.IsGenericMethod)
|> Array.map(fun mi ->
    let gargs =
        mi.GetGenericArguments()
        |> Array.sortBy(fun arg -> arg.GenericParameterPosition)
        |> Array.map(fun arg -> "'"+arg.Name)
        |> String.concat ","
    sprintf "%s<%s>" mi.Name gargs
)
|> String.concat "\r\n"
|> output.WriteLine
```

When you explore the parameters of a method, you must discern between regular types (e.g., `System.String`) and types passed as an argument to a generic type or method (e.g., T or K). This is possible because the `Type` class exposes a new `IsGenericParameter` property, which returns false in the former case and true in the latter. It is essential that you test this method before doing anything else with a `Type` value because some properties of a type used as a parameter in a generic class return meaningless values or might throw an exception. For example, this is the most correct way to assemble the signature of a method:

```F#
// (The mi variable points to a MethodInfo object.)
let mutable signature: String = mi.Name + "("
for par: ParameterInfo in mi.GetParameters() do
    if par.Position > 0 then signature <- signature + ", "
    signature <- signature + par.Name + " As " + GetTypeName(par.ParameterType)

signature <- signature + ")"
let retType: Type = mi.ReturnType
if retType.FullName <> "System.Void" then
    signature <- signature + " As " + GetTypeName(retType)

Console.WriteLine(signature)
   // => TestMethod(key As K, values As V(), count As System.Int32) As T
```

where the `GetTypeName` function is defined as follows:

```F#
let GetTypeName(tp : Type) : String =
   if tp.IsGenericParameter then
      tp.Name
   else
      tp.FullName.Replace("[", "(").Replace("]", ")").Replace("&", "")
```

#### Exploring Members That Use Generic Types

A slightly different problem occurs when you are dealing with a member of a type (either a regular or generic type) and the member uses or returns a generic type that has already been bound with nongeneric arguments, as in the following case:

```F#
[
    typeof<ResizeArray<int>>
    typeof<System.Collections.Generic.Dictionary<string,float>>
]
|> List.filter(fun ty -> ty.IsGenericType) // for all is true
|> List.map(fun ty ->ty.FullName)
|> List.iter(output.WriteLine)
```

When you reflect on the argument and the return type of the previous method, you get the following types:

```
System.Collections.Generic.List`1[[System.Int32, System.Private.CoreLib, Version=4.0.0.0, Culture=neutral, PublicKeyToken=7cec85d7bea7798e]]
System.Collections.Generic.Dictionary`2[[System.String, System.Private.CoreLib, Version=4.0.0.0, Culture=neutral, PublicKeyToken=7cec85d7bea7798e],[System.Double, System.Private.CoreLib, Version=4.0.0.0, Culture=neutral, PublicKeyToken=7cec85d7bea7798e]]
```

Three details are worth noticing:

* The type name uses the inverse quote character syntax and is followed by the names of all the types that are bound to the generic type.
* Each argument consists of the type's full name followed by the display name of the assembly where the type is defined, all enclosed in a pair of square brackets.
* The entire list of argument types is enclosed in an additional pair of square brackets.

You can easily extract the name of a generic type and its argument types by parsing this full name, for example, by using a regular expression. Alternatively, you can use the `IsGenericType` method to check whether the type is the bound version of a generic type, and, if this is the case, you can use the `GetGenericTypeDefinition` method to extract the name of the original generic type and the `GetGenericArguments` method to extract the type of individual type arguments:

```F#
let ty = typeof<System.Collections.Generic.Dictionary<System.String,System.Double>>
let name: String = ty.GetGenericTypeDefinition().FullName
let name = name.Remove(name.IndexOf('`'))
let gargs =
    ty.GetGenericArguments()
    // |> Array.sortBy(fun argType -> argType.GenericParameterPosition) //it is arg no param, order no change.
    |> Array.map(fun argType -> sprintf "%A" argType)//GetTypeName(argType)
    |> String.concat ","

let signature = sprintf "%s<%s>" name gargs
Assert.Equal("System.Collections.Generic.Dictionary<System.String,System.Double>",signature)
```

In practice, you can gather all the cases that I've illustrated so far in an expanded version of the `GetTypeName` function (which I introduced in the previous section):

```F#
let fsharpGenericParameterName(tp: Type) = "'" + tp.Name

let rec fsharpTypeName(tp: Type) : String =
    if tp.IsGenericTypeDefinition then // List<'T>
        // It's the type definition of an "open" generic type.
        let typeName = 
            let nm = tp.FullName
            nm.Remove(nm.IndexOf('`'))
        let genericParameters =
            tp.GetGenericArguments()
            |> Array.sortBy(fun p -> p.GenericParameterPosition) // only for demo, order keep no changed.
            |> Array.filter(fun p -> p.IsGenericParameter) // only for demo, always true.
            |> Array.map fsharpGenericParameterName
            |> String.concat ","
        sprintf "%s<%s>" typeName genericParameters
    elif tp.IsGenericParameter then // 'T
        // It's a parameter in an Of clause.
        fsharpGenericParameterName tp
    elif tp.IsGenericType then // List<int>
        // This is a generic type that has been bound to specific types.
        let typeName =
            let nm = tp.GetGenericTypeDefinition().FullName
            nm.Remove(nm.IndexOf('`'))
        let gargs =
            tp.GetGenericArguments()
            |> Array.map fsharpTypeName
            |> String.concat ","
        sprintf "%s<%s>" typeName gargs
    else
        // This is a regular type.
        tp.FullName
```

Thanks to its being recursive, this function is able to deal correctly even with contorted cases such as these:

```F#
let MyList = List<Dictionary<String, Double>>()
let MyDictionary = Dictionary<String, Dictionary<String, List<int>>>()
```

#### Binding a Generic Type

Sometimes you might need to bind a generic type with a set of one or more specific type arguments. This is necessary, for example, when you want to retrieve the `Type` object that corresponds to `List<String>` or `Dictionary<String,Int32>`. The key for this operation is the `Type.MakeGenericType` method:

```F#
// Retrieve the type that corresponds to MyGenericTable<String, Double>.
let typeName: String = "MyApp.MyGenericTable`2"
// Get a reference to the "open" generic type.
let genType: Type = Assembly.GetExecutingAssembly().GetType(typeName)
// Bind the "open" generic type to a set of arguments, and retrieve
// a reference to the MyGenericTable<String, Double>.
let tp: Type = genType.MakeGenericType(typeof<String>, typeof<Double>)
```

A bound generic type can be useful on at least a couple of occasions. First, you can use it when you need to create an instance of a specific type. (I cover object instantiation through reflection later in this chapter.) Second, you can use it when you are looking for a method with a signature that contains an argument of a specific type. Say you have the following class:

```F#
type TestClass() =
   member this.TestMethod(list : List<int>, x : int) = ()
   member this.TestMethod(list : List<String>, x : String) = ()
```

How can you build a `MethodInfo` object that points to the first `TestMethod` rather than the second one? Here's the solution:

```F#
// First, get a reference to the List "open" generic type.
let typeName: String = "System.Collections.Generic.List`1"
let openType: Type = typeof<Object>.Assembly.GetType(typeName)
// Bind the open List type to the int type.
let boundType: Type = openType.MakeGenericType(typeof<int>)
// Prepare the signature of the method you're interested in.
let argTypes: Type[] = [|boundType; typeof<int>|]
// Get the reference to that specific method.
let method: MethodInfo = typeof<TestClass>.GetMethod("TestMethod", argTypes)
```

When you bind an open generic type to a set of argument types, you should ensure that generic constraints are fulfilled. Reflection enables you to extract the constraints associated with each argument by means of the `GetGenericParameterConstraints` method, whereas the `GenericParameterAttributes` property returns an enum value that provides information about the New, Class, and Structure constraints:

```F#
let genType: Type = Assembly.GetExecutingAssembly().GetType("MyApp.GenericList`1")
for argType: Type in genType.GetGenericArguments() do
   // Get the class and interface constraints for this argument.
   for c: Type in argType.GetGenericParameterConstraints() do
      Console.WriteLine(c.FullName)

   // Get the New, Class, or Structure constraints.
   let attrs: GenericParameterAttributes = argType.GenericParameterAttributes
   if (attrs &&& GenericParameterAttributes.DefaultConstructorConstraint) > GenericParameterAttributes.None then
      Console.WriteLine(" New (default constructor)")

   if (attrs &&& GenericParameterAttributes.ReferenceTypeConstraint) > GenericParameterAttributes.None then
      Console.WriteLine(" Class (reference type)")

   if (attrs &&& GenericParameterAttributes.NotNullableValueTypeConstraint) > GenericParameterAttributes.None then
      Console.WriteLine(" Structure (nonnullable value type)")
```

### Reflecting on Attributes

As you might recall from Chapter 2, "Basic Language Concepts," .NET attributes provide a standard way to extend the metadata at the assembly, type, and member levels and can include additional information with a format defined by the programmer. Not surprisingly, the .NET Framework also provides the means to read this attribute-based metadata from an assembly. Because the attributes you define in your code are perfectly identical to the attributes that the .NET Framework defines for its own purposes, the mechanism for extracting either kind of attributes is the same. Therefore, even though in this section I show you how to extract .NET attributes, keep in mind that you can apply the same techniques for extracting your custom attributes. (You'll find many examples of the latter ones in Chapter 19, "Custom Attributes.")

#### Exploring Attributes

You can use several techniques to extract the custom attribute associated with a specific element.

First, you can use the `IsDefined` method exposed by the `Assembly`, `Module`, `Type`, `ParameterInfo`, and `MemberInfo` classes (and all the classes that inherit from `MemberInfo`, such as `FieldInfo` and `PropertyInfo`). This method returns true if the attribute is defined for the specified element, but doesn't let you read the attribute's fields and properties. The last argument passed to the method is a Boolean value that specifies whether attributes inherited from the base class should be returned:

```F#
// The second argument specifies whether you also want to test
// attributes inherited from the base class.
if typeof<Person>.IsDefined(typeof<SerializableAttribute>, false) then
   Console.WriteLine("The Person class is serializable")
```

(See the companion code for the complete listing of the Person type.)

Second, you can use the `GetCustomAttributes` method (note the plural) exposed by the `Assembly`, `Module`, `Type`, `ParameterInfo`, and `MemberInfo` classes (and all the classes that inherit from `MemberInfo`). This method returns an array containing all the attributes of the specified type that are associated with the specified element so that you can read their fields and properties, and you can specify whether attributes inherited from the base class should be included in the result:

```F#
// Display all the Conditional attributes associated with the Person.SendEmail method.
let mi: MethodInfo = typeof<Person>.GetMethod("SendEmail")
// GetCustomAttributes returns an array of Object elements, so you need to cast.
let miAttrs: ConditionalAttribute[] =  
    mi.GetCustomAttributes(typeof<ConditionalAttribute>, false)
    |> Array.map(fun o -> o:?> ConditionalAttribute)
// Check whether the result contains at least one element.
if miAttrs.Length > 0 then
    Console.WriteLine("SendEmail is marked with the following Conditional attribute(s):")
    // Read the properties of individual attributes.
    for attr: ConditionalAttribute in miAttrs do
        Console.WriteLine(" <Conditional(\"{0}\")>", attr.ConditionString)

```

Third, you can use an overload of the `GetCustomAttributes` method exposed by the `Assembly`, `Module`, `Type`, `ParameterInfo`, and `MemberInfo` classes (and all the classes that inherit from it) that doesn't take an attribute type as an argument. When you use this overload, the method returns an array containing all the custom attributes associated with the element:

```F#
// Display all the attributes associated with the Person.FirstName field.
let fi: FieldInfo = typeof<Person>.GetField("FirstName")
let fiAttrs: obj[] = fi.GetCustomAttributes(false)
// Check whether the result contains at least one element.

if fiAttrs.Length > 0 then
    Console.WriteLine("FirstName is marked with the following attribute(s):")
    // Display the name of all attributes (but not their properties).
    for attr in fiAttrs do
        let attr = attr :?> Attribute
        Console.WriteLine(attr.GetType().FullName)

```

To further complicate your decision, you can achieve the same results shown previously by means of static methods of the `System.Attribute` type:

```F#
// Check whether the Person class is marked as serializable.
if Attribute.IsDefined(typeof<Person>, typeof<SerializableAttribute>) then
    Console.WriteLine("The Person class is serializable")

// Retrieve the Conditional attributes associated with the Person.SendEmail method,
// including those inherited from the base class.
let mi: MethodInfo = typeof<Person>.GetMethod("SendEmail")
let miAttrs: ConditionalAttribute[] = 
    Attribute.GetCustomAttributes(mi, typeof<ConditionalAttribute>, true)
    |> Array.map(fun o -> o :?> ConditionalAttribute)
// Check whether the result contains at least one element.
if miAttrs.Length > 0 then
    ()

// Display all the attributes associated with the Person.FirstName field.
let fi: FieldInfo = typeof<Person>.GetField("FirstName")
let fiAttrs: Attribute[] = Attribute.GetCustomAttributes(fi, false)
// Check whether the result contains at least one element.
if fiAttrs.Length > 0 then
    ()

```

The `System.Attribute` class also exposes the `GetCustomAttribute` static method (note the singular), which returns the only attribute of the specified type:

```F#
// Read the Obsolete attribute associated with the Person class, if any.
let tyAttr: ObsoleteAttribute = 
    Attribute.GetCustomAttribute(typeof<Person>, typeof<ObsoleteAttribute>) 
    :?> ObsoleteAttribute
if tyAttr <> null then
    Console.WriteLine("The Person class is marked as obsolete.")
    Console.WriteLine(" IsError={0}, Message={1}", tyAttr.IsError, tyAttr.Message)

```

An important note: you should never use the `Attribute.GetCustomAttribute` method with attributes that might appear multiple times—such as the Conditional attribute—because in that case the method might throw an `AmbiguousMatchException` object.

All these alternatives are quite confusing, so let me recap when each of them should be used:

* If you just need to check whether an attribute is associated with an element, use the `Attribute.IsDefined` static method or the `IsDefined` instance method exposed by the `Assembly`, `Module`, `Type`, `ParameterInfo`, and `MemberInfo` classes. This technique doesn't actually instantiate the attribute object in memory and is therefore the fastest of the group.

* If you are checking whether a single-instance attribute is associated with an element and you want to read the attribute's fields and properties, use the `Attribute.GetCustomAttribute` static method. (Don't use this technique with attributes that might appear multiple times—such as the Conditional attribute—because in that case the method might throw an `AmbiguousMatchException` object.)

* If you are checking whether a multiple-instance attribute is associated with an element and you want to read the attribute's fields and properties, use the `Attribute.GetCustomAttributes` static method or the `GetCustomAttributes` method exposed by the `Assembly`, `Module`, `Type`, `ParameterInfo`, and `MemberInfo` classes. You must use this technique when reading all the attributes associated with an element, regardless of the attribute type.

**Version 2005 of VB or Version 2.0 of .NET** Although all the techniques discussed in this section are available in .NET Framework 1.1 as well, there is a new important change in how you use them to query some special CLR attributes, such as `Serializable`, `NonSerialized`, `DllImport`, `StructLayout`, and `FieldOffset`. To improve performance and to save space in metadata tables, previous versions of the .NET Framework stored these special attributes using a format different from all other attributes. Consequently, you couldn't reflect on these attributes by using one of the techniques I just illustrated. Instead, you had to use special properties exposed by other reflection objects, for example, the `IsSerialized` and `IsLayoutSequential` properties of the `Type` class or the `IsNonSerialized` property of the `FieldInfo` class. A welcome addition in .NET Framework 2.0 is that you don't need to use any of these properties any longer because all the special .NET attributes can be queried by means of the `IsDefined`, `GetCustomAttribute`, and `GetCustomAttributes` methods described in this section. (However, properties such as `IsSerializable` and `IsLayoutSequential` continue to be supported for backward compatibility.)

#### The `CustomAttributeData` Type

**Version 2005 of VB or Version 2.0 of .NET** Version 1.1 of the .NET Framework has a serious limitation related to custom attributes: you could search attributes buried in metadata, instantiate them, and read their properties, but you have no documented means for extracting the exact syntax used in code to define the attribute. For example, you can't determine whether an attribute field or property is assigned in the attribute's constructor using a standard (mandatory) argument or a named (optional) argument; if the field or property is equal to its default value (null or zero), you can't determine whether it happened because the property was omitted in the attribute's constructor. For example, these limitations prevent a .NET developer from building a full-featured object browser.

In addition to this limitation inherited from .NET Framework 1.1, you run into another problem under .NET Framework 2.0 when you want to extract custom attributes from assemblies that have been loaded for reflection-only purposes. In fact, both the `GetCustomAttribute` and the `GetCustomAttributes` methods instantiate the custom attribute and therefore would run some code inside the assembly, which is prohibited.

Both issues have been resolved by means of the new `CustomAttributeData` type and the auxiliary `CustomAttributeTypedArgument` class (which represents a positional argument in the attribute's constructor) and `CustomAttributeNamedArgument` class (which represents a named argument).

You create an instance of the `CustomAttributeData` type by means of the `GetCustomAttributes` static method that the type itself exposes. Each `CustomAttributeData` object has three properties: `Constructor` (the `ConstructorInfo` object that represents the attribute's constructor being used), `ConstructorArguments` (a list of `CustomAttributeTypedArgument` objects), and `NamedArguments` (a list of `CustomAttributeNamedArgument` objects):

```F#
// Retrieve the syntax used in custom attributes for the TestClass type.
let attrList: IList<CustomAttributeData> =  
    CustomAttributeData.GetCustomAttributes(typeof<TestClass>)

// Iterate over all the attributes.
for attrData: CustomAttributeData in attrList do
    // Retrieve the attribute's type, by means of the ConstructorInfo object.
    let attrType: Type = attrData.Constructor.DeclaringType
    // Start building the Visual Basic code.
    let mutable attrString: String = "<" + attrType.FullName + "("
    let mutable sep: String = ""

    // Include all mandatory arguments for this constructor.
    for typedArg: CustomAttributeTypedArgument in attrData.ConstructorArguments do
        attrString <- attrString + sep + FormatTypedArgument(typedArg)
        // A comma is used as the separator for all elements after the first one.
        sep <- ","

    // Include all optional arguments for this constructor.
    for namedArg: CustomAttributeNamedArgument in attrData.NamedArguments do
        // The TypedValue property returns a CustomAttributeTypedArgument object.
        let typedArg: CustomAttributeTypedArgument = namedArg.TypedValue
        // Use the MemberInfo property to retrieve the field or property name.
        attrString <- attrString + sep + namedArg.MemberInfo.Name + ":=" + FormatTypedArgument(typedArg)
        // A comma is used as the separator for all elements after the first one.
        sep <- ","

    // Complete the attribute syntax and display it.
    attrString <- attrString + ")>"
    Console.WriteLine(attrString)
```

The `FormatTypedArgument` method takes a `CustomAttributeTypedArgument` object and returns the corresponding Visual Basic code that can initialize it:

```F#
// Return a textual representation of a string, date, or numeric value.
let FormatTypedArgument(typedArg : CustomAttributeTypedArgument) : String =
   if typedArg.ArgumentType = typeof<String> then
        // It's a quoted string.
        "\"" + typedArg.Value.ToString() + "\""

   elif typedArg.ArgumentType = typeof<DateTime> then
        // It's a DateTime constant.
        "#" + typedArg.Value.ToString() + "#"
   elif typedArg.ArgumentType.IsEnum then
        // It's an enum value.
        typedArg.ArgumentType.Name + "." +  Enum.GetName(typedArg.ArgumentType, typedArg.Value)
   else
        // It's something else (presumably a number).
        typedArg.Value.ToString()
```

### Creating a Custom Object Browser

All the reflection properties shown enable you to create a custom object browser that can solve problems that are out of reach for the object browser included in Visual Studio. Creating a custom object browser isn't a trivial task, though, especially if you want to implement a sophisticated user interface. For this reason, in this section I focus on a simple but useful object browser implemented as a Console application.

The sample application I discuss here is able to display all the types and members in an assembly that are marked with the `Obsolete` attribute. I implemented this utility to keep an updated list of members that are in beta versions of .NET Framework 2.0 but would have been removed before the release version, as well as members that were present in .NET Framework 1.1 but have been deprecated in the current version. You can pass it the path of an assembly or launch it without passing anything on the command line: in the latter case, the utility will analyze all the assemblies in the .NET Framework main directory.

The program displays its results in the console window and takes several minutes to explore all the assemblies in the .NET Framework, but you can redirect its output to a file and then load the file in a text editor to quickly search for a type or a method. Here are the core routines:

```F#
module MainModule

open System
open System.IO
open System.Reflection
open System.Runtime.InteropServices

// Process all the types and members in an assembly.
let ShowObsoleteMembersOf(asm : Assembly) =
    let attrType: Type = typeof<ObsoleteAttribute>

    // This header is displayed only if this assembly contains obsolete members.
    let mutable asmHeader: String = String.Format("ASSEMBLY {0}{1}\n",  asm.GetName().Name)

    for tp: Type in asm.GetTypes() do
        // This header will be displayed only if the type is obsolete or
        // contains obsolete members.
        let mutable typeHeader: String = String.Format(" TYPE {0}{1}\n",  ReflectionHelpers.GetTypeName(tp))

        // Search the Obsolete attribute at the type level.
        let mutable attr: ObsoleteAttribute =
            Attribute.GetCustomAttribute(tp, attrType):?> ObsoleteAttribute
        if attr <> null then
            // This type is obsolete.
            Console.Write(asmHeader + typeHeader)
            // Display the message attached to the attribute.
            let mutable message: String = "WARNING"
            if attr.IsError then message <- "ERROR"
            Console.WriteLine("      {0}: {1}", message, attr.Message)
            // Don't display the assembly header again.
            asmHeader <- ""
        else
            // The type isn't obsolete; let's search for obsolete members.
            for mi: MemberInfo in tp.GetMembers() do
                attr <- Attribute.GetCustomAttribute(mi,  attrType):?> ObsoleteAttribute
                if attr <> null then
                    // This member is obsolete.
                    let memberHeader: String = String.Format("        {0} {1}",  mi.MemberType.ToString().ToUpper(), ReflectionHelpers.GetMemberSyntax(mi))
                    Console.WriteLine(asmHeader + typeHeader + memberHeader)
                    // Display the message attached to the attribute.
                    let mutable message: String = "WARNING"
                    if attr.IsError then message <- "ERROR"
                    Console.WriteLine("            {0}: {1}", message, attr.Message)
                    // Don't display the assembly and the type header again.
                    asmHeader <- ""
                    typeHeader <- ""

// Process an assembly at the specified file path.
let ShowObsoleteMembersFile(asmFile : String) =
    try
        let asm: Assembly = Assembly.LoadFrom(asmFile)
        ShowObsoleteMembersOf(asm)
    with (ex : Exception) ->
        // The file isn't a valid assembly.
        ()

// Process all the assemblies in the .NET Framework directory.
let ShowObsoleteMembers() =
    let path: String = RuntimeEnvironment.GetRuntimeDirectory()
    for asmFile: String in Directory.GetFiles(path, "*.dll") do
        ShowObsoleteMembersFile(asmFile)

let Main(args : String[]) =
    if args.Length = 0 then
        ShowObsoleteMembers()
    else
        ShowObsoleteMembersFile(args.[0])

```

The main program uses a few helper routines, for example, to assemble the name of a type or the signature of a method using Visual Basic syntax. I have explained how this code works in previous sections, so I won't do it again here. I have gathered these methods in a separate module so that you can reuse them easily in other reflection-intensive projects:

```F#
module ReflectionHelpers

open System
open System.Reflection

// Returns the name of a type. (Supports generics and array types.)
let rec GetTypeName(tp : Type) : String =
    let mutable tp = tp
    let mutable typeName: String = null
    let mutable suffix: String = ""

    // Account for array types.
    if tp.IsArray then
        suffix <- "()"
        tp <- tp.GetElementType()

    // Account for byref types.
    if tp.IsByRef then tp <- tp.GetElementType()

    if tp.IsGenericTypeDefinition then
        // It's the type definition of an "open" generic type.
        typeName <- tp.FullName
        typeName <- typeName.Remove(typeName.IndexOf('`')) + "(Of "
        for targ: Type in tp.GetGenericArguments() do
            if targ.GenericParameterPosition > 0 then typeName <- typeName + ","
            typeName <- typeName + targ.Name

        typeName <- typeName + ")"
    elif tp.IsGenericParameter then
        // It's a parameter in an Of clause.
        typeName <- tp.Name
    elif tp.IsGenericType then
        // This is a generic type that has been bound to specific types.
        typeName <- tp.GetGenericTypeDefinition().FullName
        typeName <- typeName.Remove(typeName.IndexOf('`')) + "(Of "
        let mutable sep: String = ""
        for argType: Type in tp.GetGenericArguments() do
            typeName <- typeName + sep + GetTypeName(argType)
            sep <- ","

        typeName <- typeName + ")"
    else
        // This is a regular type.
        typeName <- tp.FullName

    typeName + suffix

// Return the name of a member. (Recognizes constructors and generic methods.)
let GetMemberName(mi : MemberInfo) : String =
    let mutable memberName: String = mi.Name

    match mi.MemberType with
    | MemberTypes.Constructor ->
        memberName <- "New"
    | MemberTypes.Method ->
        // Account for generic methods.
        let method: MethodInfo = mi :?> MethodInfo
        if method.IsGenericMethodDefinition then
            // Include all type arguments.
            memberName <- memberName + "(Of "
            for ty: Type in method.GetGenericArguments() do
                if ty.GenericParameterPosition > 0 then memberName <- memberName + ","
                memberName <- memberName + ty.Name

            memberName <- memberName + ")"
    | mt -> failwithf "%A" mt

    memberName

// Returns the syntax of an array of parameters.
let GetParametersSyntax(parInfos : ParameterInfo[]) : String =
    let mutable paramSyntax: String = "("
    let mutable sep: String = ""
    for pi: ParameterInfo in parInfos do
        paramSyntax <- paramSyntax +  sep + GetTypeName(pi.ParameterType)
        sep <- ","

    paramSyntax + ")"

// Returns the syntax of a member
let GetMemberSyntax(mi : MemberInfo) : String =
    let mutable memberSyntax: String = GetMemberName(mi)

    match mi.MemberType with
    | MemberTypes.Property ->
        let pi: PropertyInfo = mi :?> PropertyInfo
        memberSyntax <- memberSyntax + GetParametersSyntax(pi.GetGetMethod(true).GetParameters())  + " As " + GetTypeName(pi.PropertyType)
    | MemberTypes.Method ->
        let mi: MethodInfo = mi :?> MethodInfo
        memberSyntax <- memberSyntax + GetParametersSyntax(mi.GetParameters())
        if mi.ReturnType.FullName <> "System.Void" then
            memberSyntax <- memberSyntax + " As " + GetTypeName(mi.ReturnType)
    | MemberTypes.Constructor ->
        let ci: ConstructorInfo = mi :?> ConstructorInfo
        memberSyntax <- memberSyntax + GetParametersSyntax(ci.GetParameters())
    | MemberTypes.Event ->
        let ei: EventInfo = mi :?> EventInfo
        let mi: MethodInfo = ei.EventHandlerType.GetMethod("Invoke")
        memberSyntax <- memberSyntax + GetParametersSyntax(mi.GetParameters())
    | mt -> failwithf "%A" mt
    memberSyntax
```

As provided, the utility displays output in a purely textual format. It is easy, however, to change the argument of `String.Format` methods so that it outputs XML or HTML text, which would greatly improve the appearance of the result. (The complete demo program contains modified versions of this code that outputs HTML and XML text.)

### Replace `GetGenericArguments` with `GenericTypeArguments` or `GenericTypeParameters`

`GetGenericArguments()` returns parameters (not arguments) if the type is a generic definition, and it returns arguments if the type is a constructed generic.

`GenericTypeArguments` is better if you expect the type to be constructed.

`GenericTypeParameters` is better if you expect the type to be a definition. It only exists after adding `.GetTypeInfo()` though. If you don't have an expectation about constructed vs definition, I'm not sure how you can meaningfully use `GetGenericArguments()`.

I'm still not sure if this is a good diagnostic, but the example would be `typedefof(List<_>).GetGenericArguments()` being replaced with `t1.GetTypeInfo().GenericTypeParameters` since `List<_>` doesn't technically have its type parameters filled by any type arguments.

The win is that you're using the correct terminology for what you're getting, **parameters**, not **arguments**. And if the type changes to `List<int>`, you'll get an empty array when you ask for type parameters because you should be doing `GetGenericTypeDefinition()` first if you're interested in parameters.

`typedefof(List<_>)` is an example where they differ. The property returns an empty array, while the method returns an array with a generic `'T` in it. (this `'T` has `IsGenericParameter` `true`)

From reading the documentation, I think that you can think of `GenericTypeArguments` as 

```F#
ty.GetGenericArguments()
|> Array.filter(fun t -> not t.IsGenericParameter)
```

, i.e. only the concrete types. See also `ContainsGenericParameters`.

