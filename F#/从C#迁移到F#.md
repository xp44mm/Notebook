# 从C#迁移到F#

### 委托的写法：

参考：Beginning F# 4.0 chapter 5  object-oriented programming

```C#
delegate TResult MethodCall<T, TResult>(T target, params object[] args);
```

等价的F#写法：

```F#
type MethodCall<'T, 'TResult> = delegate of 'T * [<ParamArray>]args:obj[] -> 'TResult
```

C#中的`EventHandler<'T>`在F#中有如下定义：

```F#
type EventHandler<'T> = delegate of obj * 'T -> unit
```



### 标准事件

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

```F#
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

```F#
type StockMonitor(ticker:StockTicker) =
    let _ticker = ticker:> IStockTicker
    let OnStockTick(sender:obj)(stockTick:StockTick) = ()
    let handler = EventHandler<StockTick> OnStockTick
    do _ticker.StockTick.AddHandler(handler)
    interface IDisposable with
        member this.Dispose() =
            _ticker.StockTick.RemoveHandler(handler)
```

快捷F#事件，不带`[<CLIEvent>]`，省略第一个参数（sender）的快捷方式，不是CLI通用的模式。

```F#
type IStockTicker =
    abstract StockTick:IEvent<StockTick>
    
type StockTicker() =
    let stockTick = new Event<StockTick>()
    interface IStockTicker with
        member this.StockTick = stockTick.Publish
    member this.Notify(tick:StockTick) = stockTick.Trigger(tick)
```

添加或移除事件处理器：

```F#
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

```F#
    do _ticker.StockTick.Add(OnStockTick)
```


### 自定义非标准event写法

C#中定义事件：

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

```F#
type IChatConnection =
    abstract Received:IDelegateEvent<Action<string>>
    abstract Closed:IDelegateEvent<Action>
    abstract Error:IDelegateEvent<Action<Exception>>
    abstract Disconnect:unit -> unit
```

添加或移除处理器的代码：

```F#
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

### 锁

C#代码

```C#
lock (this._stockTickLocker)
{
}
```

F#代码：

```F#
lock _stockTickLocker (fun() -> ...)
```



### 从事件到可观察

C#代码：

```C#
Observable.FromEventPattern<EventHandler<StockTick>, StockTick>(
    h => ticker.StockTick += h,
    h => ticker.StockTick -= h)
.Select(tickEvent => tickEvent.EventArgs)
```

F#代码：
```F#
ticker.StockTick :> IObservable<_>
```

在F#中，不能从C#中直译代码，而是利用事实`IEvent<>`继承自`IObservable<>`接口，向上强制转换即可。

### 写WPF程序，XAML

新建控制台应用程序，.net framework的，

添加下面引用：

```F#
#r "WindowsBase"
#r "PresentationCore"
#r "PresentationFramework"
#r "System.Xaml"
```

启动代码如下：

```F#
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

FromEventPattern(可能不对)

```F#
let clicks = 
    Observable.FromEventPattern<RoutedEventHandler, RoutedEventArgs>(
        theButton.Click.AddHandler, theButton.Click.RemoveHandler)

let _ = clicks.Subscribe(fun eventPattern -> 
    output.Text <-  output.Text + "button clicked" + Environment.NewLine
)
```





