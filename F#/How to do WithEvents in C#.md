### How to do "WithEvents" in C#?

##### Brett

What is the C# equivalent for this VB.NET code:

```VB
Private WithEvents IE_Inst As New SHDocVw.InternetExplorer
```

I get as far as:

```VB
private SHDocVw.InternetExplorer IE_Inst = new SHDocVw.InternetExplorer();
```

but am not sure how to translate the `WithEvents` part. Any suggestions?  

##### Bob Powell

Don't worry about it. It's another example of keyword diarrhea in VB. It
fits right in with the idiocy of "overloads overrides" and the VB compilers
insistence on seeing the Readonly keyword even when it can see that there is
no set accessor.

If a class exposes public events then you can add a handler to them using
the `+=` operator such as...

```C#
the Form.Paint += new PaintEventHandler(this painthandler); // This is the equivalent of AddHandler
```

you can remove events similarly with `-=` being the equivalent of `RemoveHandler`.

##### Brett

Using my code, what is the `.Paint` part? Would that be InternetExplorer?

I'm not quite sure how to build the following in C#:

```VB
Private WithEvents IE_Inst As New SHDocVw.InternetExplorer
```

Maps to things such as:

```VB
Public Sub IEDocComplete(ByVal pDisp As Object, ByRef URL As Object) Handles IE_Inst.DocumentComplete
```

##### David Anton

The usual general format of the C# add handler equivalent is:

```C#
<object>.<event> += new <eventhandler>(<event handling method>);
```

The equivalent C# code is (using our Instant C# VB to C# converter):

```C#
private SHDocVw.InternetExplorer IE_Inst = new SHDocVw.InternetExplorer();

//TODO: INSTANT C# TODO TASK: Insert the following converted event handlers at the end of the 'InitializeComponent' method for forms or into a constructor for other classes:

IE_Inst.DocumentComplete += new System.EventHandler(IEDocComplete);

public void IEDocComplete(object pDisp, ref object URL)
{
}
```

##### Bob Powell

The `.Paint` part was just an example of a well-known event.

In your particular case you can declare the object and add the handler like so:

```C#
AxSHDocVw.AxWebBrowser axWebBrowser1 = new AxSHDocVw.AxWebBrowser();

this.axWebBrowser1.DocumentComplete += new AxSHDocVw.DWebBrowserEvents2_DocumentCompleteEventHandler(this.axWebBrowser1_DocumentComplete);
```

##### Brett

I'm getting this error now on the line below:

```C#
C:\myFiles\myprogC#\Main\IE.cs(31): 
Method 'Mail.IE.IEDocComplete(object, ref object)' 
does not match 
delegate 'void System.EventHandler(object, System.EventArgs)'
```

for this code

```C#
IE_Inst.DocumentComplete += new System.EventHandler(IEDocComplete);
```

method defined here

```C#
public void IEDocComplete(object pDisp, ref object url)
{
```

What exactly is it asking for?

##### Steve Walker

A method to call with the signature `void Foo(object x, System.EventArgs y)`, not one with the signature `void IEDocComplete(object pDisp, ref object url)`.

##### Brett

I understand the signatures are different but it works in VB.NET.

```C#
Private WithEvents IE_Inst As New SHDocVw.InternetExplorer
Public Sub IEDocComplete(ByVal pDisp As Object, ByRef URL As Object) Handles IE_Inst.DocumentComplete
```

How is VB basically getting away with doing the samething I'm doing in c#? Isn't this saying the same as `Handles IE_Inst.DocumentComplete`.

```C#
IE_Inst.DocumentComplete += new System.EventHandler(IEDocComplete);
public void IEDocComplete(object pDisp, ref object url)
```

##### Steve Walker

Because it was not wiring it up to a `System.EventHandler`, it's wiring it up to a `SHDocVw.DWebBrowserEvents2_DocumentCompleteEventHandler`. Open your VB exe with *ildasm* and you'll see it doing it. You need to do the same in your C#.

You know, compiling that to have a look at the IL was the first VB I've done for years. I feel soiled :o)
