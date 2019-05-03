# Chapter 9: Object Lifetime



## Overview

An important facet of Microsoft .NET programming is understanding how the Common Language Runtime (CLR) allocates and releases memory, and above all how objects are destroyed at the end of their life cycle. In this chapter, you'll learn how you can improve your applications' performance remarkably.



## The Need for Garbage Collection

Microsoft Visual Basic classes don't have destructor methods. In other words, no method or event in the class fires when the instance is destroyed. This is one of the most controversial features of the framework and was discussed for months in forums and news groups while the .NET Framework was in beta version.

### COM and the Reference Counter

Before exploring the .NET way of dealing with object destruction, let's see how Microsoft Visual Basic 6 objects (and COM objects in general) behave in this respect. All COM objects maintain a memory location known as the *reference counter*. An object's reference counter is set to 1 when the object is created and a reference to it is assigned to a variable; the object's reference counter is incremented by 1 when a reference to the object is assigned to another variable. Finally, the object's reference counter is decremented when a variable that points to the object is set to `null`. This mechanism is hidden from Visual Basic 6 developers and is implemented behind the scenes through the `AddRef` and `Release` methods of the `IUnknown` interface, an interface that all COM objects must expose. More specifically, Visual Basic 6 calls the `AddRef` method for you when you assign an object reference to a variable with the `Set` keyword. It calls the `Release` method when you set an object variable to `null`.

At any given moment, a COM object's reference counter contains the number of variables that are pointing to that specific object. When the `Release` method is called, the object checks whether the reference counter is going to be decreased from 1 to 0, in which case the object knows it is no longer required and can destroy itself. (If the object is written in Visual Basic 6, a `Class_Terminate` event fires at this point.) In a sense, a COM object is responsible for its own life. An erroneous implementation of the `AddNew` or `Release` method or an unbalanced number of calls to these methods can be responsible for memory and resource leakage, a serious potential shortcoming in COM applications. (This isn't an issue in Visual Basic applications because this language manages the reference counter automatically, but it can be a source of bugs in languages that require that developers call `AddRef` and `Release` methods manually.) Besides, managing the reference counter and frequently calling the `AddRef` and `Release` methods can be a time-consuming process, which has a negative impact on the application's performance.

Even more important, it frequently happens that two COM objects keep themselves alive, such as when you have two `Person` objects that point to each other through their `Spouse` property. Unless you take some special steps to account for this situation, these objects will be released only when the application terminates, even if the application cleared all the variables pointing to them. This is the notorious *circular reference* problem and is the most frequent cause of memory leakage, even in relatively simple COM applications.

When Microsoft designed the .NET Framework, the designers decided to get rid of reference counting overhead and all the problems associated with it. .NET objects have no reference counter, and there is no counterpart for the `AddRef` and `Release` methods. Creating an object requires that a block of memory be allocated from the managed heap, an area in memory that holds all objects. (I introduce the heap in the section titled "Reference Types and Value Types" in Chapter 2, "Basic Language Concepts.") Assigning an object reference requires storing a 32-bit address in a variable (under 32-bit Microsoft Windows platforms, at least), whereas clearing an object variable requires storing 0 in it. These operations are extremely fast because they involve no method calls.

However, the .NET approach raises an issue that doesn't exist under COM: how can the .NET Framework determine when an object isn't used by the application and can be safely destroyed to free the memory that that object uses in the heap?

### The Garbage Collection Process

The .NET Framework memory management relies on a sophisticated process known as *garbage collection*. When an application tries to allocate memory for a new object and the heap has insufficient free memory, the .NET Framework starts the garbage collection process, which is carried out by an internal object known as the *garbage collector*. Many technical articles use the acronym GC to indicate both garbage collection and the garbage collector; strictly speaking, `System.GC` stands for garbage collector, even though in most cases it can indicate both terms.

The garbage collector visits all the objects in the heap and marks those objects that are pointed to by any variable in the application. (These variables are known as *roots* because they're at the top of the application's object graph.) The garbage collection process is quite sophisticated and recognizes objects referenced indirectly from other objects, such as when you have a `Person` object that references other `Person` instances through its `Children` property. After marking all the objects that can be reached from the application's code, the garbage collector can safely release the remaining (unmarked) objects because they're guaranteed to be unreachable by the application.

Next, the garbage collector compacts the heap and makes the resulting block of free memory available to new objects. Interestingly, this mechanism indirectly resolves the circular reference problem because the garbage collector doesn't mark unreachable objects and therefore correctly releases memory associated with objects pointed to by other objects in a circular reference fashion but not used by the main program.

In most real-world applications, the .NET way of dealing with object lifetime is significantly more efficient than the COM way is—and this is an all-important advantage because everything is an object in the .NET architecture. On the other hand, the garbage collection mechanism introduces a new problem that COM developers don't have: *nondeterministic finalization*. A COM object always knows when its reference counter goes from 1 to 0, so it knows when the main application doesn't need it any longer. When that time arrives, a Visual Basic 6 object fires the `Class_Terminate` event and the code inside the event handler can execute the necessary cleanup chores, such as closing any open files and releasing Win32 resources (pens, brushes, device contexts, and kernel objects). Conversely, a .NET object is actually released sometime after the last variable pointing to it is set to `null`. If the application doesn't create many objects, a .NET object might be collected only when the program terminates. Because of the way .NET garbage collection works, there's no way to provide a .NET class with an event equivalent to `Class_Terminate`, regardless of the language used to implement the class. For this reason, it's often necessary to distinguish between the *logical* destruction of an object (when the application clears the last variable pointing to the object) and its *physical* destruction (when the object is actually removed from memory).

If memory is the only resource an object uses, deferred destruction is never a problem. After all, if the application requires more memory, a garbage collection fires and a block of new memory is eventually made available. However, if the object allocates other types of resources—files, database connections, serial or parallel ports, internal Windows objects—you want to make such releases as soon as possible so that another application can use the resources. In some cases, the problem isn't just a shortage of resources. For example, if the object opens a window to display the value of its properties, you surely want that window to be closed as soon as the object is logically destroyed so that a user can't look at outdated information. Thus, the problem is: how can you run your cleanup code when your .NET object is destroyed?

This question has no definitive answer. A partial solution comes in the form of two special methods: `Finalize` and `Dispose`.

### The Finalize Method

The `Finalize` method is a special method that the garbage collector calls just before releasing the memory allocated to the object, that is, when the object is physically destroyed. It works more or less the same way the `Class_Terminate` event under Visual Basic 6 (or the destructor method in C++) works except that it can be called several seconds (or even minutes or hours) after the application has logically killed the object by setting the last variable pointing to the object to `null` (or by letting the variable go out of scope, which has the same effect). Because all .NET objects inherit the `Finalize` method from the `System.Object` class, this method must be declared using the `Overrides` and `Protected` keywords:

```FSharp
// (Add this to a Person class with usual FirstName and LastName properties.)
override this.Finalize()
   Debug.WriteLine("Person object is being destroyed.")
```

The following application shows that the `Finalize` method isn't called immediately when all variables pointing to the object are set to `null`:

```FSharp
let Main() =
   Debug.WriteLine("About to create a Person object.")
   let mutable aPerson = new Person()
   Debug.WriteLine("About to set the Person object to null.")
   aPerson <- null
   Debug.WriteLine("About to terminate the application.")
```

These are the messages that you'll see in the Debug window (interspersed among other diagnostic text):

```FSharp
About to create a Person object.
About to set the Person object to null
About to terminate the application.
Person object is being destroyed.
```

The sequence of messages makes it apparent that the `Person` object isn't destroyed when the `aPerson` variable is set to `null`—as would happen in Visual Basic 6—but only sometime later, when the application itself terminates.

Here's one important .NET programming guideline: never access any external object from a `Finalize` procedure because the external object might have been destroyed already. In fact, the object is being collected because it can't be reached from the main application, so a reference to another object isn't going to keep that object alive. The garbage collector can reclaim unreachable objects in any order, so the other object might be finalized before the current one. One of the few objects that can be safely accessed from a `Finalize` method is the base object of the current object, using the `base` keyword.

In general, it is safe to invoke static methods from inside the `Finalize` method, except when the application is shutting down. In the latter case, in fact, the .NET Framework might have already destroyed the `System.Type` object corresponding to the type that exposes the static method. For example, you shouldn't use the `Console.WriteLine` method because the `Console` object might be gone. The `Debug` object is one of the few objects that is guaranteed to stay alive until the very end, however, and that's why the previous code example uses `Debug.WriteLine` instead of `Console.WriteLine`. You can discern the two cases by querying the `Environment.HasShutdownStarted` method:

```FSharp
override this.Finalize()
    if not Environment.HasShutdownStarted then
        // It is safe to access static methods of other types.
```

As I mentioned earlier, you can control the garbage collector by means of the `System.GC` type, which exposes only static members. For example, you can force a collection by means of its `Collect` method:

```FSharp
let Main() =
    Debug.WriteLine("About to create a Person object.")
    let mutable aPerson = new Person()
    aPerson <- null
    Debug.WriteLine("About to fire a garbage collection.")
    GC.Collect()
    GC.WaitForPendingFinalizers()
    Debug.WriteLine("About to terminate the application.")
```

The `WaitForPendingFinalizers` method stops the current thread until all objects are correctly finalized; this action is necessary because the garbage collection process might run on a different thread. The sequence of messages in the Debug window is now different:

```FSharp
About to create a Person object.
About to fire a garbage collection.
Person object is being destroyed.
About to terminate the application.
```

However, calling the `GC.Collect` method to cause an *induced garbage collection* is usually a bad idea. If you run a garbage collection frequently, you're missing one of the most promising performance optimizations that the .NET Framework offers. The preceding code example, which uses the `GC.Collect` method only to fire the object's `Finalize` method, illustrates what you should *never* do in a real .NET application. In a server-side application—such as an `ASP.NET` application—this rule has virtually no exceptions.

In a Windows Forms application, you can invoke the `GC.Collect` method, but only when the application is idle—for example, while it waits for user input—and only if you see that unexpected (that is, not explicitly requested) garbage collections are slowing the program noticeably during time-critical operations. For example, unexpected garbage collections might be an issue when your application is in charge of controlling hardware devices that require a short response time. By inducing a garbage collection when the program is idle, you decrease the probability that a standard garbage collection slows the regular execution of your application.

Objects that expose the `Finalize` method aren't immediately reclaimed and usually require *at* *least* another garbage collection before they are swept out of the heap. The reason for this behavior is that the code in the `Finalize` method might assign the current object (using the `this` keyword) to a global variable, an advanced technique known as *object resurrection* (discussed later in this chapter). If the object were garbage collected at this point, the reference in the global variable would become invalid; the CLR can't detect this special case until the subsequent garbage collection and must wait until then to definitively release the object's memory. If you consider also that the creation of objects with a `Finalize` method requires a few more CPU cycles, you see that object finalization doesn't come for free. In general, you should implement the `Finalize` method only if you have a very good reason to do so.

### The Dispose Method

Because .NET objects don't have real destructors, well-designed classes should expose a method to let well-behaved clients manually release any resource such as files, database connections, and system objects as soon as they don't need those objects any longer—that is, just before setting the reference to `null`—rather than waiting for the subsequent garbage collection.

Classes that want to provide this feature should implement the `IDisposable` interface, which exposes only the `Dispose` method. I explain how to use disposable objects in the section titled "The Using…End Using Statement" in Chapter 3, "Control Flow and Error Handling," so here I focus on how to implement this interface in a class you design.

```FSharp
type Widget() =
    interface IDisposable with
        member self.Dispose() =
            // Close files and release other resources here.
            ()
```

The `Dispose` method is marked as `Public`, and so you can invoke it directly. This is the usage pattern for an `IDisposable` object:

```FSharp
// Create the object.
let obj = new Widget()
// Use the object.

// Ask the object to release resource before setting it to Nothing.
(obj:>IDisposable).Dispose()
//obj = null
```

Or you can use the `use` statement, a new feature of Microsoft Visual Basic 2005:

```FSharp
// Create the object.
use obj = new Widget()
// Use the object.
…
// The object is disposed of and set to null here.
```

The `use` statement ensures that the `Dispose` method is called even if the code throws an exception, but it doesn't catch the exception. If you need to handle exceptions thrown while using the object, you must give up the convenience of the `use` statement and use a regular Try…Catch…Finally block:

```FSharp
let mutable obj: Widget = null
try
    try
       // Create and use the object.
       obj <- new Widget()
    with ex -> ()
finally
   // Ensure that the Dispose method is always invoked.
   (obj:>IDisposable).Dispose()
```

Many stream- and connection-related objects in the .NET Framework, such as `FileStream` and all `ADO.NET` connection objects, have a public `Close` method that delegates to the private `Dispose` method. If you had to implement such types in Visual Basic, you would write code such as this:

```FSharp
type CustomStream() =
    interface IDisposable with
        member self.Dispose() = () // Close the stream here.
    member this.Close() = (this:>IDisposable).Dispose()
```

Interestingly, the `use` statement works correctly with objects that implement a private `Dispose` method because behind the scenes it casts the object reference to an `IDisposable` variable. In some cases, you might need to have an explicit cast to this interface, as in the following generic cleanup routine:

```FSharp
    // Set an object to null and call its Dispose method if possible.
    member this.ClearObject(obj:byref<obj>) =
       if obj.GetType() = typeof<IDisposable> then
          // You need an explicit cast. (It also works with private Dispose methods.)
          (obj :?> IDisposable).Dispose()
   
       // Next statement works because the object is passed by reference.
       obj <- null
```

.NET programming guidelines dictate that the `Dispose` method of an object should invoke the `Dispose` method of all the inner objects that the current object owns and that are hidden from the client code, and then it should call the base class's `Dispose` method (if the base class implements `IDisposable`). For example, if the `Widget` object has created a `System.Timers.Timer` object, the `Widget.Dispose` method should call the `Timer.Dispose` method. This suggestion and the fact that an object can be shared by multiple clients might cause a `Dispose` method to be called multiple times, and in fact a `Dispose` method shouldn't raise any errors when called more than once, even though all calls after the first one should be ignored. You can avoid releasing resources multiple times by using a class-level variable:

```FSharp
type Test() =
    let mutable disposed = false

    interface IDisposable with
        member self.Dispose() =
            if disposed then () else
            // Ensure that further calls are ignored.
            disposed <- true
            // Close files and release other resources here.
            ()
```

.NET programming guidelines also dictate that calling any method other than `Dispose` on an object that has already been disposed of should throw the special `ObjectDisposedException`. If you have implemented the class-level disposed field, implementing this guideline is trivial:

```FSharp
member this.AnotherMethod() =
   // Throw an exception if a client attempts to use a disposed object.
   if disposed then 
       raise <| new ObjectDisposedException("Widget")
```

### Combining the Dispose and Finalize Methods

Typically, you can allocate a resource other than memory in one of two ways: by invoking a piece of unmanaged code (for example, a Windows API function) or by creating an instance of a .NET class that wraps the resource. You need to understand this difference because the way you allocate a resource affects the decision about implementing the `Dispose` or the `Finalize` method.

You need to implement the `IDisposable` interface if a method in your type allocates resources other than memory, regardless of whether the resources are allocated directly (through a call to unmanaged code) or indirectly (through an object in the .NET Framework). Conversely, you need to implement the `Finalize` method only if your object allocates an unmanaged resource directly. Notice, however, that implementing `IDisposable` or the `Finalize` method is strictly mandatory only if your code stores a reference to the resource in a class-level field: if a method allocates a resource and releases it before exiting, for example, by means of a `use` statement, there's no need to implement either `IDisposable` or `Finalize`. In general, therefore, you must account for four different cases:

- **Neither the Dispose nor the Finalize method** 
  Your object uses only memory or other resources that don't require explicit deallocation, or the object releases any unmanaged resource before exiting the method that has allocated it. This is by far the most frequent case.
- **Dispose method only** 
  Your object allocates resources other than memory through other .NET objects, and you want to provide clients with a method to release those resources as soon as possible. This is the second most frequent case.
- **Both the Dispose and the Finalize methods** 
  Your object directly allocates a resource (typically by calling a method in an unmanaged DLL) that requires explicit deallocation or cleanup. You do such explicit deallocation in the `Finalize` method, but provide the `Dispose` method as well to provide clients with the ability to release the resource before your object's finalization.
- **Finalize method only** 
  You don't have any resource to release, but you need to perform a given action when your object is finalized. This is the least likely case, and in practice it is useful only in a few uncommon scenarios.

The first case is simple, and I have already showed how to implement the second case, so we can focus on the third case and see how the `Dispose` and `Finalize` methods can cooperate with each other. Here's an example of a `ClipboardWrapper` object that opens and closes the system Clipboard:

```FSharp
module Externs =
    [<DllImport("user32", CallingConvention = CallingConvention.Cdecl)>]
    extern Int32 OpenClipboard(Int32 hwnd)
    [<DllImport("user32", CallingConvention = CallingConvention.Cdecl)>]
    extern Int32 CloseClipboard()
open Externs

type ClipboardWrapper() =
    // Remember whether the Clipboard is currently open.
    let mutable isOpen: Boolean = false

    // Open the Clipboard and associate it with a window.
    member this.Open(hWnd : Int32) =
        // OpenClipboard returns 0 if any error.
        if OpenClipboard(hWnd) = 0 then raise <| new Exception("Unable to open clipboard")
        isOpen <- true

    // Close the Clipboard-ignore the command if not open.
    member this.Close() =
        if isOpen then CloseClipboard() |> ignore
        isOpen <- false

    interface IDisposable with
        member this.Dispose() = this.Close()

    override this.Finalize() = this.Close()
```

What the `OpenClipboard` and `CloseClipboard` methods do isn't important in this context because I just selected two of the simplest Windows API procedures that allocate and release a system resource. What really matters in this example is that an application that opens the Clipboard and associates it with a window must also release the Clipboard as soon as possible because the Clipboard is a system-wide resource and no other window can access it in the meantime. (A real-world class would surely expose other useful methods to manipulate the Clipboard, but I want to keep this example as simple as possible.)

It's a good practice to have both the `Dispose` and the `Finalize` methods call the cleanup routine that performs the actual release operation (the `Close` method, in this example) so that you don't duplicate the code. Such a method can be public, as in this case, or it can be a private helper method. You also need a class-level variable (`isOpen` in the preceding code) to ensure that cleanup code doesn't run twice, once when the client invokes `Dispose` and once when the garbage collector calls the `Finalize` method. The same variable also ensures that nothing happens if clients call the `Close` or `Dispose` method multiple times.

### A Better Dispose-Finalize Pattern

A problem with the technique just illustrated is that the garbage collector calls the `Finalize` method even if the client has already called the `Dispose` or `Close` method. As I explained previously, the `Finalize` method affects performance negatively because an additional garbage collection is required to destroy the object completely. Fortunately, you can control whether the `Finalize` method is invoked by using the `GC.SuppressFinalize` method. Using this method is straightforward. You typically call it from inside the `Dispose` method so that the garbage collector knows that it shouldn't call the `Finalize` method during the subsequent garbage collection.

Another problem that you might need to solve in a class that is more sophisticated than is the `ClipboardWrapper` demo shown earlier is that the cleanup code might access other objects referenced by the current object—for example, a control on a form—but you should never perform such access if the cleanup code runs in the finalization phase because those other objects might have been finalized already. You can resolve this issue by moving the actual cleanup code to an overloaded version of the `Dispose` method. This method takes a Boolean argument that specifies whether the object is being disposed of or finalized and avoids accessing external objects in the latter case. Here's a new version of the `ClipboardWrapper` class that uses this pattern to resolve these issues:

```FSharp
type ClipboardWrapper2() as this =
    // Remember whether the Clipboard is currently open.
    let mutable isOpen: Boolean = false

    // Remember whether the object has already been disposed of.
    // (Protected makes it available to derived classes.)
    let mutable disposed: Boolean = false
    // Don't invoke Finalize unless the Open method is actually invoked.
    do GC.SuppressFinalize(this)

    // Open the Clipboard and associate it with a window.
    member this.Open(hWnd : Int32) =
        // OpenClipboard returns 0 if any error.
        if OpenClipboard(hWnd) = 0 then raise <| new Exception("Unable to open Clipboard")
        isOpen <- true
        // Register the Finalize method, in case clients forget to call Close or Dispose.
        GC.ReRegisterForFinalize(this)

    member this.Close() =
        (this:>IDisposable).Dispose() // Delegate to private Dispose method.

    interface IDisposable with
        member this.Dispose() =
            this.Dispose(true)
            // Remember that the object has been disposed.
            disposed <- true
            // Tell .NET not to call the Finalize method.
            GC.SuppressFinalize(this)

    override this.Finalize() = this.Dispose(false)

    abstract Dispose: disposing : Boolean -> unit
    override this.Dispose(disposing : Boolean) =
        // Exit if the object has already been disposed of.
        if disposed then () else

        if disposing then
            // The object is being disposed of, not finalized. It is safe to access other
            // objects (other than the base object) only from inside this block.
            ()
        // Perform cleanup chores that must be executed in either case.
        CloseClipboard() |> ignore
        isOpen <- false
```

Notice that the constructor method invokes `GC.SuppressFinalize`, which tells the CLR not to invoke the `Finalize` method; this call accounts for the case when a client creates a `ClipboardWrapper2` object but never calls its `Open` method. The `Finalize` method is registered again by means of a call to the `GC.ReRegisterForFinalize` method in the `Open` method.

Finalization issues can become even more problematic if you consider that the `Finalize` method also runs if the object threw an exception in its constructor method. This means that the code in the `Finalize` method might access members that haven't been initialized correctly; thus, your finalization code should avoid accessing instance members if there is any chance that an error occurred in the constructor. Even better, the constructor method should use a Try…Catch block to trap errors, release any allocated resource, and then call `GC.SuppressFinalize(this)` to prevent the standard finalization code from running on uninitialized members.

---

##### Note

Visual Studio 2005 enables you to implement the `IDisposable` interface very quickly. In fact, whenever you type the Implements IDisposable statement and press Enter, Microsoft Visual Studio creates a lot of code for you: 

```FSharp
type Widget() =
    let mutable disposed = false // To detect redundant calls

    member this.Dispose(disposing : Boolean) =
        if not disposed then
            if disposing then
                // TODO: free unmanaged resources when explicitly called
                ()
            // TODO: free shared unmanaged resources
            ()
        disposed <- true

    interface IDisposable with
        member this.Dispose() =
            // Do not change this code. Put cleanup code in Dispose(disposing : Boolean)
            // method above.
            this.Dispose(true)
            GC.SuppressFinalize(this)
```

Notice that no `Finalize` method is created automatically. This is the correct approach, because—as I emphasized previously—only types that create unmanaged resources need the `Finalize` method. Most disposable objects do not. 

---




### Finalizers in Derived Classes

I have already explained that a well-written class that allocates and uses unmanaged resources (ODBC database connections, file and Windows object handles, and so on) should implement both a `Finalize` method and the `IDisposable.Dispose` method. If your application inherits from such a class, you must check whether your inherited class allocates any additional unmanaged resources. If not, you don't have to write any extra code because the derived class will inherit the base class implementation of both the `Finalize` and the `Dispose` methods. However, if the inherited class does allocate and use additional unmanaged resources, you should override the implementation of these methods, correctly release the unmanaged resources that the inherited class uses, and then call the corresponding method in the base class.

In the previous section, I illustrated a technique for correctly implementing these methods in a class, based on an overloaded `Dispose` method that contains the code for both the `IDisposable.Dispose` and the `Finalize` methods. As it happens, this overloaded `Dispose` method has a `Protected` scope, so in practice you can correctly implement the Dispose-Finalize pattern in derived classes by simply overriding one method:

```FSharp
type ClipboardWrapperEx() =
    inherit ClipboardWrapper2()
    // Remember whether the object has already been disposed of.
    // (Protected makes it available to derived classes.)
    let mutable disposed: Boolean = false

    // Insert regular methods here, some of which might allocate additional
    // unmanaged resources.

    // The only method we need to override to implement the Dispose–Finalize
    // pattern for this class.
    override this.Dispose(disposing : Boolean) =
        // Exit now if the object has already been disposed of.
        // (The disposed variable is declared as Protected in the base class.)
        if disposed then () else

        try
            if disposing then
                // The object is being disposed of, not finalized. It is safe to access other
                // objects (other than the base object) only from inside this block.
                ()
            // Perform cleanup chores that must be executed in either case.
            ()
        finally
            // Call the base class's Dispose method in all cases.
            base.Dispose(disposing)
```

If there is any chance that the code in the `Dispose` method might throw an exception, you should wrap it in a Try block and invoke the base class's `Dispose` method from the Finally block, as the previous code does. Failing to do so might result in the base class being prevented from releasing its own resources, which is something you should absolutely avoid for obvious reasons.

### A Simplified Approach to Finalization

Authoring a class that correctly implements the `Dispose` and `Finalize` methods isn't exactly a trivial task, as you've seen in previous sections. However, in most cases, you can take a shortcut and dramatically simplify the structure of your code by sticking to the following two guidelines. First, you wrap each unmanaged resource that requires finalization with a class whose only member is a field holding the handle of the unmanaged resource. Second, you nest this wrapper class inside another class that implements the `Dispose` method (but not the `Finalize` method). The nested class is marked as `Private`; therefore, it can be accessed only by the class that encloses it.

To see in practice what these guidelines mean, consider the following sample code:

```FSharp
// An invalid handle value that the wrapper class can use to check
// whether the handle is valid
let InvalidHandle = new IntPtr(-1)

// The nested private class that allocates and releases the unmanaged resource
// The constructor allocates the unmanaged resource (e.g., a file).
type UnmanagedResourceWrapper() =
    // A public field, but accessible only from inside the WinResource class
    let mutable Handle = new IntPtr(12345)

    interface IDisposable with
        member this.Dispose() =
            // Exit now if this object didn't complete its constructor correctly.
            if Handle = InvalidHandle then () else

            // Release the unmanaged resource, e.g., CloseHandle(Handle).
            ()
            // Invalidate the handle and tell the CLR not to call the Finalize method.
            Handle <- InvalidHandle
            GC.SuppressFinalize(this)

    override this.Finalize() = (this:>IDisposable).Dispose()

type WinResource() =
    // True if the object has been disposed of
    let mutable disposed : Boolean = false

    // A private field that points to the wrapper of the unmanaged resource
    // Allocate the unmanaged resource here.
    let wrapper = new UnmanagedResourceWrapper()

    // A public method that clients call to work with the unmanaged resource
    member this.DoSomething() =
        // Throw an exception if the object has already been disposed of.
        if disposed then
            raise <| new ObjectDisposedException("")

        // This code can pass the wrapper.Handle value to API calls.
        ()

    interface IDisposable with
        member this.Dispose() =
            // Avoid issues when multiple threads call Dispose at the same time.
            fun () ->
                // Do nothing if already disposed of.
                if disposed then () else
                // Dispose of all the disposable objects used by this instance,
                // including the one that wraps the unmanaged resource.
                ()
                (wrapper:>IDisposable).Dispose()
                // Remember this object has been disposed of.
                disposed <- true
            |> lock this
```

It's essential that the `UnmanagedResourceWrapper` class doesn't contain any fields, except the handle of the unmanaged resource, or any methods, except those listed. If the unmanaged resource should interact with other resources, the code that implements this interaction should be located in the `WinResource` class. The `WinResource` class must coordinate all the resources (managed and unmanaged) that it has allocated and must release them in its `Dispose` method.

Let's now discuss the advantages of these guidelines. First, and foremost, if the client code omits invoking the `WinResource.Dispose` method, all the memory used by the `WinResource` object will be cleared anyway at the first garbage collection. The inner `UnmanagedResourceWrapper` object has a `Finalize` method and therefore will be released only during a subsequent garbage collection, but this object consumes very little memory, and therefore this isn't a serious issue.

Second, the inner `UnmanagedResourceWrapper` is private and sealed; therefore, you don't need to write any complex code that takes derived classes into account. (However, you can inherit from `WinResource`, if you need to.) Being private, code outside the `WinResource` class can't get a reference to the `UnmanagedResourceWrapper` object and can't resurrect it.

Third, the `UnmanagedResourceWrapper` has only one field, and this field is a value type; therefore, the code in the `Dispose` or `Finalize` method can't mistakenly access any reference type. (As you might recall, a reference type might already be disposed of when it's accessed during the finalization stage.) Because there is just one handle to account for, you don't have to write code that deals with errors in the `UnmanagedResourceWrapper` constructor. If the constructor fails, the value of the `Handle` field continues to be equal to the `InvalidHandle` constant; the `Dispose` method can detect this condition and skip the cleanup code.

Finally, the `UnmanagedResourceWrapper` class is so simple and generic that you can often copy and paste its code (with minor edits) inside other types that must manage unmanaged resources. When it is nested inside another class, you don't even need to worry about name collisions.

---

##### Note

Version 2.0 of the .NET Framework introduces the `SafeHandle` abstract class, which makes it simpler to author classes that use unmanaged resources. Basically, a `SafeHandle` object is a wrapper for a Windows handle and is vaguely similar to the `UnmanagedResourceWrapper` in the previous example but with many additional features, such as the protection from a kind of attack known as *handle recycle attacks*. You can find more information in MSDN documentation and a few articles from the Base Class Library (BCL) Team, such as https://blogs.msdn.com/bclteam/archive/2005/03/15/396335.aspx and http://blogs.msdn.com/cbrumme/archive/2004/02/20/77460.aspx.

---

