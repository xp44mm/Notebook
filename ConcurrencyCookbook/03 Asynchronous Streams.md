# Chapter 3. Asynchronous Streams

Asynchronous streams are a way to asynchronously receive multiple data items. They’re built on asynchronous enumerables (IAsyncEnumerable<T>). An asynchronous enumerable is an asynchronous version of an enumerable; that is, it can produce items on demand for a consumer, and each item may be produced asynchronously.

I find it useful to contrast asynchronous streams with other types that may be more familiar and to consider the differences. This helps me remember when to use asynchronous streams and when other types would be more appropriate.

Asynchronous Streams and Task<T>

The standard asynchronous approach with Task<T> is only sufficient for asynchronously handling a single data value. Once a given Task<T> completes, that’s it; a single Task<T> cannot provide more than one value of T for its consumers. Even if T is a collection, the value can only be provided once. See “Introduction to Asynchronous Programming” and Chapter 2 for more on using async with Task<T>.

When comparing Task<T> to asynchronous streams, the asynchronous streams are more similar to enumerables. Specifically, an IAsyncEnumerator<T> may provide any number of T values, one at a time. Like IEnumerator<T>, an IAsyncEnumerator<T> may be infinite in length.

Asynchronous Streams and IEnumerable<T>

IAsyncEnumerable<T>, as the name would imply, is similar to IEnumerable<T>. This is perhaps not a surprise; they both enable consumers to retrieve elements from them one at a time. The big difference is in the name: one is asynchronous and the other is not.

When your code iterates over an IEnumerable<T>, it blocks as it retrieves each element from the enumerable. If the IEnumerable<T> is representing some