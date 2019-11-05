# Debug Class

## Definition

- Namespace:

  System.Diagnostics

- Assemblies:

  System.Diagnostics.Debug.dll, System.dll, netstandard.dll

Provides a set of methods and properties that help debug your code.

```csharp
public static class Debug
```

- Inheritance

  ObjectDebug

## Examples

The following example uses `Debug` to indicate the beginning and end of a program's execution. The example also uses `Indent` and `Unindent` to distinguish the tracing output.

```csharp
// Specify /d:DEBUG when compiling.

using System;
using System.Data;
using System.Diagnostics;

class Test
{
    static void Main()
    {
       //new System.Diagnostics.TextWriterTraceListener("app.log") 打印到文件
       Debug.Listeners.Add(new TextWriterTraceListener(Console.Out));
       Debug.AutoFlush = true;
       Debug.Indent();
       Debug.WriteLine("Entering Main");
       Console.WriteLine("Hello World.");
       Debug.WriteLine("Exiting Main"); 
       Debug.Unindent();
    }
}
```

## Remarks

If you use methods in the `Debug` class to print debugging information and check your logic with assertions, you can make your code more robust without impacting the performance and code size of your shipping product.

This class provides methods to display an Assert dialog box, and to emit an assertion that will always fail. This class provides write methods in the following variations: `Write`, `WriteLine`, `WriteIf` and `WriteLineIf`.

The `BooleanSwitch` and `TraceSwitch` classes provide means to dynamically control the tracing output. You can modify the values of these switches without recompiling your application. For information on using the configuration file to set a switch, see the `Switch` class and the Trace Switches topic.

You can customize the tracing output's target by adding `TraceListener` instances to or removing instances from the `Listeners` collection. The `Listeners` collection is shared by both the `Debug` and the `Trace` classes; adding a trace listener to either class adds the listener to both. By default, the `DefaultTraceListener` class emits trace output.

---

#####  Note

Adding a trace listener to the `Listeners` collection can cause an exception to be thrown while tracing, if a resource used by the trace listener is not available. The conditions and the exception thrown depend on the trace listener and cannot be enumerated in this topic. It may be useful to place calls to the `Debug` methods in `try`/`catch` blocks to detect and handle any exceptions from trace listeners.

---

You can modify the level of indentation using the `Indent` method or the `IndentLevel` property. To modify the indent spacing, use the `IndentSize` property. You can specify whether to automatically flush the output buffer after each write by setting the `AutoFlush` property to `true`.

To set the `AutoFlush` and `IndentSize` for `Debug`, you can edit the configuration file corresponding to the name of your application. The configuration file should be formatted as shown in the following example.

```xml
<configuration>  
  <system.diagnostics>  
    <trace autoflush="true" indentsize="7" />  
  </system.diagnostics>  
</configuration>  
```

The `ConditionalAttribute` attribute is applied to the methods of `Debug`. Compilers that support `ConditionalAttribute` ignore calls to these methods unless "DEBUG" is defined as a conditional compilation symbol. Refer to a compiler's documentation to determine whether `ConditionalAttribute` is supported and the syntax for defining a conditional compilation symbol.

---

#####  Note

In Visual Studio C# projects, by default, the "DEBUG" conditional compilation symbol is defined for debug builds, and the "TRACE" symbol is defined for both debug and release builds. For information about how to disable this behavior, see the Visual Studio documentation. 

To define the "DEBUG" conditional compilation symbol in C#, add the `/d:DEBUG` option to the compiler command line when you compile your code using a command line, or add `#define DEBUG` to the top of your file. 

---

## Properties

 `AutoFlush`
 Gets or sets a value indicating whether `Flush()` should be called on the `Listeners` after every write.

 `IndentLevel`
 Gets or sets the indent level.

 `IndentSize`
 Gets or sets the number of spaces in an indent.

 `Listeners`
 Gets the collection of listeners that is monitoring the debug output.

## Methods

 `Assert(Boolean)`
 Checks for a condition; if the condition is `false`, displays a message box that shows the call stack.

 `Assert(Boolean, String)`
 Checks for a condition; if the condition is `false`, outputs a specified message and displays a message box that shows the call stack.

 `Assert(Boolean, String, String)`
 Checks for a condition; if the condition is `false`, outputs two specified messages and displays a message box that shows the call stack.

 `Assert(Boolean, String, String, Object)`
 Checks for a condition; if the condition is `false`, outputs two messages (simple and formatted) and displays a message box that shows the call stack.

 `Close()`
 Flushes the output buffer and then calls the `Close` method on each of the `Listeners`.

 `Fail(String)`
 Emits the specified error message.

 `Fail(String, String)`
 Emits an error message and a detailed error message.

 `Flush()`
 Flushes the output buffer and causes buffered data to write to the `Listeners` collection.

 `Indent()`
 Increases the current `IndentLevel` by one.

 `Print(String)`
 Writes a message followed by a line terminator to the trace listeners in the `Listeners` collection.

 `Print(String, Object)`
 Writes a formatted string followed by a line terminator to the trace listeners in the `Listeners` collection.

 `Unindent()`
 Decreases the current `IndentLevel` by one.

 `Write(String, String)`
 Writes a category name and message to the trace listeners in the `Listeners` collection.

 `Write(Object, String)`
 Writes a category name and the value of the object's ToString() method to the trace listeners in the `Listeners` collection.

 `Write(Object)`
 Writes the value of the object's ToString() method to the trace listeners in the `Listeners` collection.

 `Write(String)`
 Writes a message to the trace listeners in the `Listeners` collection.

 `WriteIf(Boolean, Object)`
 Writes the value of the object's ToString() method to the trace listeners in the `Listeners` collection if a condition is `true`.

 `WriteIf(Boolean, String)`
 Writes a message to the trace listeners in the Listeners collection if a condition is `true`.

 `WriteIf(Boolean, Object, String)`
 Writes a category name and the value of the object's ToString() method to the trace listeners in the `Listeners` collection if a condition is `true`.

 `WriteIf(Boolean, String, String)`
 Writes a category name and message to the trace listeners in the `Listeners` collection if a condition is `true`.

 `WriteLine(Object)`
 Writes the value of the object's ToString() method to the trace listeners in the `Listeners` collection.

 `WriteLine(String)`
 Writes a message followed by a line terminator to the trace listeners in the `Listeners` collection.

 `WriteLine(Object, String)`
 Writes a category name and the value of the object's ToString() method to the trace listeners in the `Listeners` collection.

 `WriteLine(String, Object)`
 Writes a formatted message followed by a line terminator to the trace listeners in the `Listeners` collection.

 `WriteLine(String, String)`
 Writes a category name and message to the trace listeners in the `Listeners` collection.

 `WriteLineIf(Boolean, Object, String)`
 Writes a category name and the value of the object's ToString() method to the trace listeners in the `Listeners` collection if a condition is `true`.

 `WriteLineIf(Boolean, Object)`
 Writes the value of the object's ToString() method to the trace listeners in the `Listeners` collection if a condition is `true`.

 `WriteLineIf(Boolean, String)`
 Writes a message to the trace listeners in the `Listeners` collection if a condition is `true`.

 `WriteLineIf(Boolean, String, String)`
 Writes a category name and message to the trace listeners in the `Listeners` collection if a condition is `true`.