# 3 Functional thinking in C#

The object-oriented paradigm offers great productivity in application development. It makes projects more manageable by decomposing complex systems into classes, and objects are silos that you can concentrate on separately. Yet other paradigms have gathered attention in recent years, especially the functional programming paradigm. Functional programming languages, together with a functional way of thinking, greatly influenced the way Rx was designed and used. Specifically, the functional programming concepts of anonymous types, first-order functions, and function composition are an integral part of Rx that you’ll see used heavily throughout the book and in your everyday work with Rx.

Some of the attention functional programming is receiving arises from functional languages being good for multithreaded applications. Rx excels in creating asynchronous and concurrent processing pipelines, which were also inspired by functional thinking. Although C# is considered an object-oriented language, it has evolved over the years and added aspects that exist in functional programming.

.NET even has a functional programming language of its own, named F#, which runs on top of the Common Language Runtime (CLR) in the same way C# does. Using functional programming techniques can make your code cleaner and easier to read and can change the way you think when writing code, eventually making you more productive. Rx is highly influenced by the functional way of thinking, so it’s good to understand those concepts in order to more easily adopt the Rx way of thinking. The concepts in this chapter may not be new to you. You can skip to the next chapter if you wish, but I encourage you to at least briefly review the concepts to refresh your memory.

## 3.1 The advantages of thinking functionally

As computer science evolves, new languages appear with new concepts and techniques. All these languages share the same underlying purpose: improving developer productivity and program robustness. Productivity and robustness have many faces: shorter code, readable statements, internal resource management, and so on. Functional programming languages also try to achieve those goals. Although many types of functional programming languages exist, each with unique characteristics, we can see their similarities:

  * Declarative style of programming—This is based on the concept of “Tell what, not how.”

  * Immutability—Values can’t be modified; instead, new values are created.

  * First-class functions—Functions are the primary building block used.

With object-oriented languages, developers think about programs as a collection of objects that interact with each other. This allows you to create modular code by encapsulating data and behavior that relates to the data in an object. This modularity, again, improves the productivity of the developer and makes your program more robust because it’s easier for the developer to understand the code (less detail to remember) and concentrate effort on a specific module when writing new code or changing (or fixing) existing ones.

### 3.1.1 Declarative programming style

In the first two chapters, you saw examples of the declarative programming style. In this style, you write your program statements as a description of what you want to achieve as the result instead of specifying how you want this to be done. It’s up to the environment to figure out how to do it best. Consider this example of an English statement in an imperative style (the how) and declarative style (the what):

  * Imperative—For each customer in the list of customers, take the location and print the city name.

  * Declarative—Print the city name of every customer in the list. 

A declarative style makes it easier to grasp the code you write, which leads to better productivity and usually makes your code less error prone. The next code block shows another example of the declarative programming style, this time using HTML that produces what you see in figure 3.1.

Figure 3.1 A simple web page that has a title, a heading, and a paragraph

To create this page, you don’t have to write the rendering logic or the layout management and set the position for each element. Instead, all you have to do is to write this short HTML script:

```html
<html>
<head>
    <title>this is the page title</title>
</head>
<body>
    <h1>This is a Heading</h1> 
    <p>This is a paragraph.</p>
</body>
</html>
```

Even if you don’t know HTML, it’s easy to see that this example only declares the outcome you want to see and doesn’t deal with the technical details of making the browser do it. With a declarative language such as HTML, you can create a complex page with little effort. You want to attain the same results with the C# code you write, and you’ll see examples of that in the rest of this chapter. One thing you need to pay attention to is that because you’re indicating the what and not the how, how can you know what will happen to the system? Could there be side effects? It turns out that functional programming solves this problem from the start, as you’ll see next.

### 3.1.2 Immutability and side effects

Consider this method, which prints a message to the console:

```C#
static void WriteRedMessage(string message)
{
    Console.ForegroundColor = ConsoleColor.Red;
    Console.WriteLine(message);
}
```

This short method causes a side effect to the program: the method changes a shared state in the system, the console color in this case. Side effects can come in different flavors and are sometimes hidden inside the code. If you change the method signature and remove the word Red from the method name, as shown in the following code sample, the side effect still happens:

```C#
static void WriteMessage(string message)
```

But now it’s far more difficult to predict that this method will cause the side effect of changing the color of the console output. Side effects aren’t limited to console color, of course; they also include changes to a shared object state, such as a list of items that’s modified (as you saw in the previous chapter). Side effects can cause all kinds of bugs in your code—for example, in concurrent execution the code is reached from two places (like threads) at the same time and leads to race conditions. Side effects can also cause your code to be harder to track and predict, thus making it harder to maintain.

Functional programming languages solve the side-effect problem by preventing it in the first place. In functional programming, every object is immutable. You can’t modify the object state. After the value is set, it never changes; instead, new objects are created. This concept of immutability shouldn’t be new to you. Immutability exists in C# as well, such as in the type string. For example, try to answer what this next program will print:

```C#
string bookTitle = "Rx.NET in Action";
bookTitle.ToUpper();
Console.WriteLine("Book Title: {0}", bookTitle); 
```

If your answer is Rx.NET in Action, you’re correct. In C#, strings are immutable. All the methods that transform a string’s content don’t really change it; instead, they create a new string instance with the modifications. The previous example should’ve been written like this:

```C#
string bookTitle = "Rx.NET in Action";
string uppercaseTitle = bookTitle.ToUpper();
Console.WriteLine("Book Title: {0}", uppercaseTitle); 
```

This version of the code stores the result of the ToUpper call in a new variable. This variable holds a different string instance with the uppercase value of the book title.

The immutability implies that calling a function ends only with the function computing its result, without any other effect that the programmer needs to worry about. This takes away a major source of bugs and makes the functions idempotent—calling a function with the same input always ends with the same result, no matter whether it was applied once or multiple times.

**A WORD ABOUT CONCURRENCY**

The idempotency that you get from the immutability makes the program deterministic and predictable and makes the order in which execution happens irrelevant, making it a perfect fit for concurrent execution.

Writing concurrent code is hard, as you saw in the previous chapter. When you take sequential code and try to run it in parallel, you could find yourself facing bugs. With side effect-free and immutable code, this problem doesn’t exist. Running the code from different threads won’t cause any synchronization issues, because you have nothing to synchronize. Knowing that using functional programming languages makes writing concurrent applications easier, it’s no wonder that functional programming languages started to gain more interest in recent years and became the de facto community choice for building large-scale concurrent applications. Just to name a few, companies such as Twitter, LinkedIn, and AT&T are known to use functional programming in their systems.

### 3.1.3 First-class functions

The name functional programming is used because a function is the basic construct that you work with. It’s one of the language primitives, like an int or a string, and similar to other primitive types, the function is a first-class citizen, which means it can be passed as an argument and be returned from a function. Here’s an example in F# (a functional programming language that’s part of .NET):

```F#
let square x = x * x
let applyAndAdd f x y = f(x) + f(y)
```

Here we define a function, square, that calculates the square of its argument. We then define a new function, applyAndAdd, that takes three arguments: the first argument is a function that’s applied to the two other arguments, and then the results are summed. 

---

**NOTE** If you find this confusing, don’t worry. Read the rest of the chapter and then come back and read this short section again.

---

When you call the applyAndAdd function and pass the square function as the first argument together with two numbers as the other arguments, you get the sum of two squares. For example, applyAndAdd square 5 3 outputs the number 34, as shown in figure 3.2.

Figure 3.2 In functional programming languages, functions can be passed as arguments. This is the expression tree of the call to applyAndAdd f x y with f: square, x: 5, y: 3.  

Functions that receive functions as their arguments or return functions as their return values are called higher-order functions. With higher-order functions, you can compose new functions and add new behaviors to existing functions by changing the inner function they use, as you did in the applyAndAdd example. This way, you can extend the “language” that your code uses and adapt it to your domain.

### 3.1.4 Being concise

The core functional programming concepts mentioned earlier in the chapter serve the same purpose that makes functional thinking a powerful tool you should embrace: making your code concise and short.

Writing declaratively means that you can hide the complexity required to achieve a result and instead focus on the result that you want to achieve. This is done using the compositional nature of first-class and higher-order functions that create the glue between the various parts of your code. The expressiveness of your program is better achieved when you know that no side effects will arise and cause uncertainty in the outcome of the execution. Working with an immutable data structure enables you to be certain that the function will always end the same predictable way.

Writing code that’s concise makes you more productive when creating new code or when changing existing code, even if it’s new to you. Figure 3.3 displays the key elements for productivity.

```
Functional thinking makes you productive:
    Declarative code
    Fewer lines of code
    Reduction of errors in code
    Predictable code
```

Figure 3.3 The key benefit of functional programming is that it makes you more productive. The key elements for productivity are illustrated here.

The key elements shown in figure 3.3 are where the true benefits of functional programming lie. It’s important for you to know that so you can achieve the same advantages when writing programs in C#.

## 3.2 First-class and higher-order functions using delegates and lambdas

When C# was introduced in 2002, it was possible to make “function pointers” that you could pass as arguments and hold as class members. These function pointers are known as delegates. Over the years, C# became a multi-paradigm language that supports not only object-oriented programming but also event-driven programming or simple procedural programming. Functional programming also started to influence language as the years went by, and delegates became the underlying mechanism to support functions as first-class citizens in the language.

### 3.2.1 Delegates

In C#, a delegate is a type that represents references to methods. Delegates are most commonly used with .NET events, but in this chapter, you’ll see how to use them to spice up code with functional programming techniques.

The delegate type is defined with the exact signature of the methods you want the delegate to reference. For example, if you want to create a reference to methods that receive two string parameters and return a bool, figure 3.4 shows how to define the delegate type.

```C#
public delegate bool ComparisonTest (string first, string second);
```

Figure 3.4 Declaration of a delegate type for methods that receive two strings and return an integer

After creating the delegate type, you can reference methods with the same signature by creating a new instance of the delegate and passing the method you want to reference:

```C#
ComparisonTest delegateInstance = new ComparisonTest( <the method> ); 
```

Say you have a class that holds different methods that compare strings:

```C#
class StringComparators
{
    static bool CompareLength(string first, string second)
    {
        return first.Length == second.Length;
    }
    bool CompareContent(string first, string second)
    {
        return first == second;
    }    
}
```

You can then use your delegate type to reference the comparison methods:

```C#
string s1 = "Hello";
string s2 = "World";
var comparators = new StringComparators();
ComparisonTest test = new ComparisonTest(comparators.CompareContent);  
Console.WriteLine("CompareContent returned: {0}", test(s1, s2));
test = new ComparisonTest(StringComparators.CompareLength);       
Console.WriteLine("CompareLength returned: {0}", test(s1, s2));
```

The sample output from the previous code is as follows:

CompareContent returned: False
CompareLength returned: True

Beginning with C# 2.0 it’s much easier to create delegates. You can simply assign the method to the delegate variable (or parameter):

ComparisonTest test2 = comparators.CompareContent;

With delegates, you can make something similar to the higher-order functions that functional programming languages have. The next method checks whether two string arrays are similar by traversing the items in both collections and checking them against each other using a comparison function that was passed to a delegate reference.

Listing 3.1 AreSimilar method uses a delegate as a parameter type

```C#
bool AreSimilar(string[] leftItems, string[] rightItems, ComparisonTest tester)
{
    if (leftItems.Length != rightItems.Length)
        return false;                         
    for (int i = 0; i < leftItems.Length; i++)
    {
        if (tester(leftItems[i],rightItems[i]) == false)  
        {
            return false;
        }
    }
    return true;
}
```

The method receives the two arrays and calls the tester on every two corresponding items to check whether they’re similar. The tester is referencing a method that was sent as an argument. Here you’re calling the AreSimilar method and passing the CompareLength method as an argument:

```C#
string[] cities = new[] { "London", "Madrid", "TelAviv" };
string[] friends = new[] { "Minnie", "Goofey", "MickeyM" };
Console.WriteLine("Are friends and cities similar? {0}", 
            AreSimilar(friends,cities, StringComparators.CompareLength));
```

The output result for this sample is as follows:

```
Are friend and cities similar? True
```

### 3.2.2 Anonymous methods

The problem with delegates as you’ve seen them so far is that they force you to write a method in a class—this is called a named method. This burden slows you down and therefore hurts your productivity. Anonymous methods are a feature in C# that enable you to pass a code block as a delegate value:

```C#
ComparisonTest lengthComparer = delegate (string first, string second)
{
    return first.Length == second.Length;
};
Console.WriteLine("anonymous method returned: {0}", 
                   lengthComparer("Hello", "World"));
```

The anonymous method can also send the code block as an argument:

```C#
AreSimilar(friends, cities, 
           delegate (string s1, string s2) { return s1 == s2; });
```

Anonymous methods make it far easier to create higher-order functions in your C#program and reuse existing code, as with the AreSimilar method in the previous example. You can use the method over and over and pass different comparison methods, improving the extendibility of your program.

**CLOSURES (CAPTURED VARIABLES)**

Anonymous methods are created within a scope, such as a method scope or a class scope. The code block of your anonymous method can access anything that’s visible to it in that scope—variables, methods, and types, to name a few. An an example:

```C#
int moduloBase = 2;
var similarByMod=AreSimilar(friends, cities, delegate (string s1, string s2)
{
    return ((str1.Length % moduloBase) == (str2.Length % moduloBase));
});
```

Here the anonymous method uses a variable that’s declared in the outer scope. The variable moduloBase is called a captured variable, and its lifetime now spans the lifetime of the anonymous method that uses it.

 The anonymous method that uses captured variables is called a closure. Closures can use the captured variable even after the scope that created it has completed:

```C#
ComparisonTest comparer;
{                                               
    int moduloBase = 2;
    comparer = delegate (string s1, string s2)
    {
        Console.WriteLine("the modulo base is: {0}", moduloBase);
        return ((s1.Length % moduloBase) == (s2.Length % moduloBase));
    };
    moduloBase = 3;         
}
var similarByMod = AreSimilar(new[] { "AB" }, new[] { "ABCD" }, comparer);
Console.WriteLine("Similar by modulo: {0}", similarByMod);
```

When running this example, you get this interesting output 

```
the modulo base is: 3
Similar by modulo: False
```

The anonymous method was created in a scope that’s different from the scope that uses it, but the anonymous method still has access to a variable declared in that scope. Not only that, but the value that the anonymous method sees is the last one that the variable was holding. This leads to a powerful observation:

The value of a captured variable that a closure uses is evaluated at the time of the method execution and not at the time of declaration.

Captured variables can cause confusion from time to time, so consider this example and try to determine what will print:

```C#
delegate void ActionDelegate();               
var actions = new List<ActionDelegate>();
for (var i = 0; i < 5; i++)
{
    actions.Add(delegate () { Console.WriteLine(i); });  
}
foreach (var act in actions) act();       
```
          
The output of this example might not be what you expected. Instead of printing the numbers 0 to 4, this code prints the number 5 five times. This is because when each action is executed, it reads the value of i, and the value of i is the value it received in the last iteration of the loop, which is 4. 

### 3.2.3 Lambda expressions

To make it even simpler to create anonymous methods, you can use the lambda expression syntax introduced in C# 3.0. Lambda expressions enable you to create anonymous methods that are more concise and more closely resemble the functional style.

Here’s an example of an anonymous method written as both anonymous method syntax and with lambda expressions:

```C#
ComparisonTest x = (s1,s2) => s1==s2 ;      
ComparisonTest y = delegate (string s1,string s2) { return s1 == s2; };  
```

The lambda expression is written as a parameter list, followed by =>, which is followed by an expression or a block of statements.

If the lambda expression receives only one parameter, you can omit the parentheses:

```C#
x => Console.WriteLine(x);
```

The lambda expression also uses type inference of the parameters. You can, however, write the types of the parameters explicitly:

```C#
ComparisonTest x = (string s1, string s2) => s1==s2 ;
```

Typically, you want your lambda expression to be short and concise, but it’s not always possible to have only one statement in your lambda expression. If your lambda contains more than one expression, you need to use curly braces and write the return statement explicitly in case it needs to return a value:

```C#
() =>                                        
{
    Console.WriteLine("Hello");
    Console.WriteLine("Lambdas");
    return true;                               
};
```

Lambda expressions are used heavily with Rx because they make your processing pipeline short and expressive, which is cool! But you still have the requirement to create new delegate types each time you want to specify a method signature for the method types to which you want to receive a reference, which is far from ideal. That’s why you usually won’t create new delegate types but instead use `Action` and `Func`.

### 3.2.4 Func and Action

A delegate type is a way to enforce the method signatures you want to receive as a parameter or set as a variable. Most of the time, you’re not interested in creating a new type of delegate to enforce that constraint; you only want to state what you’re expecting. For example, the following two delegate types are the same except for the names used:

```C#
delegate bool NameValidator(string name);
delegate bool EmailValidator(string email);
```

Because the two delegate types definitions are the same, you can set both to the same lambda expression:

```C#
NameValidator nameValidator = (name) => name.Length > 3;
EmailValidator emailValidator = (email) => email.Length > 3;
```

You name the two delegate types after the functionality that the assigned code needs to have—checking the validity of a name and of an email address. You could’ve changed the name to reflect the signature:

```C#
delegate bool OneParameterReturnsBoolean(string parameter);
```

Now you have a delegate type that’s reusable, but only to code that has access to your definition, which cries for a standard implementation. The .NET Framework contains reusable delegate type definitions named `Func<>` and `Action<>`:

  * Func is a delegate type that returns a value and can receive parameters.

  * Action is a delegate type that can receive parameters but returns no value.

The .NET Framework contains 17 definitions of Func and Action, each for different numbers of parameters that the referenced method receives. The Func and Action types are located under the System namespace in the mscorlib assembly.

To reference a method that has no parameters and doesn’t return a value, you use the following definition of Action: 

```C#
delegate void Action();
```

and for a method that has two parameters and doesn’t return a value, you use this definition of Action:

```C#
delegate void Action<in T1, in T2>(T1 arg1, T2 arg2);
```

To use the Action delegate, you need to specify the types of parameters. Here’s an example of a method that traverses a collection and makes an operation on each item by using an Action of an integer:

```C#
void ForEachInt(IEnumerable<int> collection, Action<int> action)
{
    foreach (var item in collection)
    {
        action(item);
    }
}
```

Now you can call the ForEachInt method from your code like this:

```C#
var oddNumbers = new[] { 1, 3, 5, 7, 9 };
ForEachInt(oddNumbers, n => Console.WriteLine(n));
```

This code prints all the numbers in the oddNumbers collection. You can use the ForEachInt method with a different collection and different option. Because the Action delegate is generic, you can use that and create your generic version of ForEach:

```C#
public static void ForEach<T>(IEnumerable<T> collection, Action<T> action)
{
    foreach (var item in collection)
    {
        action(item);
    }
}
```

Now you can use ForEach with any collection:

```C#
ForEach(new[] { 1, 2, 3 }, n => Console.WriteLine(n));
ForEach(new[] { "a", "b", "c" }, n => Console.WriteLine(n));
ForEach(new[] { ConsoleColor.Red, ConsoleColor.Green, ConsoleColor.Blue}, 
        n => Console.WriteLine(n));
```

Because `Console.WriteLine` is a method that can accept any number of parameters, you can write the previous example this way too:

```C#
ForEach(new[] { 1, 2, 3 }, Console.WriteLine);
ForEach(new[] { "a", "b", "c" }, Console.WriteLine);
ForEach(new[] { ConsoleColor.Red, ConsoleColor.Green, ConsoleColor.Blue}, 
        Console.WriteLine));
```

Whenever you need to create a delegate for methods that return a value, you should use the type Func. This type (like Action) receives a variable number of generic parameters corresponding to the types of parameters that the method referenced by the delegate can receive. Unlike Action, in the Func definition the last generic parameter is the type of return value:

```C#
delegate TResult Func<in T1,...,T16, out TResult>(T1 arg,...,T16 arg16);
```

and there’s a definition for a Func that gets no parameters:

```C#
delegate TResult Func<out TResult>();  
```

Using Func, you can extend your implementation of the ForEach method so that it’ll accept a filter (also known as a predicate). The filter is a method that accepts an item and returns a Boolean indicating whether it’s valid:

```C#
static void ForEach<T>(IEnumerable<T> collection, Action<T> action,
                              Func<T, bool> predicate)          
{
    foreach (var item in collection)
    {
        if (predicate(item))           
        {
            action(item);
        }
    }
}
```

The filtering you added to the ForEach method can be exploited to act only on certain items in a collection—for instance, printing only even numbers:

```C#
var numbers = Enumerable.Range(1,10);          
ForEach(numbers, n => Console.WriteLine(n), n => (n % 2 == 0));
```

With Action and Func, you can build classes and methods that can be extended without modifying their code. This is a nice implementation of the Open Close Principle (OCP) that says a type should be open for extension but closed to modifications. Following design principles such as the OCP can improve your code and make it more maintainable.

### 3.2.5 Using it all together

It’s nice to see that known design patterns such as Strategy, Command, and Factory (to name a few) can be expressed differently with Func and Action and demand less code from the developer.


