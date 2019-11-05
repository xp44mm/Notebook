# Chapter 8. Interop

Asynchronous, parallel, reactive—each of these has its place, but how well do they work together?

In this chapter, we'll look at various interop scenarios where you'll learn how to combine these different approaches. You'll learn that they complement each other, rather than compete; there's very little friction at the boundaries where one approach meets another.

## 8.1 Async Wrappers for “Async” Methods with “Completed” Events

### Problem

There is an older asynchronous pattern that uses methods named OperationAsync along with events named OperationCompleted. You want to perform an operation using the older asynchronous pattern and await the result.

##### TIP

The OperationAsync and OperationCompleted pattern is called the Event-based Asynchronous Pattern (EAP). You're going to wrap those into a Task-returning method that follows the Task-based Asynchronous Pattern (TAP).

### Solution

By using the `TaskCompletionSource<TResult>` type, you can create wrappers for asynchronous operations. The `TaskCompletionSource<TResult>` type controls a `Task<TResult>` and enables you to complete the task at the appropriate time.

This example defines an extension method for `WebClient` that downloads a string. The `WebClient` type defines `DownloadStringAsync` and `DownloadStringCompleted`. Using those, you can define a `DownloadStringTaskAsync` method, like this:

```C#
static Task<string> DownloadStringTaskAsync(this WebClient client, Uri address)
{
  var tcs = new TaskCompletionSource<string>();
  // The event handler will complete the task and unregister itself.
  var handler = null as DownloadStringCompletedEventHandler;
  handler = (_, e) =>
  {
    client.DownloadStringCompleted -= handler;// rec:注销自己
    if (e.Cancelled)
      tcs.TrySetCanceled();
    else if (e.Error != null)
      tcs.TrySetException(e.Error);
    else
      tcs.TrySetResult(e.Result);
  };
  // Register for the event and *then* start the operation.
  client.DownloadStringCompleted += handler;
  client.DownloadStringAsync(address);
  return tcs.Task;
}
```

### Discussion

This particular example is not very useful because `WebClient` already is defining a `DownloadStringTaskAsync`, and there's a more async-friendly `HttpClient` that could be used. However, this same technique can be used to interface with older asynchronous code that hasn't yet been updated to use `Task`.

##### TIP

For new code, always use `HttpClient`. Only use `WebClient` if you're working with legacy code.

Normally, a TAP method for downloading strings would be named `OperationAsync` (e.g., `DownloadStringAsync`); however, that naming convention won't work in this case because EAP already defines a method with that name. Here the convention is to name the TAP method `OperationTaskAsync` (e.g., `DownloadStringTaskAsync`).

When wrapping EAP methods, there's the possibility that the “start” method may throw an exception; in the previous example, `DownloadStringAsync` may throw. In that case, you'll need to decide whether to allow the exception to propagate or to catch the exception and call `TrySetException`. Most of the time, exceptions thrown at that point are usage errors, so it doesn't matter which option you choose. If you're unsure whether the exceptions are usage errors, then I recommend catching the exception and calling `TrySetException`.

### See Also

Recipe 8.2 covers TAP wrappers for APM methods (BeginOperation and EndOperation).

Recipe 8.3 covers TAP wrappers for any kind of notification.

## 8.2 Async Wrappers for “Begin/End” Methods

### Problem

An older asynchronous pattern uses pairs of methods named BeginOperation and EndOperation, with the `IAsyncResult` representing the asynchronous operation. You have an operation that follows the older asynchronous pattern and want to consume it with await.

##### TIP

The BeginOperation and EndOperation pattern is called the Asynchronous Programming Model (APM). You're going to wrap those into a Task-returning method that follows the Task-based Asynchronous Pattern (TAP).

### Solution

The best approach for wrapping APM is to use one of the `FromAsync` methods on the `TaskFactory` type. `FromAsync` uses `TaskCompletionSource<TResult>` under the hood, but when you're wrapping APM, `FromAsync` is much easier to use.

This example defines an extension method for `WebRequest` that sends an HTTP request and gets the response. The `WebRequest` type defines `BeginGetResponse` and `EndGetResponse`; you can define a `GetResponseAsync` method like this:

```C#
public static Task<WebResponse> GetResponseAsync(this WebRequest client)
{
  return Task<WebResponse>.Factory.FromAsync(client.BeginGetResponse, client.EndGetResponse, null);
}
```

### Discussion

`FromAsync` has a downright confusing number of overloads! As a general rule, it's best to call `FromAsync`, as in the example. First, pass the BeginOperation method (without calling it), then pass the EndOperation method (without calling it). Next, pass all arguments that BeginOperation takes except for the last AsyncCallback and object arguments. Finally, pass null.

In particular, do not call the BeginOperation method before calling `FromAsync`. You can call `FromAsync`, passing the `IAsyncOperation` that you get from BeginOperation, but if you call it that way, `FromAsync` is forced to use a less efficient implementation.

You might be wondering why the recommended pattern always passes a null at the end. `FromAsync` was introduced along with the `Task` type in .NET 4.0, before async was around. At the time, it was common to use state objects in asynchronous callbacks, and the `Task` type supports this via its `AsyncState` member. In the new async pattern, state objects are no longer necessary, so it's normal to always pass null for the state parameter. These days, state is only used to avoid a closure instance when optimizing memory usage.

### See Also

Recipe 8.3 covers writing TAP wrappers for any kind of notification.

## 8.3 Async Wrappers for Anything

### Problem

You have an unusual or nonstandard asynchronous operation or event and want to consume it via await.

### Solution

The `TaskCompletionSource<T>` type can be used to construct `Task<T>` objects in any scenario. Using a `TaskCompletionSource<T>`, you can complete a task in three different ways: with a successful result, faulted, or canceled. Before async was on the scene, there were two other asynchronous patterns recommended by Microsoft: APM (Recipe 8.2) and EAP (Recipe 8.1). However, both APM and EAP were rather awkward and in some cases difficult to get right. So, an unofficial convention arose that used callbacks, with methods like the following:

```C#
public interface IMyAsyncHttpService
{
  void DownloadString(Uri address, Action<string, Exception> callback);
}
```

Methods like these follow the convention that `DownloadString` will start the (asynchronous) download, and when it completes, the callback is invoked with either the result or the exception. Usually, callback is invoked on a background thread.

A nonstandard kind of asynchronous method like the previous example can be wrapped using `TaskCompletionSource<T>` so that it naturally works with await, as this next example shows:

```C#
static Task<string> DownloadStringAsync(this IMyAsyncHttpService httpService, Uri address)
{
  var tcs = new TaskCompletionSource<string>();
  httpService.DownloadString(address, (result, exception) =>
  {
    if (exception != null)
      tcs.TrySetException(exception);
    else
      tcs.TrySetResult(result);
  });
  return tcs.Task;
}
```

### Discussion

You can use this same `TaskCompletionSource<T>` pattern to wrap any asynchronous method, no matter how nonstandard. Create the `TaskCompletionSource<T>` instance first. Next, arrange a callback so that the `TaskCompletionSource<T>` completes its task appropriately. Then, start the actual asynchronous operation. Finally, return the `Task<T>` that is attached to that `TaskCompletionSource<T>`.

It is important for this pattern that you make sure that the `TaskCompletionSource<T>` is always completed. Think through your error handling in particular, and ensure that the `TaskCompletionSource<T>` will be completed appropriately. In the last example, exceptions are explicitly passed into the callback, so you don't need a catch block; but some nonstandard patterns might need you to catch exceptions in your callbacks and place them on the `TaskCompletionSource<T>`.

### See Also

Recipe 8.1 has coverage of TAP wrappers for EAP members (OperationAsync, OperationCompleted).

Recipe 8.2 covers TAP wrappers for APM members (BeginOperation, EndOperation).

## 8.4 Async Wrappers for Parallel Code

### Problem

You have (CPU-bound) parallel processing that you want to consume using await. Usually, this is desirable so that your UI thread doesn't block waiting for the parallel processing to complete.

### Solution

The `Parallel` type and Parallel LINQ use the thread pool to do parallel processing. They will also include the calling thread as one of the parallel processing threads, so if you call a parallel method from the UI thread, the UI will be unresponsive until the processing completes.

To keep the UI responsive, wrap the parallel processing in a `Task.Run` and await the result:

```C#
await Task.Run(() => Parallel.ForEach(...));
```

The key behind this recipe is that parallel code includes the calling thread in its pool of threads that it uses to do the parallel processing. This is true for both Parallel LINQ and the `Parallel` class.

### Discussion

This is a simple recipe but one that is often overlooked. By using `Task.Run`, you're pushing all of the parallel processing off to the thread pool. `Task.Run` returns a `Task` that then represents that parallel work, and the UI thread can (asynchronously) wait for it to complete.

This recipe only applies to UI code. On the server side (e.g., ASP.NET), parallel processing is rarely done because the server host already does parallelism. For this reason, server-side code shouldn't perform parallel processing, nor should it push work off to the thread pool.

### See Also

Chapter 4 covers the basics of parallel code.

Chapter 2 covers the basics of asynchronous code.

## 8.5 Async Wrappers for System.Reactive Observables

### Problem

You have an observable stream that you want to consume using await.

### Solution

First, you need to decide which of the observable events in the event stream you're interested in. These are common situations:

  - The last event before the stream ends
  - The next event
  - All the events

To capture the last event in the stream, you can either await the result of `LastAsync` or just await the observable directly:

```C#
var observable = ...; /* as IObservable<int> */
var lastElement = await observable.LastAsync();
/* or: var lastElement = await observable; */
```

When you await an observable or `LastAsync`, the code (asynchronously) waits until the stream completes and then returns the last element. Under the covers, the await is subscribing to the stream.

To capture the next event in the stream, use `FirstAsync`. In the following code, the await subscribes to the stream and then completes (and unsubscribes) as soon as the first event arrives:

```C#
var observable = ...; // as IObservable<int>
var nextElement = await observable.FirstAsync();
```

To capture all events in the stream, you can use `ToList`:

```C#
var observable = ...; // as IObservable<int>
var allElements = await observable.ToList(); // as IList<int>
```

### Discussion

The `System.Reactive` library provides all the tools you need to consume streams using `await`. The only tricky part is that you have to think about whether the awaitable will wait until the stream completes. Of the examples in this recipe, `LastAsync`, `ToList`, and the direct await will wait until the stream completes; `FirstAsync` will only wait for the next event.

If these examples don't satisfy your needs, remember that you have the full power of LINQ as well as the `System.Reactive` manipulators. Operators such as `Take` and `Buffer` can also help you asynchronously wait for the elements you need without having to wait for the entire stream to complete.

Some of the operators for use with await—such as `FirstAsync` and `LastAsync`—don't actually return a `Task<T>`. If you plan to use `Task.WhenAll` or `Task.WhenAny`, then you'll need an actual `Task<T>`, which you can get by calling `ToTask` on any observable. `ToTask` will return a `Task<T>` that completes with the last value in the stream.

### See Also

Recipe 8.6 covers using asynchronous code within an observable stream.

Recipe 8.8 covers using observable streams as an input to a dataflow block (which can perform asynchronous work).

Recipe 6.3 covers windows and buffering for observable streams.

## 8.6 System.Reactive Observable Wrappers for async Code

### Problem

You have an asynchronous operation that you want to combine with an observable.

### Solution

Any asynchronous operation can be treated as an observable stream that does one of two things:

Produces a single element and then completes `Fault`s without producing any elements To implements this transformation, the `System.Reactive` library has a simple conversion from `Task<T>` to `IObservable<T>`. The following code starts an asynchronous download of a web page, treating it as an observable sequence:

```C#
IObservable<HttpResponseMessage> GetPage(HttpClient client)
{
  Task<HttpResponseMessage> task = client.GetAsync("http://www.example.com/");
  return task.ToObservable();
}
```

The `ToObservable` approach assumes you have already called the async method and have a `Task` to convert.

Another approach is to call `StartAsync`. `StartAsync` also calls the async method immediately but supports cancellation: if a subscription is disposed of, the async method is canceled:

```C#
IObservable<HttpResponseMessage> GetPage(HttpClient client)
{
  return Observable.StartAsync(
      token => client.GetAsync("http://www.example.com/", token));
}
```

Both `ToObservable` and `StartAsync` immediately start the asynchronous operation without waiting for a subscription; the observable is “hot.” To create a “cold” observable that only starts the operation when subscribed to, use `FromAsync` (which also supports cancellation just like `StartAsync`):

```C#
IObservable<HttpResponseMessage> GetPage(HttpClient client)
{
  return Observable.FromAsync(
      token => client.GetAsync("http://www.example.com/", token));
}
```

`FromAsync` is notably different than `ToObservable` and `StartAsync`, which return an observable for an async operation that has already started. `FromAsync` starts a new, independent async operation every time it is subscribed to. Finally, you can use special overloads of `SelectMany` to start asynchronous operations for each event in a source stream as they arrive. `SelectMany` also supports cancellation.

The following example takes an existing event stream of URLs and then initiates a request as each URL arrives:

```C#
IObservable<HttpResponseMessage> GetPages(IObservable<string> urls, HttpClient client)
{
  return urls.SelectMany(
      (url, token) => client.GetAsync(url, token));
}
```

### Discussion

`System.Reactive` existed before the introduction of async but added these operators (and others) so that it could interoperate well with async code. I recommend that you use the operators described even though you can build the same functionality using other `System.Reactive` operators.

### See Also

Recipe 8.5 covers consuming observable streams with asynchronous code.

Recipe 8.8 covers using dataflow blocks (which can contain asynchronous code) as sources of observable streams.

## 8.7 Asynchronous Streams and Dataflow Meshes

### Problem

Part of your solution uses asynchronous streams, and part of your solution uses dataflow meshes, and you need to pass data between them.

### Solution

Consuming a `Channel` as an asynchronous stream is built right into the `Channel` types; see Recipe 9.8 for details. Consuming a TPL Dataflow block as an asynchronous stream is a bit more awkward, but certainly doable. I find it easiest to first define an extension method for dataflow blocks that make their API more similar to `Channels`, and then consume them as an asynchronous stream using that extension method:

```C#
public static class DataflowExtensions
{
  public static bool TryReceiveItem<T>(this ISourceBlock<T> block,
      out T value)
  {
    if (block is IReceivableSourceBlock<T> receivableSourceBlock)
      return receivableSourceBlock.TryReceive(out value);
    try
    {
      value = block.Receive(TimeSpan.Zero);
      return true;
    }
    catch (TimeoutException)
    {
      // There is no item available right now.
      value = default;
      return false;
    }
    catch (InvalidOperationException)
    {
      // The block is complete and there are no more items.
      value = default;
      return false;
    }
  }

  public static async IAsyncEnumerable<T> ReceiveAllAsync<T>(
      this ISourceBlock<T> block,
      [EnumeratorCancellation] CancellationToken cancellationToken = default)
  {
    while (await block.OutputAvailableAsync(cancellationToken).ConfigureAwait(false))
    {
      while (block.TryReceiveItem(out var value))
      {
        yield return value;
      }
    }
  }
}
```

See Recipe 3.4 for the details on the `EnumeratorCancellation` attribute. Using the extension method in the previous code example, it's possible to consume any output dataflow block as an asynchronous stream:

```C#
var multiplyBlock = new TransformBlock<int, int>(value => value * 2);
multiplyBlock.Post(5);
multiplyBlock.Post(2);
multiplyBlock.Complete();
await foreach (int item in multiplyBlock.ReceiveAllAsync())
{
  Console.WriteLine(item);
}
```

It is also possible to use an asynchronous stream as a source of items for a dataflow block. All you need is a loop to pull the items out and place them into the block. There are a couple of assumptions in the following code that may not be appropriate in every scenario. First, the code assumes you want the block to complete when the stream completes. Second, it begins running on its calling thread; some scenarios may want to always run the entire loop on a threadpool thread:

```C#
public static async Task WriteToBlockAsync<T>(this IAsyncEnumerable<T> enumerable,
    ITargetBlock<T> block,
    CancellationToken token = default)
{
  try
  {
    await foreach (var item in enumerable.WithCancellation(token).ConfigureAwait(false))
    {
      await block.SendAsync(item, token).ConfigureAwait(false);
    }
    block.Complete();
  }
  catch (Exception ex)
  {
    block.Fault(ex);
  }
}
```

### Discussion

The extension methods in this recipe are intended as a starting point. In particular, the `WriteToBlockAsync` extension method does make some assumptions; be sure to consider the behavior of these methods and ensure that their behavior is appropriate for your scenario before using them.

### See Also

Recipe 9.8 covers consuming a `Channel` as an asynchronous stream.

Recipe 3.4 covers canceling asynchronous streams.

Chapter 5 covers recipes for TPL Dataflow.

Chapter 3 covers recipes for asynchronous streams.

## 8.8 System.Reactive Observables and Dataflow Meshes

### Problem

Part of your solution uses `System.Reactive` observables, and part of your solution uses dataflow meshes, and you need them to communicate.

`System.Reactive` observables and dataflow meshes each have their own uses, with some conceptual overlap; this recipe shows how easily they work together so that you can use the best tool for each part of the job.

### Solution

First, let's consider using a dataflow block as an input to an observable stream. The following code creates a buffer block (which does no processing) and creates an observable interface from that block by calling `AsObservable`:

```C#
var buffer = new BufferBlock<int>();
var integers = buffer.AsObservable(); // as IObservable<int>
integers.Subscribe(
    data => Trace.WriteLine(data),
    ex => Trace.WriteLine(ex),
    () => Trace.WriteLine("Done"));
buffer.Post(13);
```

Buffer blocks and observable streams can be completed normally or with error, and the `AsObservable` method will translate the block completion (or fault) into the completion of the observable stream. However, if the block faults with an exception, that exception will be wrapped in an `AggregateException` when it's passed to the observable stream. This is similar to how linked blocks propagate their faults.

It's only a little more complicated to take a mesh and treat it as a destination for an observable stream. The following code calls `AsObserver` to enable a block to subscribe to an observable stream:

```C#
IObservable<DateTimeOffset> ticks =
    Observable.Interval(TimeSpan.FromSeconds(1))
        .Timestamp()
        .Select(x => x.Timestamp)
        .Take(5);
var display = new ActionBlock<DateTimeOffset>(x => Trace.WriteLine(x));
ticks.Subscribe(display.AsObserver());
try
{
  display.Completion.Wait();
  Trace.WriteLine("Done.");
}
catch (Exception ex)
{
  Trace.WriteLine(ex);
}
```

Just as before, the completion of the observable stream is translated to the completion of the block, and any errors from the observable stream are translated to a fault of the block.

### Discussion

Dataflow blocks and observable streams share a lot of conceptual ground. They both have data pass through them, and they both understand completion and faults. They were designed for different scenarios; TPL Dataflow is intended for a mixture of asynchronous and parallel programming, while `System.Reactive` is intended for reactive programming. However, the conceptual overlap is compatible enough that they work very well and naturally together.

### See Also

Recipe 8.5 covers consuming observable streams with asynchronous code.

Recipe 8.6 covers using asynchronous code within an observable stream.

## 8.9 Converting System.Reactive Observables to Asynchronous Streams

### Problem

Part of your solution uses `System.Reactive` observables, and you want to consume them as asynchronous streams.

### Solution

`System.Reactive` observables are push-based, and asynchronous streams are pull-based. So right off the bat, you need to realize there's a conceptual mismatch. You need a way to remain responsive to the observable stream, storing its notifications until the consuming code requests them.

The most straightforward solution is already included in the `System.Linq.Async` library:

```C#
var observable = // as IObservable<long>
    Observable.Interval(TimeSpan.FromSeconds(1));
// WARNING: May consume unbounded memory; see discussion!
var enumerable = // as IAsyncEnumerable<long>
    observable.ToAsyncEnumerable();
```

##### TIP

The `ToAsyncEnumerable` extension method is in the `System.Linq.Async` NuGet package.

However, it's important to recognize that this simple `ToAsyncEnumerable` extension method is using an unbounded producer/consumer queue under the hood. It is essentially the same as an extension method you can write yourself using a `Channel` as an unbounded producer/consumer queue:

```C#
// WARNING: May consume unbounded memory; see discussion!
public static async IAsyncEnumerable<T> ToAsyncEnumerable<T>(this IObservable<T> observable)
{
  var buffer = Channel.CreateUnbounded<T>(); // as Channel<T>
  using (observable.Subscribe(
      value => buffer.Writer.TryWrite(value),
      error => buffer.Writer.Complete(error),
      () => buffer.Writer.Complete()))
  {
    await foreach (T item in buffer.Reader.ReadAllAsync())
      yield return item;
  }
}
```

These are simple solutions, but they use unbounded queues, so they should only be used if you're sure that the consumer can (eventually) keep up with the observable events. It's fine if the producer runs faster than the consumer for a while; during that time, the observable events go into the buffer. As long as the producer eventually catches up, the preceding solutions will work. But if the producer always runs faster than the consumer, the observable events will continue to arrive, expanding the buffer, and eventually use up all the memory for the process.

You can avoid the memory issue by using a bounded queue. The trade-off is that you must decide how to handle extra items if the observable events fill up the queue. One option is to discard the extra items; the following example code uses a bounded channel to throw away the oldest observable notification when the buffer is full:

```C#
// WARNING: May discard items; see discussion!
public static async IAsyncEnumerable<T> ToAsyncEnumerable<T>(this IObservable<T> observable,
    int bufferSize)
{
  var bufferOptions = new BoundedChannelOptions(bufferSize)
  {
    FullMode = BoundedChannelFullMode.DropOldest,
  };
  Channel<T> buffer = Channel.CreateBounded<T>(bufferOptions);
  using (observable.Subscribe(
      value => buffer.Writer.TryWrite(value),
      error => buffer.Writer.Complete(error),
      () => buffer.Writer.Complete()))
  {
    await foreach (T item in buffer.Reader.ReadAllAsync())
      yield return item;
  }
}
```

### Discussion

When you have a producer that runs faster than a consumer, your options are to either buffer the producer items (assuming that the producer eventually catches up), or limit the producer's items. The second solution in this recipe limits the producer's items by dropping ones that don't fit in the buffer. You can also limit the producer's items by using observable operators designed for that, such as `Throttle` or `Sample`; see Recipe 6.4 for details. Depending on your needs, it may be best to `Throttle` or `Sample` the input observable before converting it to an `IAsyncEnumerable<T>` using one of the techniques in this recipe.

Aside from bounded queues and unbounded queues, there's a third option not covered here: use backpressure to notify the observable stream that it must stop producing notifications until the buffer is ready to receive them. Unfortunately, `System.Reactive` hasn't yet standardized on a backpressure pattern, so this isn't a viable option at the time of writing. Backpressure is complex and nuanced, and reactive libraries for other languages have implemented different patterns for backpressure. It remains to be seen whether `System.Reactive` will adopt one of these, invent its own backpressure pattern, or just leave backpressure unsolved.

### See Also

Recipe 6.4 covers `System.Reactive` operators designed to throttle input.

Recipe 9.8 covers using `Channel` as an unbounded producer/consumer queue.

Recipe 9.10 covers using `Channel` as a sampling queue, dropping items when it is full.