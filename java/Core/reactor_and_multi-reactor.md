# Reactor and Multi-Reactor

我们在前文中提到过Vert.x是基于事件驱动的，当Vert.x事件准备好之后，就会向事件传递给被设置的`handler`上

在大多数情况下，Vert.x会使用一个称为`event loop`的线程来调用你的handler

鉴于Vert.x以及你的应用程序不会产生任何阻塞操作，`event loop`会快速地将事件分发到不同的`handler`上

因为我们的任何操作都不会带来任何阻塞，因此一个`event loop`就可以在非常短的时间内，分发出去居多的事件。例如一个`event loop`就可以非常快速地处理数千个HTTP请求。

我们把这种模式称为[Reactor Pattern](http://en.wikipedia.org/wiki/Reactor_pattern).

你也许以前就听说过这种模式，例如`Node.js`就是这种模式的一种实现

在一个标准`reactor`实现中,会有一个单独的`event loop`线程进行可用事件轮询，只要有事件接听到，就将它发送到全部的handler上

但是这种实现有个小缺点，在任一时刻，它都会只运行在一个核心上，因此如果你想要你的单线程`reactor`应用程序在多核心服务器上进行拓展，那么你就需要启动并管理多个不同的`reactor`应用程序进程

但是Vert.x的工作模式与之不同。相比单线程`event loop`,每个`Vertx`实例都包含数个`event loop`. 在默认情况下,我们会根据所在机器的可用核心数来设置`event loop`数量,当然你也可以自己指定这个数量

我们把这种模式称为`Multi-Reactor Pattern`

> 注意：尽管`Vertx`实例会持有多个`event loop`,但是每一个`handler`都不会被并发执行, 而且在大多数情况下(工作者verticle除外),`handler`会被同一个`event loop执行`