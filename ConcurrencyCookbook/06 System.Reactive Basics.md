# Chapter 6. System.Reactive Basics

LINQ is a set of language features that enable developers to query sequences. The two most common LINQ providers are the built-in LINQ to Objects (which is based on `IEnumerable<T>`) and LINQ to Entities (based on `IQueryable<T>`). There are many other providers available, and most providers have the same general structure. Queries are lazily evaluated, and the sequences produce values as necessary. Conceptually, this is a pull model; during evaluation, value items are pulled from the query one at a time.

`System.Reactive` (Rx) treats events as sequences of data that arrive over time. As such, you can think of Rx as LINQ to Events (based on `IObservable<T>`). The main difference between observables and other LINQ providers is that Rx is a “push” model, meaning that the query defines how the program reacts as events arrive. Rx builds on top of LINQ, adding some powerful new operators as extension methods.

This chapter looks at some of the more common Rx operations. Bear in mind that all of the LINQ operators are also available, so simple operations, such as filtering (Where) and projection (Select), work conceptually the same as they do with any other LINQ provider. We won't cover these common LINQ operations here; we'll focus on the new capabilities that Rx builds on top of LINQ, particularly those dealing with time.

To use `System.Reactive`, install the NuGet package for `System.Reactive` into your application.

## 6.1 Converting .NET Events

### Problem

You have an event that you need to treat as a `System.Reactive` input stream, producing some data via `OnNext` each time the event is raised.

### Solution

The `Observable` class defines several event converters. Most .NET framework events are compatible with `FromEventPattern`, but if you have events that don't follow the common pattern, you can use `FromEvent` instead.

`FromEventPattern` works best if the event delegate type is `EventHandler<T>`.

Many newer framework types use this event delegate type. For example, the `Progress<T>` type defines a `ProgressChanged` event, which is of type `EventHandler<T>`, so it can be easily wrapped with `FromEventPattern`:

```C#
var progress = new Progress<int>();
IObservable<EventPattern<int>> progressReports =
    Observable.FromEventPattern<int>(
        handler => progress.ProgressChanged += handler,
        handler => progress.ProgressChanged -= handler);
progressReports.Subscribe(data => Trace.WriteLine("OnNext: " + data.EventArgs));
```

Note here that the `data.EventArgs` is strongly typed to be an int. The type argument to `FromEventPattern` (int in the previous example) is the same as the type T in `EventHandler<T>`. The two lambda arguments to `FromEventPattern` enable `System.Reactive` to subscribe and unsubscribe from the event.

The newer user interface frameworks use `EventHandler<T>`, and can easily be used with `FromEventPattern`, but older types often define a unique delegate type for each event. These can also be used with `FromEventPattern`, but it takes a bit more work. For example, the `System.Timers.Timer` type defines an `Elapsed` event, which is of type `ElapsedEventHandler`. You can wrap older events like this with `FromEventPattern`:

```C#
var timer = new System.Timers.Timer(interval: 1000) { Enabled = true };
IObservable<EventPattern<ElapsedEventArgs>> ticks =
    Observable.FromEventPattern<ElapsedEventHandler, ElapsedEventArgs>(
        handler => (s, a) => handler(s, a),
        handler => timer.Elapsed += handler,
        handler => timer.Elapsed -= handler);
ticks.Subscribe(data => Trace.WriteLine("OnNext: " + data.EventArgs.SignalTime));
```

Note that in this example that `data.EventArgs` is still strongly typed. The type arguments to `FromEventPattern` are now the unique handler type and the derived `EventArgs` type. The first lambda argument to `FromEventPattern` is a converter from `EventHandler<ElapsedEventArgs>` to `ElapsedEventHandler`; the converter should do nothing more than pass along the event.

That syntax is definitely getting awkward. Here's another option, which uses reflection:

```C#
var timer = new System.Timers.Timer(interval: 1000) { Enabled = true };
IObservable<EventPattern<object>> ticks =
    Observable.FromEventPattern(timer, nameof(Timer.Elapsed));
ticks.Subscribe(data => Trace.WriteLine("OnNext: " + ((ElapsedEventArgs)data.EventArgs).SignalTime));
```

With this approach, the call to `FromEventPattern` is much easier. Note that there's one drawback to this approach: the consumer doesn't get strongly typed data. Because `data.EventArgs` is of type object, you have to cast it to `ElapsedEventArgs` yourself.

### Discussion

Events are a common source of data for `System.Reactive` streams. This recipe covers wrapping any events that conform to the standard event pattern (where the first argument is the sender and the second argument is the event arguments type). If you have unusual event types, you can still use the `Observable.FromEvent` method overloads to wrap them into an observable.

When events are wrapped into an observable, `OnNext` is called each time the event is raised. When you're dealing with `AsyncCompletedEventArgs`, this can cause surprising behavior, because any exception is passed along as data (`OnNext`), not as an error (`OnError`). Consider this wrapper for `WebClient.DownloadStringCompleted`, for example:

```C#
var client = new WebClient();
IObservable<EventPattern<object>> downloadedStrings =
    Observable.
    FromEventPattern(client, nameof(WebClient.DownloadStringCompleted));
downloadedStrings.Subscribe(
    data =>
    {
      var eventArgs = (DownloadStringCompletedEventArgs)data.EventArgs;
      if (eventArgs.Error != null)
        Trace.WriteLine("OnNext: (Error) " + eventArgs.Error);
      else
        Trace.WriteLine("OnNext: " + eventArgs.Result);
    },
    ex => Trace.WriteLine("OnError: " + ex.ToString()),
    () => Trace.WriteLine("OnCompleted"));
client.DownloadStringAsync(new Uri("http://invalid.example.com/"));
```

When `WebClient.DownloadStringAsync` completes with an error, the event is raised with an exception in `AsyncCompletedEventArgs.Error`. Unfortunately, `System.Reactive` sees this as a data event, so if you then run the preceding code you will see `OnNext: (Error)` printed instead of `OnError:`.

Some event subscriptions and unsubscriptions must be done from a particular context. For example, events on many UI controls must be subscribed to from the UI thread. `System.Reactive` provides an operator that will control the context for subscribing and unsubscribing: `SubscribeOn`. The `SubscribeOn` operator isn't necessary in most situations because most of the time a UI-based subscription is done from the UI thread.

##### TIP

`SubscribeOn` controls the context for the code that adds and removes the event handlers. Don't confuse this with `ObserveOn`, which controls the context for the observable notifications (the delegates passed to `Subscribe`).

### See Also

Recipe 6.2 covers how to change the context in which events are raised.

Recipe 6.4 covers how to throttle events so subscribers aren't overwhelmed.

## 6.2 Sending Notifications to a Context

### Problem

`System.Reactive` does its best to be thread agnostic. So, it'll raise notifications (e.g., `OnNext`) in whatever thread happens to be current. Each `OnNext` notification will happen sequentially, but not necessarily on the same thread.

You often want these notifications raised in a particular context. For example, UI elements should only be manipulated from the UI thread that owns them, so if you're updating a UI in response to a notification that is arriving on a threadpool thread, then you'll need to move over to the UI thread.

### Solution

`System.Reactive` provides the `ObserveOn` operator to move notifications to another scheduler.

Consider the following example, which uses the `Interval` operator to create `OnNext` notifications once a second:

```C#
private void Button_Click(object sender, RoutedEventArgs e)
{
  Trace.WriteLine($"UI thread is {Environment.CurrentManagedThreadId}");
  Observable.Interval(TimeSpan.FromSeconds(1))
      .Subscribe(x => Trace.WriteLine(
          $"Interval {x} on thread {Environment.CurrentManagedThreadId}"));
}
```

On my machine, the output looks like the following:

```C#
UI thread is 9
Interval 0 on thread 10
Interval 1 on thread 10
Interval 2 on thread 11
Interval 3 on thread 11
Interval 4 on thread 10
Interval 5 on thread 11
Interval 6 on thread 11
```

Since `Interval` is based on a timer (without a specific thread), the notifications are raised on a threadpool thread, rather than the UI thread. If you need to update a UI element, you can pipe those notifications through `ObserveOn` and pass a synchronization context representing the UI thread:

```C#
private void Button_Click(object sender, RoutedEventArgs e)
{
  SynchronizationContext uiContext = SynchronizationContext.Current;
  Trace.WriteLine($"UI thread is {Environment.CurrentManagedThreadId}");
  Observable.Interval(TimeSpan.FromSeconds(1))
      .ObserveOn(uiContext)
      .Subscribe(x => Trace.WriteLine(
          $"Interval {x} on thread{Environment.CurrentManagedThreadId}"));
}
```

Another common usage of `ObserveOn` is to move off the UI thread when necessary. Consider a situation where you need to do some CPU-intensive computation whenever the mouse moves. By default, all mouse moves are raised on the UI thread, so you can use `ObserveOn` to move those notifications to a threadpool thread, do the computation, and then move the result notifications back to the UI thread:

```C#
SynchronizationContext uiContext = SynchronizationContext.Current;
Trace.WriteLine($"UI thread is {Environment.CurrentManagedThreadId}");
Observable.FromEventPattern<MouseEventHandler, MouseEventArgs>(
        handler => (s, a) => handler(s, a),
        handler => MouseMove += handler,
        handler => MouseMove -= handler)
    .Select(evt => evt.EventArgs.GetPosition(this))
    .ObserveOn(Scheduler.Default)
    .Select(position =>
    {
      // Complex calculation
      Thread.Sleep(100);
      var result = position.X + position.Y;
      var thread = Environment.CurrentManagedThreadId;
      Trace.WriteLine($"Calculated result {result} on thread {thread}");
      return result;
    })
    .ObserveOn(uiContext)
    .Subscribe(x => Trace.WriteLine(
        $"Result {x} on thread {Environment.CurrentManagedThreadId}"));
```

If you execute this sample, you'll see the calculations done on a threadpool thread and the results printed on the UI thread. However, you'll also notice that the calculations and results will lag behind the input; they'll queue up because the mouse location updates more often than every 100 ms. `System.Reactive` has several techniques for handling this situation; one common one covered in Recipe 6.4 is throttling the input.

### Discussion

`ObserveOn` actually moves notifications to a `System.Reactive` scheduler. This recipe covered the default (thread pool) scheduler and one way of creating a UI scheduler. The most common uses for the `ObserveOn` operator are moving on or off the UI thread, but schedulers are also useful in other scenarios. A more advanced scenario where schedulers are useful is faking the passage of time when unit testing, which you'll find covered in Recipe 7.6.

##### TIP

`ObserveOn` controls the context for the observable notifications. This is not to be confused with `SubscribeOn`, which controls the context for the code that adds and removes the event handlers.

### See Also

Recipe 6.1 covers how to create sequences from events, and using `SubscribeOn`.

Recipe 6.4 covers throttling event streams.

Recipe 7.6 covers the special scheduler used for testing your `System.Reactive` code.

## 6.3 Grouping Event Data with Windows and Buffers

### Problem

You have a sequence of events, and you want to group the incoming events as they arrive. As an example, you need to react to pairs of inputs. As another example, you need to react to all inputs within a two-second window.

### Solution

`System.Reactive` provides a pair of operators that group incoming sequences: `Buffer` and `Window`. `Buffer` will hold on to the incoming events until the group is complete, at which time it forwards them all at once as a collection of events. `Window` will logically group the incoming events but will pass them along as they arrive. The return type of `Buffer` is `IObservable<IList<T>>` (an event stream of collections); the return type of `Window` is `IObservable<IObservable<T>>` (an event stream of event streams).

The following example uses the `Interval` operator to create `OnNext` notifications once a second and then buffers them two at a time:

```C#
Observable.Interval(TimeSpan.FromSeconds(1))
    .Buffer(2)
    .Subscribe(x => Trace.WriteLine(
        $"{DateTime.Now.Second}: Got {x[0]} and {x[1]}"));
```

On my machine, this code produces a pair of outputs every two seconds:

```C#
13: Got 0 and 1
15: Got 2 and 3
17: Got 4 and 5
19: Got 6 and 7
21: Got 8 and 9
```

The following is a similar example of using `Window` to create groups of two events:

```C#
Observable.Interval(TimeSpan.FromSeconds(1))
    .Window(2)
    .Subscribe(group =>
    {
      Trace.WriteLine( $"{DateTime.Now.Second}: Starting new group");
      group.Subscribe(
          x => Trace.WriteLine($"{DateTime.Now.Second}: Saw {x}"),
          () => Trace.WriteLine($"{DateTime.Now.Second}: Ending group"));
    });
```

On my machine, this `Window` example produces this output:

```C#
17: Starting new group
18: Saw 0
19: Saw 1
19: Ending group
19: Starting new group
20: Saw 2
21: Saw 3
21: Ending group
21: Starting new group
22: Saw 4
23: Saw 5
23: Ending group
23: Starting new group
```

These examples illustrate the difference between `Buffer` and `Window`. `Buffer` waits for all the events in its group and then publishes a single collection. `Window` groups events the same way, but publishes the events as they come in; `Window` immediately publishes an observable that will publish the events for that window.

Both `Buffer` and `Window` also work with time spans. The following code is an example where all mouse move events are collected in windows of one second:

```C#
private void Button_Click(object sender, RoutedEventArgs e)
{
  Observable.FromEventPattern<MouseEventHandler, MouseEventArgs>(
          handler => (s, a) => handler(s, a),
          handler => MouseMove += handler,
          handler => MouseMove -= handler)
      .Buffer(TimeSpan.FromSeconds(1))
      .Subscribe(x => Trace.WriteLine(
          $"{DateTime.Now.Second}: Saw {x.Count} items."));
}
```

Depending on how you move the mouse, you should see output like the following:

```C#
49: Saw 93 items.
50: Saw 98 items.
51: Saw 39 items.
52: Saw 0 items.
53: Saw 4 items.
54: Saw 0 items.
55: Saw 58 items.
```

### Discussion

`Buffer` and `Window` are some of the tools you have for taming input and shaping it the way you want it to look. Another useful technique is throttling, which you'll learn about in Recipe 6.4.

Both `Buffer` and `Window` have other overloads that can be used in more advanced scenarios. The overloads with `skip` and `timeShift` parameters enable you to create groups that overlap other groups or skip elements in between groups. There are also overloads that take delegates, which enable you to dynamically define the boundary of the groups.

### See Also

Recipe 6.1 covers how to create sequences from events.

Recipe 6.4 covers throttling event streams.

## 6.4 Taming Event Streams with Throttling and Sampling

### Problem

A common problem with writing reactive code is when the events come in too quickly. A fast-moving stream of events can overwhelm your program's processing.

### Solution

`System.Reactive` provides operators specifically for dealing with a flood of event data. The `Throttle` and `Sample` operators give us two different ways to tame fast input events.

The `Throttle` operator establishes a sliding timeout window. When an incoming event arrives, it resets the timeout window. When the timeout window expires, it publishes the last event value that arrived within the window.

The following example monitors mouse movements and uses `Throttle` to only report updates once the mouse has stayed still for a full second:

```C#
private void Button_Click(object sender, RoutedEventArgs e)
{
  Observable.FromEventPattern<MouseEventHandler, MouseEventArgs>(
          handler => (s, a) => handler(s, a),
          handler => MouseMove += handler,
          handler => MouseMove -= handler)
      .Select(x => x.EventArgs.GetPosition(this))
      .Throttle(TimeSpan.FromSeconds(1))
      .Subscribe(x => Trace.WriteLine(
          $"{DateTime.Now.Second}: Saw {x.X + x.Y}"));
}
```

The output varies considerably based on mouse movement, but one example run on my machine looked like this:

```C#
47: Saw 139
49: Saw 137
51: Saw 424
56: Saw 226
```

`Throttle` is often used in situations such as autocomplete, when the user is typing text into a text box, and you don't want to do the actual lookup until the user stops typing.

`Sample` takes a different approach to taming fast-moving sequences. `Sample` establishes a regular timeout period and publishes the most recent value within that window each time the timeout expires. If no values were received within the sample period, then no results are published for that period.

The following example captures mouse movements and samples them in one-second intervals. Unlike the `Throttle` example, this `Sample` example doesn't require you to hold the mouse still to see data:

```C#
private void Button_Click(object sender, RoutedEventArgs e)
{
  Observable.FromEventPattern<MouseEventHandler, MouseEventArgs>(
          handler => (s, a) => handler(s, a),
          handler => MouseMove += handler,
          handler => MouseMove -= handler)
      .Select(x => x.EventArgs.GetPosition(this))
      .Sample(TimeSpan.FromSeconds(1))
      .Subscribe(x => Trace.WriteLine(
          $"{DateTime.Now.Second}: Saw {x.X + x.Y}"));
}
```

Here's the output on my machine when I first left the mouse still for a few seconds and then continuously moved it:

```C#
12: Saw 311
17: Saw 254
18: Saw 269
19: Saw 342
20: Saw 224
21: Saw 277
```

### Discussion

Throttling and sampling are essential tools for taming the flood of input. Don't forget that you can also easily do filtering with the standard LINQ `Where` operator. You can think of the `Throttle` and `Sample` operators as similar to `Where`, only they filter on time windows instead of filtering on event data. All three of these operators help you tame fast-moving input streams in different ways.

### See Also

Recipe 6.1 covers how to create sequences from events.

Recipe 6.2 covers how to change the context in which events are raised.

## 6.5 Timeouts

### Problem

You expect an event to arrive within a certain time and need to ensure that your program will respond in a timely fashion, even if the event doesn't arrive. Most commonly, this kind of expected event is a single asynchronous operation (e.g., expecting the response from a web service request).

### Solution

The `Timeout` operator establishes a sliding timeout window on its input stream. Whenever a new event arrives, the timeout window is reset. If the timeout expires without seeing an event in that window, the `Timeout` operator will end the stream with an `OnError` notification containing a `TimeoutException`.

The following example issues a web request for the example domain and applies a timeout of one second. To get the web request started, the code uses `ToObservable` to convert a `Task<T>` to an `IObservable<T>` (see Recipe 8.6):

```C#
void GetWithTimeout(HttpClient client)
{
  client.GetStringAsync("http://www.example.com/").ToObservable()
      .Timeout(TimeSpan.FromSeconds(1))
      .Subscribe(
          x => Trace.WriteLine($"{DateTime.Now.Second}: Saw {x.Length}"),
          ex => Trace.WriteLine(ex));
}
```

`Timeout` is ideal for asynchronous operations, such as web requests, but it can be applied to any event stream. The following example applies `Timeout` to mouse movements, which are easier to play around with:

```C#
private void Button_Click(object sender, RoutedEventArgs e)
{
  Observable.FromEventPattern<MouseEventHandler, MouseEventArgs>(
          handler => (s, a) => handler(s, a),
          handler => MouseMove += handler,
          handler => MouseMove -= handler)
      .Select(x => x.EventArgs.GetPosition(this))
      .Timeout(TimeSpan.FromSeconds(1))
      .Subscribe(
          x => Trace.WriteLine($"{DateTime.Now.Second}: Saw {x.X + x.Y}"),
          ex => Trace.WriteLine(ex));
}
```

On my machine, I moved the mouse a bit and then kept it still for a second, and got these results:

```C#
16: Saw 180
16: Saw 178
16: Saw 177
16: Saw 176
System.TimeoutException: The operation has timed out.
```

Note that once the `TimeoutException` is sent to `OnError`, the stream is finished. No more mouse movements come through. You may not want exactly this behavior, so the `Timeout` operator has overloads that substitute a second stream when the timeout occurs instead of ending the stream with an exception.

The code in the following example observes mouse movements until there's a timeout. After the timeout, the code observes mouse clicks:

```C#
private void Button_Click(object sender, RoutedEventArgs e)
{
  IObservable<Point> clicks =
      Observable.FromEventPattern<MouseButtonEventHandler,MouseButtonEventArgs>(
          handler => (s, a) => handler(s, a),
          handler => MouseDown += handler,
          handler => MouseDown -= handler)
      .Select(x => x.EventArgs.GetPosition(this));
      
  Observable.FromEventPattern<MouseEventHandler, MouseEventArgs>(
          handler => (s, a) => handler(s, a),
          handler => MouseMove += handler,
          handler => MouseMove -= handler)
      .Select(x => x.EventArgs.GetPosition(this))
      .Timeout(TimeSpan.FromSeconds(1), clicks)
      .Subscribe(
          x => Trace.WriteLine($"{DateTime.Now.Second}: Saw {x.X}, {x.Y}"),
          ex => Trace.WriteLine(ex));
}
```

On my machine, I moved the mouse a bit, then held it still for a second, and then clicked a couple of different points. The following outputs shows the mouse movements quickly moving through until the timeout, and then the two clicks:

```C#
49: Saw 95,39
49: Saw 94,39
49: Saw 94,38
49: Saw 94,37
53: Saw 130,141
55: Saw 469,4
```

### Discussion

`Timeout` is an essential operator in nontrivial applications because you always want your program to be responsive even if the rest of the world isn't. It's particularly useful when you have asynchronous operations, but it can be applied to any event stream. Note that the underlying operation is not actually canceled; in the case of a timeout, the operation will continue executing until it succeeds or fails.

### See Also

Recipe 6.1 covers how to create sequences from events.

Recipe 8.6 covers wrapping asynchronous code as an observable event stream.

Recipe 10.6 covers unsubscribing from sequences as a result of a `CancellationToken`.

Recipe 10.3 covers using a `CancellationToken` as a timeout.
