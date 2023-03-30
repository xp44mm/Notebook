# CountdownEvent

[System.Threading.CountdownEvent](https://learn.microsoft.com/en-us/dotnet/api/system.threading.countdownevent) is a synchronization primitive that unblocks its waiting threads after it has been signaled a certain number of times. For example, in a fork/join scenario, you can just create a [CountdownEvent](https://learn.microsoft.com/en-us/dotnet/api/system.threading.countdownevent) that has a signal count of 5, and then start five work items on the thread pool and have each work item call [Signal](https://learn.microsoft.com/en-us/dotnet/api/system.threading.countdownevent.signal) when it completes. Each call to [Signal](https://learn.microsoft.com/en-us/dotnet/api/system.threading.countdownevent.signal) decrements the signal count by 1. On the main thread, the call to [Wait](https://learn.microsoft.com/en-us/dotnet/api/system.threading.countdownevent.wait) will block until the signal count is zero.

[CountdownEvent](https://learn.microsoft.com/en-us/dotnet/api/system.threading.countdownevent) has these additional features:

- The wait operation can be canceled by using cancellation tokens.
- Its signal count can be incremented after the instance is created.
- Instances can be reused after [Wait](https://learn.microsoft.com/en-us/dotnet/api/system.threading.countdownevent.wait) has returned by calling the [Reset](https://learn.microsoft.com/en-us/dotnet/api/system.threading.countdownevent.reset) method.

## Basic Usage

The following example demonstrates how to use a [CountdownEvent](https://learn.microsoft.com/en-us/dotnet/api/system.threading.countdownevent) with [ThreadPool](https://learn.microsoft.com/en-us/dotnet/api/system.threading.threadpool) work items.

```fsharp
let test () =
    let source = seq []
    use e = new CountdownEvent(1)
    // fork work:
    for element in source do
        // Dynamically increment signal count.
        e.AddCount()
        ThreadPool.QueueUserWorkItem((fun (state) ->
                try
                    Console.WriteLine(state)
                finally
                    e.Signal() |> ignore
            ),
            element) |> ignore
    e.Signal() |> ignore

    // The first element could be run on this thread.

    // Join with work.
    e.Wait()
```

## `CountdownEvent` With Cancellation

The following example shows how to cancel the wait operation on [CountdownEvent](https://learn.microsoft.com/en-us/dotnet/api/system.threading.countdownevent) by using a cancellation token. The basic pattern follows the model for unified cancellation, which was introduced in .NET Framework 4. For more information, see [Cancellation in Managed Threads](https://learn.microsoft.com/en-us/dotnet/standard/threading/cancellation-in-managed-threads).

```fsharp

type Data = { Num:int }

type DataWithToken = { Token: CancellationToken; Data: Data }

let GetData() =
    [1..5]
    |> Seq.map(fun i -> {Num=i})

let ProcessData (obj:obj) =
    let dataWithToken = unbox<DataWithToken> obj

    if dataWithToken.Token.IsCancellationRequested then
        Console.WriteLine("Canceled before starting {0}", dataWithToken.Data.Num)
    else 
        let rec loop i =
            if i = 0 then
                Console.WriteLine("Processed {0}", dataWithToken.Data.Num)
            else
                if dataWithToken.Token.IsCancellationRequested then
                    Console.WriteLine("Cancelling while executing {0}", dataWithToken.Data.Num)
                else
                    Thread.SpinWait(100000)
                    loop (i-1)
        loop 10000

let EventWithCancel() =
    let source = GetData()
    use cts = new CancellationTokenSource()

    //Enable cancellation request from a simple UI thread.
    Task.Factory.StartNew(fun () ->
                if (Console.ReadKey().KeyChar = 'c') then
                    cts.Cancel()
            ) |> ignore

    // Event must have a count of at least 1
    use e = new CountdownEvent(1)

    // fork work:
    for element in source do
        let item = {Data = element; Token= cts.Token}
        // Dynamically increment signal count.
        e.AddCount();
        ThreadPool.QueueUserWorkItem((fun state ->
                ProcessData(state)
                if not cts.Token.IsCancellationRequested then
                    e.Signal() |> ignore
            ),
            item) |> ignore
    // Decrement the signal count by the one we added
    // in the constructor.
    e.Signal() |> ignore

    // The first element could be run on this thread.

    // Join with work or catch cancellation.
    try
        e.Wait(cts.Token)
    with 
    | :? OperationCanceledException as oce when oce.CancellationToken = cts.Token ->
            Console.WriteLine("User canceled.");
    | ex -> 
        Console.Write("We don't know who canceled us!")
        raise ex

let Main() =
    EventWithCancel()
    Console.WriteLine("Press enter to exit.")
    Console.ReadLine()
```

Note that the wait operation does not cancel the threads that are signaling it. Typically, cancellation is applied to a logical operation, and that can include waiting on the event as well as all the work items that the wait is synchronizing. In this example, each work item is passed a copy of the same cancellation token so that it can respond to the cancellation request.