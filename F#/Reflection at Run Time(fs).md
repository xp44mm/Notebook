## Reflection at Run Time

So far, I've shown how to use reflection to enumerate all the types and members in an assembly, an activity that is central to applications such as object browsers or code generators. If you write mostly business applications, you might object that reflection doesn't have much to offer you, but this isn't correct. In fact, reflection also allows you to actually create objects and invoke methods in a sort of "late-bound" mode, that is, without you having to burn the type name and the method name in code. In this section, I show a series of techniques based on this capability.

### Creating an Object Dynamically

Let's start by seeing how you can instantiate an object given its type name. You can choose from three ways to create a .NET object using reflection: by using the `CreateInstance` method of the `System.Activator` class, by using the `InvokeMember` method of the Type class, or by invoking one of the type's constructor methods.

If the type has a parameterless constructor, creating an instance is simple:

``` F#
// Next statement assumes that the Person class is defined in
// an assembly named @"MyApp".
let myType: Type = Assembly.GetExecutingAssembly().GetType(@"MyApp.Person")
let o: Object = Activator.CreateInstance(myType)
// Prove that we created a Person.
Console.WriteLine(@"A {0} object has been created", o.GetType().Name)
```

To call a constructor that takes one or more parameters, you must prepare an array of values:

``` F#
// (We reuse the myType variable from previous code…)
// Use the constructor that takes two arguments.
let args2: Object[] = [|@"Joe", @"Evans"|]
// Call the constructor that matches the parameter signature.
let o2: Object = Activator.CreateInstance(myType, args2)
```

You can use `InvokeMember` to create an instance of the class and even pass arguments to its constructor, as in the following code:

``` F#
// Prepare the array of parameters.
let args3: Object[] = [|@"Joe", @"Evans"|]

// Constructor methods have no name and take null in the second to last argument.
let o3: Object = myType.InvokeMember(@"", BindingFlags.CreateInstance,  null, null, args3)
```

Creating an object through its constructor method is a bit more convoluted, but I'll demonstrate the technique here for the sake of completeness:

``` F#
// Prepare the argument signature as an array of types (two strings).
let argTypes: Type[] = [|typeof<String>, typeof<String>|]
// Get a reference to the correct constructor.
let ci: ConstructorInfo = myType.GetConstructor(argTypes)
// Prepare the parameters.
let args4: Object[] = [|@"Joe", @"Evans"|]
// Invoke the constructor and assign the result to a variable.
let o4: Object = ci.Invoke(args4)
```

Regardless of the technique you used to create an instance of the type, you usually assign the instance you've created to an Object variable, as opposed to a strongly typed variable. (If you knew the name of the type at compile time, you wouldn't need to use reflection in the first place.) There is only one relevant exception to this rule: when you know in advance that the type being instantiated derives from a specific base class (or implements a given interface), you can cast the Object variable to a variable typed after that base class (or interface) and access all the members that the object inherits from the base class (or interface).

**Version 2005 of VB or Version 2.0 of .NET** The new `MakeArrayType` method of the Type class makes it very simple to instantiate arrays using reflection, as you can see in this code:

``` F#
// Create an array of Double. (You can pass an integer argument to the MakeArrayType
// method to specify the rank of the array, for multidimensional arrays.
let arrType: Type = typeof<Double>.MakeArrayType()
// The new array has 10 elements.
let arr: Array = Activator.CreateInstance(arrType, 10) :?> Array
// Prove that an array of 10 elements has been created.
Console.WriteLine(@"{0} {1} elements", arr.Length, arr.GetValue(0).GetType.Name)
```

When you work with an array created using reflection, you typically assign its elements with the `SetValue` method and read them back with the `GetValue` method:

``` F#
// Assign the first element and read it back.
arr.SetValue(123.45, 0)
Console.WriteLine(arr.GetValue(0))             // => 123.45
```

### Accessing Members

In the most general case, after you've created an instance by using reflection, all you have is an Object variable pointing to a type and no direct way to access one of its members. The easiest operation you can perform is reading or writing a field by means of the `GetValue` and `SetValue` methods of the `FieldInfo` object:

``` F#
// Create a Person object and reflect on it.
let myType: Type = Assembly.GetExecutingAssembly().GetType(@"MyApp.Person")

let args: Object[] = [|@"Joe", @"Evans"|]
let o: Object = Activator.CreateInstance(myType, args)

// Get a reference to its FirstName field.
let fi: FieldInfo = myType.GetField(@"FirstName")
// Display its current value, and then change it.
Console.WriteLine(fi.GetValue(o))       // => Joe
fi.SetValue(o, @"Robert")

// Prove that it changed, by casting to a strong-myType variable.
let pers: Person = o :?> Person
Console.WriteLine(pers.FirstName)       // => Robert
```

Like `FieldInfo`, the `PropertyInfo` type exposes the `GetValue` and `SetValue` methods, but properties can take arguments, and thus these methods take an array of arguments. You must pass `null` in the second argument if you're calling parameterless properties.

``` F#
// (Continuing previous example…)
// This code assumes that the Person myType exposes a 16-bit Age property.
// Get a reference to the PropertyInfo object.
let pi: PropertyInfo = myType.GetProperty(@"Age")
// Note that the myType of value must match exactly.
// (Int32 constants must be converted to Short, in this case.)
pi.SetValue(pers, 35S, null)
// Read it back.
Console.WriteLine(pi.GetValue(pers, null)) // => 35
```

If the property takes one or more arguments, you must pass an Object array containing one element for each argument:

``` F#
// Get a reference to the PropertyInfo object.
let pi2: PropertyInfo = myType.GetProperty(@"Notes")
// Prepare the array of parameters.
let args2: Object[] = [|1|]
// Set the property.
pi2.SetValue(o, @"Tell John about the briefing", args2)
// Read it back.
Console.WriteLine(pi2.GetValue(o, args2))
```

A similar thing happens when you're invoking methods, except that you use the Invoke method instead of `GetValue` or `SetValue`:

``` F#
// Get the MethodInfo for this method.
let mi: MethodInfo = myType.GetMethod(@"SendEmail")
// Prepare an array for expected arguments.
let arguments: Object[] = [|@"This is a message", 3|]
// Invoke the method.
mi.Invoke(o, arguments)
```

Things are more interesting when optional arguments are involved. In this case, you pass the `Type.Missing` special value, as in this code:

``` F#
// …(Initial code as above)…
// Don't pass the second argument (optional).

arguments = [|@"This is a message", myType.Missing|]
mi.Invoke(o, arguments)// Don't pass the second argument.
```

Alternatively, you can query the `DefaultValue` property of corresponding `ParameterInfo` to learn the default value for that specific argument:

``` F#
// …(Initial code as above)…
// Retrieve the DefaultValue from the ParameterInfo object.
arguments = [|@"This is a message", mi.GetParameters(1).DefaultValue|]
mi.Invoke(o, arguments)
```

The Invoke method traps all the exceptions thrown in the called method and converts them into `TargetInvocationException`; you must check the `InnerException` property of the caught exception to retrieve the real exception:

``` F#
try
   mi.Invoke(o, arguments)
with | :? TargetInvocationException as ex -> 
   Console.WriteLine(ex.InnerException.Message)
with | :? Exception as ex -> 
   Console.WriteLine(ex.Message)
```

### The InvokeMember Method

In some cases, you might find it easier to set properties dynamically and invoke methods by means of the Type object's `InvokeMember` method. This method takes the name of the member; a flag that says whether it's a field, property, or method; the object for which the member should be invoked; and an array of Objects for the arguments if there are any. Here are a few examples:

``` F#
// Create an instance of the Person myType using InvokeMember.
let myType: Type = Assembly.GetExecutingAssembly().GetType(@"MyApp.Person")
let arguments: Object[] = [|@"John", @"Evans"|]
let obj: Object = myType.InvokeMember(@"", BindingFlags.CreateInstance, null, null, arguments)

// Set the FirstName field.
let args: Object[] = [|@"Francesco"|]        // One argument
myType.InvokeMember(@"FirstName", BindingFlags.SetField, null, obj, args)
// Read the FirstName field. (Pass null for the argument array.)
let value: Object = myType.InvokeMember(@"FirstName", BindingFlags.GetField,  null, obj, null)

// Set the Age property, create the argument array on the fly.
myType.InvokeMember(@"Age", BindingFlags.SetProperty, null, obj,  [|35S|])

// Call the SendEMail method.
let args2: Object[] = [|@"This is a message", 2|]
myType.InvokeMember(@"SendEmail", BindingFlags.InvokeMethod, null, obj, args2)
```

It is very important that you pass the correct value for the `BindingFlags` argument. All the examples shown so far access public instance members, but you must explicitly add the `NonPublic` and/or Static modifiers if the member is private or static:

``` F#
// Read the m_Age private field.
value = myType.InvokeMember(@"m_Age", BindingFlags.GetField ||| BindingFlags.NonPublic  ||| BindingFlags.Instance, null, obj, null)
```

When you invoke a static member, you must pass `null` in the second to last argument. The same rule applies when you use `InvokeMember` to call a constructor method because you don't yet have a valid instance in that case.

The `InvokeMember` method does a case-sensitive search for the member with the specified name, but it's quite forgiving when it matches the type of the arguments because it will perform any necessary conversion for you if the types don't correspond exactly. You can change this default behavior by means of the `BindingFlags.IgnoreCase` (for case-insensitive searches) and the `BindingFlags.ExactBinding` (for exact type matches) values.

`InvokeMember` works correctly if one or more arguments are passed by reference. For example, if the `SendEmail` method would take the priority in a `ByRef` argument, on return from the method call the args2(1) element would contain the new value assigned to that argument.

Even though `InvokeMember` can make your code more concise—because you don't have to get a reference to a specific `FieldInfo`, `PropertyInfo`, or `MethodInfo` object—it surely doesn't make your code faster. In fact, the `InvokeMember` method must perform two distinct operations internally: the discovery phase (looking for the member with the specified signature) and the execution phase. If you use `InvokeMember` to call the same method a hundred times, it will "rediscover" the same method a hundred times, which clearly adds overhead that you can avoid if you reflect on the member once and then access the member through a `FieldInfo`, `PropertyInfo`, or `MethodInfo` object. For this reason, you shouldn't use `InvokeMember` when repeatedly accessing the same member, especially in time-critical code.

### Creating a Universal Comparer

As you might recall from Chapter 10, "Interfaces," you implement the `IComparer` interface in auxiliary classes that work as comparers for other types, whether they are .NET types or custom types you've defined. The main problem with comparer types is that you must define a distinct comparer type for each possible sort criterion. Clearly, this requirement can soon become a nuisance. Reflection gives you the opportunity to implement a *universal comparer*, a class capable of working with any type of object and any combination of fields and properties and that supports both ascending and descending sorts.





Before discussing how the `UniversalComparer` type works, let me show you how you can use it. You create a `UniversalComparer` instance by passing its constructor a string argument that resembles an ORDER BY clause in SQL.

``` F#
let persons: Person[] = null
// Init the array here.
…
// Sort the array on the LastName and FirstName fields.
let comp = new UniversalComparer<Person>(@"LastName, FirstName ")
Array.Sort<Person>(persons, comp)
```

You can even sort in descending mode separately on each field:

``` F#
let comp = new UniversalComparer<Person>(@"LastName DESC, FirstName DESC")
Array.Sort<Person>(persons, comp)
```

Not surprisingly, the `UniversalComparer` class relies heavily on reflection to perform its magic. Here's its complete source code:

``` F#
class UniversalComparer<'T>() =
   Implements IComparer, IComparer<'T>

   Private sortKeys() As SortKey

   let New(sort : String) =
      let myType: Type = typeof<'T>
      // Split the list of properties.
      let props: String[] = sort.Split(',')
      // Prepare the array that holds information on sort criteria.
      ReDim sortKeys(props.Length - 1)

      // Parse the sort string.
      for i: Int32 in [0..props.Length - 1] do
         // Get the Nth member name.
         let memberName: String = props(i).Trim()
         if memberName.ToLower().EndsWith(@" desc") then
            // Discard the DESC qualifier.
            sortKeys(i).Descending = true
            memberName = memberName.Remove(memberName.Length - 5).TrimEnd()
         
         // Search for a field or a property with this name.
         sortKeys(i).FieldInfo = myType.GetField(memberName)
         if sortKeys(i).FieldInfo = null then
            sortKeys(i).PropertyInfo = myType.GetProperty(memberName)
         
      
   

   // Implementation of IComparer.Compare
   let Compare(o1 : Object, o2 : Object) : Int32  Implements IComparer.Compare =
      Compare(CType(o1, 'T), CType(o2, 'T))
   

   // Implementation of IComparer<'T>.Compare
   let Compare(o1 : 'T, o2 : 'T) : Int32  Implements IComparer<'T>.Compare =
      // Deal with simplest cases first.
      if o1 = null then
         // Two null objects are equal.
         if o2 = null then Return 0
         // A null object is less than any non-null object.
         -1
      elif o2 = null then
         // Any non-null object is greater than a null object.
         1
      

      // Iterate over all the sort keys.
      for i: Int32 in [0..sortKeys.Length - 1] do
         let value1: Object, value2 As Object
         let sortKey: SortKey = sortKeys(i)
         // Read either the field or the property.
         if sortKey.FieldInfo <> null then
            value1 = sortKey.FieldInfo.GetValue(o1)
            value2 = sortKey.FieldInfo.GetValue(o2)
         else
            value1 = sortKey.PropertyInfo.GetValue(o1, null)
            value2 = sortKey.PropertyInfo.GetValue(o2, null)
         

         let res: Int32
         if value1 = null &&& value2 = null then
            // Two null objects are equal.
            res = 0
         elif value1 = null then
            // A null object is always less than a non-null object.
            res = -1
         elif value2 = null then
            // Any object is greater than a null object.
            res = 1
         else
            // Compare the two values, assuming that they support IComparable.
            res = value1 :?> IComparable).CompareTo(value2
         

            // If values are different, return this value to caller.
            if res <> 0 then
               // Negate it if sort direction is descending.
               if sortKey.Descending then res = -res
               res
            
         
         // If we get here, the two objects are equal.
         0
      

      // Nested myType to store detail on sort keys
      Private Structure SortKey
         Public FieldInfo As FieldInfo

      Public PropertyInfo As PropertyInfo
      // true if sort is descending.
      Public Descending As Boolean
```

As the comments in the source code explain, the universal comparer supports comparisons on both fields and properties. Because this class uses reflection intensively, it isn't as fast as a more specific comparer can be, but in most cases the speed difference isn't noticeable.

### Dynamic Registration of Event Handlers

Another programming technique you can implement through reflection is the dynamic registration of an event handler. For example, let's say that the `Person` class exposes a `GotEmail` event and you have an event handler in the `MainModule` type:

``` F#
class Person() =
   Event GotEmail(sender : Object, e : EventArgs)
   …
   // A method that fires the GotEmail event
   let SendEmail(text : String, Optional priority : Int32 = 1) =
      …
      let e = new GotEmailEventArgs(text, priority)
      RaiseEvent GotEmail(Me, e)
   


module MainModule =
   let GotEmail_Handler(sender : Object, e : GotEmailEventArgs) =
      Console.WriteLine(@"GotEmail event fired")
   
   …
```

Here's the code that registers the procedure for this event, using reflection exclusively:

``` F#
// obj and myType initialized as in previous examples…
// Get a reference to the GotEmail event.
let ei: EventInfo = myType.GetEvent(@"GotEmail")
// Get a reference to the delegate that defines the event.
let handlerType: Type = ei.EventHandlerType
// Create a delegate of this myType that points to a method in this module.
let handler: Delegate = Delegate.CreateDelegate( handlerType, typeof<MainModule>, @"GotEmail_Handler")
// Register this handler dynamically.
ei.AddEventHandler(obj, handler)
// Call the method that fires the event, using reflection.
Dim args() as Object = {@"Hello Joe", 2}
Type.InvokeMember(@"SendEmail", BindingFlags.InvokeMethod, null, obj, args)
```

A look at the console window proves that the `EventHandler` procedure in the `MainModule` type was invoked when the code in the `Person.SendEmail` method raised the `GotEmail` event. If the event handler is an instance method, the second argument to the `Delegate.CreateDelegate` method must be an instance of the class that defines the method; if the event handler is a static method (as in the previous example), this argument must be a `Type` object corresponding to the class where the method is defined.

The previous code doesn't really add much to what you can do by registering an event by means of the `AddHandler` operator. But wait, there's more. To show how this technique can be so powerful, I must make a short digression on delegates.

#### Delegate Covariance and Contravariance in C# 2.0

.NET Framework 2.0 has enhanced delegates with two important features: covariance and contravariance. Unfortunately, however, these features are available only in C#, and therefore I am forced to illustrate these concepts in that language.

Both these features relax the requirement that a delegate object must match exactly the signature of its target method. More specifically, delegate covariance means that you can have a delegate point to a method with a return value that inherits from the return type specified by the delegate. Let's say we have the following delegate:

``` C#
// A delegate that can point to a method that takes a TextBox and returns an object.
delegate object GetControlData(TextBox ctrl);
```

The `GetControlData` delegate specifies object as the return value; therefore, the covariance property tells that this delegate can point to any method that takes a `TextBox` control, regardless of the method's return value, because all .NET types inherit from `System.Object`. The only requirement is that the method actually returns something; therefore, you can't have this delegate point to a C# void method ~~(a Sub method, in Visual Basic parlance)~~. For example, a `GetControlData` delegate might point to the following method because the `String` type inherits from `System.Object`:

``` F#
// A function that takes a TextBox control and returns a String
string GetText(TextBox ctrl)
{ return ctrl.Text; }
```

Delegate contravariance means that a delegate can point to a method with an argument that is a base class of the argument specified in the delegate's signature. For example, a `GetControlData` delegate might point to a method that takes one argument of the `Control` or `Object` type because both these types are base classes for the `TextBox` argument that appears in the delegate:

``` F#
// A function that takes a Control and returns an Object value.
object GetTag(Control ctrl)
{ return ctrl.Tag; }
```

It's important to realize that covariance and contravariance relax the constraint that a delegate can point only to a method with a signature that doesn't exactly match the delegate's signature, but they don't make the code less robust because no type mismatch exception can occur at run time.





#### Delegate Covariance and Contravariance in Visual Basic 2005

Don't look for delegate covariance and contravariance in Visual Basic documentation because you won't find any information. As a matter of fact, Visual Basic 2005 doesn't support these features. Period.

Well, not exactly. Granted, Visual Basic doesn't support these features directly, but you can achieve them nevertheless. Covariance and contravariance are supported at the CLR level, and you can create a delegate that leverages both of them through reflection. Let's say you have the following delegate and the following method in a Windows Forms class:

``` F#
Delegate Function GetControlData(ctrl : TextBox) As Object

let GetText(ctrl : Control) : String =
   ctrl.Text
```

You know that you can't create a `GetControlData` delegate that points to the `GetText` method directly in Visual Basic because the language supports neither covariance nor contravariance. However, you can create a `MethodInfo` object that points to the `GetText` method and then pass this object to the `Delegate.CreateDelegate` static method:

``` F#
// The target method
let method: MethodInfo = Me.GetType().GetMethod(@"GetText")
// Build the delegate through reflection.
let deleg: GetControlData = Delegate.CreateDelegate( typeof<GetControlData>, Me, method) :?> GetControlData
// Show that the delegate works correctly.
Console.WriteLine(deleg(Me.TextBox1)) // Displays the TextBox1.Text property.
```

This code is only marginally slower than the C# counterpart, but this isn't a serious issue because you typically create a delegate once and use it repeatedly.

The most interesting application of this feature is the ability to have an individual method handle all the events coming from one or more objects, provided that the event has the canonical .NET syntax `(sender, e)`, where the second argument can be any type that derives from `EventArgs`. Consider the following event handler:

``` F#
// (Inside a Form class)
let MyEventHandler(sender : Object, e : EventArgs) =
   Console.WriteLine(@"An event has fired")
```

The following code can make all the events exposed by an object point to the "universal handler":

``` F#
// (Inside the same Form class…)
// The control we want to trap events from
let ctrl: Object = TextBox1
for ei: EventInfo in ctrl.GetType().GetEvents() do
   let handlerType: Type = ei.EventHandlerType

   // The universal event handler method
   let method: MethodInfo = Me.GetType().GetMethod(@"MyEventHandler")
   // Leverage contravariance to create a delegate that points to the method.
   let handler: Delegate = Delegate.CreateDelegate(handlerType, Me, method)
   // Use reflection to register the event.
   ei.AddEventHandler(ctrl, handler)

// Prove that it works by causing a TextChanged event.
ctrl.Text <- Text + @"*"
```

This code proves that it is technically possible to use reflection to have all the events of an object point to an individual handler, but this technique doesn't look very promising. After all, the `MyEventHandler` method has no means to understand which event was fired. To get that information, we need to do more.

#### A Universal Event Handler

What we need is an object that is able to "mediate" between the event source and the object where the event is handled. Writing this object requires some significant code, but the result is well worth the effort. The `EventInterceptor` class exposes only one event, `ObjectEvent`, defined by the `ObjectEventHandler` delegate:

``` F#
Public Delegate Sub ObjectEventHandler(sender : Object, e : ObjectEventArgs)

class EventInterceptor() =
   // The public event
   Public Event ObjectEvent As ObjectEventHandler

   // This is invoked from inside the EventInterceptorHandler auxiliary class.
   Protected Sub OnObjectEvent(e : ObjectEventArgs)
      RaiseEvent ObjectEvent(Me, e)
   
   …
```

The `EventInterceptor` class uses the nested `EventInterceptorHandler` type to trap events coming from the object source. More precisely, an `EventInterceptorHandler` instance is created for each event that the event source can raise. The `EventInterceptor` class supports multiple event sources; therefore, the number of `EventInterceptorHandler` instances can be quite high: for example, if you trap the events coming from 20 TextBox controls, the `EventInterceptor` object will create as many as 1,540 `EventInterceptorHandler` instances because each TextBox control exposes 77 events. For this reason, the `AddEventSource` method supports a third argument that enables you to specify which events should be intercepted:

``` F#
let AddEventSource(eventSource : Object, includeChildren : Boolean, filterPattern : String) =
   for ei: EventInfo in eventSource.GetType().GetEvents() do
      // Skip this event if its name doesn't match the pattern.
      if not <| String.IsNullOrEmpty(filterPattern) && not <|  Regex.IsMatch(ei.Name, @"^" + filterPattern + @"$") then Continue For

      // Get the signature of the underlying delegate.

         let mi: MethodInfo = ei.EventHandlerType.GetMethod(@"Invoke")
         let pars: ParameterInfo[] = mi.GetParameters()
         // Check that event signature is in the form (sender, e).
         if mi.ReturnType.FullName = @"System.Void" && pars.Length = 2  && pars(0).ParameterType = typeof<Object> &&  typeof<EventArgs>.IsAssignableFrom(pars(1).ParameterType) then
            // Create an EventInterceptorHandler that handles this event.
            let interceptor = new EventInterceptorHandler(eventSource, ei, Me)
         
      
      // Recurse on child controls if so required.
      if eventSource.GetType() = Control && includeChildren then
         for ctrl: Control in eventSource :?> Control.Controls do
            AddEventSource(ctrl, includeChildren, filterPattern)
         
      
   
   …
    // End of EventInterceptor class
```

The `EventInterceptorHandler` nested class does a very simple job: it uses reflection to register its `EventHandler` method as a listener for the specified event coming from the specified event source. When the event is fired, the `EventHandler` method calls back the `OnObjectEvent` method in the parent `EventInterceptor` object, which in turn fires the `ObjectEvent` event:

``` F#
class EventInterceptorHandler() =
   // The event being intercepted
   Public ReadOnly EventInfo As EventInfo
   // The parent EventInterceptor
   Public ReadOnly Parent As EventInterceptor

   let New(eventSource : Object, eventInfo : EventInfo,  parent : EventInterceptor) =
      Me.EventInfo = eventInfo
      Me.Parent = parent
      // Create a delegate that points to the EventHandler method.
      let method: MethodInfo = Me.GetType().GetMethod(@"EventHandler")
      let handler: Delegate =  Delegate.CreateDelegate(eventInfo.EventHandlerType, Me, method)
      // Register the event.
      eventInfo.AddEventHandler(eventSource, handler)
   

   let EventHandler(sender : Object, e : EventArgs) =
      // Notify the parent EventInterceptor object.
      let objEv = new ObjectEventArgs(sender, EventInfo.Name, e)
      Parent.OnObjectEvent(objEv)
```

Here's how a Windows Forms application can use the `EventInterceptor` object to get a notification when any event of any control fires:

``` F#
Dim WithEvents Interceptor As New EventInterceptor

let Form1_Load(sender : Object, e : EventArgs) Handles MyBase.Load =

   Interceptor.AddEventSource(Me, true, @"")


let Interceptor_ObjectEvent(sender : Object, e : ObjectEventArgs)  Handles Interceptor.ObjectEvent =
   let msg: String = String.Format(@"Event {0} from control {1}",  e.EventSource, Control).Name :?> e.EventName
   Debug.WriteLine(msg)
```

You can limit the number of events that you receive by passing a regular expression pattern to the `AddEventSource` method:

``` F#
// Trap only xxxChanged events.
Interceptor.AddEventSource(Me, true, @".+Changed")
// Trap only mouse and keyboard events.
Interceptor.AddEventSource(Me, true, @"(Mouse|Key).+)")
```

For more information, see the source code of the complete demo program. (See Figure 18-2.)

 ![img](C:/Program Files/Typora/images/fig794_01.jpg)
Figure 18-2: The `EventInterceptor` demo application

### Scheduling a Sequence of Actions

Reflection allows you to implement techniques that would be very difficult (and sometimes impossible) to implement using a more traditional approach. For example, the ability to invoke methods through `MethodInfo` objects gives you the ability to deal with call sequences as if they were just another data type that your application processes. To see what I mean, let's say that your application must perform a series of actions—for example, file creation, registry manipulation, variable assignments—as an atomic operation. If any one of the involved actions fails, all the actions performed so far should be undone in an orderly manner: files should be deleted, registry keys should be restored, variables should be assigned their original value, and so forth.

Implementing an undo strategy in the most general case isn't a simple task, especially if some of the actions are performed only conditionally when other conditions are met. What you need is a generic solution to this problem, and you'll see how elegantly you can solve this programming task through reflection. To begin with, let's define an Action class, which represents a method—either a static or an instance method:

``` F#
class Action() =
   Public ReadOnly Message As String       // Description of the action
   Public ReadOnly [Object] As Object      // Instance on which the method is called
   Public ReadOnly Method As MethodInfo    // The method to be invoked
   Public ReadOnly Arguments() As Object   // Arguments for the method

   // Second argument can be an object (for instance methods) or a Type (for static methods).
   let New(message As String, obj : Object,  methodName : String, ByVal ParamArray arguments : Object[]) =
      Me.Message = message
      Me.Arguments = arguments
      // Determine the myType this method belongs to.
      let myType: Type = TryCast(obj, Type)
      if myType = null then
         Me.Object = obj
         myType = obj.GetType()
      
      // Prepare the list of argument types, to call GetMethod without any ambiguity.
      Dim argTypes(arguments.Length - 1) As Type
      for index: Int32 in [0..arguments.Length - 1] do
         if arguments(index) <> null then
            argTypes(index) = arguments(index).GetType()
         
      
      // Retrieve the actual MethodInfo object, throw an exception if not found.
      Me.Method = myType.GetMethod(methodName, argTypes)
      if Me.Method = null then
         raise <| new ArgumentException(@"Missing method")
      
   

   // Execute this method.
   let Execute() =
      Me.Method.Invoke(Me.Object, Me.Arguments)
```

Instead of performing a method directly, you can create an Action instance and then invoke its Execute method:

``` F#
// Create c:\backup directory.
let act = new Action(@"Create c:\backup directory",  typeof<Directory>, @"CreateDirectory", @"c:\backup")
act.Execute()
```

Of course, executing a method in this way doesn't bring any benefit. You see the power of this technique, however, if you define another type that works as a container for Action instances and that can also remember the "undo" action for each method being executed:

``` F#
class ActionSequence() =
   // The parallel lists of actions and undo actions
   Private Actions As New List<Action>

   Private UndoActions As New List<Action>
   // This delegate must point to a method that takes a string.
   Private DisplayMethod As Action<String>

   // The constructor takes a delegate to a method that can output a message.
   let New(displayMethod : Action<String>) =
      Me.DisplayMethod = displayMethod
   

   // Add an action and an undo action to the list.
   let Add(action : Action, undoAction : Action) =
      Actions.Add(action)
      UndoActions.Add(undoAction)
   

   // Insert an action and an undo action at a specific index in the list.
   let Insert(index : Int32, action : Action, undoAction : Action) =
      Actions.Insert(index, action)
      UndoActions.Insert(index, undoAction)
   

   // Execute all pending actions, return true if no exception occurred.
   let Execute(ignoreExceptions : Boolean) : Boolean =
      // This is the list of undo actions to execute in case of error.
      let undoSequence = new ActionSequence(Me.DisplayMethod)

      for index: Int32 in [0..Actions.Count - 1] do
         let act: Action = Actions(index)
         // Skip over null actions.
         if act = null then Continue For

         try
            // Display the message and execute the action.
            DisplayMessage(act.Message)
            act.Execute()
            // If successful, remember the undo action. The undo action is placed
            // in front of all others so that it will be the first to be executed in case of error.
            undoSequence.Insert(0, UndoActions(index), null)
         with | :? TargetInvocationException as ex -> 
            // Ignore exceptions if so required.
            if ignoreExceptions then Continue For
            // Display the error message.
            DisplayMessage(@"ERROR: " + ex.InnerException.Message)
            // Perform the undo sequence. (Ignore exceptions while undoing.)
            DisplayMessage(@"UNDOING OPERATIONS...")
            undoSequence.Execute(true)
            // Signal that an exception occurred.
            false
         
      
      // Signal that no exceptions occurred.
      true
   

   // Report a message through the delegate passed to the constructor.
   let DisplayMessage(text : String) =

      if Me.DisplayMethod <> null then
         Me.DisplayMethod(text)
```

The constructor of the `ActionSequence` type takes an Action(Of `String`) object, which is a delegate to a Sub method that takes a string as an argument. This method will be used to display all the messages that are produced during the action sequence: it can point to a method such as the `Console.WriteLine` method (to display messages in the console window), the `WriteLine` method of a `StreamWriter` object (to write messages to a log file), the `AppendText` method of a `TextBox` control (to display messages inside a `TextBox` control), or a method you define in your application:

``` F#
// Prepare to write diagnostic messages to a log file.
let sw = new StreamWriter(@"c:\logfile.txt")
let actionSequence = new ActionSequence(AddressOf sw.WriteLine)
…
// Close the stream when you're done with the ActionSequence object.
sw.Close()
```

The following code shows how you can schedule a sequence of actions and undo them if an exception is thrown during the process:

``` F#
// Schedule the creation of c:\backup directory.
let actionSequence = new ActionSequence(AddressOf Console.WriteLine)
let act = new Action(@"Create c:\backup directory", typeof<Directory>,  @"CreateDirectory", @"c:\backup")
let undoAct = new Action(@"Delete c:\backup directory", typeof<Directory>,  @"Delete", @"c:\backup", true)
actionSequence.Add(act, undoAct)

// Create a readme.txt file in the c:\ root directory.
let contents: String = @"Instructions for myapp.exe..."
act = New Action(@"Create the c:\myapp_readme.txt", typeof<File>,  @"WriteAllText", @"c:\myapp_readme.txt", contents)
// Notice that this action has no undo action.
actionSequence.Add(act, null)

// Move the readme file to the backup directory.
act = New Action(@"Move the c:\myapp_readme.txt to c:\backup\readme.txt", typeof<File>,  @"Move", @"c:\myapp_readme.txt", @"c:\backup\readme.txt")
undoAct = New Action(@"Move c:\backup\readme.txt to c:\myapp_readme.txt", typeof<File>,  @"Move", @"c:\backup\readme.txt", @"c:\myapp_readme.txt")
actionSequence.Add(act, undoAct)

// Execute the action sequence. (false means that an exception undoes all actions.)
actionSequence.Execute(false)
```

After running the previous code snippet, you should find a new c:\backup directory containing the files readme.txt and win.ini. To see how the `ActionSequence` type behaves in case of error, delete the c:\backup directory and intentionally cause an error in the sequence by attempting to copy a file that doesn't exist:

``` F#
// (Insert the lines in bold myType before the call to the Execute method.)
…
// Copy the c:\missing.txt file to the c:\backup directory.
act = New Action(@"Copy c:\missing.txt to c:\backup", typeof<File>,  @"Copy", @"c:\missing.text", @"c:\backup\missing.txt")
undoAct = New Action(@"Delete c:\backup\missing.text", typeof<File>,  @"Delete", @"c:\backup\missing.text")
actionSequence.Add(act, null)

// Execute the action sequence. (false means that an exception undoes all actions.)
actionSequence.Execute(false)
```

The intentional error causes the `ActionSequence` object to undo all the actions before the one that caused the exception, and in fact at the end of the process you won't find any c:\backup directory, as confirmed by the text that appears in the console window:

``` F#
Create c:\backup directory
Create the c:\myapp_readme.txt
Move the c:\myapp_readme.txt to c:\backup\readme.txt
Copy c:\missing.txt to c:\backup
ERROR: Could not find file 'c:\missing.text'.
UNDOING OPERATIONS...
Move the c:\backup\readme.txt file back to c:\myapp_readme.txt
Delete c:\backup directory
```

In this example, I used the `ActionSequence` type to undo a sequence of actions that are hard-coded in the program, but I could have used a similar technique to implement an Undo menu command in your applications. Or I could have read the series of actions from a file instead, to implement undoable scripts.

You can expand the `ActionSequence` type with new features. For example, you might have the series of actions be performed on a background thread (see Chapter 20, "Threads," for more information) and specify multiple undo methods for a given action. You might add properties to the Action type to specify whether a failed method should abort the entire sequence. Also, you might extend the Action class with the ability to create new instances (that is, to call constructors in addition to regular methods) and to pass instances created in this way as arguments to other methods down in the action sequence. As usual, the only limit is your imagination.

### On-the-Fly Compilation

Earlier in this chapter, I mentioned the `System.Reflection.Emit` namespace, which has classes that let you create an assembly on the fly. The .NET Framework uses these classes internally in a few cases—for example, when you pass the `RegexOptions.Compiled` option to the constructor of the `Regex` object (see Chapter 14, "Regular Expressions"). Using reflection emit, however, isn't exactly the easiest .NET programming task, and I'm glad I've never had to use it heavily in a real-world application.





Nevertheless, at times the ability to create an assembly out of thin air can be quite tantalizing because it opens up a number of programming techniques that are otherwise impossible. For example, consider building a routine that takes a math expression entered by the end user (as a string), evaluates it, and returns the result. In the section titled "Parsing and Evaluating Expressions" in Chapter 14, I showed how you can parse and evaluate an expression at run time, but that approach is several orders of magnitude slower than evaluating a compiled expression is and can't be used in time-critical code, such as for doing function plotting or finding the roots of a higher-degree equation (see Figure 18-3). In this case, your best option is to generate the source code of a Visual Basic program, compile it on the fly, and then instantiate one of its classes.

![img](C:/Program Files/Typora/images/fig799_01.jpg)
Figure 18-3: The demo application, which uses on-the-fly compilation to evaluate functions and find the roots of any equation that uses the X variable

The types that allow us to compile an assembly at run time are in the `Microsoft.VisualBasic` namespace (or in the `Microsoft.CSharp` namespace, if you want to generate and compile C# source code) and in the `System.`CodeDom`.Compiler` namespace, so you need to add proper Imports statements to your code to run the code samples that follow.

The first thing to do is generate the source code for the program to be compiled dynamically. In the expression evaluator demo application, such source code is obtained by inserting the expression that the end user enters in the `txtExpression` field in the middle of the Eval method of an Evaluator public class:

``` F#
let source: String = String.Format( @"Imports Microsoft.VisualBasic{0}"  + @"Imports System.Math{0}"  + @"Public Class Evaluator{0}"  + @"   Public Function Eval(x : Double) As Double{0}"  + @"      Return {1}{0}"  + @"   End Function{0}"  + @"End Class{0}",  ControlChars.CrLf, txtExpression.Text)
```

Next, you create a `CompilerParameters` object (in the `System.`CodeDom`.Compiler` namespace) and set its properties; this object broadly corresponds to the options you'd pass to the VBC command-line compiler:

``` F#
let params = new CompilerParameters
// Generate a DLL, not an EXE executable.
// (not <| really necessary because false is the default.)
params.GenerateExecutable = false

#If DEBUG Then
   // Include debug information.
   params.IncludeDebugInformation = true
   // Debugging works if we generate an actual DLL and keep temporary files.
   params.TempFiles.KeepFiles = true
   params.GenerateInMemory = false
#Else
   // Treat warnings as errors, don't keep temporary source files.
   params.TreatWarningsAsErrors = true
   params.TempFiles.KeepFiles = false
   // Optimize the code for faster execution.
   params.CompilerOptions = @"/Optimize+"
   // Generate the assembly in memory.
   params.GenerateInMemory = true
#End If
   // Add a reference to necessary strong-named assemblies.
   params.ReferencedAssemblies.Add(@"Microsoft.VisualBasic.Dll")
   params.ReferencedAssemblies.Add(@"System.Dll")
```

The preceding code snippet shows the typical actions you perform to prepare a Compiler-Parameters object, as well as its most important properties. The statements inside the #If block are especially interesting. You can include debug information in a dynamic assembly and debug it from inside Visual Studio by setting the `IncludeDebugInformation` property to `true`. To enable debugging, however, you must generate an actual .dll or .exe file (`GenerateInMemory` must be `false`) and must not delete temporary files at the end of the compilation process (the `KeepFiles` property of the `TempFiles` collection must be `true`). If debugging is correctly enabled, you can force a break in the generated assembly by inserting the following statement in the code you generate dynamically:

``` F#
System.Diagnostics.Debugger.Break()
```

You are now ready to compile the assembly:

``` F#
// Create the VB compiler.
let provider = new VBCodeProvider
let compRes: CompilerResults = provider.CompileAssemblyFromSource(params, source)

// Check whether we have errors.
if compRes.Errors.Count > 0 then
   // Gather all error messages and display them.
   let msg: String = @""
   for compErr: CompilerError in compRes.Errors do
      msg <- msg + compErr.ToString + ControlChars.CrLf
   
   MessageBox.Show(msg, @"Compilation Failed", MessageBoxButtons.OK, MessageBoxIcon.Error)
else
   // Compilation was successful.
   …
```

If the compilation was successful, you use the `CompilerResults.CompiledAssembly` property to get a reference to the created assembly. Once you have this Assembly object, you can create an instance of its Evaluator class and invoke its Eval method by using standard reflection techniques:

``` F#
let asm: Assembly = compRes.CompiledAssembly
let evaluator: Object = asm.CreateInstance(@"Evaluator")
let evalMethod: MethodInfo = evaluator.GetType.GetMethod(@"Eval")
let args: Object[] = [|float (123)|]    // Pass x = 123
let result: Object = evalMethod.Invoke(evaluator, args)
```

Notice that you can't reference the Evaluator class by a typed variable because this class (and its container assembly) doesn't exist yet when you compile the main application. For this reason, you must use reflection both to create an instance of the class and to invoke its members.

Another tricky thing to do when applying this technique is to have the dynamic assembly call back a method in a class defined in the main application by means of reflection, for example, to let the main application update a progress bar during a lengthy routine. Alternatively, you can define a public interface in a DLL and must have the class in the main application implement the interface; being defined in a DLL, the dynamic assembly can create an interface variable and therefore it can call methods in the main application through that interface.

You must be aware of another detail when you apply on-the-fly compilation in a real application: once you load the dynamically created assembly, that assembly will take memory in your process until the main application ends. In most cases, this problem isn't serious and you can just forget about it. But you can't ignore it if you plan to build many assemblies on the fly. The only solution to this problem is to create a separate `AppDomain`, load the dynamic assembly in that `AppDomain`, use the classes in the assembly, and finally unload the `AppDomain` when you don't need the assembly any longer. On the other hand, loading the assembly in another `AppDomain` means that you can't use reflection to manage its types (reflection works only with types in the same `AppDomain` as the caller). Please see the demo application in the companion code for a solution to this complex issue.

### Performance Considerations

I have warned about the slow performance of reflection-based techniques often in this chapter. Using reflection to invoke methods is similar to using the late-binding techniques that are available in script languages such as Microsoft Visual Basic Scripting Edition (`VBScript`), in Visual Basic 6 when you use a Variant variable, or even in Visual Basic 2005 when you invoke a method using an Object variable and Option Strict is Off.

In general, invoking a method by using reflection is many times slower than a direct call is, and therefore you shouldn't use these techniques in time-critical portions of your application. In some scenarios, however, you need to defer the decision about which method to call until run time, and therefore a direct call is out of the question. Even then, reflection should be your last resort and should be used only if you can't solve the problem with another technique based on indirection, for example, an interface or a delegate.





If you decide to use reflection and you must invoke a method more than once or twice, you should use `Type.GetMethod` to get a reference to a `MethodInfo` object and then use the `MethodInfo.Invoke` method to do the actual call, rather than using the `Type.InvokeMember` method because the former technique requires that you perform the discovery phase only once.

Don't use the `BindingFlags.IgnoreCase` value with the Get*Xxxx* method (singular form), if you know the exact spelling of the member you're looking for, and specify the `BindingFlags` .`ExactBinding` value if possible because it speeds up the search. The latter flag suppresses implicit type conversions; therefore, you must supply the exact type of each argument:

``` F#
// This code doesn't work—the GetMethod method returns null.
// You must either use Int32 instead of Short in the argTypes signature
// or drop the BindingFlags.ExactBinding bit in the GetMethod call.
let argTypes: Type[] = [|typeof<Char>, typeof<Short>|]
let mi: MethodInfo = typeof<String>.GetMethod(@"IndexOf", BindingFlags.ExactBinding |||  BindingFlags.Public ||| BindingFlags.Instance, null, argTypes, null)
```

**Version 2005 of VB or Version 2.0 of .NET** .NET Framework 2.0 improves performance in many ways. For example, in previous versions of the .NET Framework, a call to the `Type.Get`*Xxxx* (singular) adds a noticeable overhead because all the type's members are queried anyway, as if `Type.GetMembers` were called. The results from this first call are cached, so at least you pay this penalty only once, but this approach has a serious issue: if you reflect on many types, all the resulting `MemberInfo` objects are kept in memory and are never discarded until the application terminates.

In .NET Framework 2.0, a `Type.Get`*Xxxx* method (singular form) doesn't cause the exploration of the entire Type object, and therefore execution is faster and memory consumption is kept to a minimum. Also, the cache used by reflection is subject to garbage collection; therefore, type and member information is discarded unless you keep it alive by storing a reference in a Type or `MemberInfo` field at the class level.

An alternative technique for storing information about a large number of types and members without taxing the memory is based on the `RuntimeTypeHandle` and `RuntimeMethodHandle` classes that you can use instead of the Type and `MemberInfo` classes. Handle-based objects use very little memory, yet they allow you to rebuild a reference to the actual Type or `MemberInfo`-based object very quickly, as this code demonstrates:

``` F#
// Store information about a method in the Person myType.
let hType: RuntimeTypeHandle = typeof<Person>.TypeHandle
let hMethod: RuntimeMethodHandle = typeof<Person>.GetMethod(@"SendEmail").MethodHandle
…
// (Later in the application…)
// Rebuild the Type and MethodBase objects.
let ty: Type = Type.GetTypeFromHandle(hType)
let mb: MethodBase = MethodBase.GetMethodFromHandle(hMethod, hType)
// Use them as needed.
…
```

### Security Issues

A warning about using reflection at run time is in order. As you've seen in previous sections, reflection allows you to access any type and any member, regardless of their scope. Therefore, you can use reflection even to instantiate private types, call private methods, or read private variables.

More precisely, your code can perform these operations if it is fully trusted or at least has `ReflectionPermission`. All the applications that run from the local hard disk have this permission, whereas applications running from the Internet don't. In general, if you don't have `ReflectionPermission`, you can perform only the following reflection-related techniques:

* Enumerate assemblies and modules.
* Enumerate public types and obtain information about them and their public members.
* Set public fields and properties and invoke public members.
* Access and enumerate family (Protected) members of a base class of the calling code.
* Access and enumerate assembly (`internal`) members from inside the assembly in which the calling code runs.

You see that code can access through reflection only those types and members that it can access directly anyway. For example, a piece of code can't access a private field in another type or invoke a private member in another type, even if that type is inside the same assembly as the calling code. In other words, reflection doesn't give code more power than it already has; it just adds flexibility because of the additional level of indirection that it provides.

This discussion on reflection is important for one reason: never rely on the Private scope keyword to hide confidential data from unauthorized eyes because a malicious user might create a simple application that uses reflection to read your data. If the application runs from the local hard disk, it has `ReflectionPermission` and can therefore access all the members of your assembly, regardless of whether it's an EXE or a DLL. (If you also consider that decompiling a .NET assembly is as easy, you see that the only way to protect confidential data is by means of cryptography.)

The ability to invoke nonpublic members can be important in some scenarios. For example, in the section titled "The `ICloneable` Interface" in Chapter 10, I show how a type can implement the Clone method by leveraging the `MemberwiseClone` protected method, but in some cases you'd like to clone an object for which you don't have any source code. Provided that your application runs in full trust mode and has `ReflectionPermission`, you can clone any object quite easily with reflection. Here's a reusable routine that performs a (shallow) copy of the object passed as an argument:

``` F#
let CloneObject<'T>(obj : 'T) : 'T =
   if obj = null then
      // Cloning a null object is easy.
      null

   elif TypeOf CObj(obj) = ICloneable then
      // Take advantage of the ICloneable interface, if possible.
      let iclone: ICloneable = obj :?> ICloneable
      CType(iclone.Clone(), 'T)
   else
      // Use reflection if everything else failed.
      // (Throws if application doesn't have ReflectionPermission.)
      let mi: MethodInfo = obj.GetType().GetMethod(@"MemberwiseClone",  BindingFlags.ExactBinding ||| BindingFlags.NonPublic ||| BindingFlags.Instance)
      CType(mi.Invoke(obj, null), 'T)
```

Assuming that you have a Person type—with the usual `FirstName`, `LastName`, and Spouse properties—the following code tests that the `CloneObject` method works correctly:

``` F#
let john = new Person(@"John", @"Evans")
let ann = new Person(@"Ann", @"Beebe")
john.Spouse = ann
ann.Spouse = john
// We need no CType or DirectCast, thanks to generics.
let john2: Person = CloneObject(john)
// Prove that it worked.
Console.WriteLine(@"{0} {1}, spouse is {2} {3}", john2.FirstName, john2.LastName,  john2.Spouse.FirstName, john2.Spouse.LastName)
   // => John Evans, spouse is Ann Beebe
```