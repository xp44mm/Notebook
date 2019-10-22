# appendix A Writing asynchronous code in .NET

For modern applications to be responsive, writing asynchronous code is crucial, and it's a key trait for being reactive. This appendix summarizes what asynchronous code is, what it's good for, how you can write asynchronous code in .NET, and the best practices for doing so.

## A.1 Writing asynchronous code

Imagine you want to ask your friend to send you important information from a document (such as the content of the ReactiveX.io portal). You have two options: you can use the phone to ask your friend to read the information to you, or you can send an email with your request so you both can work on getting the information later. Figure A.1 shows the two options.

Figure A.1 Two approaches to get the content of a document from a friend. The left sequence shows the synchronous way via phone call. The right sequence shows the asynchronous way via email. When you make a phone call to retrieve information (the left sequence in the figure), you're using the synchronous approach. With this method, you have to wait until the call is answered and then you need to wait for the other side to complete the requested task (such as retrieving the document).

When you choose to send an email (the right sequence in the figure), you're using the asynchronous approach. The benefit of the asynchronous operation is obvious: while your message is being sent to the other participant and while it's being handled, you can continue doing other things.

Running code in an asynchronous way is crucial for the modern application for two main reasons:

  - Responsiveness—Imagine your client application copies a big file. It'd be awful if the UI is blocked the entire time. The user might think the application is stuck. It'd be much better if the long copy operation were asynchronous, with the UI showing progress until the operation finishes.

  - Scalability—Nowadays almost every computer has more than one core, so running tasks or jobs in real-time parallelism lets your application handle multiple tasks at the same time. Suppose your application needs to handle multiple user requests. Each request that arrives can run asynchronously, and your application can scale accordingly.

## A.2 Asynchronous code in .NET

Writing asynchronous code isn't hard; all it takes is to delegate the work to another thread, process, or machine, and then not to wait for it to complete. Sounds simple, doesn't it? Unfortunately, writing asynchronous code tends to be much more difficult than that.

For starters, you need to decide how to create a thread, a process, or a task. Or you need to decide how to communicate with another machine to run your code. Then, after you decide how to run your asynchronous code, you need to determine whether it has finished successfully or failed. If it finishes, you'll want to capture the result (or the error).

Not every asynchronous operation (such as reading a file from the hard drive or running a query against a database) is CPU bound, which is great because the CPU is free to process other threads.

.NET has always had ways to run code in an asynchronous fashion, and that has evolved throughout the years.

Here's a simple application that runs a lengthy task in an asynchronous way:

```C#
class Program
{
    static void Main(string[] args)
    {
        var thread = new Thread(() =>
        {
            //Performing a very long task

            Console.WriteLine("Long work is done, the result is ...");
        });
        thread.Start();
        Console.ReadLine();
    }
}
```

In this simple program, you create a thread that runs the long-running job. After starting the thread, the main thread is free to proceed with another task. In this case, you wait for user input. When the long-running job is done, the result is written to the console. Figure A.2 shows how the two threads work concurrently.

Figure A.2 Creating a background thread. After the thread is created, the main thread continues its execution concurrently to the background thread.

This sample works, but it's far from ideal. Creating a thread is a relatively expensive operation because the OS needs to allocate the thread. For this example, together with all data structures involved, it's approximately 1 MB of memory. Not only is the creation of the thread expensive, when the thread is destroyed and given back to the OS, your application suffers again. For every thread you create, you lose precious resources (time and memory) that the application could've used for additional work.

Inside .NET, you have ways to improve the sample code. The `System.Threading.ThreadPool` class, for example, handles the creation and destruction of threads, and it does so in a way that reuses threads and adapts to the workload of your application. Thus, the long-running job could've been written like this:

```C#
ThreadPool.QueueUserWorkItem((_) =>
{
    //Performing a very long task
    Console.WriteLine("Long work is done, the result is ...");
});
```

In this code, you assign the work that needs to be done asynchronously to the `ThreadPool` that adds it to an internal queue. Then, a worker thread that's managed by the `ThreadPool` will pick the work item and execute it.

But creating threads to run every code you want to run asynchronously can work against you. For example, in the previous code, you call to the Console.WriteLine method to keep the output from displaying characters that arrive from simultaneous calls. The Console class holds a lock to prevent multiple threads from executing at the same time. If you have many threads to write to the console, they'll be blocked until the current write is finished. If you take into consideration the overhead of managing the threads and the code that runs them, you might find that the performance of your application decreases, which is counterproductive to what you want to achieve.

Beside the performance issues, there's another downside to using the thread approach: There's no standard and easy way to know whether the thread completes and no way to receive the response. This makes this approach unfriendly in many situations. In many cases, you'll want to delegate calculations to other threads, and when the calculation is done, you'll want to take the results and combine them. Sometimes this is also done in another thread (or threads).

.NET provides a few patterns to achieve this behavior. The following list describes two that are now considered obsolete, but there's still a chance you'll run into them:

  - The Asynchronous Programming Model (APM) pattern: In this pattern, two methods (`Begin`[OperationName] and `End`[OperationName]) begin and end the operation. After calling Begin, an object of a type that implements IAsyncResult is returned immediately. The calling thread isn't blocked and can continue processing the next line. The application can be notified that the operation completed, either by checking the IsCompleted property of the IAsyncResult object or by a callback that's supplied to the Begin[Operation Name] method. When the operation completes, the application calls the End[OperationName] method and provides the IAsyncResult as an argument; that is, End[OperationName] returns the operation's result.

  - The Event-Based Asynchronous Pattern (EAP): In this pattern, you call the method that's making the time-consuming work, [MethodName]Async. The containing class will have a corresponding event called [MethodName]Completed, which will be raised when the operation completes.

Beginning with .NET Framework 4, the recommended pattern for creating asynchronous code is the Task-Based Asynchronous Pattern (TAP), which is based on the Task Parallel Library (TPL). And, because all the other patterns can be converted to the TPL, I use it in the rest of the appendix.

TIP If you do bump into the older asynchronous patterns, the easiest (also recommended) way to work with them is to create a task that abstracts them. For the APM pattern, you can use the `Task.FromAsync` static method, but for EAP you need to work a bit and use the type `TaskCompletionSource<TResult>`. You can find more information in an MSDN article at http://mng.bz/dJ6K.

## A.3 Task-Based Asynchronous Pattern

TAP is based on two important types that exist in the .NET Framework: `System.Threading.Tasks.Task` and `System.Threading.Tasks.Task<TResult>`.

Tasks are the .NET implementation of futures. A future is a stand-in for a computational result that's initially unknown but becomes available at a later time (hence the name future), as shown in figure A.3.

Figure A.3 The task is a .NET implementation of a future: a stand-in for a computational result that's initially unknown but becomes available at a later time.

With TAP, methods that perform an asynchronous computation return tasks, and the task is the contact point from which you can know the state of the computation and the final result.

The process of calculating the result can occur in parallel with other computations. `Task` represents a computation that yields no value, while `Task<TResult>` yields a value of type TResult. After the computation completes, you can get the result from the task's property `Result`. Here's a small example that asynchronously gets the headers of the ReactiveX portal home page. For brevity, error handling wasn't added to this example:

```C#
var httpClient = new HttpClient();
var requestTask = //Task<HttpResponseMessage>
    httpClient.GetAsync("http://ReactiveX.io");
Console.WriteLine("the request was sent, status:{0}",requestTask.Status);
Console.WriteLine(requestTask.Result.Headers);
```

When I ran the sample on my machine, this is what I got:

```C#
the request was sent, status:WaitingForActivation
Access-Control-Allow-Origin: *
Accept-Ranges: bytes
Cache-Control: max-age=600
Date: Mon, 27 Jul 2015 19:17:26 GMT
Server: GitHub.com
```

A little explanation is in order. When a task is created and started, it's added to a queue that's managed by the `TaskScheduler`. At this point, the task state is `WaitingForActivation`, which is why the first line that prints shows this status. The `TaskScheduler` assigns the task into a thread, and this changes the task status to Running. When you try to get the Result from the task while it's still running, the calling thread is blocked until the task finishes, which causes its state to change to `RanToCompletion`. Upon completion, the calling thread resumes, and the task result is returned. This is when the headers are printed to the console.

Waiting for the task to finish by calling Result (or Wait) is a little counterproductive for what you want to achieve—getting the headers asynchronously without blocking the calling threads. You can achieve this thanks to the continuation functionality, which tasks support. Here's what the previous example looks like when using the continuation functionality:

```C#
var httpClient = new HttpClient();
httpClient.GetAsync("http://ReactiveX.io")
    .ContinueWith(requestTask =>
    {
        Console.WriteLine("the request was sent, status:{0}",requestTask.Status);
        Console.WriteLine(requestTask.Result.Headers);
    });
```

Running this version of the code produces this output:

```C#
the request was sent, status:RanToCompletion
Access-Control-Allow-Origin: *
Accept-Ranges: bytes
Cache-Control: max-age=600
Date: Tue, 28 Jul 2015 20:08:55 GMT
Server: GitHub.com
```

You use the `ContinueWith` method to attach code that'll be executed when the task completes (successfully or not), and the task you're attaching to is sent as an argument (the requestTask in the lambda expression). This is why the output you received shows that the status of the task is `RanToCompletion`. Now, in the time that it takes to get the headers and process them (writing to the console), your code works in an asynchronous way. The main thread isn't blocked, and everything runs in the background.

Continuation makes the creation of asynchronous code nice, but it can be lengthy. To demonstrate, let's see what happens when you want to print the content of the page as well as the headers:

```C#
var httpClient = new HttpClient();
httpClient.GetAsync("http://ReactiveX.io")
    .ContinueWith(requestTask =>
    {
        var httpContent = requestTask.Result.Content;
        httpContent.ReadAsStringAsync()
            .ContinueWith(contentTask =>
            {
                Console.WriteLine(contentTask.Result);
            });
    });
```

As you can see, the more asynchronous methods you use on the way to your target, the more continuations you'll need and the less readable your code becomes. To help with that, C# provides language-based support to hide the complexity of continuations by awaiting the tasks while maintaining regular control flow. This is also known as the async-await pattern.



## A.4 Simplifying asynchronous code with async-await

Instead of repeating the pattern of continuing a task and getting the result when it finishes and then making another continuation on another task, the async-await pattern lets you write your asynchronous code as if it were simple and sequential. When calling the asynchronous method (which returns a task), you can instruct the compiler that you want to await it, meaning you want the rest of the code to execute when the async-await pattern finishes and its result is returned.

Let's look at an example of getting the reactivex.io page with async-await:

```C#
async void GetPageAsync()
{
    var httpClient = new HttpClient();
    var response = await httpClient.GetAsync("http://ReactiveX.io");
    var page = await response.Content.ReadAsStringAsync();
    Console.WriteLine(page);
}

```

To use async-await, the method that contains `await` must be marked with the `async` modifier. Inside the method, you can now call the asynchronous methods that return either `Task` or `Task<TResult>` and await them (any type can be awaited as long as it provides an awaiter). When the task is complete, the rest of the code is executed and, if you awaited `Task<TResult>`, the TResult value is returned without the need to explicitly use the Result property on the task. If the task throws an exception, it will be caught in the calling code as if it were a synchronous call. 

TIP Any type can be awaited as long as it provides an awaiter by exposing a `GetAwaiter` method. An awaiter is any class that conforms to a particular pattern. You can find more details in the Async/Await FAQ: http://mng.bz/27mZ.

注意，任务的结果`Task.Result`应该改用如下方法获得，这更安全，阅读起来更合逻辑：

```C#
result = requestTask.GetAwaiter().GetResult();
```

Suppose the method you wrote needs to return the downloaded string content. async-await is viral! Because you call async methods inside your code, this makes your code asynchronous by itself, which means that the value you want to return will be available when the async calls complete. To expose that information to the caller, and to allow it to await on the completeness of your code, you must also return a task (in this case, `Task<string>`). Methods marked with async can only return `Task`, `Task<TResult>`, or `void`. This is how the method signature looks now:

```C#
async Task<string> GetPageAsync()
```


When you need to return a value inside an async method that returns a task, you don't need to return the task explicitly. You can regularly return the value as if it were a simple synchronous method. The compiler makes the transformations behind the scenes:

```C#
private static async Task<string> GetPageAsync()
{
    var httpClient = new HttpClient();
    var response = await httpClient.GetAsync("http://ReactiveX.io");
    var page = await response.Content.ReadAsStringAsync();
    return page;
}
```


The GetPageAsync method calls to other async methods and then returns the end result that's of type string. The method is asynchronous because it uses other asynchronous methods, but you have no real idea of what's going on inside and whether it's truly asynchronous. Because we haven't discussed how to write methods that run their code inside an asynchronous task, we'll look at that next.

## A.5 Creating tasks

In the previous examples, I haven't talked about how to create tasks in the code that you write. All you saw is that when you add the async modifier to your methods, the compiler generates a returned task for you. This can be misleading, as the next example shows.

Look at the following code and predict what will be printed:

```C#
async void AsyncMethodCaller()
{
    bool isSame = await MyAsyncMethod(Thread.CurrentThread.ManagedThreadId);
    Console.WriteLine(isSame);
}
async Task<bool> MyAsyncMethod(int callingThreadId)
{
    return Thread.CurrentThread.ManagedThreadId == callingThreadId;
}

```

The method AsyncMethodCaller calls MyAsyncMethod and passes the thread ID. Because MyAsyncMethod returns a task, the call can be awaited. The MyAsyncMethod checks whether the ID of the thread it's running on is the same as the thread ID it received as a parameter and returns the result, which is then printed by the caller method.

When you run this program, you'll see that the printed value will be true, which might surprise you. You see, marking a method as async and returning a task doesn't by itself make the code inside the method perform asynchronously. It's a way to instruct the compiler that the code inside might perform as an asynchronous operation and that you request its help to make a continuation when that happens.

In the previous method, you haven't done anything asynchronous, so that's why the calling thread is the same thread as the one running the method's code. The only thing you did is hurt the performance of your simple code because, behind the scenes, the compiler created a state machine that has only one state, and the overhead of managing this state machine has a performance hit.

To make your method asynchronous, you need to span the work to another task, and this can be done by using `Task.Run`. Here's a real asynchronous version of MyAsyncMethod:

```C#
async void AsyncMethodCaller()
{
    Console.WriteLine();
    Console.WriteLine("----- Using Task.Run(...) to create async code ----");
    bool isSame = await MyAsyncMethod(Thread.CurrentThread.ManagedThreadId);
    Console.WriteLine("Caller thread is the same as executing thread: {0}",isSame);
}
async Task<bool> MyAsyncMethod(int callingThreadId)
{
    return await Task.Run(() => Thread.CurrentThread.ManagedThreadId == callingThreadId);
}
```

In this version, you create a new task and start it by passing the lambda expression to the `Task.Run` method. This causes the `TaskScheduler` to assign a thread that will run your code. The printed value you see now is false.

In this appendix, I've tried to show a few techniques that you can use to write asynchronous code. I haven't touched upon many other techniques that we could talk about for hours. But this book is about Rx and not about how to write multithreaded applications; many books have been written to explain that. Because this topic is interesting to me and, I'm sure, to you too, you can learn more about what .NET provides when dealing with multithreading by reading Jeffrey Richter's CLR via C# (Microsoft Press, 2012). I also recommend Concurrency in C# Cookbook 2nd by Stephen Cleary (O'Reilly, 2019).

## A.6 Summary

This appendix provided a short recap about writing asynchronous code in .NET. Asynchronicity plays a major role in the Rx world, and the material in this book relies on the concepts explained here.

Here's a summary of what you learned:

  - Asynchronous code execution is crucial for the modern application to be both scalable and responsive.

  - .NET provides a few ways to achieve asynchronicity, which rely on thread creation and I/O operations.

  - The recommended approach is to use the Task Parallel Library (TPL), which allows spanning new tasks, and to create continuations on tasks (completed or failed).

  - The async-await pattern lets you write asynchronous code in a way that makes it look sequential and natural.

