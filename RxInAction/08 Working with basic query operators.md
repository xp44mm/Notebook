# 8 Working with basic query operators

After a source observable emits a notification, there's often a pipeline the notification goes through before it reaches the destined observer. Almost every example in this book shows how operators are used in some way to manipulate a generated sequence of notifications, which your observers eventually observe. This chapter categorizes and explains the basic operators that you'll use to create queries on the observables at hand. These include transformations and mappings, filtering and flattening, and aggregation operators that generate sums, averages, and other types of quantifiable results, as shown in figure 8.1. Some of the operators shown in this chapter were introduced in previous chapters, but you haven't yet seen how they're defined and the capabilities they provide.

Figure 8.1 Example of an observable pipeline. Each block may or may not be present, and the order of blocks may change as well. Sometimes one type of block may be present more than once.

NOTE For those who speak LINQ as a second language, this chapter might seem trivial at times. But more than once I've found that previous knowledge may lead to false conclusions, so it's better to be on the safe side and make sure that a standard query operator works the way you'd expect it to.

## 8.1 Selecting what's important (mapping)

Observables emit notifications—so far, so good. Sometimes the notifications you receive aren't exactly what you were looking for, and not always in the form that's the easiest for your program to process. For example, when working with a remote end-point, the object that travels from one side to the other is usually simple and light, containing a minimal amount of data. This is commonly known as a data transfer object (DTO). DTOs carry only the must-have information (for example, identifiers) to your application in order to perform its logic; however, when the DTO enters your application, it's often easier for you to work with your own data type or for you to fetch the corresponding entities from your datastore. For that, you need to create a transforming method with the `Select` operator.

The `Select` operator, illustrated in figure 8.2, lets you transform a notification that's flowing in your observable pipeline (in functional programming, this operator is also called `Map`) to a data format that's more usable for your purposes.

Figure 8.2 The Select operator projects each element of an observable sequence into a new form.

TIP Instead of Select, you can use the Map operator if you feel it's more natural, but you'll need to add NuGet's `System.Reactive.Observable.Aliases` package (www.nuget.org/packages/System.Reactive.Observable.Aliases). Both operators are the same implementation, just with different names.

The next bit of code shows how to use Select to transform a received ChatMessage that contains the sending user identifier. You load a User object from the database and create a ViewModel that will be easier to work with in the UI layer. Figure 8.3 illustrates this process.

Figure 8.3 Using the Select operator to convert an incoming message DTO to a corresponding ViewModel with a loaded user object from the database

```C#
var messages = ... as IObservable<ChatMessage>

var messagesViewModels =
    messages
        .Select(m => new ChatMessageViewModel
        {
            MessageContent = m.Content,
            User = LoadUserFromDb(m.Sender)
        });
```

The transformation done with the `Select` operator creates a new observable that emits the ViewModel you created.

Using Select, you can make multiple transformations in a declarative and readable form. The Select operator has two overloads—one accepts a simple selector function that receives the notification data, and the second receives the notification index:

```C#
IObservable<TResult> Select<TSource, TResult>(this IObservable<TSource> source,
    Func<TSource, TResult> selector);
IObservable<TResult> Select<TSource, TResult>(this IObservable<TSource> source,
    Func<TSource, int, TResult> selector);
```

The transformation function (the selector) that the Select operator receives is free to do whatever transformations you like. It can create an object based on the notification values, extract a subproperty of the received notification and return it, or ignore the notification and return a different value altogether (although this probably won't be that helpful).

An interesting scenario occurs when you want to select a subproperty that's an enumerable or an observable. This creates an observable of enumerables (or observables). If you want to apply more operators to the enumerables (or observables), you'll have to do that independently for each enumerable (or observable). For these cases, it's better to work with the `SelectMany` operator. As you'll see in a short while, overloads let you keep track of the source object (the event) that created the enumerable.

## 8.2 Flattening observables

Observables that carry observables (or enumerables) as their notification objects make it harder to apply operators that need to work on each one of the inner elements. Let's say an observable carries collections of numbers, and you want to filter all the odd numbers. What's the best way to filter the collection? Adding a `Where` operator for every inner collection is possible but makes your code less readable. A better approach is to flatten the observable with `SelectMany` and use the Where operator on the resulting observable. Now, let's see it in action.

### 8.2.1 Flattening observables of enumerables

The `SelectMany` operator is used to flatten an observable. If each element in the observable sequence is a collection, as shown in figure 8.4, the resulting observable will emit all the elements from each collection.

Figure 8.4 The SelectMany operator flattens an observable of collections to an observable of items.

SelectMany is also called `FlatMap` (as an alias) because it maps each item to a collection and then flattens those collections into one stream. Suppose you have an observable of news items and every item has a collection of images that you want to show on the screen, but only if they're rated PG-13 (child friendly). You can write the Rx query like this:

```C#
var news = ... as IObservable<NewsItem>

news.SelectMany(n => n.Images)
    .Where(img => img.IsChildFriendly)
    .Subscribe(img => AddToHeadlines(img));
```

The `SelectMany` operator used here has the following signature:

```C#
IObservable<TResult> SelectMany<TSource, TResult>(this IObservable<TSource> source,
    Func<TSource, IEnumerable<TResult>> selector)
```

It receives the source observable and, using a selector function, generates an enumerable of type TResult for each item in the observable sequence. The resulting observable flattens all the generated enumerables, so that's why the return type is `IObservable<TResult>`.

You can see in this example that the selector function of `SelectMany` returns the Images collection. The resulting observable of `SelectMany` emits all the images from all the news items; therefore, you can write the `Where` clause at the same level as the SelectMany operator. You don't need to add the Where clause to each collection.

Figure 8.5 shows an example of news items that your application might receive, followed by the application's output for those news items.

```C#
var theNews = new[] {
    new NewsItem {
        Title = "NewsItem1",
        Url = new Url("https://news.com/NewsItem1"),
        Images = new[] { 
            new NewsImage {ImageName = "Item1Image1"},
            new NewsImage {ImageName = "Item1Image2", IsChildFriendly = false}
        },
    },
    new NewsItem {
        Title = "NewsItem2",
        Url = new Url("https://news.com/NewsItem2"),
        Images = new[] {
            new NewsImage {ImageName = "Item2Image1"}
        },
    }
}
```

Figure 8.5 The test news items. The first news item contains two images, but only one that is child friendly, and the second news item contains a single image.

The program output is: 

```
News headline image: Item1Image1 
News headline image: Item2Image1 
```

Note that when `SelectMany` is applied to an observable of enumerables, the items' order is kept; so, in this scenario, all the images of the first news item will be processed before any images of the second news item.

`SelectMany` also has an overload that lets you receive the index of each notification from the source observable:

```C#
IObservable<TResult> SelectMany<TSource, TResult>(this IObservable<TSource> source,
    Func<TSource, int, IEnumerable<TResult>> selector)
```

You can use this overload to change the way you generate the enumerable, based on the position of the source item in the observable sequence. Next, I'll show you how to keep track of the source notifications that create the enumerable to which the observed items belong.

#### PROCESSING THE SOURCE AND THE RESULT

`SelectMany` does a great job flattening the observable, but each inner item that's now emitted loses its connection to the source object that generated the collection. For example, the images generated from the news items are separated from that news item object. Luckily, `SelectMany` offers another overload that can be helpful.

Suppose for each image in the headline view you want to add a link to the news item it belongs to. This is how you'd use `SelectMany` now:

```C#
var news = ... as IObservable<NewsItem>

news.SelectMany(n => n.Images,
    (newsItem, img) => new NewImageViewModel
    {
        ItemUrl = newsItem.Url,
        NewsImage = img
    })
    .Where(vm => vm.NewsImage.IsChildFriendly)
    .Subscribe(img => AddToHeadlines(img));
```

The overloads for the `SelectMany` operator that accept a resultSelector function are shown in the following code snippet. For each item in every collection, the resultSelector function is invoked, together with the source element that generates the collection. The value returned from resultSelector is the value that the resulting observable emits. The resultSelector in the second overload also receives the index of the emitted notification.

```C#
IObservable<TResult> SelectMany<TSource, TCollection, TResult>(this IObservable<TSource> source,
    Func<TSource, IEnumerable<TCollection>> collectionSelector,
    Func<TSource, TCollection, TResult> resultSelector)
    
IObservable<TResult> SelectMany<TSource, TCollection, TResult>(this IObservable<TSource> source,
    Func<TSource, int, IEnumerable<TCollection>> collectionSelector,
    Func<TSource, int, TCollection, int, TResult> resultSelector)
```

A nice feature that the C# compiler provides is the ability to add the SelectMany operator to the query without a plethora of code when done manually. To achieve this, use the `Let` operator when writing the query. Here's how to manually create the view models of the child-friendly news images with query syntax:

```C#
var news = ... as IObservable<NewsItem>
var newsImages =
    from n in news
    from img in n.Images
    where img.IsChildFriendly
    select new NewImageViewModel {
        ItemUrl = n.Url,
        NewsImage = img
    };
```

The two from statements (and you can use as many from statements as you like) will cause the compiler to generate a SelectMany that will wrap the news item and the news image inside an object. Thereafter, all references to the news item and news image will take its value from that object behind the scenes.

SelectMany works not only on observables of enumerables but on observables of observables as well.

### 8.2.2 Flattening observables of observables

The same difficulty of applying operators to each emitted observable applies to observables that carry other observables as well (as in figure 8.6).

Figure 8.6 The SelectMany operator flattens an observable of observables to an observable of emitted items from all the observables.

Suppose your Chat application supports different chat rooms that can be opened by you or by other users that add you as a participant. Each chat room is represented as the type ChatRoom, which holds an observable for the messages sent in each room. In the application, you want to show a dashboard view with the recent messages (no matter from which room they were sent). This is shown in figure 8.7.

Figure 8.7 Flattening messages from various chat rooms to one stream of messages

Here's how you can do this with Rx:

```C#
var rooms = ... as IObservable<ChatRoom>

rooms
    .Log("Rooms")
    .SelectMany(r => r.Messages) // mergeMap ?
    .Select(m => new ChatMessageViewModel(m))
    .Subscribe(vm => AddToDashboard(vm));
```

To simulate a situation in which two chat rooms are opened and messages are sent, I created this simple code where the rooms observable and each Messages observable is a `Subject`.

Listing 8.1 Test program to simulate the creation of chat rooms and message emissions

```C#
var roomsSubject = new Subject<ChatRoom>();
var room1 = new Subject<ChatMessage>();
var room2 = new Subject<ChatMessage>();

roomsSubject.OnNext(new ChatRoom {Id = "Room1", Messages = room1});
room1.OnNext(new ChatMessage{Content = "First Message", Sender = "1"});
room1.OnNext(new ChatMessage{Content = "Second Message", Sender = "1"});
roomsSubject.OnNext(new ChatRoom{Id = "Room2", Messages = room2});
room2.OnNext(new ChatMessage{Content = "Hello World", Sender = "2" });
room1.OnNext(new ChatMessage{Content = "Another Message", Sender = "1" });

var rooms = roomsSubject.AsObservable() as IObservable<ChatRoom>; //接上个代码的rooms
```


Running this test program against the query produces this output:

```C#
Rooms - OnNext(ChatRoom: Room1)
Room: Room1 , Message: "First Message" was sent by Id=1 Name:User1
Room: Room1 , Message: "Second Message" was sent by Id=1 Name:User1
Rooms - OnNext(ChatRoom: Room2)
Room: Room2 , Message: "Hello World" was sent by Id=2 Name:User2
Room: Room1 , Message: "Another Message" was sent by Id=1 Name:User1
```

In chapter 6, you created the Log operator that, when called, prints a message for every `OnNext`, `OnError`, and `OnCompleted` method. Log is used here to display a message when a new room is opened (the bolded lines in the output). You can see in the output that all the messages that were sent, no matter from which room, are displayed in a centralized fashion. The view model created for each message is the one that formats the output line for each message, and the AddToDashboard method simply writes it to the console.

#### PROCESSING THE SOURCE AND THE RESULT

As with observables of enumerables, when applying the `SelectMany` operator on an observable of observables, you can specify a resultSelector function that will be invoked for each notification (regardless of which observable originates the notification), together with the source item that creates the observable that emits the notification (figure 8.8). This allows you to track the connection between the notifications and their origin and to produce a value based on the results. For example, for chat messages that are emitted concurrently from multiple chat rooms, it's important to know what the source chat room is so you can show it on screen or highlight it if it's a room of importance to the user.

Figure 8.8 When the SelectMany operator is applied to an observable of observables, the resultSelector will be invoked for each notification, together with the source item that created the observable it was emitted from.

Here's how to use SelectMany to add the ChatRoom identifier to the chat message ViewModel:

```C#
var rooms = roomsSubject.AsObservable() as IObservable<ChatRoom>;

rooms
    .Log("Rooms")
    .SelectMany(r => r.Messages,
                (room, msg) => 
                    new ChatMessageViewModel(msg) {Room = room.Id})
    .Subscribe(vm => AddToDashboard(vm));
```

Unlike enumerables, observables can emit asynchronous notifications, so the order in which the resultSelector is invoked is nondeterministic. This means that SelectMany will have to cache all the items from the source observable in order to pass them with each notification emitted by the observable they created. This is, of course, only until the observables have completed. Consequently, using SelectMany may affect the memory footprint of your application.

NOTE The SelectMany operator is powerful when adding asynchronous method invocations as part of the observable pipeline. Chapter 5 discusses this technique, along with other techniques for working with asynchronous operations.

## 8.3 Filtering an observable

Not all notifications emitted by an observable are meaningful to your application; therefore, they need to be filtered out of the observable sequence. These might be notifications whose values are higher than a specific threshold, or chat messages sent from a user you blocked, or news items already received from another news source.

### 8.3.1 Filtering with the Where operator

I've used the `Where` operator in almost every example in this book; it's one of the fundamental operators for most query languages. `Where` receives a predicate function that's invoked for each emitted value and returns a Boolean, which indicates whether the value is allowed to proceed in the pipeline and be observed by the observer. The `Where` operator, depicted in figure 8.9, is also known as the `Filter` operator.

Figure 8.9 The Where operator takes a predicate function and filters the elements of an observable sequence.

The next example filters an observable of strings, so only the strings that start with a capital A are emitted:

```C#
var strings = new[] {"aa", "Abc", "Ba", "Ac"}.ToObservable();

strings.Where(s => s.StartsWith("A"))
    .SubscribeConsole();
```

This produces the following output:

```
- OnNext(Abc)
- OnNext(Ac)
- OnCompleted()
```

The `Where` operator checks each emitted notification solely; it doesn't hold a view of the entire observable sequence generated before the current notification. This makes it harder to create predicate functions that make decisions based on past events. Therefore, it's the developer's responsibility to keep track of that information and to use it. For example, you may need a view of what happened previously if you need to get a distinct observable sequence (where each value is emitted at most once). Luckily, Rx provides this kind of operator for your use.

### 8.3.2 Creating a distinct sequence

The `Distinct` operator permits a restrictive policy on the resulting observable, so values appear only once in the sequence. If the observable emits news items that come from multiple news sources, but you want to see a news item only once, for example, the Distinct operator makes this easy to achieve, assuming the news item has an identifier. Figure 8.10 shows a marble diagram of Distinct.

Figure 8.10 The Distinct operator suppresses duplicate items emitted by an observable.

Note that the Distinct operator emits values as they come (and not when the source observable completes), unless they were already emitted:

```C#
var subject = new Subject<NewsItem>();
subject.Log()
    .Distinct(n=>n.Title)
    .SubscribeConsole("Distinct");
subject.OnNext(new NewsItem() {Title = "Title1"});
subject.OnNext(new NewsItem() {Title = "Title2"});
subject.OnNext(new NewsItem() {Title = "Title1"});
subject.OnNext(new NewsItem() {Title = "Title3"});
```

In the output of this program the lines prefixed with Distinct are emitted by the observable after removing the duplicates (with the Distinct operator):

```C#
- OnNext(Title1)
Distinct - OnNext(Title1)
- OnNext(Title2)
Distinct - OnNext(Title2)
- OnNext(Title1)
- OnNext(Title3)
Distinct - OnNext(Title3)
- OnCompleted()
Distinct - OnCompleted()
```

As you can see, the second time the Title1 news item is emitted, it's filtered out.

The Distinct operator has several overloads. If the emitted data type overrides the `Equals` method, you can leave the Distinct operator empty (with no arguments), and it'll check equality based on the implementation of Equals. Alternatively, you can provide an `EqualityComparer` that determines the equality between the items:

```C#
IObservable<TSource> Distinct<TSource>(this IObservable<TSource> source)
IObservable<TSource> Distinct<TSource>(this IObservable<TSource> source,
    IEqualityComparer<TSource> comparer)
IObservable<TSource> Distinct<TSource, TKey>(this IObservable<TSource> source,
    Func<TSource, TKey> keySelector)
IObservable<TSource> Distinct<TSource, TKey>(this IObservable<TSource> source, 
    Func<TSource, TKey> keySelector,
    IEqualityComparer<TKey> comparer)
```

NOTE In order for the Distinct operator to behave as expected, it must save the entire emitted distinct sequence internally. This affects the memory footprint of your application, so you must use it with care.

### 8.3.3 Removing duplicate contiguous values

Suppose you have a search form, and every time the user changes a search term, a search request is sent to the search service. Because the search request is expensive in terms of time and service load, you want to reduce the number of calls made if they're duplicate queries, as shown in figure 8.11.

Figure 8.11 To reduce load on the service, avoid sending the same term more than once contiguously.

The `DistinctUntilChanged` operator returns an observable sequence that emits only distinct contiguous elements. If the source observable emits the same element consecutively, the value is emitted only once (the first appearance) by the observable returned from DistinctUntilChanged. But, unlike Distinct, if the value is emitted again after other values are emitted in between, the value is emitted again. Figure 8.12 shows the marble diagram for this operator.

Figure 8.12 The DistinctUntilChanged operator filters consecutive duplicate items from the observable.

The next example uses `DistinctUntilChanged` to prevent the same search term from being sent to the search service if it has already been sent. To make this approach more realistic, I'm using another important operator called `Throttle`.

This operator emits a value only if a particular timespan has passed without another value being emitted. In this case, a search term is sent only if no other search term is provided within 400 milliseconds:

```C#
Observable.FromEventPattern(SearchTerm, "TextChanged")
    .Select(_ => SearchTerm.Text)
    .Throttle(TimeSpan.FromMilliseconds(400))
    .DistinctUntilChanged()
    .Subscribe(s => /* Sending the search term to the WebService */);

```

You can find a sample WPF application that uses this code at http://mng.bz/84bh. In the sample application, I added all the search terms to a list instead of querying a real web service. Figure 8.13 shows the output when I wrote Rx, Reactive and then wrote ReactiveX but deleted the X in less than 400 ms. Note that the list isn't cleared between search terms and, with each term, grows over time.

Figure 8.13 With DistinctUntilChanged, the word Reactive appears only once, even though it was provided twice.

Just like the `Distinct` operator, `DistinctUntilChanged` provides a few overloads that let you specify the way equality is determined by keySelector and/or EqualityComparer between values emitted by the observable:

```C#
IObservable<TSource> DistinctUntilChanged<TSource>(this IObservable<TSource> source,
    IEqualityComparer<TSource> comparer)
IObservable<TSource> DistinctUntilChanged<TSource, TKey>(this IObservable<TSource> source,
    Func<TSource, TKey> keySelector)
IObservable<TSource> DistinctUntilChanged<TSource, TKey>(this IObservable<TSource> source,
    Func<TSource, TKey> keySelector,
    IEqualityComparer<TKey> comparer)

```

The result of an Rx query isn't always an observable sequence of items. Occasionally, you may want a single result, such as the sum of the items or a count of them. The Rx aggregation operators let you do that.

## 8.4 Aggregating the observable sequence

Rx lets you take an observable sequence and reduce it to an aggregated value from the entire sequence or from the sequence up to the current point. This type of aggregation includes summing the sequence, averaging, and finding maximum and minimum values, depending on your aggregation algorithm.

### 8.4.1 Using basic aggregation operators

Those who are familiar with SQL and LINQ know that you can easily include basic aggregate functions in the query, and the underlying system will do the work for you. Rx operators let you utilize the same technique.

#### SUM

The `Sum` operator sums all values in the source observable sequence and emits the summation on the resulting observable when the source completes. Figure 8.14 shows a marble diagram of Sum.

Figure 8.14 The Sum operator calculates the sum of numbers emitted by an observable and then emits the sum.

The Sum operator supports the summation of all primitive number types (integers, floats, and so on), as well as their nullable forms, where the null values will be discarded. Here's an example that sums integers from an observable sequence containing the numbers 1 to 5:

```C#
Observable.Range(1, 5)
    .Sum()
    .SubscribeConsole("Sum");
```

The output is as follows:

```C#
Sum - OnNext(15)
Sum - OnCompleted()
```

You can see that sum (15) was emitted when the source observable completed. Using a selector function and overloads for the Sum operator, you can specify which operand to use for the summation. This allows you to select a subproperty of the emitted object. Here's the signature of the overload that accepts integers (int); the same signature exists for the other primitive types as well:

```C#
IObservable<int> Sum<TSource>(this IObservable<TSource> source,
    Func<TSource, int> selector)
```

#### COUNT

To count the number of values emitted by an observable, apply the `Count` operator, depicted in figure 8.15.

Figure 8.15 The Count operator counts the number of items emitted by the source observable and emits this value.

The observable returned from Count emits the count when the source observable completes:

```C#
Observable.Range(1, 5)
    .Count()
    .SubscribeConsole("Count");
```

The output is as follows:

```C#
Count - OnNext(5)
Count - OnCompleted()
```

The Count operator also lets you specify a predicate that determines which emitted value will be counted. This is equivalent to using a Where operator followed by the parameterless Count operator. This is how you count only the even numbers in an observable sequence:

```C#
Observable.Range(1, 5)
    .Count(x => x % 2 == 0)
    .SubscribeConsole("Count of even numbers");
```

Here's the output:

```C#
Count of even numbers - OnNext(2)
Count of even numbers - OnCompleted()
```

#### AVERAGE

The `Average` operator, illustrated in figure 8.16, creates an observable that emits the average of the values emitted from the source observable when it completes.

Figure 8.16 The Average operator calculates the average of numbers emitted by an observable and emits this average.

`Average` supports averaging all primitive number types (integers, floats, and so on), as well as their nullable forms, where the null values will be discarded:

```C#
Observable.Range(1, 5)
    .Average()
    .SubscribeConsole("Average");
```

The output is as follows:

```C#
Average - OnNext(3)
Average - OnCompleted()
```

Using a selector function and overloads for the Average operator, you can specify which operand to use for averaging. This allows you to select a subproperty of the emitted object. Here's the signature of the overload that accepts integers (int); the same signature exists for the other primitive types as well:

```C#
IObservable<double> Average<TSource>(this IObservable<TSource> source,
    Func<TSource, int> selector)
```

#### MAX AND MIN

The `Max` and `Min` operators let you find the maximum and minimum values in an observable sequence and emit them when it completes, as shown in figure 8.17. 

Figure 8.17 The Max operator emits the maximum value in an observable sequence. Here's an example of finding the maximal and minimal values:

```C#
Observable.Range(1, 5)
    .Max()
    .SubscribeConsole("Max");

Observable.Range(1, 5)
   .Min()
   .SubscribeConsole("Min");
```

The output is as follows:

```C#
Max - OnNext(5)
Max - OnCompleted()
Min - OnNext(1)
Min - OnCompleted()
```

.NET provides the default comparer that Max and Min use for your data type; however, if the default comparison condition isn't suitable for your needs, you can provide an IComparer and/or a selector function. The following shows the list of overloads for the Max operator; the Min operator provides the same overloads:

```C#
IObservable<TSource> Max<TSource>(this IObservable<TSource> source,
    IComparer<TSource> comparer)
IObservable<TResult> Max<TSource, TResult>(this IObservable<TSource> source,
    Func<TSource, TResult> selector)
IObservable<TResult> Max<TSource, TResult>(this IObservable<TSource> source,
    Func<TSource, TResult> selector,
    IComparer<TResult> comparer)
```

Note that the values returned by the selector are the ones from which the maximum/minimum values will be chosen and subsequently emitted. The source item (the containing object, for example) producing the values won't be emitted.

If you have an observable sequence of students' grades, and you want to find the student with the maximum grade, for example, the Max operator won't help because you receive the maximum grade only as a number and not the contained object. This is shown in the following example:

```C#
var grades = new Subject<StudentGrade>();
grades.Max(g => g.Grade)
    .SubscribeConsole("Maximal grade");

grades.OnNext(new StudentGrade() {Id = "1",Name = "A", Grade = 85.0});
grades.OnNext(new StudentGrade() {Id = "2",Name = "B", Grade = 90.0});
grades.OnNext(new StudentGrade() {Id = "3",Name = "C", Grade = 80.0});
grades.OnCompleted();
```

This example generates the following output:

```C#
Maximal grade - OnNext(90)
Maximal grade - OnCompleted()
```

As you can see, the Max operator (and selector) emits the value 90 and not the StudentGrade object that contained the maximum values. If you want to print the name of the student with the maximum grade, you won't be able to do that. To reach the behavior you want (emitting the maximum/minimum object and not just the maximum/minimum value), you need to use the `MaxBy` or `MinBy` operator.

### 8.4.2 Finding the maximum and minimum items by condition

The operators MaxBy and MinBy let you search an observable sequence to find the items containing the maximum and the minimum values, respectively, and then emit that value when the search completes, as shown in figure 8.18. You set the maximum or minimum values by invoking a keySelector function on each item emitted by the source observable.

Figure 8.18 The MaxBy operator, based on the values provided by the keySelector function, emits the maximum value as an item when the source observable completes.

Because multiple items might have the same maximum or minimum value, the operators MaxBy or MinBy return an observable of lists:

```C#
IObservable<IList<TSource>> MaxBy<TSource, TKey>(this IObservable<TSource> source,
    Func<TSource, TKey> keySelector)
```

The following example finds the StudentGrade object that has the maximum Grade property:

```C#
var grades = new Subject<StudentGrade>();
grades.MaxBy(s => s.Grade)
    .SelectMany(max => max)
    .SubscribeConsole("Maximal object by grade");

grades.OnNext(new StudentGrade() { Id = "1", Name = "A", Grade = 85.0 });
grades.OnNext(new StudentGrade() { Id = "2", Name = "B", Grade = 90.0 });
grades.OnNext(new StudentGrade() { Id = "3", Name = "C", Grade = 80.0 });
grades.OnCompleted();
```

After running the example, this is the output:

```C#
Maximal object by grade - OnNext(Id: 2, Name: B, Grade: 90)
Maximal object by grade - OnCompleted()
```

This example and its output show that you succeeded in finding the student object with the maximum grade—student B with grade 90.

### 8.4.3 Writing your aggregation logic with Aggregate and Scan

In Rx, you can create your own aggregation functions and apply them to an observable sequence. The aggregate functions are invoked for each item that's emitted, together with the aggregated value up to that point. The computed value is the input for the next invocation with the next item.

You can use two operators with aggregate functions:

  - Aggregate—Applies a function to each item emitted by an observable, and then emits the computed value upon the source observable completion.

  - Scan—Applies a function to each item emitted by a sequential observable, and then emits each successive value.

Figure 8.19 depicts the Aggregate operator, and figure 8.20 depicts the Scan operator.

Figure 8.19 The Aggregate operator applies a function to each item emitted by an observable, and then emits the computed value upon the source observable completion.

Figure 8.20 The Scan operator applies a function to each item emitted by sequential observables and emits each successive value.

Here's an example that creates the multiplication of all values in an observable sequence, first with Aggregate and then with Scan:

```C#
Observable.Range(1, 5)
    .Aggregate(1,
              (accumulate, currItem) => accumulate * currItem)
    .SubscribeConsole("Aggregate");

Observable.Range(1, 5)
    .Scan(1,
          (accumulate, currItem) => accumulate * currItem)
    .SubscribeConsole("Scan");
```

This is the output produced:

```C#
Aggregate - OnNext(120)
Aggregate - OnCompleted()
Scan - OnNext(1)
Scan - OnNext(2)
Scan - OnNext(6)
Scan - OnNext(24)
Scan - OnNext(120)
Scan - OnCompleted()
```

In this example, the input observable emits a sequence of values 1 to 5, the Aggregate operator emits the factorial value 120, and Scan emits the factorials 1! 2! 3! 4! 5!

Beside the seed value and the accumulator function, the Aggregate operator provides an overload that you can use to pass a resultSelector function that's invoked for the last value—the aggregate result:

```C#
IObservable<TResult> Aggregate<TSource, TAccumulate, TResult>(this IObservable<TSource> source,
    TAccumulate seed,
    Func<TAccumulate, TSource, TAccumulate> accumulator,
    Func<TAccumulate, TResult> resultSelector)
```

Suppose you need to retrieve the second largest item in an observable. Instead of creating your own variables to store the relevant state and use them inside the aggregate operators by capturing them as closures (mutating state as part of the observable is usually a code smell), you can use the Aggregate operator to encapsulate the mutated state for you. The next example retains the two largest items emitted by an observable (so far) in a sorted collection. And, when the observable is complete, it emits the second largest item.

Listing 8.2 Creating observable with Aggregate operator emitting second-largest item

```C#
var numbers = new Subject<int>();
numbers.Aggregate(
    new SortedSet<int>(),
    (largest, item) => {
        largest.Add(item);
        if (largest.Count > 2)
        {
            largest.Remove(largest.First());
        }
        return largest;
    },
    largest => largest.FirstOrDefault())
    .SubscribeConsole();
numbers.OnNext(3);
numbers.OnNext(1);
numbers.OnNext(4);
numbers.OnNext(2);
numbers.OnCompleted();
```

This example uses an empty `SortedSet` as a seed. This class helps keep the items sorted and ensures that there won't be duplicate items in the set. For each item emitted from the source observable, the accumulator function adds an item to the set, and then compacts it to hold two items at most.

When the source observable is complete, the resultSelector function takes the first item in the set (if it exists) and returns it. Because SortedSet is sorted, and you want to make sure there will be two items at most, the first item is the second greatest that you want to find.

The output from the preceding example is shown here:

```C#
- OnNext(3)
- OnCompleted()
```

Using Aggregate and Scan allows you to create your own powerful aggregation functions. In a way, they're the reactive equivalent to a loop, which you would've used for collections in order to produce a single value from it.

## 8.5 Summary

The querying abilities that Rx provides are rich and extensive. This can sometimes be overwhelming and complex to understand, so I made this chapter easy to digest in order to teach you the fundamentals of writing an Rx query by using some of the most used operators. Here's what I covered:

  - The `Select` operator transforms the emitted notification to another form. This includes taking only a subproperty or creating a new object (for example, a ViewModel).

  - Observables emit other observables or other collections (or items that contain them). The `SelectMany` operator merges the inner observables (or collections) to a flat stream; pass a collectionSelector function and you're done.

  - The `SelectMany` operator also takes a resultSelector function that you can call for every emitted item, together with its source item.

  - `SelectMany` is the power force behind the `Let` operator that you can use when manually writing a query.

  - The `Where` operator filters emitted notifications, which receive a predicate function to test each notification.

  - The `Distinct` operator gets an observable of distinct items.

  - The `DistinctUntilChanged` operator gets an observable of distinct consecutive items.

  - Rx provides the common statistical aggregation functions that you can apply to an observable. These are `Sum`, `Count`, `Average`, `Max`, and `Min`.

  - The `MaxBy` and `MinBy` operators get the maximum and minimum item, respectively, based on a subproperty.

  - You can use the `Aggregate` and `Scan` operators to customize the aggregation logic.

  - The `Aggregate` operator emits the aggregated result only when the source observable completes.

  - The `Scan` operator emits a sequential aggregated result each time a notification is emitted by the source observable.

In this chapter, we dealt only with operators that act in the scope of a single observable. The next chapter describes the operators used to break the observable into finer observables (groups) and to combine multiple observables.


