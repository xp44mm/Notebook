# Chapter 14. Scenarios

In this chapter, we'll take a look at a variety of types and techniques to address some common scenarios when writing concurrent programs. These kinds of scenarios could fill up another entire book, so I've selected just a few that I've found the most useful.

## 14.1 Initializing Shared Resources

### Problem

You have a resource that is shared between multiple parts of your code. This resource needs to be initialized the first time it is accessed.

### Solution

The .NET framework includes a type specifically for this purpose: `Lazy<T>`.

You construct an instance of the `Lazy<T>` type with a factory delegate that is used to initialize the instance. The instance is then made available via the `Value` property. The following code illustrates the `Lazy<T>` type:

```C#
static int _simpleValue;
static readonly Lazy<int> MySharedInteger = new Lazy<int>(() => _simpleValue++);
void UseSharedInteger()
{
  int sharedValue = MySharedInteger.Value;
}
```

No matter how many threads call `UseSharedInteger` simultaneously, the factory delegate is only executed once, and all threads wait for the same instance. Once it's created, the instance is cached and all future access to the `Value` property returns the same instance (in the preceding example, `MySharedInteger.Value` will always be 0).

A very similar approach can be used if the initialization requires asynchronous work; in this case, you can use a `Lazy<Task<T>>`:

```C#
static int _simpleValue;
static readonly Lazy<Task<int>> MySharedAsyncInteger =
    new Lazy<Task<int>>(async () =>
    {
      await Task.Delay(TimeSpan.FromSeconds(2)).ConfigureAwait(false);
      return _simpleValue++;
    });
async Task GetSharedIntegerAsync()
{
  int sharedValue = await MySharedAsyncInteger.Value;
}
```

In this example, the delegate returns a `Task<int>`, that is, an integer value determined asynchronously. No matter how many parts of the code call `Value` simultaneously, the `Task<int>` is only created once and returned to all callers. Each caller then has the option of (asynchronously) waiting until the task completes by passing the task to await.

The preceding code is an acceptable pattern, but there are some additional considerations. For one, the asynchronous delegate may be executed on any thread that calls `Value`, and that delegate will execute within that context. If there are different thread types that may call `Value` (e.g., a UI thread and a threadpool thread, or two different ASP.NET request threads), then it may be better to always execute the asynchronous delegate on a threadpool thread. This is easy enough to do by wrapping the factory delegate in a call to `Task.Run`:

```C#
static int _simpleValue;
static readonly Lazy<Task<int>> MySharedAsyncInteger =
  new Lazy<Task<int>>(() => Task.Run(async () =>
  {
    await Task.Delay(TimeSpan.FromSeconds(2));
    return _simpleValue++;
  }));
async Task GetSharedIntegerAsync()
{
  int sharedValue = await MySharedAsyncInteger.Value;
}
```

Another consideration is that the `Task<T>` instance is only created once. If the asynchronous delegate throws an exception, then the `Lazy<Task<T>>` will cache that faulted task. This is seldom desirable; usually it's better to re-execute the delegate the next time the lazy value is requested rather than to cache the exception. There isn't a way to “reset” the `Lazy<T>`, but you can create a new class that handles re-creating the `Lazy<T>` instance:

```C#
public sealed class AsyncLazy<T>
{
  private readonly object _mutex;
  private readonly Func<Task<T>> _factory;
  private Lazy<Task<T>> _instance;
  public AsyncLazy(Func<Task<T>> factory)
  {
    _mutex = new object();
    _factory = RetryOnFailure(factory);
    _instance = new Lazy<Task<T>>(_factory);
  }
  private Func<Task<T>> RetryOnFailure(Func<Task<T>> factory)
  {
    return async () =>
    {
      try
      {
        return await factory().ConfigureAwait(false);
      }
      catch
      {
        lock (_mutex)
        {
          _instance = new Lazy<Task<T>>(_factory);
        }
        throw;
      }
    };
  }
  public Task<T> Task
  {
    get
    {
      lock (_mutex)
        return _instance.Value;
    }
  }
}
static int _simpleValue;
static readonly AsyncLazy<int> MySharedAsyncInteger =
  new AsyncLazy<int>(() => Task.Run(async () =>
  {
    await Task.Delay(TimeSpan.FromSeconds(2));
    return _simpleValue++;
  }));
async Task GetSharedIntegerAsync()
{
  int sharedValue = await MySharedAsyncInteger.Task;
}
```


### Discussion

The final code sample in this recipe is a general code pattern for asynchronous lazy initialization, and it's a bit awkward. The `Nito.AsyncEx` library includes an `AsyncLazy<T>` type that acts just like a `Lazy<Task<T>>` that executes its factory delegate on the thread pool and has an option for retrying on failure. It can also be awaited directly, so the declaration and usage look like the following:

```C#
static int _simpleValue;
private static readonly AsyncLazy<int> MySharedAsyncInteger =
  new AsyncLazy<int>(async () =>
  {
    await Task.Delay(TimeSpan.FromSeconds(2));
    return _simpleValue++;
  },
  AsyncLazyFlags.RetryOnFailure);
public async Task UseSharedIntegerAsync()
{
  int sharedValue = await MySharedAsyncInteger;
}
```

##### TIP

The `AsyncLazy<T>` type is in the `Nito.AsyncEx` NuGet package.

### See Also

Chapter 1 covers basic async/await programming.

Recipe 13.1 covers scheduling work to the thread pool.

## 14.2 System.Reactive Deferred Evaluation

### Problem

You want to create a new source observable whenever someone subscribes to it. For example, you want each subscription to represent a different request to a web service.

### Solution

The `System.Reactive` library has an operator `Observable.Defer`, which will execute a delegate each time the observable is subscribed to. This delegate acts as a factory that creates an observable. The following code uses `Defer` to call an asynchronous method every time someone subscribes to the observable:

```C#
void SubscribeWithDefer()
{
  var invokeServerObservable = Observable.Defer(
      () => GetValueAsync().ToObservable());
  invokeServerObservable.Subscribe(_ => { });
  invokeServerObservable.Subscribe(_ => { });
  Console.ReadKey();
}
async Task<int> GetValueAsync()
{
  Console.WriteLine("Calling server...");
  await Task.Delay(TimeSpan.FromSeconds(2));
  Console.WriteLine("Returning result...");
  return 13;
}
```

If you execute this code, you should see this output:

```C#
Calling server...
Calling server...
Returning result...
Returning result...
```

### Discussion

Your own code usually does not subscribe to an observable more than once, but some `System.Reactive` operators do in their implementation. For example, the `Observable.While` operator will resubscribe to a source sequence as long as its condition is true. `Defer` enables you to define an observable that is reevaluated every time a new subscription comes in. This is useful if you need to refresh or update the data for that observable.

### See Also

Recipe 8.6 covers wrapping asynchronous methods in observables.

## 14.3 Asynchronous Data Binding

### Problem

You are retrieving data asynchronously and need to data-bind the results (e.g., in the ViewModel of a Model-View-ViewModel design).

### Solution

When a property is used in data binding, it must immediately and synchronously return some kind of result. If the actual value needs to be determined asynchronously, you can return a default result and later update the property with the correct value.

Keep in mind that asynchronous operations may fail as well as succeed. Since you're writing a ViewModel, you could use data binding to update the UI for an error condition as well.

The `Nito.Mvvm.Async` library has a type `NotifyTask` that can be used for this:

```C#
class MyViewModel
{
  public MyViewModel()
  {
    MyValue = NotifyTask.Create(CalculateMyValueAsync());
  }
  public NotifyTask<int> MyValue { get; private set; }
  private async Task<int> CalculateMyValueAsync()
  {
    await Task.Delay(TimeSpan.FromSeconds(10));
    return 13;
  }
}
```

It's possible to data-bind to various properties on the `NotifyTask<T>` property, as this example shows:

```xaml
<Grid>
  <Label Content="Loading..."
      Visibility="{Binding MyValue.IsNotCompleted,
          Converter={StaticResource BooleanToVisibilityConverter}}"/>
  <Label Content="{Binding MyValue.Result}"
      Visibility="{Binding MyValue.IsSuccessfullyCompleted,
          Converter={StaticResource BooleanToVisibilityConverter}}"/>
  <Label Content="An error occurred" Foreground="Red"
      Visibility="{Binding MyValue.IsFaulted,
          Converter={StaticResource BooleanToVisibilityConverter}}"/>
</Grid>
```

The `MvvmCross` library has a `MvxNotifyTask` that is much the same as `NotifyTask<T>`.

### Discussion

It's also possible to write your own data-binding wrapper instead of using the one from the libraries. The following code gives the basic idea:

```C#
class BindableTask<T> : INotifyPropertyChanged
{
  private readonly Task<T> _task;
  public BindableTask(Task<T> task)
  {
    _task = task;
    var _ = WatchTaskAsync();
  }
  private async Task WatchTaskAsync()
  {
    try
    {
      await _task;
    }
    catch
    {
    }
    OnPropertyChanged("IsNotCompleted");
    OnPropertyChanged("IsSuccessfullyCompleted");
    OnPropertyChanged("IsFaulted");
    OnPropertyChanged("Result");
  }
  public bool IsNotCompleted { get { return !_task.IsCompleted; } }
  public bool IsSuccessfullyCompleted
  {
    get { return _task.Status == TaskStatus.RanToCompletion; }
  }
  public bool IsFaulted { get { return _task.IsFaulted; } }
  public T Result
  {
    get { return IsSuccessfullyCompleted ? _task.Result : default; }
  }
  public event PropertyChangedEventHandler PropertyChanged;
  protected virtual void OnPropertyChanged(string propertyName)
  {
    PropertyChanged?.Invoke(this, new
PropertyChangedEventArgs(propertyName));
  }
}
```

Note that this has an empty catch clause on purpose: that code specifically does want to catch all exceptions and handle those conditions via data binding.

Also, the code explicitly does not want to use `ConfigureAwait(false)` because the `PropertyChanged` event should be raised on the UI thread.

##### TIP

The `NotifyTask` type is in the `Nito.Mvvm.Async` NuGet package. The `MvxNotifyTask` type is in the `MvvmCross` NuGet package.

### See Also

Chapter 1 covers basic async/await programming.

Recipe 2.7 covers using ConfigureAwait.

## 14.4 Implicit State

### Problem

You have some state variables that need to be accessible at different points in your call stack. For example, you have a current operation identifier that you want to use for logging but that you don't want to add as a parameter to every method.

### Solution

The best solution is to add parameters to your methods, store data as members of a class, or use dependency injection to provide data to the different parts of your code. In some situations, however, that would overcomplicate the code.

The `AsyncLocal<T>` type enables you to give your state an object where it can live on a logical “context.” The following code shows how to use `AsyncLocal<T>` to set an operation identifier that is later read by a logging method:

```C#
private static AsyncLocal<Guid> _operationId = new AsyncLocal<Guid>();
async Task DoLongOperationAsync()
{
  _operationId.Value = Guid.NewGuid();
  await DoSomeStepOfOperationAsync();
}
async Task DoSomeStepOfOperationAsync()
{
  await Task.Delay(100); // Some async work
  // Do some logging here.
  Trace.WriteLine("In operation: " + _operationId.Value);
}
```

Many times, it's useful to have a more complex data structure (such as a stack of values) in a single `AsyncLocal<T>` instance. This is possible, with one caveat: you should only store immutable data in the `AsyncLocal<T>`.

Whenever you need to update the data, then you should overwrite the existing value. It is often helpful to hide the `AsyncLocal<T>` inside a helper type that ensures the stored data is immutable and updated correctly:

```C#
internal sealed class AsyncLocalGuidStack
{
  private readonly AsyncLocal<ImmutableStack<Guid>> _operationIds =
      new AsyncLocal<ImmutableStack<Guid>>();
  private ImmutableStack<Guid> Current =>
      _operationIds.Value ?? ImmutableStack<Guid>.Empty;
  public IDisposable Push(Guid value)
  {
    _operationIds.Value = Current.Push(value);
    return new PopWhenDisposed(this);
  }
  private void Pop()
  {
    ImmutableStack<Guid> newValue = Current.Pop();
    if (newValue.IsEmpty)
      newValue = null;
    _operationIds.Value = newValue;
  }
  public IEnumerable<Guid> Values => Current;
  private sealed class PopWhenDisposed : IDisposable
  {
    private AsyncLocalGuidStack _stack;
    public PopWhenDisposed(AsyncLocalGuidStack stack) =>
        _stack = stack;
    public void Dispose()
    {
      _stack?.Pop();
      _stack = null;
    }
  }
}
private static AsyncLocalGuidStack _operationIds = new
AsyncLocalGuidStack();
async Task DoLongOperationAsync()
{
  using (_operationIds.Push(Guid.NewGuid()))
    await DoSomeStepOfOperationAsync();
}
async Task DoSomeStepOfOperationAsync()
{
  await Task.Delay(100); // some async work
  // Do some logging here.
  Trace.WriteLine("In operation: " +
      string.Join(":", _operationIds.Values));
}
```

The wrapper type ensures that the underlying data is immutable and that new values are pushed onto the stack. It also provides a convenient `IDisposable` way of popping values off the stack.

### Discussion

Older code may use the `ThreadStatic` attribute for contextual state used by synchronous code. When converting older code to be asynchronous, `AsyncLocal<T>` is a prime candidate for replacing `ThreadStaticAttribute`.

`AsyncLocal<T>` works for both synchronous and asynchronous code, and should be the default choice for implicit state in modern applications.

### See Also

Chapter 1 covers basic async/await programming.

Chapter 9 covers several immutable collections, for when you need to store complex data as implicit state.

## 14.5 Identical Synchronous and Asynchronous Code

### Problem

You have some code that needs to be exposed through both synchronous and asynchronous APIs, but you don't want to duplicate the logic. You'll often encounter this situation when updating code to be asynchronous, but existing synchronous consumers cannot (yet) be changed.

### Solution

If you can, try to organize your code along modern design guidelines, like Ports and Adapters (Hexagonal Architecture), which separate your business logic from side effects such as I/O. If you can get into that situation, then there's no need to expose both synchronous and asynchronous APIs for anything; your business logic would always be synchronous, and the I/O would always be asynchronous.

However, that's a very lofty goal, and in The Real World, brownfield code can be messy, and there's rarely time to make it perfect before adopting asynchronous code. Existing APIs often need to be maintained for backwards compatibility, even if they were poorly designed.

There is no perfect solution in this scenario. Many developers attempt to have the synchronous code call the asynchronous code, or have the asynchronous code call the synchronous code, but both of those approaches are anti-patterns.

The Boolean Argument Hack is the one that I tend to prefer in this situation.

It's a way to keep all the logic in a single method while exposing both synchronous and asynchronous APIs.

The primary idea of the Boolean Argument Hack is that there's a private core method containing the logic. That core method has an asynchronous signature and takes a boolean argument determining whether the core method should be asynchronous or not. If the boolean argument specifies that the core method should be synchronous, then it must return an already-completed task. Then you can write both asynchronous and synchronous API methods that forward to the core method:

```C#
private async Task<int> DelayAndReturnCore(bool sync)
{
  int value = 100;
  // Do some work.
  if (sync)
    Thread.Sleep(value); // Call synchronous API.
  else
    await Task.Delay(value); // Call asynchronous API.
  return value;
}
// Asynchronous API
public Task<int> DelayAndReturnAsync() =>
    DelayAndReturnCore(sync: false);
// Synchronous API
public int DelayAndReturn() =>
    DelayAndReturnCore(sync: true).GetAwaiter().GetResult();
```

The asynchronous API DelayAndReturnAsync invokes DelayAndReturnCore with the boolean sync parameter set to false; this means that DelayAndReturnCore may behave asynchronously, and it uses await on the underlying asynchronous “delay” API `Task.Delay`. The task returned from DelayAndReturnCore is returned directly to the caller of DelayAndReturnAsync.

The synchronous API DelayAndReturn invokes DelayAndReturnCore with the boolean sync parameter set to true; this means that DelayAndReturnCore must behave synchronously, and it uses the underlying synchronous “delay” API `Thread.Sleep`. The task returned by DelayAndReturnCore must already be complete, so it's safe to extract the result. DelayAndReturn uses `GetAwaiter().GetResult()` to retrieve the result from the task; this avoids an `AggregateException` wrapper that can happen if it were to use the `Task<T>.Result` property.

### Discussion

This isn't an ideal solution, but it's one that can help with real-world applications.

Now, a few caveats for this solution. The most disastrous problems will arise if the Core method doesn't properly honor its sync parameter. If the Core method ever returns an incomplete task when sync is true, then the synchronous API can easily deadlock; the only reason the synchronous API can block on its task is that it knows that the task is already complete. Similarly, if the Core method blocks a thread when sync is false, then the application isn't as efficient as it should be.

One improvement that could be made to this solution is to add a check in the synchronous API, validating that the returned task is in fact completed. If it's ever not completed, then there is a serious coding bug.

### See Also

Chapter 1 covers basic async/await programming, including a discussion of deadlocks that can happen when blocking on asynchronous code in general.

## 14.6 Railway Programming with Dataflow Meshes

### Problem

You have a dataflow mesh set up, but some data items fail to process. You want to respond to these errors in a way that keeps your dataflow mesh operational.

### Solution

By default, if a block encounters an exception when processing a data item, that block will fault, preventing it from processing any more data items. The core idea of this solution is to treat exceptions as just another kind of data. If the dataflow mesh operates on types that can be either an exception or data, then the mesh can remain operational even when exceptions occur and continue to process other data items.

This is sometimes called “railway” programming because the items in the mesh can be viewed as traveling on one of two separate tracks. There's the normal “data” track: if everything goes perfectly, the item stays on the “data” track and travels through the mesh, being transformed and operated on, until it reaches the end of the mesh. The second track is the “error” track; in any block, if an exception is raised when processing an item, that exception transfers to the “error” track and travels through the mesh. Exception items aren't processed; they are just passed on from block to block, so they also reach the end of the mesh. The terminal blocks in the mesh end up receiving a sequence of items, each of which is either a data item or exception item; a data item represents data that has completed the entire mesh successfully, and an exception item represents a processing error at some point in the mesh.

In order to set up this kind of “railway” programming, you first need to define a type that represents either a data item or an exception. If you want to use a pre-built one, there are a few available. This kind of type is common in the functional programming community, where it's commonly called Try or Error or Exceptional, and is a special case of the Either monad. I've defined my own Try<T> type that you can use as an example; it's in the Nito.Try NuGet package and the source code is on GitHub.

Once you have some kind of Try<T> type, setting up the mesh is a bit tedious but not terrible. The type of each dataflow block should be changed from T to Try<T>, and any processing in that block should be done by mapping one Try<T> value to another. With my Try<T> type, this is done by calling Try<T>.Map. I find it helpful to define small factory methods for railway-oriented dataflow blocks instead of having that extra code inline. The following code is an example of a helper method that constructs a TransformBlock that operates on Try<T> values by calling Try<T>.Map:

```C#
private static TransformBlock<Try<TInput>, Try<TOutput>>
    RailwayTransform<TInput, TOutput>(Func<TInput, TOutput> func)
{
  return new TransformBlock<Try<TInput>, Try<TOutput>>(t =>
t.Map(func));
}
```

With helpers like these in place, the dataflow mesh creation code is more straightforward:

```C#
var subtractBlock = RailwayTransform<int, int>(value => value - 2);
var divideBlock = RailwayTransform<int, int>(value => 60 / value);
var multiplyBlock = RailwayTransform<int, int>(value => value * 2);
var options = new DataflowLinkOptions { PropagateCompletion = true };
subtractBlock.LinkTo(divideBlock, options);
divideBlock.LinkTo(multiplyBlock, options);
// Insert data items into the first block.
subtractBlock.Post(Try.FromValue(5));
subtractBlock.Post(Try.FromValue(2));
subtractBlock.Post(Try.FromValue(4));
subtractBlock.Complete();
// Receive data/exception items from the last block.
while (await multiplyBlock.OutputAvailableAsync())
{
  Try<int> item = await multiplyBlock.ReceiveAsync();
  if (item.IsValue)
    Console.WriteLine(item.Value);
  else
    Console.WriteLine(item.Exception.Message);
}
```

### Discussion

Railway programming is a great way to avoid faulting dataflow blocks. Since railway programming is a functional programming construct based on monads, it's a bit awkward when translated to .NET, but it is usable. If you have a dataflow mesh that needs to be fault-tolerant, then railway programming is certainly worth it.

### See Also

Recipe 5.2 covers the normal way exceptions fault blocks and can propagate through a mesh if railway programming is not used.

## 14.7 Throttling Progress Updates

### Problem

You have a long-running operation that reports progress, and you display progress updates in the UI. But the progress updates arrive too rapidly, causing your UI to be unresponsive.

### Solution

Consider the following code, which reports progress very quickly:

```C#
private string Solve(IProgress<int> progress)
{
  // Count as quickly as possible for 3 seconds.
  var endTime = DateTime.UtcNow.AddSeconds(3);
  int value = 0;
  while (DateTime.UtcNow < endTime)
  {
    value++;
    progress?.Report(value);
  }
  return value.ToString();
}
```

You can execute this code from a GUI application by wrapping it in Task.Run and passing in an IProgress<T>. The following example code is for WPF, but the same concepts apply regardless of GUI platform (WPF, Xamarin, or Windows Forms):

```C#
// For simplicity, this code updates a label directly.
// In a real-world MVVM application, those assignments
//  would instead be updating a ViewModel property
//  which is data-bound to the actual UI.
private async void StartButton_Click(object sender, RoutedEventArgs e)
{
  MyLabel.Content = "Starting...";
  var progress = new Progress<int>(value => MyLabel.Content = value);
  var result = await Task.Run(() => Solve(progress));
  MyLabel.Content = $"Done! Result: {result}";
}
```

This code will cause the UI to become unresponsive for quite some time, about 20 seconds on my machine, and then suddenly the UI is responsive again and only displays the "Done! Result:" message. The intermediate progress reports were never seen. What is happening is that the background code is sending progress reports to the UI thread extremely quickly, so fast that after running for only 3 seconds, it takes the UI thread another 17 seconds or so just to process all those progress reports, updating that label over and over. Lastly, the UI thread updates the label one last time with the "Done! Result:" values, and then finally has time to repaint the screen, displaying the updated label value to the user.

The first thing to realize is that we need to throttle the progress reports. It's the only way to ensure the UI has enough time to repaint itself between progress updates. The next thing to realize is that we want to throttle based on time, not the number of reports. While you may be tempted to throttle the progress reports by only sending one out of every hundred or so, this isn't ideal for reasons discussed in the “Discussion” section.

The fact that we want to deal with time indicates that we should consider System.Reactive. And, in fact, System.Reactive has operators specifically designed to throttle on time. So, it sounds like System.Reactive will play a role in this solution.

To get started, you can define an IProgress<T> implementation that raises an event for each progress report, and then create an observable that receives those progress reports by wrapping that event:

```C#
public static class ObservableProgress
{
  private sealed class EventProgress<T> : IProgress<T>
  {
    void IProgress<T>.Report(T value) => OnReport?.Invoke(value);
    public event Action<T> OnReport;
  }
  public static (IObservable<T>, IProgress<T>) Create<T>()
  {
    var progress = new EventProgress<T>();
    var observable = Observable.FromEvent<T>(
        handler => progress.OnReport += handler,
        handler => progress.OnReport -= handler);
    return (observable, progress);
  }
}
```

The method ObservableProgress.Create<T> will create a pair: one IObservable<T> and one IProgress<T>, where all progress reports sent to the IProgress<T> will be sent to the subscribers of the IObservable<T>. We now have an observable stream for our progress reports; the next step is to throttle it.

We want to update the UI slowly enough that it can remain responsive, and we want to update the UI quickly enough that users can see the updates. Human perception is considerably slower than computer displays, so there's a large window of possible values. If you prefer true readability, throttling to one update every second or so may be sufficient. If you prefer more real-time feedback, I find that one update every 100 or 200 milliseconds (ms) is fast enough that the user sees that something is happening fast and gets a general sense of the progress details, while still being slow enough for the UI to remain responsive.

Another point to keep in mind is that progress reports can be raised from other threads—in this case, they are raised from a background thread. The throttling should be done as close to the source as possible, so we want to keep the throttling on the background thread. However, the code that updates the UI needs to be run on the UI thread. With this in mind, you can define a CreateForUi method that handles both the throttling and the transition to the UI thread:

```C#
public static class ObservableProgress
{
  // Note: this must be called from the UI thread.
  public static (IObservable<T>, IProgress<T>) CreateForUi<T>(
      TimeSpan? sampleInterval = null)
  {
    var (observable, progress) = Create<T>();
    observable = observable
        .Sample(sampleInterval ?? TimeSpan.FromMilliseconds(100))
        .ObserveOn(SynchronizationContext.Current);
    return (observable, progress);
  }
}
```

Now you have a helper method that will throttle your progress updates before they hit the UI. You can use the helper method in the previous code example in your button click handler:

```C#
// For simplicity, this code updates a label directly.
// In a real-world MVVM application, those assignments
//  would instead be updating a ViewModel property
//  which is data-bound to the actual UI.
private async void StartButton_Click(object sender, RoutedEventArgs e)
{
  MyLabel.Content = "Starting...";
  var (observable, progress) = ObservableProgress.CreateForUi<int>();
  string result;
  using (observable.Subscribe(value => MyLabel.Content = value))
    result = await Task.Run(() => Solve(progress));
  MyLabel.Content = $"Done! Result: {result}";
}
```

The new code calls our helper method ObservableProgress.CreateForUi, which creates the IObservable<T> and IProgress<T> pair. The code subscribes to the progress updates and keeps that going until Solve is done.

Finally, it passes the IProgress<T> to the long-running Solve method. As Solve calls IProgress<T>.Report, those reports are first sampled within a 100 ms time window, with one update every 100-ms being forwarded to the UI thread and used to update the label text. The UI is now fully responsive!

### Discussion

This recipe is a fun combination of other recipes in this book! No new techniques were introduced; we just walked through which recipes to combine to come up with this solution.

An alternative solution to this problem that you may see a lot in the wild is the “modulus solution.” The idea behind this solution is that Solve itself has to throttle its own progress updates; for example, if the code only wanted to process one update for every 100 actual updates, then the code may use some modulus technique like if (value % 100 == 0) progress?.Report(value);.

There are a couple of problems with the modulus approach. The first is that there's no “correct” modulus value; usually, the developer tries various values until it works well on their own laptop. The same code, however, may not behave well when running on a client's massive server or inside an underpowered virtual machine. In addition, different platforms and environments cache very differently, which can make code run much faster (or slower) than expected. And, of course, the capabilities of the “latest” computer hardware do change over time. So the modulus value only ends up being a guess; it's not going to be correct everywhere and throughout all time.

The other problem with the modulus approach is that it's trying to fix the problem in the wrong part of the code. This problem is purely a UI issue; it's the UI that has a problem, and it's the UI layer that should provide the fix for it.

In the example code for this recipe, Solve represents some background business processing logic; it shouldn't be concerned with UI-specific issues. A Console app may want to use a very different modulus than a WPF app.

The one thing that the modulus approach is correct on is that it's best to throttle the updates before sending the updates to the UI thread. The solution in this recipe also does this: it throttles the updates immediately and synchronously on the background thread before sending them to the UI thread. By injecting its own IProgress<T> implementation, the UI is able to do its own throttling without requiring any changes to the Solve method itself.

### See Also

Recipe 2.3 covers using IProgress<T> to report progress from long-running operations.

Recipe 13.1 covers using Task.Run to run synchronous code on a threadpool thread.

Recipe 6.1 covers using FromEvent to wrap .NET events into observables.

Recipe 6.4 covers using Sample to throttle observables by time.

Recipe 6.2 covers using ObserveOn to move observable notifications to another context.