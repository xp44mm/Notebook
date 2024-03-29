# 从CSharp迁移到FSharp

## 委托的写法

参考：Beginning F# 4.0 chapter 5  object-oriented programming

```C#
delegate TResult MethodCall<T, TResult>(T target, params object[] args);
```

等价的F#写法：

```fsharp
type MethodCall<'T, 'TResult> = delegate of 'T * [<ParamArray>]args:obj[] -> 'TResult
```

C#中的`EventHandler<'T>`在F#中有如下定义：

```fsharp
type EventHandler<'T> = delegate of obj * 'T -> unit
```

## 标准事件

C#中有`event`关键字，放在声明事件处理器的前面，定义事件的代码：

```C#
public interface IStockTicker
{
    event EventHandler<StockTick> StockTick;
}

public class StockTicker : IStockTicker
{
    public event EventHandler<StockTick> StockTick = delegate { };
    public void Notify(StockTick tick)
    {
        StockTick(this, tick);
    }
}
```

C#中添加或移除事件处理器：

```C#
class StockMonitor : IDisposable
{
    private readonly StockTicker _ticker;

    public StockMonitor(StockTicker ticker)
    {
        this._ticker = ticker;
        ticker.StockTick += this.OnStockTick;
    }

    void OnStockTick(object sender, StockTick stockTick)
    {
    }

    public void Dispose()
    {
        this._ticker.StockTick -= this.OnStockTick;
    }
}
```

C#定义的事件是是CLI通用的模式。在F#需要带`[<CLIEvent>]`的标准事件，下面代码与C#等价：

```fsharp
type IStockTicker =
    [<CLIEvent>]
    abstract StockTick:IEvent<EventHandler<StockTick>,StockTick>

type StockTicker() =
    let stockTick = new Event<EventHandler<StockTick>,StockTick>()
    interface IStockTicker with
        [<CLIEvent>]
        member this.StockTick = stockTick.Publish
    member this.Notify(tick:StockTick) = stockTick.Trigger(this,tick)
```

添加或移除事件处理器：

```fsharp
type StockMonitor(ticker:StockTicker) =
    let _ticker = ticker :> IStockTicker
    let OnStockTick(sender:obj)(stockTick:StockTick) = ()
    let handler = EventHandler<StockTick> OnStockTick
    do _ticker.StockTick.AddHandler(handler)
    interface IDisposable with
        member this.Dispose() =
            _ticker.StockTick.RemoveHandler(handler)
```

快捷F#事件，不带`[<CLIEvent>]`，省略第一个参数（sender）的快捷方式，不是CLI通用的模式。

```fsharp
type IStockTicker =
    abstract StockTick:IEvent<StockTick>
    
type StockTicker() =
    let stockTick = new Event<StockTick>()
    interface IStockTicker with
        member this.StockTick = stockTick.Publish
    member this.Notify(tick:StockTick) = stockTick.Trigger(tick)
```

添加或移除事件处理器：

```fsharp
type StockMonitor(ticker:StockTicker) =
    let _ticker = ticker :> IStockTicker
    let OnStockTick(sender:obj)(stockTick:StockTick) = ()
    let handler = Handler<StockTick> OnStockTick
    do _ticker.StockTick.AddHandler(handler)
    interface IDisposable with
        member this.Dispose() =
            _ticker.StockTick.RemoveHandler(handler)
```

可以使用`Add`快捷方法添加事件处理器，但是这个快捷方法没有反方法，不能移除事件处理器：

```fsharp
    do _ticker.StockTick.Add(OnStockTick)
```

## 委托event写法

C#中定义事件的接口类型：

```c#
public interface IChatConnection
{
    event Action<string> Received;
    event Action Closed;
    event Action<Exception> Error;
    void Disconnect();
}
```

等价的F#代码：

```fsharp
type IChatConnection =
    abstract Received:IDelegateEvent<Action<string>>
    abstract Closed:IDelegateEvent<Action>
    abstract Error:IDelegateEvent<Action<Exception>>
    abstract Disconnect:unit -> unit
```

实现C#代码：

```C#
public class ChatConnection : IChatConnection
{
    public event Action<string> Received = delegate { };

    public event Action Closed = delegate { };

    public event Action<Exception> Error = delegate { };

    public void NotifyRecieved(string msg)
    {
        Received(msg);
    }

    public void NotifyClosed()
    {
        Closed();
    }

    public void NotifyError()
    {
        Error(new OutOfMemoryException());
    }

    public void Disconnect()
    {
        Console.WriteLine("Disconnect");
    }
}
```

等价的F#代码：

```fsharp
type ChatConnection() =
    let received = DelegateEvent<Action<string>>()
    let closed = DelegateEvent<Action>()
    let error = DelegateEvent<Action<Exception>>()

    interface IChatConnection with
        override _.Received = received.Publish
        override _.Closed = closed.Publish
        override _.Error = error.Publish

        override _.Disconnect() =
            Console.WriteLine("Disconnect")

    member _.NotifyRecieved(msg:string) = received.Trigger [|msg|]

    member _.NotifyClosed() = closed.Trigger [||]

    member _.NotifyError() = error.Trigger[|OutOfMemoryException()|]
```

[DelegateEvent<'Delegate>](https://fsharp.github.io/fsharp-core-docs/reference/fsharp-control-fsharpdelegateevent-1.html)

Event implementations for an arbitrary type of delegate.

添加或移除处理器的C#代码：

```C#
public class ObservableConnection : ObservableBase<string>
{
    private readonly IChatConnection _chatConnection;

    public ObservableConnection(IChatConnection chatConnection)
    {
        this._chatConnection = chatConnection;
    }

    protected override IDisposable SubscribeCore(IObserver<string> observer)
    {
        Action<string> received = message => {
            observer.OnNext(message);
        };

        Action closed = () => {
            observer.OnCompleted();
        };

        Action<Exception> error = ex => {
            observer.OnError(ex);
        };

        this._chatConnection.Received += received;
        this._chatConnection.Closed += closed;
        this._chatConnection.Error += error;

        return Disposable.Create(() => {
            this._chatConnection.Received -= received;
            this._chatConnection.Closed -= closed;
            this._chatConnection.Error -= error;
            this._chatConnection.Disconnect();
        });
    }
}
```

F#代码：

```fsharp
type ObservableConnection(chatConnection:IChatConnection) =
    inherit ObservableBase<string>()

    override this.SubscribeCore(observer:IObserver<string>) =
        let received = new Action<_>(fun message->observer.OnNext(message))
        let closed = new Action(fun () -> observer.OnCompleted())
        let error = new Action<_>(fun ex ->observer.OnError(ex))

        chatConnection.Received.AddHandler(received)
        chatConnection.Closed.AddHandler(closed)
        chatConnection.Error.AddHandler(error)

        Disposable.Create(fun () ->
            chatConnection.Received.RemoveHandler(received)
            chatConnection.Closed.RemoveHandler(closed)
            chatConnection.Error.RemoveHandler(error)
            chatConnection.Disconnect()
        )
```

测试代码，仅列出F#代码：

```fsharp
let chatConnection = ChatConnection() 
let observableConnection = ObservableConnection (chatConnection:> IChatConnection )
let subscription =
    observableConnection.Subscribe(ConsoleObserver "receiver")

chatConnection.NotifyRecieved("Hello")
chatConnection.NotifyClosed()
subscription.Dispose()
```

## 锁

C#代码

```C#
lock (this._stockTickLocker)
{
}
```

F#代码：

```fsharp
lock _stockTickLocker (fun() -> ...)
```

## 从事件(IEvent)到可观察(IObservable)

标准事件模式，C#代码：

```C#
Observable.FromEventPattern<EventHandler<StockTick>, StockTick>(
    h => ticker.StockTick += h,
    h => ticker.StockTick -= h)
.Select(tickEvent => tickEvent.EventArgs)
```

F#代码：

```fsharp
ticker.StockTick :> IObservable<_>
```

在F#中，不能从C#中直译代码，而是利用事实`IEvent<'t>`继承自`IObservable<'t>`接口，向上强制转换即可。

## 委托事件转化为`Observable`

详见 rx.net in action第4章，非标准事件。

## 写WPF程序，XAML

新建控制台应用程序。非(.net framework)

修改项目属性，项目文件代码如下：

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>WinExe</OutputType>
    <TargetFramework>net7.0-windows</TargetFramework>
    <UseWPF>true</UseWPF>
    <UseWindowsForms>False</UseWindowsForms>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="MahApps.Metro" Version="2.4.9" />
    <PackageReference Include="System.Reactive" Version="5.0.0" />
  </ItemGroup>

</Project>

```

启动代码如下：

```fsharp
open System
open System.Windows
open System.Windows.Controls

let w = new Window()
let b = new Button(Content = "Hello from WPF!")
b.Click.Add(fun _ -> w.Close())
w.Content <- b

let a = new Application()

[<STAThread>]
do a.Run(w) |> ignore
```

LOADING XAML INTO F# APPLICATIONS

```fsharp
let a = 
    let reader = XmlReader.Create(Path.Combine(__SOURCE_DIRECTORY__,"App.xaml"))
    XamlReader.Load(reader) :?> Application

let w = 
    let reader = XmlReader.Create(Path.Combine(__SOURCE_DIRECTORY__,"FirstMetroWindow.xaml"))
    let this = XamlReader.Load(reader) :?> Window
    let btn = this.FindName("btn") :?> Button
    btn.Click.Add(fun _ ->
        MessageBox.Show("click from code behind.") |> ignore
    )
    this

[<STAThread>]
do a.Run(w) |> ignore

```

FromEventPattern

```fsharp
let clicks = 
    (theButton.Click :> IObservable<_>)
        .Synchronize();

let _ = clicks.Subscribe(fun eventPattern -> 
    output.Text <-  output.Text + "button clicked" + Environment.NewLine
)
```
