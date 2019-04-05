F# event写法

事件C#：

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



F#写WPF程序：

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

FromEventPattern

```F#
let clicks = 
    Observable.FromEventPattern<RoutedEventHandler, RoutedEventArgs>(
        theButton.Click.AddHandler, theButton.Click.RemoveHandler)

let _ = clicks.Subscribe(fun eventPattern -> 
    output.Text <-  output.Text + "button clicked" + Environment.NewLine
)
```



C#中的委托：

参考：Beginning F# 4.0 chapter 5  object-oriented programming

```C#
delegate TResult MethodCall<T, TResult>(T target, params object[] args);
```

等价的F#写法：

```F#
type MethodCall<'T, 'TResult> = delegate of 'T * [<ParamArray>]args:obj[] -> 'TResult
```

