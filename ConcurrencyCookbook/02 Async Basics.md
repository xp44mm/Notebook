# Chapter 2. Async Basics

This chapter introduces you to the basics of using `async` and `await` for asynchronous operations. Here, we'll only deal with naturally asynchronous operations, which are operations such as HTTP requests, database commands, and web service calls.

If you have a CPU-intensive operation that you want to treat as though it were asynchronous (e.g., so that it doesn't block the UI thread), then see Chapter 4 and Recipe 8.4. Also, this chapter only deals with operations that are started once and complete once; if you need to handle streams of events, then see Chapters 3 and 6.

## 2.1 Pausing for a Period of Time

### Problem

You need to (asynchronously) wait for a period of time. This is a common scenario when unit testing or implementing retry delays. It also comes up when coding simple timeouts.

### Solution

The `Task` type has a static method `Delay` that returns a task that completes after the specified time.

The following example code defines a task that completes asynchronously. When faking an asynchronous operation, it's important to test synchronous success and asynchronous success, as well as asynchronous failure. The following example returns a task used for the asynchronous success case:

```C#
async Task<T> DelayResult<T>(T result, TimeSpan delayTime)
{
  await Task.Delay(delayTime);
  return result;
}
```

**Exponential backoff** is a strategy in which you increase the delays between retries. Use it when working with web services to ensure that the server doesn't get flooded with retries. The next example is a simple implementation of exponential backoff:

```C#
async Task<string> DownloadStringWithRetries(HttpClient client, string uri)
{
  // Retry after 1 second, then after 2 seconds, then 4.
  var nextDelay = TimeSpan.FromSeconds(1);
  for (int i = 0; i < 3; i++)
  {
    try
    {
      return await client.GetStringAsync(uri);
    }
    catch
    {
    }
    await Task.Delay(nextDelay);
    nextDelay = nextDelay * 2;
  }
  // Try one last time, allowing the error to propagate.
  return await client.GetStringAsync(uri);
}
```

##### TIP

For production code, I'd recommend a more thorough solution, such as the `Polly` NuGet library; this code is just a simple example of `Task.Delay` usage.

You can also use `Task.Delay` as a simple timeout. `CancellationTokenSource` is the normal type used to implement a timeout (Recipe 10.3). You can wrap a cancellation token in an infinite `Task.Delay` to provide a task that cancels after a specified time. Finally, use that timer task with `Task.WhenAny` (Recipe 2.5) to implement a “soft timeout.” The following example code returns null if the service doesn't respond within three seconds:

```C#
async Task<string> DownloadStringWithTimeout(HttpClient client, string uri)
{
  using var cts = new CancellationTokenSource(TimeSpan.FromSeconds(3));
  Task<string> downloadTask = client.GetStringAsync(uri);
  Task timeoutTask = Task.Delay(Timeout.InfiniteTimeSpan, cts.Token);
  Task completedTask = await Task.WhenAny(downloadTask, timeoutTask);
  if (completedTask == timeoutTask)
    return null;
  return await downloadTask;
}
```

While it's possible to use `Task.Delay` as a “soft timeout,” this approach has limitations. If the operation times out, it's not canceled; in the previous example, the download task continues downloading and will download the full response before discarding it. The preferred approach is to use a cancellation token as the timeout and pass it directly to the operation (`GetStringAsync` in the last example). That said, sometimes the operation is not cancelable, and in that case `Task.Delay` may be used by other code to act like the operation timed out.

### Discussion

`Task.Delay` is a fine option for unit testing asynchronous code or for implementing retry logic. However, if you need to implement a timeout, a `CancellationToken` is usually a better choice.

### See Also

Recipe 2.5 covers how `Task.WhenAny` is used to determine which task completes first.

Recipe 10.3 covers using `CancellationToken` as a timeout.

## 2.2 Returning Completed Tasks

### Problem

You need to implement a synchronous method with an asynchronous signature. This situation can arise if you're inheriting from an asynchronous interface or base class but want to implement it synchronously. This technique is particularly useful when unit testing asynchronous code, when you need a simple stub or mock for an asynchronous interface.

### Solution

You can use `Task.FromResult` to create and return a new `Task<T>` that is already completed with the specified value:

```C#
interface IMyAsyncInterface
{
  Task<int> GetValueAsync();
}
class MySynchronousImplementation : IMyAsyncInterface
{
  public Task<int> GetValueAsync()
  {
    return Task.FromResult(13);
  }
}
```

For methods that don't have a return value, you can use `Task.CompletedTask`, which is a cached `Task` that is successfully completed:

```C#
interface IMyAsyncInterface
{
  Task DoSomethingAsync();
}
class MySynchronousImplementation : IMyAsyncInterface
{
  public Task DoSomethingAsync()
  {
    return Task.CompletedTask;
  }
}
```

`Task.FromResult` provides completed tasks only for successful results. If you need a task with a different kind of result (e.g., a task that is completed with a `NotImplementedException`), then you can use `Task.FromException`:

```C#
Task<T> NotImplementedAsync<T>()
{
  return Task.FromException<T>(new NotImplementedException());
}
```

Similarly, there's a `Task.FromCanceled` for creating tasks that have already been canceled from a given `CancellationToken`:

```C#
Task<int> GetValueAsync(CancellationToken cancellationToken)
{
  if (cancellationToken.IsCancellationRequested)
    return Task.FromCanceled<int>(cancellationToken);
  return Task.FromResult(13);
}
```

If it is possible for your synchronous implementation to fail, then you should capture exceptions and use `Task.FromException` to return them, as such:

```C#
interface IMyAsyncInterface
{
  Task DoSomethingAsync();
}
class MySynchronousImplementation : IMyAsyncInterface
{
  public Task DoSomethingAsync()
  {
    try
    {
      DoSomethingSynchronously();
      return Task.CompletedTask;
    }
    catch (Exception ex)
    {
      return Task.FromException(ex);
    }
  }
}
```

### Discussion

If you're implementing an asynchronous interface with synchronous code, avoid any form of blocking. It isn't ideal for an asynchronous method to block and then return a completed task, when it is possible for the method to be implemented asynchronously. For a counterexample, consider the `Console` text readers in the .NET BCL. `Console.In.ReadLineAsync` will actually block the calling thread until a line is read, and then it will return a completed task. This behavior isn't intuitive and has surprised many developers. If an asynchronous method blocks, it prevents the calling thread from starting other tasks, which interferes with concurrency and may even cause a deadlock.

If you regularly use `Task.FromResult` with the same value, consider caching the actual task. For example, if you create a `Task<int>` with a zero result once, then you avoid creating extra instances that will have to be garbage-collected:

```C#
private static readonly Task<int> zeroTask = Task.FromResult(0);
Task<int> GetValueAsync()
{
  return zeroTask;
}
```

Logically, `Task.FromResult`, `Task.FromException`, and `Task.FromCanceled` are all helper methods and shortcuts for the general-purpose `TaskCompletionSource<T>`. `TaskCompletionSource<T>` is a lower-level type that is useful for interoperating with other forms of asynchronous code. Generally, you should use the shorthand `Task.FromResult` and friends if you want to return a task that's already been completed. Use `TaskCompletionSource<T>` to return a task that is completed at some future time.

### See Also

Recipe 7.1 covers unit testing asynchronous methods.

Recipe 11.1 covers inheritance of async methods.

Recipe 8.3 shows how `TaskCompletionSource<T>` can be used for general-purpose interop with other asynchronous code.

## 2.3 Reporting Progress

### Problem

You need to respond to progress while an operation is executing.

### Solution

Use the provided `IProgress<T>` and `Progress<T>` types. Your async method should take an `IProgress<T>` argument; the `T` is whatever type of progress you need to report:

```C#
async Task MyMethodAsync(IProgress<double> progress = null)
{
  var done = false;
  var percentComplete = 0.0;
  while (!done)
  {
    ...
    progress?.Report(percentComplete);
  }
}
```

Calling code can use it as such:

```C#
async Task CallMyMethodAsync()
{
  var progress = new Progress<double>();
  progress.ProgressChanged += (sender, args) =>
  {
    ...
  };
  await MyMethodAsync(progress);
}
```

### Discussion

By convention, the `IProgress<T>` parameter may be null if the caller doesn't need progress reports, so be sure to check for this in your async method. Bear in mind that the `IProgress<T>.Report` method is usually asynchronous. This means that MyMethodAsync may continue executing before the progress is reported. For this reason, it's best to define T as an immutable type or at least a value type. If T is a mutable reference type, then you'll have to create a separate copy yourself each time you call `IProgress<T>.Report`.

`Progress<T>` will capture the current context when it is constructed and will invoke its callback within that context. This means that if you construct the `Progress<T>` on the UI thread, then you can update the UI from its callback, even if the asynchronous method is invoking `Report` from a background thread.

When a method supports progress reporting, it should also make a best effort to support cancellation.

`IProgress<T>` is not exclusively for asynchronous code; both progress and cancellation can (and should) be used in long-running synchronous code as well.

### See Also

Recipe 10.4 covers how to support cancellation in an asynchronous method.

## 2.4 Waiting for a Set of Tasks to Complete

### Problem

You have several tasks and need to wait for them all to complete.

### Solution

The framework provides a `Task.WhenAll` method for this purpose. This method takes several tasks and returns a task that completes when all of those tasks have completed:

```C#
Task task1 = Task.Delay(TimeSpan.FromSeconds(1));
Task task2 = Task.Delay(TimeSpan.FromSeconds(2));
Task task3 = Task.Delay(TimeSpan.FromSeconds(1));
await Task.WhenAll(task1, task2, task3);
```

If all the tasks have the same result type and they all complete successfully, then the `Task.WhenAll` task will return an array containing all the task results:

```C#
Task<int> task1 = Task.FromResult(3);
Task<int> task2 = Task.FromResult(5);
Task<int> task3 = Task.FromResult(7);
int[] results = await Task.WhenAll(task1, task2, task3);
// "results" contains { 3, 5, 7 }
```

There is an overload of `Task.WhenAll` that takes an `IEnumerable` of tasks; however, I don't recommend that you use it. Whenever I mix asynchronous code with LINQ, I find the code is clearer when I explicitly “reify” the sequence (i.e., evaluate the sequence, creating a collection):

```C#
async Task<string> DownloadAllAsync(HttpClient client, IEnumerable<string> urls)
{
  // Define the action to do for each URL.
  var downloads = urls.Select(url => client.GetStringAsync(url));
  // Note that no tasks have actually started yet
  //  because the sequence is not evaluated.
  // Start all URLs downloading simultaneously.
  Task<string>[] downloadTasks = downloads.ToArray();
  // Now the tasks have all started.
  // Asynchronously wait for all downloads to complete.
  string[] htmlPages = await Task.WhenAll(downloadTasks);
  return string.Concat(htmlPages);
}
```

### Discussion

If any of the tasks throws an exception, then `Task.WhenAll` will fault its returned task with that exception. If multiple tasks throw an exception, then all of those exceptions are placed on the `Task` returned by `Task.WhenAll`. However, when that task is awaited, only one of them will be thrown. If you need each specific exception, you can examine the `Exception` property on the `Task` returned by `Task.WhenAll`:

```C#
async Task ThrowNotImplementedExceptionAsync()
{
  throw new NotImplementedException();
}
async Task ThrowInvalidOperationExceptionAsync()
{
  throw new InvalidOperationException();
}
async Task ObserveOneExceptionAsync()
{
  var task1 = ThrowNotImplementedExceptionAsync();
  var task2 = ThrowInvalidOperationExceptionAsync();
  try
  {
    await Task.WhenAll(task1, task2);
  }
  catch (Exception ex)
  {
    // "ex" is either NotImplementedException or InvalidOperationException.
    ...
  }
}
async Task ObserveAllExceptionsAsync()
{
  var task1 = ThrowNotImplementedExceptionAsync();
  var task2 = ThrowInvalidOperationExceptionAsync();
  var allTasks = Task.WhenAll(task1, task2);
  try
  {
    await allTasks;
  }
  catch
  {
    AggregateException allExceptions = allTasks.Exception;
    ...
  }
}
```

Most of the time, I do not observe all the exceptions when using `Task.WhenAll`. It's usually sufficient to respond to only the first error that was thrown, rather than all of them.

Note that in the preceding example, the `ThrowNotImplementedExceptionAsync` and `ThrowInvalidOperationExceptionAsync` methods don't throw their exceptions directly; they use the `async` keyword, so their exceptions are captured and placed on a task that is returned normally. This is the normal and expected behavior of methods that return awaitable types.

### See Also

Recipe 2.5 covers a way to wait for any of a collection of tasks to complete.

Recipe 2.6 covers waiting for a collection of tasks to complete and performing actions as each one completes.

Recipe 2.8 covers exception handling for async Task methods.

## 2.5 Waiting for Any Task to Complete

### Problem

You have several tasks and need to respond to just one of them that's completing. You'll encounter this problem most commonly when you have multiple independent attempts at an operation, with a first-one-takes-all kind of structure. For example, you could request stock quotes from multiple web services simultaneously, but you only care about the first one that responds.

### Solution

Use the `Task.WhenAny` method. The `Task.WhenAny` method takes a sequence of tasks and returns a task that completes when any of the tasks complete. The result of the returned task is the task that completed. Don't worry if that sounds confusing; it's one of those things that's difficult to explain but is easier to understand with code:

```C#
// Returns the length of data at the first URL to respond.
async Task<int> FirstRespondingUrlAsync(HttpClient client, string urlA, string urlB)
{
  // Start both downloads concurrently.
  Task<byte[]> downloadTaskA = client.GetByteArrayAsync(urlA);
  Task<byte[]> downloadTaskB = client.GetByteArrayAsync(urlB);
  // Wait for either of the tasks to complete.
  Task<byte[]> completedTask =
      await Task.WhenAny(downloadTaskA, downloadTaskB);
  // Return the length of the data retrieved from that URL.
  byte[] data = await completedTask;
  return data.Length;
}
```

### Discussion

The task returned by `Task.WhenAny` never completes in a faulted or canceled state. This “outer” task always completes successfully, and its result value is the first `Task` to complete (the “inner” task). If the inner task completed with an exception, then that exception is not propagated to the outer task (the one returned by `Task.WhenAny`). You should usually await the inner task after it has completed to ensure any exceptions are observed.

When the first task completes, consider whether to cancel the remaining tasks. If the other tasks aren't canceled but are also never awaited, then they are abandoned. Abandoned tasks will run to completion, and their results will be ignored. Any exceptions from those abandoned tasks will also be ignored. If these tasks aren't canceled, they do continue to run and can use resources unnecessarily, such as HTTP connections, DB connections, or timers.

It is possible to use `Task.WhenAny` to implement timeouts (e.g., using `Task.Delay` as one of the tasks), but it's not recommended. It's more natural to express timeouts with cancellation, and cancellation has the added benefit that it can actually cancel the operation(s) if they time out.

Another anti-pattern for `Task.WhenAny` is handling tasks as they complete. At first it seems reasonable to keep a list of tasks and remove each task from the list as it completes. The problem with this approach is that it executes in O(N²) time, when an O(N) algorithm exists. The proper O(N) algorithm is discussed in Recipe 2.6.

### See Also

Recipe 2.4 covers asynchronously waiting for all of a collection of tasks to complete.

Recipe 2.6 covers waiting for a collection of tasks to complete and performing actions as each one completes.

Recipe 10.3 covers using a cancellation token to implement timeouts.

## 2.6 Processing Tasks as They Complete

### Problem

You have a collection of tasks to await, and you want to do some processing on each task after it completes. However, you want to do the processing for each one as soon as it completes, not waiting for any of the other tasks.

The following example code kicks off three delay tasks and then awaits each one:

```C#
async Task<int> DelayAndReturnAsync(int value)
{
  await Task.Delay(TimeSpan.FromSeconds(value));
  return value;
}
// Currently, this method prints "2", "3", and "1".
// The desired behavior is for this method to print "1", "2", and "3".
async Task ProcessTasksAsync()
{
  // Create a sequence of tasks.
  Task<int> taskA = DelayAndReturnAsync(2);
  Task<int> taskB = DelayAndReturnAsync(3);
  Task<int> taskC = DelayAndReturnAsync(1);
  Task<int>[] tasks = new[] { taskA, taskB, taskC };
  // Await each task in order.
  foreach (Task<int> task in tasks)
  {
    var result = await task;
    Trace.WriteLine(result);
  }
}
```

The code currently awaits each task in sequence order, even though the third task in the sequence is the first one to complete. You want the code to do the processing (e.g., `Trace.WriteLine`) as each task completes without waiting for the others.

### Solution

There are a few different approaches you can take to solve this problem. The one described first in this recipe is the recommended approach; another is described in the “Discussion” section.

The easiest solution is to restructure the code by introducing a higher-level async method that handles awaiting the task and processing its result. Once the processing is factored out, the code is significantly simplified:

```C#
async Task<int> DelayAndReturnAsync(int value)
{
  await Task.Delay(TimeSpan.FromSeconds(value));
  return value;
}
async Task AwaitAndProcessAsync(Task<int> task)
{
  int result = await task;
  Trace.WriteLine(result);
}
// This method now prints "1", "2", and "3".
async Task ProcessTasksAsync()
{
  // Create a sequence of tasks.
  Task<int> taskA = DelayAndReturnAsync(2);
  Task<int> taskB = DelayAndReturnAsync(3);
  Task<int> taskC = DelayAndReturnAsync(1);
  Task<int>[] tasks = new[] { taskA, taskB, taskC };
  IEnumerable<Task> taskQuery =
      from t in tasks
      select AwaitAndProcessAsync(t);
  Task[] processingTasks = taskQuery.ToArray();
  // Await all processing to complete
  await Task.WhenAll(processingTasks);
}
```

Alternatively, this code can be written like this:

```C#
async Task<int> DelayAndReturnAsync(int value)
{
  await Task.Delay(TimeSpan.FromSeconds(value));
  return value;
}
// This method now prints "1", "2", and "3".
async Task ProcessTasksAsync()
{
  // Create a sequence of tasks.
  Task<int> taskA = DelayAndReturnAsync(2);
  Task<int> taskB = DelayAndReturnAsync(3);
  Task<int> taskC = DelayAndReturnAsync(1);
  Task<int>[] tasks = new[] { taskA, taskB, taskC };
  Task[] processingTasks = tasks.Select(async t =>
  {
    var result = await t;
    Trace.WriteLine(result);
  }).ToArray();
  // Await all processing to complete
  await Task.WhenAll(processingTasks);
}
```

The refactoring shown is the cleanest and most portable way to solve this problem. Note that it is subtly different than the original code. This solution will do the task processing concurrently, whereas the original code would do the task processing one at a time. Typically this isn't a problem, but if it's not acceptable for your situation, then consider using locks (Recipe 12.2) or the following alternative solution.

### Discussion

If refactoring isn't a palatable solution, then there is an alternative. Stephen Toub and Jon Skeet have both developed an extension method that returns an array of tasks that will complete in order. Stephen Toub's solution is available on the Parallel Programming with .NET blog, and Jon Skeet's solution is available on his coding blog.

##### TIP

The `OrderByCompletion` extension method is also available in the open source `AsyncEx` library, in the `Nito.AsyncEx` NuGet package.

Using an extension method like `OrderByCompletion` minimizes the changes to the original code:

```C#
async Task<int> DelayAndReturnAsync(int value)
{
  await Task.Delay(TimeSpan.FromSeconds(value));
  return value;
}
// This method now prints "1", "2", and "3".
async Task UseOrderByCompletionAsync()
{
  // Create a sequence of tasks.
  Task<int> taskA = DelayAndReturnAsync(2);
  Task<int> taskB = DelayAndReturnAsync(3);
  Task<int> taskC = DelayAndReturnAsync(1);
  Task<int>[] tasks = new[] { taskA, taskB, taskC };
  // Await each one as they complete.
  foreach (Task<int> task in tasks.OrderByCompletion())
  {
    int result = await task;
    Trace.WriteLine(result);
  }
}
```

### See Also

Recipe 2.4 covers asynchronously waiting for a sequence of tasks to complete.

## 2.7 Avoiding Context for Continuations

### Problem

When an async method resumes after an await, by default it will resume executing within the same context. This can cause performance problems if that context was a UI context and a large number of async methods are resuming on the UI context.

### Solution

To avoid resuming on a context, await the result of `ConfigureAwait` and pass false for its `continueOnCapturedContext` parameter:

```C#
async Task ResumeOnContextAsync()
{
  await Task.Delay(TimeSpan.FromSeconds(1));
  // This method resumes within the same context.
}
async Task ResumeWithoutContextAsync()
{
  await Task.Delay(TimeSpan.FromSeconds(1)).ConfigureAwait(false);
  // This method discards its context when it resumes.
}
```

表示离去了就不回来了，忘记来时路。

### Discussion

Having too many continuations run on the UI thread can cause a performance problem. This type of performance problem is difficult to diagnose, since it's not a single method that is slowing down the system. Rather, the UI performance begins to suffer from “thousands of paper cuts” as the application grows more complex.

The real question is, how many continuations on the UI thread are too many? There's no hard-and-fast answer, but Lucian Wischik of Microsoft has publicized the guideline used by the Universal Windows team: a hundred or so per second is OK, but a thousand or so per second is too many.

It's best to avoid this problem right at the beginning. For every async method you write, if it doesn't need to resume to its original context, then use `ConfigureAwait`. There's no disadvantage to doing so.

It's also a good idea to be aware of context when writing async code. Normally, an async method should either require context (dealing with UI elements or ASP.NET requests/responses) or be free from context (doing background operations). If you have an async method that has parts requiring context and parts free from context, consider splitting it up into two (or more) async methods. This approach helps keep your code better organized into layers.

### See Also

Chapter 1 covers an introduction to asynchronous programming.

## 2.8 Handling Exceptions from async Task Methods

### Problem

Exception handling is a critical part of any design. It's easy to design for the success case, but a design isn't correct until it also handles the failure cases. Fortunately, handling exceptions from async Task methods is straightforward.

### Solution

Exceptions can be caught by a simple try/catch, just like you would do for synchronous code:

```C#
async Task ThrowExceptionAsync()
{
  await Task.Delay(TimeSpan.FromSeconds(1));
  throw new InvalidOperationException("Test");
}
```

Exceptions raised from async Task methods are placed on the returned `Task`. They are only raised when the returned `Task` is awaited:

```C#
async Task TestAsync()
{
  // The exception is thrown by the method and placed on the `tsk`.
  var tsk = ThrowExceptionAsync();
  try
  {
    // The exception is re-raised here, where the `tsk` is awaited.
    await tsk;
  }
  catch (InvalidOperationException e)
  {
    // The exception is correctly caught here.
  }
}
```

### Discussion

When an exception is thrown out of an async Task method, that exception is captured and put on the returned `Task`. Since async void methods don't have a `Task` to put their exception on, their behavior is different; catching exceptions from async void methods is covered in Recipe 2.9.

When you await a faulted `Task`, the first exception on that task is re-thrown. If you're familiar with the problems of re-throwing exceptions, you may be wondering about stack traces. Rest assured: when the exception is re-thrown, the original stack trace is correctly preserved.

This setup sounds somewhat complicated, but all this complexity works together so that the simple scenario has simple code. Most of the time, your code should propagate exceptions from asynchronous methods that it calls; all it has to do is await the task returned from that asynchronous method, and the exception will be propagated naturally.

There are some situations (such as `Task.WhenAll`) where a `Task` may have multiple exceptions, and await will only re-throw the first one. See Recipe 2.4 for an example of handling all exceptions.

### See Also

Recipe 2.4 covers waiting for multiple tasks.

Recipe 2.9 covers techniques for catching exceptions from async void methods.

Recipe 7.2 covers unit testing exceptions thrown from async Task methods.

## 2.9 Handling Exceptions from async void Methods

### Problem

You have an async void method and need to handle exceptions propagated out of that method.

### Solution

There is no good solution. If at all possible, change the method to return `Task` instead of `void`. In some situations, doing that isn't possible; for example, let's say you need to unit test an `ICommand` implementation (which must return `void`). In this case, you can provide a Task-returning overload of your `Execute` method:

```C#
sealed class MyAsyncCommand : ICommand
{
  async void ICommand.Execute(object parameter)
  {
    await Execute(parameter);
  }
  public async Task Execute(object parameter)
  {
    ... // Asynchronous command implementation goes here.
  }
  ... // Other members (CanExecute, etc.)
}
```

It's best to avoid propagating exceptions out of async void methods. If you must use an async void method, consider wrapping all of its code in a try block and handling the exception directly.

There is another solution for handling exceptions from async void methods. When an async void method propagates an exception, that exception is then raised on the `SynchronizationContext` that was active at the time the async void method started executing. If your execution environment provides a `SynchronizationContext`, then it usually has a way to handle these top-level exceptions at a global scope. For example, WPF has `Application.DispatcherUnhandledException`, Universal Windows has `Application.UnhandledException`, and ASP.NET has the `UseExceptionHandler` middleware.

It is also possible to handle exceptions from async void methods by controlling the `SynchronizationContext`. Writing your own `SynchronizationContext` isn't easy, but you can use the `AsyncContext` type from the free `Nito.AsyncEx` NuGet helper library. `AsyncContext` is particularly useful for applications that don't have a built-in `SynchronizationContext`, such as Console applications and Win32 services. The next example uses `AsyncContext` to run and handle exceptions from an async void method:

```C#
static class Program
{
  static void Main(string[] args)
  {
    try
    {
      AsyncContext.Run(() => MainAsync(args));
    }
    catch (Exception ex)
    {
      Console.Error.WriteLine(ex);
    }
  }
  // BAD CODE!!!
  // In the real world, do not use async void unless you have to.
  static async void MainAsync(string[] args)
  {
    ...
  }
}
```

### Discussion

One reason to prefer `async Task` over `async void` is that Task-returning methods are easier to test. At the very least, overloading void-returning methods with Task-returning methods will give you a testable API surface. If you do need to provide your own `SynchronizationContext` type (for example, `AsyncContext`), be sure not to install that `SynchronizationContext` on any threads that don't belong to you. As a general rule, you shouldn't place this type on any thread that already has one (such as UI or ASP.NET classic request threads); nor should you place a `SynchronizationContext` on `threadpool` threads. The main thread of a Console application does belong to you, and so do any threads you manually create yourself.

##### TIP

The `AsyncContext` type is in the `Nito.AsyncEx` NuGet package.

### See Also

Recipe 2.8 covers exception handling with async Task methods.

Recipe 7.3 covers unit testing async void methods.

## 2.10 Creating a ValueTask

### Problem

You need to implement a method that returns `ValueTask<T>`.

### Solution

`ValueTask<T>` is used as a return type in scenarios where there's usually a synchronous result that can be returned and asynchronous behavior is more rare. As a general rule, for your own application code, you should use `Task<T>` as a return type and not `ValueTask<T>`. Only consider using `ValueTask<T>` as a return type in your own application after profiling shows that you'd see a performance increase. That said, there are situations where you need to implement a method that returns `ValueTask<T>`. One such situation is `IAsyncDisposable`, whose DisposeAsync method returns `ValueTask`. See Recipe 11.6 for a more detailed discussion of asynchronous disposal.

The easiest way to implement a method that returns `ValueTask<T>` is to use async and await just like a normal async method:

```C#
public async ValueTask<int> MethodAsync()
{
  await Task.Delay(100); // asynchronous work.
  return 13;
}
```

Many times a method returning `ValueTask<T>` is capable of returning a value immediately; in that case, you can optimize for that scenario using the `ValueTask<T>` constructor, and then forward to the slow asynchronous method only if necessary:

```C#
public ValueTask<int> MethodAsync()
{
  if (CanBehaveSynchronously)
    return new ValueTask<int>(13);
  return new ValueTask<int>(SlowMethodAsync());
}
private Task<int> SlowMethodAsync();
```

A similar approach is possible for the non-generic `ValueTask`. Here, the `ValueTask` default constructor is used to return a successfully completed `ValueTask`. The following example shows an `IAsyncDisposable` implementation that only runs its asynchronous disposal logic once; on future invocations, the DisposeAsync method completes successfully and synchronously:

```C#
private Func<Task> _disposeLogic;
public ValueTask DisposeAsync()
{
  if (_disposeLogic == null)
    return default;
  // Note: this simple example is not threadsafe;
  //  if multiple threads call DisposeAsync,
  //  the logic could run more than once.
  Func<Task> logic = _disposeLogic;
  _disposeLogic = null;
  return new ValueTask(logic());
}
```

### Discussion

Most of your methods should return `Task<T>`, since consuming `Task<T>` has fewer pitfalls than consuming `ValueTask<T>`. See Recipe 2.11 for details on these pitfalls.

Most often, if you're just implementing interfaces that use `ValueTask` or `ValueTask<T>`, then you can simply use async and await. The more advanced implementations are for when you want to use `ValueTask<T>` yourself.

The approaches covered in this recipe are the simpler and more common approaches to creating `ValueTask<T>` and `ValueTask` instances. There is another approach more suitable to more advanced scenarios, when you need to absolutely minimize the allocations used. This more advanced approach enables you to cache or pool an `IValueTaskSource<T>` implementation and reuse it for multiple asynchronous method invocations. To get started with the advanced scenario, see the Microsoft docs for the `ManualResetValueTaskSourceCore<T>` type.

### See Also

Recipe 2.11 covers limitations of consuming `ValueTask<T>` and `ValueTask` types.

Recipe 11.6 covers asynchronous disposal.

## 2.11 Consuming a ValueTask

### Problem

You need to consume a `ValueTask<T>` value.

### Solution

Using await is the most straightforward and common way to consume a `ValueTask<T>` or `ValueTask` value. The majority of the time, this is all you need to do:

```C#
ValueTask<int> MethodAsync();
async Task ConsumingMethodAsync()
{
  int value = await MethodAsync();
}
```

You can also do the await after doing a concurrent operation, as with `Task<T>`:

```C#
ValueTask<int> MethodAsync();
async Task ConsumingMethodAsync()
{
  ValueTask<int> valueTask = MethodAsync();
  ... // other concurrent work
  int value = await valueTask;
}
```

Both of these are appropriate because the `ValueTask` is only awaited a single time. This is one of the restrictions of `ValueTask`.

##### WARNING

A `ValueTask` or `ValueTask<T>` may only be awaited once.

To do anything more complex, convert the `ValueTask<T>` into a `Task<T>` by calling `AsTask`:

```C#
ValueTask<int> MethodAsync();
async Task ConsumingMethodAsync()
{
  Task<int> task = MethodAsync().AsTask();
  ... // other concurrent work
  int value = await task;
  int anotherValue = await task;
}
```

It's perfectly safe to await a `Task<T>` multiple times. You can do other things with it, too, like asynchronously wait for multiple operations to complete (see Recipe 2.4):

```C#
ValueTask<int> MethodAsync();
async Task ConsumingMethodAsync()
{
  Task<int> task1 = MethodAsync().AsTask();
  Task<int> task2 = MethodAsync().AsTask();
  int[] results = await Task.WhenAll(task1, task2);
}
```

However, for each `ValueTask<T>`, you can only call `AsTask` once. The usual approach is to convert it to a `Task<T>` immediately and then ignore the `ValueTask<T>`. Also note that you cannot both await and call `AsTask` on the same `ValueTask<T>`.

Most code should either immediately await a `ValueTask<T>` or convert it to a `Task<T>`.

### Discussion

Other properties on `ValueTask<T>` are for more advanced usage. They don't tend to act like other properties you may be familiar with; in particular, `ValueTask<T>.Result` has more restrictions than `Task<T>.Result`. Code that synchronously retrieves a result from a `ValueTask<T>` may call `ValueTask<T>.Result` or `ValueTask<T>.GetAwaiter().GetResult()`, but these members must not be called until the `ValueTask<T>` is complete. Synchronously retrieving a result from `Task<T>` blocks the calling thread until the task completes; `ValueTask<T>` makes no such guarantees.

Synchronously getting results from a `ValueTask` or `ValueTask<T>` may only be done once, after the `ValueTask` has completed, and that same `ValueTask` cannot be awaited or converted to a task.

At the risk of being repetitive, when your code calls a method returning `ValueTask` or `ValueTask<T>`, it should either immediately await that `ValueTask` or immediately call `AsTask` to convert it to a `Task`. This simple guideline doesn't cover all the advanced scenarios, but most applications will never need to do more than that.

### See Also

Recipe 2.10 covers how to return `ValueTask<T>` and `ValueTask` values from your methods.

Recipes 2.4 and 2.5 cover waiting for multiple tasks simultaneously.
