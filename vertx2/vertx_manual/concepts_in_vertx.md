# Concepts in Vert.x

### verticle

* `Vert.x` 执行的代码单元称为 `verticle`.
* `verticle`可以由`JavaScript, Ruby, Java, Groovy or Python`等语言编写(Scala和Clojure支持还在开发中)
* 许多`verticle`可以在同一个`Vert.x`实例中并发执行
* 应用程序可以由部署同一网络在多个不同的节点上的`verticle`组成, 然后在``Vert.x` event bus`上进行消息交换。
* 在一些不重要的应用程序中, 可以在命令行中直接运行`verticle`, 但是更加通常的做法是将他们打包进`module`里，然后运行该`module`

### module

* `Vert.x`应用程序通常是由一个或者多个`module`组成. 一个`module`中可以包含多个`verticle`(不同的`verticle`s`可能由不同的语言编写). `module`允许功能性的封装和复用
* `module`可以被放入`Maven`或者其他[Bintray]()仓库里, 而且也可以注册到`Vert.x` [module registry]()上.
* 通过`Vert.x`社区, `Vert.x module`系统发展出了一个完整的`Vert.x module`生态系统

更多关于`module`的信息,参考[module`s manual]().

### Vert.x Instances

`verticle`s运行在`Vert.x`实例中. `Vert.x`实例运行在它自己的JVM实例里. 在同一时间,一个`Vert.x`实例中可以运行多个`verticle`s.在同一时刻同一个主机上可以运行多个`Vert.x`实例, 这些`Vert.x`实例可以被配制成一个集群, 集群中的`Vert.x`实例可以在一个分布式的event bus中进行交互.

> 注：是否每一个`Vert.x`实例都要启动一个JVM实例？

### Polyglot

`Vert.x`允许你通过`JavaScript, Ruby, Java, Groovy and Python`这几种语言编写`verticle`，而且在未来我们还会支持`Clojure and Scala`. 不管是采用什么语言编写的`verticle`，都可以进行无缝交互。

### Concurrency

`Vert.x`保证每一个`verticle`实例在任一时间点上只会被一个线程执行. 这样一来，在实际的开发过程中，你就不需要考虑你的代码会并发执行了(就好像你的代码永远只会被单线程执行).

如果你曾经使用传统的多线程并发模型进程编程, 那`Vert.x`的这种做法会将你从那种编程模型中解救出来，从此你不用再同步你的状态访问了. 这意味着条件竞争和OS线程死锁都成为了过去式.

不同的`verticle`实例可以通过event bus进行消息交换. `Vert.x`应用程序是并发的，因为`Vert.x`允许有多个单线程的`verticle`实例并发执行，以及他们之间相互交换数据，而且单个`verticle`实例并不会被多个线程并发执行。

因此`Vert.x`的并发模型和Actor模型非常像(`verticle`和`actor`大致是一致的)。但是他们之间还是有一些不同点的，例如，在代码结构单元上，`verticle`的粒度趋于比`actor`的要大

### Asynchronous Programming Model

`Vert.x`提供了一套异步API,这意味着在`Vert.x`中你需要做的大部分事情就是设置event handler(进行异步回调). 例如你设置了一个handler从TCP Socket接受数据,当数据来的时候,这个handler就会自动被调用.

你还可以设置handler,让其从event bus接受消息, 或者接受HTTP请求然后回应该请求, 以及当一个连接被关闭时收到通知, 或者当定时器到达时接受通知. 这种设置handler的模式在`Vert.x`中普遍存在(因为我们要异步，要回调).

由于我们使用异步API,因此我们可以仅仅使用少量的os thread便可以支持多个`verticle`s. 实际上, `Vert.x`设置的线程数与主机上的可用核心数相等.作为一个非常优秀的非阻塞应用程序,你设置线程数没有必要超过可用核心数.

在传统的异步API中,线程会在API操作中阻塞住, 而线程被阻塞之后, 他们就不能再做其他的工作了.例如,从socket中读取数据. 当该线在Socket上程等待数据到达时,它就不能再做其他事.这意味着,当我们需要支持百万并发连接的时候,那我们需要一百万个线程.

在开发项目的时候，有时异步API会受到些批评,尤其是当你不得不从多个event handler上获取结果的时候

我们可以采取下面的方式缓和这种情况,例如,使用[mod-rx-vertx]()`module`,这个`module`允许你构建异步事件流.这个`module`使用了[RxJava]()类库,这个类库受到了`.net`的[Reactive extensions]()启发.

### event loop

每个`Vert.x`实例内部都管理着一些线程(线程数与主机上可用核心上的线程数相等). 我们称那些线程为`event loop`， 因为那些线程都或多或少地进行着循环检查——是否有事件传递过来，如果接受到事件，就将它发送给适当的handler处理。例如，事件可以是已经从socket中读取到的数据，或者一个定时器的时间到了，或者一个HTTP response已经结束。

当一个标准的`verticle`实例被部署后, 服务器将选择一个`event loop`指派给该`verticle`实例。要由那个`verticle`实例处理的工作，都会调用分配给它的线程转发给它。当然，由于在同一时刻可能存在着好几千个处于运行状态的`verticle`，一个`event loop`可能同时会被分配到多个`verticle`上

我们管这叫做`multi-reactor`模式。它和[reactor pattern]()很像，但是它却拥有多个`event loop`

###### The Golden Rule - Don't block the event loop!

一个特定的`event loop`经常会被用于服务多个`verticle`实例，所以你千万不能将`verticle`实例阻塞住。一旦`verticle`阻塞住，那么分配给他线程就不能将接下来的事件分发给其他的handler了，然后你的应用程序慢慢地就被拖死了。

在`verticle`中任何占用`event loop`的事情以及`event loop`不能继续快速处理其他事件的事情都会造成`event loop`阻塞。可能阻塞`event loop`的事件可能包括。

* `Thread.sleep()`
* `Object.wait()`
* `CountDownLatch.await()` 或者`java.util.concurrent`中其他的阻塞操作.
* 在loop中进行自旋
* 执行一个长时间的计算密集型操作，例如数字计算
* 调用一个阻塞的第三方库操作，例如JDBC查询

### Writing blocking code - introducing Worker verticles

在一个标准的`verticle`中，`event loop`是不建议阻塞发生的，但是，在实际情况中我们极可能有需要阻塞`event loop`的场景，或者你确实有计算密集型操作执行。一个典型的例子就是调用像JDBC这样的API。

你也许会想写一些直接的阻塞代码，例如，你打算开发一个简单的webserver，但是你知道自己不会有很大的流量，那么你也就不需要处理很多并发连接了。

像刚才描述的那些场景，`Vert.x`允许你使用一种特殊的`verticle`实例——工作者`verticle`(`woker verticle`)。工作者`verticle`与标准`verticle`的不同之处在于，工作者`verticle`不会被分配到一个`Vert.x` `event loop`线程上，而是在一个称为工作者线程池的内部线程池上运行。

和标准`verticle`一样，工作者`verticle`也不会被多个线程同时执行，但和标准`verticle`不同的是，工作者`verticle`可以被不同的线程在不同的时刻执行(标准`verticle`总是被同一个线程执行)。

在一个工作者`verticle`中阻塞线程是可以被接受的。

为了支持标准非阻塞`verticle`和阻塞工作者`verticle`，`Vert.x`提供了一种混合线程模型，因此你可以在你的应用程序使用合适的`verticle`书写你的逻辑。这相比于一些其他只要求使用阻塞或者非阻塞的平台更加实用。

但是当你使用工作者`verticle`时也要小心，如果你想要处理很多的并发网络连接时，阻塞的`verticle`并不能拓展你的应用程序的并发处理能力

### Shared data

消息传递是非常有用的，但是它并不适用于所有的应用程序的并发处理。于是我们提供了一种共享数据结构，可以使使同一个`Vert.x`实例中不同`verticle`实例直接访问它。

`Vert.x`提供了一个共享的`Map`和一个共享的`Set`。为了避免条件竞争，我们建议所有被存储共享的数据都应该是不可变的。

### Vert.x APIs

`Vert.x`提供了一个可以在`verticle`中可以直接调用的小巧的静态API。

`Vert.x` API不会轻易地发生变动，新功能的添加会通过`module`的方式添加。

这意味着`Vert.x` core会一直保持小巧和紧凑，如果你想要使用新功能，只需要添加相关`module`即可。

`Vert.x` API被分为`container API`和`core API`俩部分。

###### Container API

下面给出了`Vert.x container`对象的操作列表：

* 对`verticle`进行部署和解除部署
* 对`module`进行部署和解除部署
* 恢复`verticle`配置
* Logging

###### Core API

This API provides functionality for:

* `TCP/SSL` 服务器和客户端
* `HTTP/HTTPS` 服务器和客户端
* `WebSockets`服务器和客户端
* 分布式 `event bus`
* Periodic and one-off timers
* `Buffers`
* `Flow control`
* `File-system access`
* `Shared map and sets`
* `Accessing configuration`
* `SockJS`