# Using blocking handlers

在某些环境下,你也许想要在handler执行时,将`event loop`阻塞住, 例如调用一个阻塞API或者执行一些密集型计算. 这种情况下,你不能在普通`handler`中进行操作,我们为`route`提供了一个阻塞的`handler`.

阻塞式和非阻塞式`handler`非常像,只不过阻塞式是由`Vert.x`从`worker pool`中借出一个线程进行任务执行,而非是从`event loop`中.

下例进行了说明：
```java
router.route().blockingHandler(routingContext -> {

  // Do something that might take some time synchronously
  service.doSomethingThatBlocks();

  // Now call the next handler
  routingContext.next();

});
```
在默认情况下, Vert.x中任何的阻塞`handler`都是在相同的上下文中(例如`verticle`实例中)顺序执行的,这意味着当前一个handler未完成之前,下一个handler是不会执行的. 如果你不关心任务的执行顺序,而且不介意阻塞`handler`并行执行,你可以在调用`blockingHandler`方法时传递一个`false`的参数,让其不按照任务的指定顺序进行执行.

