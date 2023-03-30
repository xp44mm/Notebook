# Chapter 3. Asynchronous Streams

Asynchronous streams are a way to asynchronously receive multiple data items. They're built on asynchronous enumerables (`IAsyncEnumerable<T>`). An asynchronous enumerable is an asynchronous version of an enumerable; that is, it can produce items on demand for a consumer, and each item may be produced asynchronously.

I find it useful to contrast asynchronous streams with other types that may be more familiar and to consider the differences. This helps me remember when to use asynchronous streams and when other types would be more appropriate.

#### Asynchronous Streams and `Task<T>`

The standard asynchronous approach with `Task<T>` is only sufficient for asynchronously handling a single data value. Once a given `Task<T>` completes, that's it; a single `Task<T>` cannot provide more than one value of T for its consumers. Even if T is a collection, the value can only be provided once. See “Introduction to Asynchronous Programming” and Chapter 2 for more on using async with `Task<T>`.

When comparing `Task<T>` to asynchronous streams, the asynchronous streams are more similar to enumerables. Specifically, an `IAsyncEnumerator<T>` may provide any number of T values, one at a time. Like `IEnumerator<T>`, an `IAsyncEnumerator<T>` may be infinite in length.

#### Asynchronous Streams and `IEnumerable<T>`

`IAsyncEnumerable<T>`, as the name would imply, is similar to `IEnumerable<T>`. This is perhaps not a surprise; they both enable consumers to retrieve elements from them one at a time. The big difference is in the name: one is asynchronous and the other is not.

When your code iterates over an `IEnumerable<T>`, it blocks as it retrieves each element from the enumerable. If the `IEnumerable<T>` is representing some I/O-bound operation, such as a database query or API call, then the consuming code ends up blocking on I/O, which is not ideal. `IAsyncEnumerable<T>` works just like an `IEnumerable<T>`, except that it asynchronously retrieves each next element.

#### Asynchronous Streams and `Task<IEnumerable<T>>`

It is entirely possible to asynchronously return a collection with more than one item; one common example is `Task<List<T>>`. Still, async methods that return `List<T>` only get one return statement; the collection must be completely populated before it is returned. Even methods returning `Task<IEnumerable<T>>` may asynchronously return an enumerable, but then that enumerable is evaluated synchronously. Consider that LINQ-to-Entities has a `ToListAsync` LINQ method that returns `Task<List<T>>`. When a LINQ provider executes this, it has to communicate with the database and get all the matching responses back before it can finish populating the list and return it.

The limitation of `Task<IEnumerable<T>>` is that it cannot return items as it gets them; if returning a collection, it has to load all of its items into memory, populate the collection, and then return the entire collection all at once. Even if it returns a LINQ query, it can asynchronously build that query, but once the query is returned, each item is retrieved from that query synchronously. `IAsyncEnumerable<T>` also returns multiple items asynchronously, but the difference is that `IAsyncEnumerable<T>` can act asynchronously for each item returned. It's a true asynchronous stream of items.

#### Asynchronous Streams and `IObservable<T>`

Observables are a true notion of asynchronous streams; they produce their notifications one at a time with true support for asynchronous production (no blocking). But the consumption pattern for `IObservable<T>` is completely different than that of `IAsyncEnumerable<T>`. See Chapter 6 for more details about `IObservable<T>`.

To consume an `IObservable<T>`, code needs to define a LINQ-like query through which the observable notifications will flow, and then subscribe to the observable in order to start the flow. When working with observables, the code first defines how it will react to the incoming notifications, and then it turns them on (hence the name “reactive”). In contrast, consuming an `IAsyncEnumerable<T>` is done very similarly to consuming an `IEnumerable<T>`, except that the consumption is asynchronous.

There is also a backpressure problem; all notifications in `System.Reactive` are synchronous, so as soon as one item notification is sent to its subscribers, the observable continues execution and retrieves the next item to publish, possibly calling the API again. If the consuming code is consuming the stream asynchronously (i.e., doing some asynchronous action for each notification as it arrives), then the observable will race ahead of the consuming code.

A nice way of thinking about the difference between them is that `IObservable<T>` is push-based and `IAsyncEnumerable<T>` is pull-based. An observable stream will push notifications at your code, but an asynchronous stream will passively let your code (asynchronously) pull data items out of it. Only when the consuming code requests the next item does the observable stream resume execution.

#### Summary

A theoretical example may be useful. Many APIs take offset and limit parameters to enable paging of results. Let's say we wanted to define a method that retrieves results from an API that does paging, and we want our method to handle the paging so that our higher-level methods don't have to deal with that. If our method returns `Task<T>`, we are limited to returning only a single T. This is fine for a single call to the API where the T is the result of the API, but it doesn't work well as a return type if we want our method to call the API multiple times.

If our method returns `IEnumerable<T>`, we can create a loop, paging through the API results by calling it multiple times. Each time the method calls the API, it would yield return the results of that page. Further API calls are only necessary if the enumeration continues. Unfortunately, methods returning `IEnumerable<T>` cannot be asynchronous, so all our API calls are forced to be synchronous.

If our method returns `Task<List<T>>`, then we can have a loop that pages through the API results, calling the API asynchronously. However, the code cannot return each item as it gets the response; it would have to build up all the results and return them all at once.

If our method returns `IObservable<T>`, we can use `System.Reactive` to implement an observable stream that begins requests when subscribed to and publishes each item as we get them. The abstraction is push-based; it appears to consuming code that the API results are being pushed to them, which is more awkward to handle. `IObservable<T>` would be a better fit for scenarios like receiving and responding to WebSocket/SignalR messages.

If our method returns `IAsyncEnumerable<T>`, we can have a natural loop that uses both await and yield return to create a true pull-based asynchronous stream. `IAsyncEnumerable<T>` is the natural fit for this kind of scenario. Table 3-1 summarizes the different roles of common types.

Table 3-1. Type classifications

| Type                  | Single or multiple value | Asynchronous or synchronous | Push or pull |
| --------------------- | ------------------------ | --------------------------- | ------------ |
| T                     | Single value             | Synchronous                 | N/A          |
| `IEnumerable<T>`      | Multiple values          | Synchronous                 | N/A          |
| `Task<T>`             | Single value             | Asynchronous                | Pull         |
| `IAsyncEnumerable<T>` | Multiple values          | Asynchronous                | Pull         |
| `IObservable<T>`      | Single or multiple       | Asynchronous                | Push         |

##### WARNING

As this book goes to press, .NET Core 3.0 is still in beta, so the details around asynchronous streams may change.

## 3.1 Creating Asynchronous Streams

### Problem

You need to return multiple values, and each value may require some asynchronous work. This point is commonly reached from one of two paths: You have multiple values to return (as an `IEnumerable<T>`), and then need to add asynchronous work.

You have a single asynchronous return (as a `Task<T>`), and then need to add other return values.

### Solution

Returning multiple values from a method can be done with yield return, and asynchronous methods use async and await. With asynchronous streams, you can combine these two; just use a return type of `IAsyncEnumerable<T>`:

```C#
async IAsyncEnumerable<int> GetValuesAsync()
{
  await Task.Delay(1000); // some asynchronous work
  yield return 10;
  await Task.Delay(1000); // more asynchronous work
  yield return 13;
}
```

This simple example illustrates how await can be used with yield return to create an asynchronous stream.

A more real-world example is asynchronously enumerating over all the results of an API that uses parameters for paging:

```C#
async IAsyncEnumerable<string> GetValuesAsync(HttpClient client)
{
  int offset = 0;
  const int limit = 10;
  while (true)
  {
    // Get the current page of results and parse them.
    string result = await client.GetStringAsync(
        $"https://example.com/api/values?offset={offset}&limit={limit}");
    string[] valuesOnThisPage = result.Split('\n');
    // Produce the results for this page.
    foreach (string value in valuesOnThisPage)
      yield return value;
    // If this is the last page, we're done.
    if (valuesOnThisPage.Length != limit)
      break;
    // Otherwise, proceed to the next page.
    offset += limit;
  }
}
```

When GetValuesAsync starts, it does an asynchronous request for the first page of data, and then produces the first element. When the second element is then requested, GetValuesAsync produces it immediately, since it is also in that same first page of data. The next element is also in that page, and so on, up to 10 elements. Then, when the 11th element is requested, all the values in valuesOnThisPage will have been produced, so there are no more elements on the first page. GetValuesAsync will continue executing its while loop, proceed to the next page, do an asynchronous request for the second page of data, receive back a new batch of values, and then it'll produce the 11th element.

### Discussion

Ever since async and await were introduced, users have been wondering how to use them with yield return. For many years, that wasn't possible, but asynchronous streams has now brought this capability to C# and modern versions of .NET.

One thing you may notice with the more realistic example is that only some of the results need any asynchronous work. In that example, with a page length of 10, only about 1 out of every 10 elements will need asynchronous work. If the page size is 20, then only 1 out of every 20 elements will need asynchronous work.

This is a normal pattern with asynchronous streams. For many streams, the majority of asynchronous iteration is actually synchronous; asynchronous streams merely allow any next item to be retrieved asynchronously.

Asynchronous streams were designed with both asynchronous and synchronous code in mind; this is why asynchronous streams are built on `ValueTask<T>`. By using `ValueTask<T>` under the hood, asynchronous streams maximize their efficiency, whether items are retrieved synchronously or asynchronously. See Recipe 2.10 for more about `ValueTask<T>` and when it is appropriate to use.

When you do implement asynchronous streams, consider supporting cancellation. See Recipe 3.4 for a detailed discussion of cancellation with asynchronous streams. Some scenarios do not require actual cancellation; the consuming code can always choose not to retrieve the next element. That is a perfectly fine approach if there's no external source for the cancellation. If you have an asynchronous stream where you want to cancel the asynchronous stream, even if it's in the middle of getting the next element, then you'd want to support proper cancellation using a CancellationToken.

### See Also

Recipe 3.2 covers consuming asynchronous streams.

Recipe 3.4 covers handling cancellation for asynchronous streams.

Recipe 2.10 has more detail about `ValueTask<T>` and when it is appropriate to use.

## 3.2 Consuming Asynchronous Streams

### Problem

You need to process the results of an asynchronous stream, also known as an asynchronous enumerable.

### Solution

Consuming an asynchronous operation is done via await, and consuming an enumerable is usually done via foreach. Consuming an asynchronous enumerable is done by combining these two into await foreach. For example, given an asynchronous enumerable that pages over API responses, you can consume it and write each element to the console:

```C#
IAsyncEnumerable<string> GetValuesAsync(HttpClient client);
public async Task ProcessValueAsync(HttpClient client)
{
  await foreach (string value in GetValuesAsync(client))
  {
    Console.WriteLine(value);
  }
}
```

Conceptually, what is happening here is that GetValuesAsync is invoked, and it returns an `IAsyncEnumerable<T>`. The foreach then creates an asynchronous enumerator from that asynchronous enumerable. Asynchronous enumerators are logically similar to regular enumerators, except that their “get next element” operation may be asynchronous. So, the await foreach will await for the next element to arrive or for the asynchronous enumerator to complete. If an element arrived, await foreach will execute its loop body; if the asynchronous enumerator is complete, then the loop will exit. It is also natural to do asynchronous processing of each element:

```C#
IAsyncEnumerable<string> GetValuesAsync(HttpClient client);
public async Task ProcessValueAsync(HttpClient client)
{
  await foreach (string value in GetValuesAsync(client))
  {
    await Task.Delay(100); // asynchronous work
    Console.WriteLine(value);
  }
}
```

In this case, the await foreach won't proceed to the next element until the loop body is complete. So, the await foreach will asynchronously receive the first element, then asynchronously execute the loop body for that first element, then asynchronously receive the next element, then asynchronously execute the loop body for that next element, and so on.

There is an await buried in the await foreach: the “get next element” operation is awaited. With a regular await, you can avoid the implicitly captured context by using ConfigureAwait(false), as described in Recipe 2.7. Asynchronous streams also support `ConfigureAwait(false)`, which is passed to the hidden await statements:

```C#
IAsyncEnumerable<string> GetValuesAsync(HttpClient client);
public async Task ProcessValueAsync(HttpClient client)
{
  await foreach (string value in GetValuesAsync(client).ConfigureAwait(false))
  {
    await Task.Delay(100).ConfigureAwait(false); // asynchronous work
    Console.WriteLine(value);
  }
}
```

### Discussion

await foreach is the most natural way to consume asynchronous streams. The language supports `ConfigureAwait(false)` for avoiding context in await foreach.

It's also possible to pass in cancellation tokens; this is a bit more advanced due to the complexity of asynchronous streams, so you can find it covered in Recipe 3.4.

While it's possible and natural to use await foreach to consume asynchronous streams, there's an exhaustive library of asynchronous LINQ operators available; some of the more popular ones are covered in Recipe 3.3. The body of await foreach can be either synchronous or asynchronous. For the asynchronous example in particular, this is something that is much trickier to get right when working with other streaming abstractions, such as `IObservable<T>`. This is because observable subscriptions must be synchronous, but await foreach permits natural asynchronous processing. The await foreach generates an await used for the “get next element” operation; it also generates an await used to asynchronously dispose the enumerable.

### See Also

Recipe 3.1 covers producing asynchronous streams.

Recipe 3.4 covers handling cancellation for asynchronous streams.

Recipe 3.3 covers common LINQ methods for asynchronous streams.

Recipe 11.6 covers asynchronous disposal.

## 3.3 Using LINQ with Asynchronous Streams

### Problem

You want to process an asynchronous stream using well-defined and well-tested operators.

### Solution

`IEnumerable<T>` has LINQ to Objects, and `IObservable<T>` has LINQ to Events. Both of these have libraries of extension methods that define operators you can use to build queries. `IAsyncEnumerable<T>` also has LINQ support, provided by the .NET community in the System.Linq.Async NuGet package.

As an example, one of the common questions about LINQ is how to use the Where operator if the predicate for Where is asynchronous. In other words, you want to filter a sequence based on some asynchronous condition—e.g., you need to look up each element in a database or API to see if it should be included in the result sequence. Where doesn't work with an asynchronous condition because the Where operator requires that its delegate return an immediate, synchronous answer.

Asynchronous streams have a support library that defines many useful operators. In the following example, WhereAwait is the proper choice:

```C#
IAsyncEnumerable<int> values = SlowRange().WhereAwait(
    async value =>
    {
      // Do some asynchronous work to determine
      //  if this element should be included.
      await Task.Delay(10);
      return value % 2 == 0;
    });
await foreach (int result in values)
{
  Console.WriteLine(result);
}
// Produce sequence that slows down as it progresses.
async IAsyncEnumerable<int> SlowRange()
{
  for (int i = 0; i != 10; ++i)
  {
    await Task.Delay(i * 100);
    yield return i;
  }
}
```

LINQ operators for asynchronous streams also include synchronous versions; it does make sense to apply a synchronous Where (or Select, or whatever) to an asynchronous stream. The result is still an asynchronous stream:

```C#
IAsyncEnumerable<int> values = SlowRange().Where(
    value => value % 2 == 0);
await foreach (int result in values)
{
  Console.WriteLine(result);
}
```

All of your old LINQ friends are here: Where, Select, SelectMany, and even Join. Most LINQ operators now also take asynchronous delegates, like the WhereAwait example above.

### Discussion

Asynchronous streams are pull-based, so there's no time-related operators like there are for observables. Throttle and Sample don't make sense in this world, since the elements are pulled out of the asynchronous stream on demand.

LINQ methods for asynchronous streams can also be useful for regular enumerables. If you find yourself in this situation, you can call `ToAsyncEnumerable()` on any `IEnumerable<T>`, and then you'll have an asynchronous stream interface that you can use with WhereAwait, SelectAwait, and other operators that support asynchronous delegates. Before you dive in, a word on naming is in order. The example in this recipe used WhereAwait as the asynchronous equivalent of Where. As you explore the LINQ operators for asynchronous streams, you'll find that some end in Async and others end in Await. The operators that end in Async return an awaitable; they represent a regular value, not an asynchronous sequence. The operators that end in Await take an asynchronous delegate; the Await in their name implies that they actually perform an await on the delegate you pass to them. We already looked at an example of the Await suffix with Where and WhereAwait. The Async suffix only applies to termination operators—operators that extract some value or perform some calculation and return an asynchronous scalar value instead of an asynchronous sequence. An example of a termination operator is CountAsync, the asynchronous stream version of Count, which can count the number of elements that match some predicate:

```C#
int count = await SlowRange().CountAsync(
    value => value % 2 == 0);
```

That predicate can also be asynchronous, in which case you would then use the `CountAwaitAsync` operator, since it both takes an asynchronous delegate (which it will await) and produces a single terminal value, the count:

```C#
int count = await SlowRange().CountAwaitAsync(
    async value =>
    {
      await Task.Delay(10);
      return value % 2 == 0;
    });
```

In summary, operators that can take delegates have two names: one with an Await suffix and one without. In addition, operators that return a terminal value rather than an asynchronous stream end in Async. If an operator takes an asynchronous delegate and returns a terminal value, then it has both suffixes.

##### TIP

The LINQ operators for asynchronous streams are in the NuGet package for System.Linq.Async. Additional LINQ operators for asynchronous streams can be found in the NuGet package for System.Interactive.Async.

### See Also

Recipe 3.1 covers producing asynchronous streams.

Recipe 3.2 covers consuming asynchronous streams.

## 3.4 Asynchronous Streams and Cancellation

### Problem

You want a way to cancel asynchronous streams.

### Solution

Not all asynchronous streams require cancellation. It's possible to simply stop enumerating when a condition is reached. If that is the only kind of “cancellation” necessary, then a true cancellation isn't required, as the following example shows:

```C#
await foreach (int result in SlowRange())
{
  Console.WriteLine(result);
  if (result >= 8)
    break;
}
// Produce sequence that slows down as it progresses.
async IAsyncEnumerable<int> SlowRange()
{
  for (int i = 0; i != 10; ++i)
  {
    await Task.Delay(i * 100);
    yield return i;
  }
}
```

That said, it's often useful to cancel asynchronous streams, as some operators pass cancellation tokens to their source streams. In this scenario, you' would want to use a `CancellationToken` to stop the await foreach from external code.

An async method returning `IAsyncEnumerable<T>` may take a cancellation token by defining a parameter marked with the `EnumeratorCancellation` attribute. It can then use the token naturally, which is usually done by passing it to other APIs that take cancellation tokens, like this:

```C#
using var cts = new CancellationTokenSource(500);
CancellationToken token = cts.Token;
await foreach (int result in SlowRange(token))
{
  Console.WriteLine(result);
}
// Produce sequence that slows down as it progresses.
async IAsyncEnumerable<int> SlowRange(
    [EnumeratorCancellation] CancellationToken token = default)
{
  for (int i = 0; i != 10; ++i)
  {
    await Task.Delay(i * 100, token);
    yield return i;
  }
}
```

### Discussion

The example solution here passes the `CancellationToken` directly to the method returning the asynchronous enumerator. This is the most common usage.

There are other scenarios where your code will be given an asynchronous enumerator and will want to apply a `CancellationToken` to the enumerators it uses. Cancellation tokens are used when starting a new enumeration of an enumerable, so it makes sense to apply a `CancellationToken` in this way. The enumerable itself is defined by the `SlowRange` method, but it's not started until it is consumed. There are even some scenarios where different cancellation tokens should be passed to different enumerations of the enumerable. Briefly; it is not the enumerable that is cancelable, but the enumerator created by that enumerable. This is an uncommon but important use case, and it's the reason asynchronous streams support a `WithCancellation` extension method that you can use to attach a `CancellationToken` to a specific iteration of an asynchronous stream:

```C#
async Task ConsumeSequence(IAsyncEnumerable<int> items)
{
  using var cts = new CancellationTokenSource(500);
  CancellationToken token = cts.Token;
  await foreach (int result in items.WithCancellation(token))
  {
    Console.WriteLine(result);
  }
}
// Produce sequence that slows down as it progresses.
async IAsyncEnumerable<int> SlowRange(
    [EnumeratorCancellation] CancellationToken token = default)
{
  for (int i = 0; i != 10; ++i)
  {
    await Task.Delay(i * 100, token);
    yield return i;
  }
}
await ConsumeSequence(SlowRange());
```

With the `EnumeratorCancellation` parameter attribute in place, the compiler takes care of passing the token from `WithCancellation` to the token parameter marked by `EnumeratorCancellation`, and the cancellation request now causes await foreach to raise an `OperationCanceledException` after it has processed the first few items.

The `WithCancellation` extension method doesn't prevent `ConfigureAwait(false)`. Both extension methods can be chained together:

```C#
async Task ConsumeSequence(IAsyncEnumerable<int> items)
{
  using var cts = new CancellationTokenSource(500);
  CancellationToken token = cts.Token;
  await foreach (int result in items
      .WithCancellation(token).ConfigureAwait(false))
  {
    Console.WriteLine(result);
  }
}
```

### See Also

Recipe 3.1 covers producing asynchronous streams.

Recipe 3.2 covers consuming asynchronous streams.

Chapter 10 covers cooperative cancellation across multiple technologies.

