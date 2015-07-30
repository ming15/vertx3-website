# Running blocking code

在一个完美的世界里，那么没有战争和饥饿，全部的API也都会被书写成异步形式的，

###### But.. the real world is not like that. (Have you watched the news lately?)

Fact is, many, if not most libraries, especially in the JVM ecosystem have synchronous APIs and many of the methods are likely to block. A good example is the JDBC API - it’s inherently asynchronous, and no matter how hard it tries, Vert.x cannot sprinkle magic pixie dust on it to make it asynchronous.

但事实上，许多类库，尤其是在JVM生态系统中拥有大量同步API而且许多方法都会产生阻塞。一个非常著名的例子就是`JDBC API`,它被设计出来就是同步的，Vert.x无论怎么优化都不能将它变成异步的。

我们并不准备将所有的东西都重写成异步的，因此我们提供了一种方式，以便你可以在Vert.x应用程序中使用传统的阻塞API

正如像在前面讨论的那样，你不能在`event loop`直接调用阻塞操作,那么你要如何去执行一个阻塞操作呢？

我们可以通过调用`executeBlocking`方法来执行阻塞代码, 同样当这个方法执行完阻塞代码之后，会异步地调用`result handler`.
```
vertx.executeBlocking(future -> {
  // Call some blocking API that takes a significant amount of time to return
  String result = someAPI.blockingMethod("hello");
  future.complete(result);
}, res -> {
  System.out.println("The result is: " + res.result());
});
```
还有一种执行阻塞代码的方式，那就是使用`worker verticle`
