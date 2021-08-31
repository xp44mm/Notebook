# Why Ramda?



When [buzzdecafe](http://buzzdecafe.github.io/) recently [introduced](http://buzzdecafe.github.io/code/2014/05/16/introducing-ramda) [Ramda](https://github.com/ramda/ramda) to the world, there were two distinct groups of responses. Those accustomed to functional techniques — in Javascript or in other languages — mostly responded with, "Cool". They may have been excited by it or just casually noting another potential tool, but they understood what it was for.

The second group responded with a resounding, "Huh?"

To those not used to functional programming, Ramda seems to serve no purpose whatsoever. Most of its major capabilities are already covered by libraries like [Underscore](https://github.com/jashkenas/underscore) and [LoDash](https://github.com/lodash/lodash).

These folks are right. If you want to keep coding with the same imperative and object-oriented styles you've been using, Ramda does not have much to offer you.

However, it does offer a different style of coding, a style that's taken for granted in purely functional programming languages: Ramda makes it simple for you to build complex logic through functional composition. Note that any library with a `compose` function will allow you do functional composition; the real point here is: *"makes it simple"*.

Let's see how that works in Ramda.

"TODO lists" seem to be the big point of comparison for web frameworks, so we'll use them for our purposes, too: Let's start by imagining we want to be able to filter a TODO list to remove all the completed items.

With the built-in Array prototype methods, we might do something like this:

```js
// Plain JS
let incompleteTasks = tasks.filter(function(task) {
    return !task.complete;
});
```

With LoDash, it would be a bit simpler:

```js
// Lo-Dash
let incompleteTasks = _.filter(tasks, {complete: false});
```

In either case, we get a filtered list of tasks.

In Ramda, we might do it like this:

```js
let incomplete = R.filter(R.where({complete: false});
```

(**Update**: the `where` function has since been [split into two](https://github.com/ramda/ramda/pull/1036): [`where`](http://ramdajs.com/docs/#where) and [`whereEq`](http://ramdajs.com/docs/#whereEq), and that code won't quite work as stands.)

Do you notice something missing? There's no mention of the list of tasks. This Ramda code just gives us a function.

We'd still have to call it with the list of tasks in order to get the filtered set.

And that's the point.

Because we now have a function we can easily combine it with others to operate on whatever sets of data we choose. Imagine we had a function `groupByUser` that grouped the TODO items by user. Then we could simply create a new function:

```js
let activeByUser = R.compose(groupByUser, incomplete);
```

which selects the incomplete tasks and groups them by user.

Or it would, if we ever got around to supplying it with data, because, again, this is simply a function. If we were to write it out by hand, it might look something like this:

```js
// (if created by hand)
let activeByUser = function(tasks) {
    return groupByUser(incomplete(tasks));
};
```

That we don't have to do it by hand is the point of composition. And composition is one key technique of functional programming. Let's see what happens if we carry it a little further. What if we then need to sort each of these users' TODO lists by due date?

```js
let sortUserTasks = R.compose(R.map(R.sortBy(R.prop("dueDate"))), activeByUser);
```

## All in one?

The observant reader might have noticed that we could combine all the above. Since our `compose` function allows more than two parameters, why not do all of this in a single step?

```js
let sortUserTasks = R.compose(
    R.mapObj(R.sortBy(R.prop('dueDate'))),
    groupByUser,
    R.filter(R.where({complete: false})
);
```

My answer is that this might be reasonable if you have no other call for the intermediate functions `activeByUser` and `incomplete`. But it can make debugging harder, and it doesn't really add much to code readability.

In fact, I'd argue that we should go in the other direction. We used a fairly complicated section internally that might itself be reusable. Perhaps we'd be better off if we did this:

```js
let sortByDate = R.sortBy(R.prop('dueDate'));
let sortUserTasks = R.compose(R.mapObj(sortByDate), activeByUser);
```

Now we could use `sortByDate` to sort any collection of tasks by due date. (In fact, it's more flexible than that; it will sort any collection of objects containing sortable "dueDate" properties.)

Oh, but wait, did someone say we should be sorting dates descending?

```js
let sortByDateDescend = R.compose(R.reverse, sortByDate);
let sortUserTasks = R.compose(R.mapObj(sortByDateDescend), activeByUser);
```

If we knew for certain that we only ever wanted to sort by most recent date first, we could combine these into a single definition of `sortByDateDescend`. I personally would keep both around in case I decide to sort the data in either ascending or descending order. But that's up to you.

## Where's the Data?

We **still** don't have any data. What's going on here? Data processing without the data is just... well, processing. I'm afraid you're going to have to be patient. When you work with functional programming, all you get is functions forming a pipeline. One function feeds data to the next, which feeds it to the next, and so on until the results you need flow out the end.

What we've built so far is this collection of functions:

```haskell
incomplete: [Task] -> [Task]
sortByDate: [Task] -> [Task]
sortByDateDescend: [Task] -> [Task]
activeByUser: [Task] -> {String: [Task]}
sortUserTasks: {String: [Task]} -> {String: [Task]}
```

And although we've used the earlier functions to build up `sortUserTasks`, they are all potentially useful on their own. We did gloss over thing, though. I only asked you to imagine we had the function `byUser` in order to build `activeByUser`; we didn't actually see it. Did I sneak that by? Or did you notice? How would we build that one?

Here's one technique:

```js
let groupByUser = R.partition(R.prop('username'));
```

The `partition` function uses Ramda's version of `reduce`, one that is very similar to the one on `Array.prototype.reduce`. We're not going to discuss this further here. Our `partition` simply uses `reduce` to group a list into sublists that share the same key, as determined by a function run against each one of them, in this case `prop('username')`, which simply extracts the "username" property from each item.

(So, did I manage to distract you with the shiny new function? I'm still not mentioning the data here! Sorry. And look, a few more shiny new functions are coming up!)

## But Wait, There's More

We can carry this on as far as we like. If we want to choose the top five elements from a list we could use the Ramda function `take`. So to get the first five elements from each task for each user, we could do this:

```js
let topFiveUserTasks = R.compose(R.mapObj(R.take(5)), sortUserTasks);
```

(Anyone else thinking [Brubeck and Desmond](http://en.wikipedia.org/wiki/Take_Five) here?)

Then we could reduce the returned object to just a subset of the properties, say the title and the due date. Username is obviously redundant in this data structure, and perhaps the others are simply overhead we don't want to pass through to other systems.

This we could do with Ramda's analog to the SQL `select` function, one called `project`:

```js
let importantFields = R.project(['title', 'dueDate']);
let topDataAllUsers = R.compose(R.mapObj(importantFields), topFiveUserTasks);
```

Some of the functions we've created along the way seem genuinely reusable for other purposes inside a TODO application. Others are perhaps only placeholders that could be combined into the major ones. So if we were to revisit now, perhaps we might combine the code like this:

```js
let incomplete = R.filter(R.where({complete: false}));
let sortByDate = R.sortBy(R.prop('dueDate'));
let sortByDateDescend = R.compose(R.reverse, sortByDate);
let importantFields = R.project(['title', 'dueDate']);
let groupByUser = R.partition(R.prop('username'));
let activeByUser = R.compose(groupByUser, incomplete);
let topDataAllUsers = R.compose(R.mapObj(R.compose(importantFields, 
    R.take(5), sortByDateDescend)), activeByUser);
```

## All Right, Already! May I See Some Data?

Yes. Yes you may.

Now is the time to pass data into our functions. But the point is that these functions all accept the same sort of data, an array of TODO items. We haven't specifically described the structure of those items, but we do know that they must have at least the following properties:

- `complete`: Boolean
- `dueDate`: String, formatted YYYY-MM-DD
- `title`: String
- `userName`: String

So, if we have an array of tasks, how do we use it? Simply:

```js
let results = topDataAllUsers(tasks);
```

That's it?

All that build-up, and that's it?

I'm afraid so. The results will be an object something like:

```js
{
    Michael: [
        {dueDate: '2014-06-22', title: 'Integrate types with main code'},
        {dueDate: '2014-06-15', title: 'Finish algebraic types'},
        {dueDate: '2014-06-06', title: 'Types infrastucture'},
        {dueDate: '2014-05-24', title: 'Separating generators'},
        {dueDate: '2014-05-17', title: 'Add modulo function'}
    ],
    Richard: [
        {dueDate: '2014-06-22', title: 'API documentation'},
        {dueDate: '2014-06-15', title: 'Overview documentation'}
    ],
    Scott: [
        {dueDate: '2014-06-22', title: 'Complete build system'},
        {dueDate: '2014-06-15', title: 'Determine versioning scheme'},
        {dueDate: '2014-06-09', title: 'Add `mapObj`'},
        {dueDate: '2014-06-05', title: 'Fix `and`/`or`/`not`'},
        {dueDate: '2014-06-01', title: 'Fold algebra branch back in'}
    ]
}
```

But here's an interesting thing. You can also pass that same initial list of tasks into `incomplete` and get a filtered list:

```js
let incompleteTasks = incomplete(tasks);
```

Perhaps this might return something like the following:

```js
[
    {
        username: 'Scott',
        title: 'Add `mapObj`',
        dueDate: '2014-06-09',
        complete: false,
        effort: 'low',
        priority: 'medium'
    }, {
        username: 'Michael',
        title: 'Finish algebraic types',
        dueDate: '2014-06-15',
        complete: false,
        effort: 'high',
        priority: 'high'
    } /*, ... */
]
```

And, of course, you could also pass the list of tasks to `sortBydate`, to `sortByDateDescend`, to `importantFields`, to `byUser`, or to `activeByUser`. Because these all operate on similar types — an array of tasks — we can build up an large collection of tools just through simple combinations.

## New Requirements

Late in the game, you've just learned that you need to support another feature. You need to filter the tasks down to just those for a specific user, then run the same sort of filtering, sorting, and subsetting for that one user that you did earlier for the mapping of usernames to task lists.

This logic is currently embedded in `topDataAllUsers`... which probably shows us that our combining of functions was too aggressive. But it's quite easy to refactor that. As is often the case, the hardest thing is to come up with a good name. "`gloss`" probably isn't it, but it's late at night, and it's the best I can do:

```js
let gloss = R.compose(importantFields, R.take(5), sortByDateDescend);
let topData = R.compose(gloss, incomplete);
let topDataAllUsers = R.compose(R.mapObj(gloss), activeByUser);
let byUser = R.use(R.filter).over(R.propEq("username"));
```

Then when you want to use it, you can call

```js
let results = topData(byUser('Scott', tasks));
```

## I Just Want My Data, Thanks

"Okay," you say, "maybe that's cool, but for now, I really just want my data. I don't want functions that will one day return my data... perhaps... maybe. Can I even *use* Ramda?"

Of course you can.

Let's return to that very first function:

```js
let incomplete = R.filter(R.where({complete: false}));
```

How do we turn this into something which just gets the data? It's very simple:

```js
let incompleteTasks = R.filter(R.where({complete: false}), tasks);
```

And the same is true of all the other major functions: just add a `tasks` parameter to the end of the call, and you get data back.

## What Just Happened?

This is another major point of Ramda. All the key functions of Ramda are automatically curried. This means that if you don't supply all the parameters the function is expecting, instead of trying to call the function, we return you a new function that is expecting the remaining ones. So the definition of `filter` involves the array of values as well as the predicate function used to filter them. In the initial version, we didn't supply the values, so `filter` simply returned a new function that was looking for that array. In the second version, we did pass the array, and it was used together with the predicate to calculate the response.

The auto-currying of Ramda's functions combine with it's unswerving function-first, data-last API design is what makes Ramda so easy to use for this style of functional composition.

But the details of currying in Ramda are material for another article: [Favoring Curry](http://fr.umio.us/favoring-curry/). In the meantime, it's definitely worth reading Hugh Jackson's excellent post, [Why Curry Helps](http://hughfdjackson.com/javascript/why-curry-helps/).
