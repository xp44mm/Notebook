# appendix B The Rx Disposables library

The `System.Reactive.Core` package includes an additional treat that can be helpful: the Rx Disposables library. It's a set of types that implement the `IDisposable` interface and provides generic implementations to recurring patterns to help you accomplish most of the things you'll need your disposable to do, without creating your own type! All the types listed in this appendix reside in the `System.Reactive.Disposables` namespace.

~~NOTE The `System.Reactive.Core` package is the main Rx package. For details about other Rx packages, see chapter 2.~~

Table B.1 will help you remember what the disposable utilities and types do.

Table B.1 The tenets of the Rx Disposables library

Disposable.Create

A static method used to create a disposable that executes a given code when disposed of.

Disposable.Empty

Creates an empty disposable.

ContextDisposable

Runs the disposal of its underlying disposable resource on a given `SynchronizationContext`.

ScheduledDisposable

Runs the disposal of its underlying disposable resource on a given `IScheduler`.

SerialDisposable

Holds a replaceable underlying disposable and disposes of the previous disposable when replaced.

MultipleAssignmentDisposable

Holds a replaceable underlying disposable but doesn't dispose of the previous disposable when replaced.

RefCountDisposable

Disposes of the underlying disposable when all the referencing disposables are disposed of.

CompositeDisposable

Combines multiple disposables into a single disposable object that will dispose of all the disposables together.

CancellationDisposable

Cancels a given `CancellationTokenSource` when disposed of.

BooleanDisposable

Sets a Boolean flag when disposed of. You can query whether the `BooleanDisposable` type was disposed of by using the `IsDisposed` property.

## B.1 Disposable.Create

The most flexible way to create a disposable is with the `Disposable.Create` static factory method. All you need to do is pass the action you want the disposable to execute upon calling the `Dispose` method. The next example creates a disposable that changes the state of a screen (from busy to nonbusy). The screen shows news items after they're downloaded. While the screen is in the busy state, the UI can display a progress bar to show the user that something is happening in the background.

```C#
private async Task RefreshNewsAsync()
{
    this.IsBusy = true;
    NewsItems = Enumerable.Empty<string>();
    using(Disposable.Create(() => this.IsBusy = false))
    {
        NewsItems = await DownloadNewsItems();
    }
}
```

The `IsBusy` property is bound to a busy indicator on the screen, so when it's set to true, the busy indicator is shown, and when it's set to false, the busy indicator is invisible. The nice thing about working with disposables is that the using statement ensures that the `Dispose` method is executed even if the code throws an exception, so you can be assured that the screen won't get stuck in a busy state.

## B.2 Disposable.Empty

The static property `Disposable.Empty` returns a disposable object that has an empty `Dispose` method. This can be handy for initializing an `IDisposable` variable or member so you won't have to write code to check for null and risk forgetting it. It can also serve to return a disposable object, such as when you create your own Rx operators that must return a disposable object from their `Subscribe` method, but you don't need any special disposing functionality. Here's a simplified version of the `Observable.Return` operator that uses `Disposable.Empty`:

```C#
public static IObservable<T> Return<T>(T value)
{
    return Observable.Create<T>(o =>
    {
        o.OnNext(value);
        o.OnCompleted();
        return Disposable.Empty;
    });
}

```

## B.3 ContextDisposable

The `ContextDisposable` class wraps a disposable object and executes its `Dispose` method on a specified `SynchronizationContext`. Executing the `Dispose` method on a `SynchronizationContext` is important when the operation is tied to a specific context (for example, when changing a UI element). This code creates a `StartBusy` method to create a disposable that turns off the busy indicator on the UI's `SynchronizationContext`:

```C#
public IDisposable StartBusy()
{
    this.IsBusy = true;
    return new ContextDisposable(
        SynchronizationContext.Current,
        Disposable.Create(() => this.IsBusy = false));
}
```

## B.4 ScheduledDisposable

The `ScheduledDisposable` class works similarly to `ContextDisposable`, but instead of specifying a `SynchronizationContext`, you specify an `IScheduler` on which the disposal invocation is scheduled. For example, when you use the `SubscribeOn` operator on an observable, the returned disposable from your subscription is wrapped with a `ScheduledDisposable` that uses the `IScheduler` provided. See the next section for an example.

## B.5 SerialDisposable

The `SerialDisposable` class lets you wrap a replaceable disposable object. Upon replacing the inner disposable, the previous one is automatically disposed of. Besides that, `SerialDisposable` remembers whether it's been disposed of, and if it was and the inner disposable is replaced, then the inner disposable will also be disposed of. An example is shown in a simplified version of the `SubscribeOn` operator. Because I can't predict exactly when the scheduler will execute work that I schedule, I'm creating a `SerialDisposable` and set its inner disposable inside the scheduled operation:
```C#
public static IObservable<TSource> MySubscribeOn<TSource>(
     this IObservable<TSource> source,
     IScheduler scheduler)
{
    return Observable.Create<TSource>(observer =>
    {

        var d = new SerialDisposable();
        d.Disposable = scheduler.Schedule(() =>
        {
            d.Disposable = new ScheduledDisposable(scheduler, source.SubscribeSafe(observer));
        });
        return d;
    });
}
```

After the scheduled task executes the subscription to the source observable, the underlying disposable of `SerialDisposable` is set to the subscription that's wrapped with a `ScheduledDisposable`, so its disposal takes place on the scheduler. If it's already disposed of, the assigned disposable will also be disposed of.

## B.6 RefCountDisposable

The `RefCountDisposable` class wraps a disposable object and disposes of it only after all referencing disposables have been disposed of. A referencing disposable is created by calling the method `GetDisposable` on the `RefCountDisposable`.

Here's an example that shows that the inner disposable is disposed of only after the two referencing disposables are disposed of:

```C#
var inner = Disposable.Create(
     () => Console.WriteLine("Disposing inner-disposable"));
var refCountDisposable = new RefCountDisposable(inner);
var d1 = refCountDisposable.GetDisposable();
var d2 = refCountDisposable.GetDisposable();
refCountDisposable.Dispose();
Console.WriteLine("Disposing 1st");
d1.Dispose();
Console.WriteLine("Disposing 2nd");
d2.Dispose();
```

The output is as follows:

```C#
Disposing 1st
Disposing 2nd
Disposing inner-disposable
```

## B.7 MultipleAssignmentDisposable

The `MultiAssignmentDisposable` class holds an underlying disposable object that can be replaced at any time, but unlike the `SerialDisposable`, replacing the underlying disposable doesn't automatically dispose of the previous one. But `MultiAssignmentDisposable` remembers whether the disposable has been disposed of. If so, and the underlying disposable is replaced, `MultiAssignmentDisposable` will automatically dispose of it.

## B.8 CompositeDisposable

The `CompositeDisposable` class lets you group multiple disposable objects into one, so that when the `CompositeDisposable` is disposed of, all its inner disposables are disposed of as well.
```C#
var compositeDisposable = new CompositeDisposable(
    Disposable.Create(() => Console.WriteLine("1st disposed")),
    Disposable.Create(() => Console.WriteLine("2nd disposed")));
compositeDisposable.Dispose();
```

The same can also be written using the Add method:

```C#
var compositeDisposable = new CompositeDisposable();
compositeDisposable.Add(Disposable.Create(
     () => Console.WriteLine("1st disposed")));
compositeDisposable.Add(Disposable.Create(
     () => Console.WriteLine("2nd disposed")));
compositeDisposable.Dispose();
```

Often when I subscribe to multiple observables inside my class, I want to group all the subscriptions together so I can dispose of them at the same time. To keep my observable pipelines fluent, I created this handy extension method:

```C#
static CompositeDisposable AddToCompositeDisposable(this IDisposable @this,
    CompositeDisposable compositeDisposable)
{
    if (compositeDisposable==null)
        throw new ArgumentNullException(nameof(compositeDisposable));
    compositeDisposable.Add(@this);
    return compositeDisposable;
}
```

Then I can use it like this:

```C#
var observable = ... as IObservable<string>;
observable.Where(x => x.Length%2 == 0)
    .Select(x => x.ToUpper())
    .Subscribe(x => Console.WriteLine(x))
    .AddToCompositeDisposable(compositeDisposable);
observable.Where(x => x.Length % 2 == 1)
    .Select(x => x.ToLower())
    .Subscribe(x => Console.WriteLine(x))
    .AddToCompositeDisposable(compositeDisposable);
```

## B.9 SingleAssignmentDisposable

The `SingleAssignmentDisposable` class allows only a single assignment of its underlying disposable object. If there's an attempt to set the underlying disposable object when it's already set, an `InvalidOperationException` is thrown.

## B.10 CancellationDisposable

The `CancellationDisposable` class is an adapter between the `IDisposable` world and the `CancellationTokenSource` world. When `CancellationDisposable` is disposed of, the underlying `CancellationTokenSource` is canceled. This is used, for example, in the Rx `TaskPoolScheduler` so the returned disposable from `Schedule` will be tied to the `CancellationToken` that's sent to the `TaskScheduler`. Here's a simplified version of how it looks:
```C#
IDisposable Schedule<TState>(TState state, Func<IScheduler, TState, IDisposable> action)
{
    var d = new SerialDisposable();
    var cancelable = new CancellationDisposable();
    d.Disposable = cancelable;
    Task.Run(() =>
    {
        d.Disposable = action(this, state);
    }, cancelable.Token);
    return d;
}
```

The `CancellationToken` created by the underlying `CancellationTokenSource` of the `CancellationDisposable` is sent to the `TaskScheduler` to prevent it from running the `Task` if the user disposed of the disposable that was returned from the method.

## B.11 BooleanDisposable

The `BooleanDisposable` class holds a Boolean flag that lets you check whether it has already been disposed of. For example:

```C#
var booleanDisposable = new BooleanDisposable();
Console.WriteLine("Before dispose, booleanDisposable.IsDisposed = {0}",
    booleanDisposable.IsDisposed);

booleanDisposable.Dispose();
Console.WriteLine("After dispose, booleanDisposable.IsDisposed = {0}",
    booleanDisposable.IsDisposed);
```

The output is as follows:

```C#
Before dispose, booleanDisposable.IsDisposed = False
After dispose, booleanDisposable.IsDisposed = True
```

## B.12 Summary

The Rx package provides not only Rx-specific types and utilities, but also a rich library to ease your life when creating disposables.

  - To create a disposable that executes a given code when disposed of, use the `Disposable.Create` static method.

  - To create an empty disposable, use the `Disposable.Empty` static property.

  - To make sure that a disposable will be disposed of in a specific `SynchronizationContext`, wrap it with an instance of `ContextDisposable`.

  - To make sure that a disposable will be disposed of in a specific `IScheduler`, wrap it with an instance of `ScheduledDisposable`.

  - Use the `SerialDisposable` class when you need a disposable that holds an underlying disposable resource that can be replaced, causing an automatic disposal of the previous underlying disposable resource.

  - When you need a disposable whose underlying disposable resource can be replaced but without disposing the previous one, use the `MultipleAssignmentDisposable` class.

  - To make sure that a disposable object will be disposed of only after all referencing disposables are disposed of, use the `RefCountDisposable` class.

  - To combine multiple disposables into a single disposable object that will dispose of all the disposables together, use the `CompositeDisposable` class.

  - Use the `SingleAssignmentDisposable` when you need to make sure that only a single underlying disposable will be set.

  - Use the `CancellationDisposable` to cancel a `CancellationTokenSource` upon disposal.

  - Use the `BooleanDisposable` when you need to query a disposable about whether it was disposed of.
