When you want to write parallel and concurrent applications in Scala, you *could* still use the native Java `Thread` — but the Scala [Future](https://www.scala-lang.org/api/current/scala/concurrent/Future$.html) makes parallel/concurrent programming much simpler, and it’s preferred.

Here’s a description of `Future` from its Scaladoc:

> “A Future represents a value which may or may not *currently* be available, but will be available at some point, or an exception if that value could not be made available.”



## Thinking in futures

To help demonstrate this, in single-threaded programming you bind the result of a function call to a variable like this:

```SCALA
def aShortRunningTask(): Int = 42
val x = aShortRunningTask
```

With code like that, the value `42` is bound to the variable `x` immediately.

When you’re working with a `Future`, the assignment process looks similar:

```scala
def aLongRunningTask(): Future[Int] = ???
val x = aLongRunningTask
```

But because `aLongRunningTask` takes an indeterminate amount of time to return, the value in `x` may or may not be *currently* available, but it will be available at some point (in the future).

Another important point to know about futures is that they’re intended as a one-shot, “Handle this relatively slow computation on some other thread, and call me back with a result when you’re done” construct.

In this lesson you’ll see how to use futures, including how to run multiple futures in parallel and combine their results in a for-expression, along with other methods that are used to handle the value in a future once it returns.

> Tip: If you’re just starting to work with futures and find the name `Future` *to be confusing in the following examples, replace it with the name* `ConcurrentResult`*, which might be easier to understand initially.*

## Source code

You can find the source code for this lesson at this URL:

- [github.com/alvinj/HelloScalaFutures](https://github.com/alvinj/HelloScalaFutures)

## An example in the REPL

A Scala `Future` is used to create a temporary pocket of concurrency that you use for one-shot needs. You typically use it when you need to call an algorithm that runs an indeterminate amount of time — such as calling a web service or executing a long-running algorithm — so you therefore want to run it off of the main thread.

To demonstrate how this works, let’s start with an example of a `Future` in the Scala REPL. First, paste in these `import` statements:

```SCALA
import scala.concurrent.Future
import scala.concurrent.ExecutionContext.Implicits.global
import scala.util.{Failure, Success}
```

Now, you’re ready to create a future. For example, here’s a future that sleeps for ten seconds and then returns the value `42`:

```scala
scala> val a = Future { Thread.sleep(10*1000); 42 }
a: scala.concurrent.Future[Int] = Future(<not completed>)
```

Because a `Future` has a `map` function, you use it as usual:

```scala
scala> val b = a.map(_ * 2)
b: scala.concurrent.Future[Int] = Future(<not completed>)
```

Initially this shows `Future(<not completed>)`, but if you check `b`’s value you’ll see that it eventually contains the expected result of `84`:

```SCALA
scala> b
res1: scala.concurrent.Future[Int] = Future(Success(84))
```

Therefore, when working with the result of a future, use the usual `Try`-handling techniques, or one of the other `Future` callback methods.

```SCALA
a.onComplete {
    case Success(value) => println(s"Got the callback, value = $value")
    case Failure(e) => e.printStackTrace
}
```

When you paste that code in the REPL you’ll see the result:

```
Got the callback, value = 42
```

There are other ways to process the results from futures, and the most common methods are listed later in this lesson.

## An example application

The following application (`App`) provides an introduction to using multiple futures. It shows several key points about how to work with futures:

- How to create futures
- How to combine multiple futures in a `for` expression to obtain a single result
- How to work with that result once you have it

### A potentially slow-running method

First, imagine you have a method that accesses a web service to get the current price of a stock. Because it’s a web service it can be slow to return, and even fail. As a result, you create a method to run as a `Future`. It takes a stock symbol as an input parameter and returns the stock price as a `Double` inside a `Future`, so its signature looks like this:

```SCALA
def getStockPrice(stockSymbol: String): Future[Double] = ???
```

To keep this tutorial simple we won’t access a real web service, so we’ll mock up a method that has a random run time before returning a result:

```SCALA
def getStockPrice(stockSymbol: String): Future[Double] = Future {
    val r = scala.util.Random
    val randomSleepTime = r.nextInt(3000)
    val randomPrice = r.nextDouble * 1000
    sleep(randomSleepTime)
    randomPrice
}
```

That method sleeps a random time up to 3000 ms, and also returns a random stock price. Notice how simple it is to create a method that runs as a `Future`: Just pass a block of code into the `Future` constructor to create the method body.



| words                     | words            |
| ------------------------- | ---------------- |
| 通勤：parallel/concurrent | 远程工作：native |
| Thread                    | preferred        |
| point                     | single-threaded  |
| function call             | variable         |
| bind                      | assignment       |
| indeterminate             | run              |
| work  -> use              | take             |
| paste                     | statements       |
| construct                 | wrapped          |
| signature                 |                  |

