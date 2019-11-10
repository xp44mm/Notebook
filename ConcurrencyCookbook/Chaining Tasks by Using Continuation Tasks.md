# Chaining Tasks by Using Continuation Tasks

https://docs.microsoft.com/en-us/dotnet/standard/parallel-programming/chaining-tasks-by-using-continuation-tasks?view=netcore-3.0

In asynchronous programming, it is common for one asynchronous operation, on completion, to invoke a second operation and pass data to it. Traditionally, continuations have been done by using callback methods. In the Task Parallel Library, the same functionality is provided by *continuation tasks*. A continuation task (also known just as a continuation) is an asynchronous task that is invoked by another task, which is known as the *antecedent*, when the antecedent finishes.

Continuations are relatively easy to use, but are nevertheless powerful and flexible. For example, you can:

- Pass data from the antecedent to the continuation.
- Specify the precise conditions under which the continuation will be invoked or not invoked.
- Cancel a continuation either before it starts or cooperatively as it is running.
- Provide hints about how the continuation should be scheduled.
- Invoke multiple continuations from the same antecedent.
- Invoke one continuation when all or any one of multiple antecedents complete.
- Chain continuations one after another to any arbitrary length.
- Use a continuation to handle exceptions thrown by the antecedent.

## About continuations

A continuation is a task that is created in the `WaitingForActivation` state. It is activated automatically when its antecedent task(s) complete. Calling `Task.Start` on a continuation in user code throws an `System.InvalidOperationException` exception.

A continuation is itself a `Task` and does not block the thread on which it is started. Call the `Task.Wait` method to block until the continuation task finishes.

笔记：`Task`位于`System.Threading.Tasks`名字空间

## Creating a continuation for a single antecedent

You create a continuation that executes when its antecedent has completed by calling the `Task.ContinueWith` method. The following example shows the basic pattern (for clarity, exception handling is omitted). It executes an antecedent task, `taskA`, that returns a `DayOfWeek` object that indicates the name of the current day of the week. When the antecedent completes, the continuation task, `continuation`, is passed the antecedent and displays a string that includes its result.

##### Note

> The C# samples in this article make use of the `async` modifier on the `Main` method. 

```csharp
   public static async Task Main()
   {
      // Execute the antecedent.
      Task<DayOfWeek> taskA = Task.Run( () => DateTime.Today.DayOfWeek );

      // Execute the continuation when the antecedent finishes.
      await taskA.ContinueWith( antecedent => Console.WriteLine("Today is {0}.", antecedent.Result) );
   }
```

The example displays output like the following output:

```cs
Today is Monday.
```

## Creating a continuation for multiple antecedents

You can also create a continuation that will run when any or all of a group of tasks has completed. To execute a continuation when all antecedent tasks have completed, you call the static `Task.WhenAll` method or the instance `TaskFactory.ContinueWhenAll` method. To execute a continuation when any of the antecedent tasks has completed, you call the static  `Task.WhenAny` method or the instance `TaskFactory.ContinueWhenAny` method.

Note that calls to the `Task.WhenAll` and `Task.WhenAny` overloads do not block the calling thread. However, you typically call all but the `Task.WhenAll(IEnumerable)` and `Task.WhenAll(Task[])` methods to retrieve the returned `Task.Result` property, which does block the calling thread.

The following example calls the `Task.WhenAll(IEnumerable)` method to create a continuation task that reflects the results of its 10 antecedent tasks. Each antecedent task squares an index value that ranges from 1 to 10. If the antecedents complete successfully (their `Task.Status` property is `TaskStatus.RanToCompletion`), the `Task.Result` property of the continuation is an array of the `Task.Result` values returned by each antecedent. The example adds them to compute the sum of squares for all numbers between 1 and 10.

```csharp
   public static void Main()
   {
      var tasks = new List<Task<int>>();
      for (int ctr = 1; ctr <= 10; ctr++) {
         int baseValue = ctr;
         tasks.Add(Task.Factory.StartNew( (b) => { var i = (int) b;
                                                   return i * i; }, baseValue));
      }
      var continuation = Task.WhenAll(tasks);

      long sum = 0;
      for (int ctr = 0; ctr <= continuation.Result.Length - 1; ctr++) {
         Console.Write("{0} {1} ", continuation.Result[ctr],
                       ctr == continuation.Result.Length - 1 ? "=" : "+");
         sum += continuation.Result[ctr];
      }
      Console.WriteLine(sum);
   }
```

The example displays the following output:

```cs
1 + 4 + 9 + 16 + 25 + 36 + 49 + 64 + 81 + 100 = 385
```

## Continuation options

When you create a single-task continuation, you can use a `ContinueWith` overload that takes a `TaskContinuationOptions` enumeration value to specify the conditions under which the continuation starts. For example, you can specify that the continuation is to run only if the antecedent completes successfully, or only if it completes in a faulted state. If the condition is not true when the antecedent is ready to invoke the continuation, the continuation transitions directly to the `TaskStatus.Canceled` state and subsequently cannot be started.

笔记：`TaskContinuationOptions`位于`System.Threading.Tasks`名字空间

A number of multi-task continuation methods, such as overloads of the `TaskFactory.ContinueWhenAll` method, also include a `TaskContinuationOptions` parameter. Only a subset of all `TaskContinuationOptions` enumeration members are valid, however. You can specify `TaskContinuationOptions` values that have counterparts in the `TaskCreationOptions` enumeration, such as `TaskContinuationOptions.AttachedToParent`, `TaskContinuationOptions.LongRunning`, and `TaskContinuationOptions.PreferFairness`. If you specify any of the `NotOn` or `OnlyOn` options with a multi-task continuation, an `ArgumentOutOfRangeException` exception will be thrown at run time.

笔记：`TaskCreationOptions`位于`System.Threading.Tasks`名字空间

## Passing Data to a Continuation

The `Task.ContinueWith` method passes a reference to the antecedent to the user delegate of the continuation as an argument. If the antecedent is a `Task` object, and the task ran until it was completed, then the continuation can access the `Task.Result` property of the task.

The `Task.Result` property blocks until the task has completed. However, if the task was canceled or faulted, attempting to access the `Result` property throws an `AggregateException` exception. You can avoid this problem by using the `OnlyOnRanToCompletion` option, as shown in the following example.

```csharp
using System;
using System.Threading.Tasks;

public class Example
{
   public static async Task Main()
   {
      var t = Task.Run( () => { DateTime dat = DateTime.Now;
                                if (dat == DateTime.MinValue)
                                   throw new ArgumentException("The clock is not working.");
                                   
                                if (dat.Hour > 17)
                                   return "evening";
                                else if (dat.Hour > 12)
                                   return "afternoon";
                                else
                                   return "morning"; });
      await t.ContinueWith( (antecedent) => { Console.WriteLine("Good {0}!",
                                                                  antecedent.Result);
                                                Console.WriteLine("And how are you this fine {0}?",
                                                                  antecedent.Result); },
                              TaskContinuationOptions.OnlyOnRanToCompletion);
   }
}
// The example displays output like the following:
//       Good afternoon!
//       And how are you this fine afternoon?
```

If you want the continuation to run even if the antecedent did not run to successful completion, you must guard against the exception. One approach is to test the `Task.Status` property of the antecedent, and only attempt to access the `Result` property if the status is not `Faulted` or `Canceled`. You can also examine the `Exception` property of the antecedent. For more information, see `Exception Handling`. The following example modifies the previous example to access antecedent's `Task.Result` property only if its status is `TaskStatus.RanToCompletion`.


```csharp
using System;
using System.Threading.Tasks;

public class Example
{
   public static void Main()
   {
      var t = Task.Run( () => { DateTime dat = DateTime.Now;
                                if (dat == DateTime.MinValue)
                                   throw new ArgumentException("The clock is not working.");
                                   
                                if (dat.Hour > 17)
                                   return "evening";
                                else if (dat.Hour > 12)
                                   return "afternoon";
                                else
                                   return "morning"; });
      var c = t.ContinueWith( (antecedent) => { if (t.Status == TaskStatus.RanToCompletion) {
                                                   Console.WriteLine("Good {0}!",
                                                                     antecedent.Result);
                                                   Console.WriteLine("And how are you this fine {0}?",
                                                                  antecedent.Result);
                                                }
                                                else if (t.Status == TaskStatus.Faulted) {
                                                   Console.WriteLine(t.Exception.GetBaseException().Message);
                                                }} );
   }
}
// The example displays output like the following:
//       Good afternoon!
//       And how are you this fine afternoon?
```

## Canceling a Continuation

The `Task.Status` property of a continuation is set to `TaskStatus.Canceled` in the following situations:

- It throws an `OperationCanceledException` exception in response to a cancellation request. As with any task, if the exception contains the same token that was passed to the continuation, it is treated as an acknowledgement of cooperative cancellation.
- The continuation is passed a `System.Threading.CancellationToken` whose `IsCancellationRequested` property is `true`. In this case, the continuation does not start, and it transitions to the `TaskStatus.Canceled` state.
- The continuation never runs because the condition set by its `TaskContinuationOptions` argument was not met. For example, if an antecedent goes into a `TaskStatus.Faulted` state, its continuation that was passed the `TaskContinuationOptions.NotOnFaulted` option will not run but will transition to the `Canceled` state.

If a task and its continuation represent two parts of the same logical operation, you can pass the same cancellation token to both tasks, as shown in the following example. It consists of an antecedent that generates a list of integers that are divisible by 33, which it passes to the continuation. The continuation in turn displays the list. Both the antecedent and the continuation pause regularly for random intervals. In addition, a `System.Threading.Timer` object is used to execute the `Elapsed` method after a five-second timeout interval. This example calls the `CancellationTokenSource.Cancel` method, which causes the currently executing task to call the `CancellationToken.ThrowIfCancellationRequested` method. Whether the `CancellationTokenSource.Cancel` method is called when the antecedent or its continuation is executing depends on the duration of the randomly generated pauses. If the antecedent is canceled, the continuation will not start. If the antecedent is not canceled, the token can still be used to cancel the continuation.


```csharp
using System;
using System.Collections.Generic;
using System.Threading;
using System.Threading.Tasks;

public class Example
{
   public static void Main()
   {
      Random rnd = new Random();
      var cts = new CancellationTokenSource();
      CancellationToken token = cts.Token;
      Timer timer = new Timer(Elapsed, cts, 5000, Timeout.Infinite);

      var t = Task.Run( () => { List<int> product33 = new List<int>();
                                for (int ctr = 1; ctr < Int16.MaxValue; ctr++) {
                                   if (token.IsCancellationRequested) {
                                      Console.WriteLine("\nCancellation requested in antecedent...\n");
                                      token.ThrowIfCancellationRequested();
                                   }
                                   if (ctr % 2000 == 0) {
                                      int delay = rnd.Next(16,501);
                                      Thread.Sleep(delay);
                                   }

                                   if (ctr % 33 == 0)
                                      product33.Add(ctr);
                                }
                                return product33.ToArray();
                              }, token);

      Task continuation = t.ContinueWith(antecedent => { Console.WriteLine("Multiples of 33:\n");
                                                         var arr = antecedent.Result;
                                                         for (int ctr = 0; ctr < arr.Length; ctr++)
                                                         {
                                                            if (token.IsCancellationRequested) {
                                                               Console.WriteLine("\nCancellation requested in continuation...\n");
                                                               token.ThrowIfCancellationRequested();
                                                            }

                                                            if (ctr % 100 == 0) {
                                                               int delay = rnd.Next(16,251);
                                                               Thread.Sleep(delay);
                                                            }
                                                            Console.Write("{0:N0}{1}", arr[ctr],
                                                                          ctr != arr.Length - 1 ? ", " : "");
                                                            if (Console.CursorLeft >= 74)
                                                               Console.WriteLine();
                                                         }
                                                         Console.WriteLine();
                                                       } , token);

      try {
          continuation.Wait();
      }
      catch (AggregateException e) {
         foreach (Exception ie in e.InnerExceptions)
            Console.WriteLine("{0}: {1}", ie.GetType().Name,
                              ie.Message);
      }
      finally {
         cts.Dispose();
      }

      Console.WriteLine("\nAntecedent Status: {0}", t.Status);
      Console.WriteLine("Continuation Status: {0}", continuation.Status);
  }

   private static void Elapsed(object state)
   {
      CancellationTokenSource cts = state as CancellationTokenSource;
      if (cts == null) return;

      cts.Cancel();
      Console.WriteLine("\nCancellation request issued...\n");
   }
}
// The example displays the following output:
//    Multiples of 33:
//
//    33, 66, 99, 132, 165, 198, 231, 264, 297, 330, 363, 396, 429, 462, 495, 528,
//    561, 594, 627, 660, 693, 726, 759, 792, 825, 858, 891, 924, 957, 990, 1,023,
//    1,056, 1,089, 1,122, 1,155, 1,188, 1,221, 1,254, 1,287, 1,320, 1,353, 1,386,
//    1,419, 1,452, 1,485, 1,518, 1,551, 1,584, 1,617, 1,650, 1,683, 1,716, 1,749,
//    1,782, 1,815, 1,848, 1,881, 1,914, 1,947, 1,980, 2,013, 2,046, 2,079, 2,112,
//    2,145, 2,178, 2,211, 2,244, 2,277, 2,310, 2,343, 2,376, 2,409, 2,442, 2,475,
//    2,508, 2,541, 2,574, 2,607, 2,640, 2,673, 2,706, 2,739, 2,772, 2,805, 2,838,
//    2,871, 2,904, 2,937, 2,970, 3,003, 3,036, 3,069, 3,102, 3,135, 3,168, 3,201,
//    3,234, 3,267, 3,300, 3,333, 3,366, 3,399, 3,432, 3,465, 3,498, 3,531, 3,564,
//    3,597, 3,630, 3,663, 3,696, 3,729, 3,762, 3,795, 3,828, 3,861, 3,894, 3,927,
//    3,960, 3,993, 4,026, 4,059, 4,092, 4,125, 4,158, 4,191, 4,224, 4,257, 4,290,
//    4,323, 4,356, 4,389, 4,422, 4,455, 4,488, 4,521, 4,554, 4,587, 4,620, 4,653,
//    4,686, 4,719, 4,752, 4,785, 4,818, 4,851, 4,884, 4,917, 4,950, 4,983, 5,016,
//    5,049, 5,082, 5,115, 5,148, 5,181, 5,214, 5,247, 5,280, 5,313, 5,346, 5,379,
//    5,412, 5,445, 5,478, 5,511, 5,544, 5,577, 5,610, 5,643, 5,676, 5,709, 5,742,
//    5,775, 5,808, 5,841, 5,874, 5,907, 5,940, 5,973, 6,006, 6,039, 6,072, 6,105,
//    6,138, 6,171, 6,204, 6,237, 6,270, 6,303, 6,336, 6,369, 6,402, 6,435, 6,468,
//    6,501, 6,534, 6,567, 6,600, 6,633, 6,666, 6,699, 6,732, 6,765, 6,798, 6,831,
//    6,864, 6,897, 6,930, 6,963, 6,996, 7,029, 7,062, 7,095, 7,128, 7,161, 7,194,
//    7,227, 7,260, 7,293, 7,326, 7,359, 7,392, 7,425, 7,458, 7,491, 7,524, 7,557,
//    7,590, 7,623, 7,656, 7,689, 7,722, 7,755, 7,788, 7,821, 7,854, 7,887, 7,920,
//    7,953, 7,986, 8,019, 8,052, 8,085, 8,118, 8,151, 8,184, 8,217, 8,250, 8,283,
//    8,316, 8,349, 8,382, 8,415, 8,448, 8,481, 8,514, 8,547, 8,580, 8,613, 8,646,
//    8,679, 8,712, 8,745, 8,778, 8,811, 8,844, 8,877, 8,910, 8,943, 8,976, 9,009,
//    9,042, 9,075, 9,108, 9,141, 9,174, 9,207, 9,240, 9,273, 9,306, 9,339, 9,372,
//    9,405, 9,438, 9,471, 9,504, 9,537, 9,570, 9,603, 9,636, 9,669, 9,702, 9,735,
//    9,768, 9,801, 9,834, 9,867, 9,900, 9,933, 9,966, 9,999, 10,032, 10,065, 10,098,
//    10,131, 10,164, 10,197, 10,230, 10,263, 10,296, 10,329, 10,362, 10,395, 10,428,
//    10,461, 10,494, 10,527, 10,560, 10,593, 10,626, 10,659, 10,692, 10,725, 10,758,
//    10,791, 10,824, 10,857, 10,890, 10,923, 10,956, 10,989, 11,022, 11,055, 11,088,
//    11,121, 11,154, 11,187, 11,220, 11,253, 11,286, 11,319, 11,352, 11,385, 11,418,
//    11,451, 11,484, 11,517, 11,550, 11,583, 11,616, 11,649, 11,682, 11,715, 11,748,
//    11,781, 11,814, 11,847, 11,880, 11,913, 11,946, 11,979, 12,012, 12,045, 12,078,
//    12,111, 12,144, 12,177, 12,210, 12,243, 12,276, 12,309, 12,342, 12,375, 12,408,
//    12,441, 12,474, 12,507, 12,540, 12,573, 12,606, 12,639, 12,672, 12,705, 12,738,
//    12,771, 12,804, 12,837, 12,870, 12,903, 12,936, 12,969, 13,002, 13,035, 13,068,
//    13,101, 13,134, 13,167, 13,200, 13,233, 13,266,
//    Cancellation requested in continuation...
//
//
//    Cancellation request issued...
//
//    TaskCanceledException: A task was canceled.
//
//    Antecedent Status: RanToCompletion
//    Continuation Status: Canceled
```

You can also prevent a continuation from executing if its antecedent is canceled without supplying the continuation a cancellation token by specifying the `TaskContinuationOptions.NotOnCanceled` option when you create the continuation. The following is a simple example.


```csharp
using System;
using System.Threading;
using System.Threading.Tasks;

public class Example
{
   public static void Main()
   {
      var cts = new CancellationTokenSource();
      CancellationToken token = cts.Token;
      cts.Cancel();

      var t = Task.FromCanceled(token);
      var continuation = t.ContinueWith( (antecedent) => {
                                            Console.WriteLine("The continuation is running.");
                                          } , TaskContinuationOptions.NotOnCanceled);
      try {
         t.Wait();
      }
      catch (AggregateException ae) {
         foreach (var ie in ae.InnerExceptions)
            Console.WriteLine("{0}: {1}", ie.GetType().Name, ie.Message);

         Console.WriteLine();
      }
      finally {
         cts.Dispose();
      }

      Console.WriteLine("Task {0}: {1:G}", t.Id, t.Status);
      Console.WriteLine("Task {0}: {1:G}", continuation.Id,
                        continuation.Status);
   }
}
// The example displays the following output:
//       TaskCanceledException: A task was canceled.
//
//       Task 1: Canceled
//       Task 2: Canceled
```

After a continuation goes into the `Canceled` state, it may affect continuations that follow, depending on the `TaskContinuationOptions` that were specified for those continuations.

Continuations that are disposed will not start.

## Continuations and Child Tasks

A continuation does not run until the antecedent and all of its attached child tasks have completed. The continuation does not wait for detached child tasks to finish. The following two examples illustrate child tasks that are attached to and detached from an antecedent that creates a continuation. In the following example, the continuation runs only after all child tasks have completed, and running the example multiple times produces identical output each time. The example launches the antecedent by calling the `TaskFactory.StartNew` method, since by default the `Task.Run` method creates a parent task whose default task creation option is `TaskCreationOptions.DenyChildAttach`.


```csharp
using System;
using System.Threading;
using System.Threading.Tasks;

public class Example
{
   public static void Main()
   {
      var t = Task.Factory.StartNew( () => { Console.WriteLine("Running antecedent task {0}...",
                                                  Task.CurrentId);
                                             Console.WriteLine("Launching attached child tasks...");
                                             for (int ctr = 1; ctr <= 5; ctr++)  {
                                                int index = ctr;
                                                Task.Factory.StartNew( (value) => {
                                                                       Console.WriteLine("   Attached child task #{0} running",
                                                                                         value);
                                                                       Thread.Sleep(1000);
                                                                     }, index, TaskCreationOptions.AttachedToParent);
                                             }
                                             Console.WriteLine("Finished launching attached child tasks...");
                                           });
      var continuation = t.ContinueWith( (antecedent) => { Console.WriteLine("Executing continuation of Task {0}",
                                                                             antecedent.Id);
                                                         });
      continuation.Wait();
   }
}
// The example displays the following output:
//       Running antecedent task 1...
//       Launching attached child tasks...
//       Finished launching attached child tasks...
//          Attached child task #5 running
//          Attached child task #1 running
//          Attached child task #2 running
//          Attached child task #3 running
//          Attached child task #4 running
//       Executing continuation of Task 1
```

If child tasks are detached from the antecedent, however, the continuation runs as soon as the antecedent has terminated, regardless of the state of the child tasks. As a result, multiple runs of the following example can produce variable output that depends on how the task scheduler handled each child task.


```csharp
using System;
using System.Threading;
using System.Threading.Tasks;

public class Example
{
   public static void Main()
   {
      var t = Task.Factory.StartNew( () => { Console.WriteLine("Running antecedent task {0}...",
                                                  Task.CurrentId);
                                             Console.WriteLine("Launching attached child tasks...");
                                             for (int ctr = 1; ctr <= 5; ctr++)  {
                                                int index = ctr;
                                                Task.Factory.StartNew( (value) => {
                                                                       Console.WriteLine("   Attached child task #{0} running",
                                                                                         value);
                                                                       Thread.Sleep(1000);
                                                                     }, index);
                                             }
                                             Console.WriteLine("Finished launching detached child tasks...");
                                           }, TaskCreationOptions.DenyChildAttach);
      var continuation = t.ContinueWith( (antecedent) => { Console.WriteLine("Executing continuation of Task {0}",
                                                                             antecedent.Id);
                                                         });
      continuation.Wait();
   }
}
// The example displays output like the following:
//       Running antecedent task 1...
//       Launching attached child tasks...
//       Finished launching detached child tasks...
//          Attached child task #1 running
//          Attached child task #2 running
//          Attached child task #5 running
//          Attached child task #3 running
//       Executing continuation of Task 1
//          Attached child task #4 running
```

The final status of the antecedent task depends on the final status of any attached child tasks. The status of detached child tasks does not affect the parent. For more information, see `Attached and Detached Child Tasks`.

## Associating State with Continuations

You can associate arbitrary state with a task continuation. The `ContinueWith` method provides overloaded versions that each take an `Object` value that represents the state of the continuation. You can later access this state object by using the `Task.AsyncState` property. This state object is `null` if you do not provide a value.

Continuation state is useful when you convert existing code that uses the `Asynchronous Programming Model (APM)` to use the TPL. In the APM, you typically provide object state in the **Begin***Method* method and later access that state by using the `IAsyncResult.AsyncState` property. By using the `ContinueWith` method, you can preserve this state when you convert code that uses the APM to use the TPL.

Continuation state can also be useful when you work with `Task` objects in the Visual Studio debugger. For example, in the **Parallel Tasks** window, the **Task** column displays the string representation of the state object for each task. For more information about the **Parallel Tasks** window, see `Using the Tasks Window`.

The following example shows how to use continuation state. It creates a chain of continuation tasks. Each task provides the current time, a `DateTime` object, for the `state` parameter of the `ContinueWith` method. Each `DateTime` object represents the time at which the continuation task is created. Each task produces as its result a second `DateTime` object that represents the time at which the task finishes. After all tasks finish, this example displays the creation time and the time at which each continuation task finishes.


```csharp
using System;
using System.Collections.Generic;
using System.Threading;
using System.Threading.Tasks;

// Demonstrates how to associate state with task continuations.
class ContinuationState
{
   // Simluates a lengthy operation and returns the time at which
   // the operation completed.
   public static DateTime DoWork()
   {
      // Simulate work by suspending the current thread 
      // for two seconds.
      Thread.Sleep(2000);

      // Return the current time.
      return DateTime.Now;
   }

   static void Main(string[] args)
   {
      // Start a root task that performs work.
      Task<DateTime> t = Task<DateTime>.Run(delegate { return DoWork(); });

      // Create a chain of continuation tasks, where each task is 
      // followed by another task that performs work.
      List<Task<DateTime>> continuations = new List<Task<DateTime>>();
      for (int i = 0; i < 5; i++)
      {
         // Provide the current time as the state of the continuation.
         t = t.ContinueWith(delegate { return DoWork(); }, DateTime.Now);
         continuations.Add(t);
      }

      // Wait for the last task in the chain to complete.
      t.Wait();

      // Print the creation time of each continuation (the state object)
      // and the completion time (the result of that task) to the console.
      foreach (var continuation in continuations)
      {
         DateTime start = (DateTime)continuation.AsyncState;
         DateTime end = continuation.Result;

         Console.WriteLine("Task was created at {0} and finished at {1}.",
            start.TimeOfDay, end.TimeOfDay);
      }
   }
}

/* Sample output:
Task was created at 10:56:21.1561762 and finished at 10:56:25.1672062.
Task was created at 10:56:21.1610677 and finished at 10:56:27.1707646.
Task was created at 10:56:21.1610677 and finished at 10:56:29.1743230.
Task was created at 10:56:21.1610677 and finished at 10:56:31.1779883.
Task was created at 10:56:21.1610677 and finished at 10:56:33.1837083.
*/
```

## Handling Exceptions Thrown from Continuations

An antecedent-continuation relationship is not a parent-child relationship. Exceptions thrown by continuations are not propagated to the antecedent. Therefore, handle exceptions thrown by continuations as you would handle them in any other task, as follows:

- You can use the `Wait`, `WaitAll`, or `WaitAny` method, or its generic counterpart, to wait on the continuation. You can wait for an antecedent and its continuations in the same `try` statement, as shown in the following example.

```csharp
  using System;
  using System.Threading.Tasks;
  
  public class Example
  {
     public static void Main()
     {
        var task1 = Task<int>.Run( () => { Console.WriteLine("Executing task {0}",
                                                             Task.CurrentId);
                                           return 54; });
        var continuation = task1.ContinueWith( (antecedent) =>
                                               { Console.WriteLine("Executing continuation task {0}",
                                                                   Task.CurrentId);
                                                 Console.WriteLine("Value from antecedent: {0}",
                                                                   antecedent.Result);
                                                 throw new InvalidOperationException();
                                              } );
  
        try {
           task1.Wait();
           continuation.Wait();
        }
        catch (AggregateException ae) {
            foreach (var ex in ae.InnerExceptions)
                Console.WriteLine(ex.Message);
        }
     }
  }
  // The example displays the following output:
  //       Executing task 1
  //       Executing continuation task 2
  //       Value from antecedent: 54
  //       Operation is not valid due to the current state of the object.
```

- You can use a second continuation to observe the `Exception` property of the first continuation. In the following example, a task attempts to read from a non-existent file. The continuation then displays information about the exception in the antecedent task.

```csharp
  using System;
  using System.IO;
  using System.Threading.Tasks;
  
  public class Example
  {
     public static void Main()
     {
        var t = Task.Run( () => { string s = File.ReadAllText(@"C:\NonexistentFile.txt");
                                  return s;
                                });
  
        var c = t.ContinueWith( (antecedent) =>
                                { // Get the antecedent's exception information.
                                  foreach (var ex in antecedent.Exception.InnerExceptions) {
                                     if (ex is FileNotFoundException)
                                        Console.WriteLine(ex.Message);
                                  }
                                }, TaskContinuationOptions.OnlyOnFaulted);
  
        c.Wait();
     }
  }
  // The example displays the following output:
  //        Could not find file 'C:\NonexistentFile.txt'.
```

  Because it was run with the `TaskContinuationOptions.OnlyOnFaulted` option, the continuation executes only if an exception occurs in the antecedent, and therefore it can assume that the antecedent's `Exception` property is not `null`. If the continuation executes whether or not an exception is thrown in the antecedent, it would have to check whether the antecedent's `Exception` property is not `null` before attempting to handle the exception, as the following code fragment shows.

```csharp
  // Determine whether an exception occurred.
  if (antecedent.Exception != null) {
     foreach (var ex in antecedent.Exception.InnerExceptions) {
        if (ex is FileNotFoundException)
           Console.WriteLine(ex.Message);
     }
  }
```

  For more information, see `Exception Handling`.

- If the continuation is an attached child task that was created by using the `TaskContinuationOptions.AttachedToParent` option, its exceptions will be propagated by the parent back to the calling thread, as is the case in any other attached child. For more information, see `Attached and Detached Child Tasks`.