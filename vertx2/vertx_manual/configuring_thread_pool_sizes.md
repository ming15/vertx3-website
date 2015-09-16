# Configuring thread pool sizes


`Vert.x`主要包含俩个线程池：
1. `event loop`池
2. 后台(工作者)线程池

### The event loop pool

`event loop池` 用于向标准`verticle`提供`event loop`。默认大小是根据你机器上的核心决定的，通过`Runtime.getRuntime().availableProcessors()`的方法获取你机器上的可用核心.

如果你想要进行优化，改变线程池的大小，你可以重新设置系统属性值——`vertx.pool.eventloop.size`.

### The background pool

这个线程池是用于提供工作者`verticle`使用的线程，以及用于其他阻塞任务。由于工作者线程提供阻塞功能，我们往往会比`event loop`线程池使用的更多。默认的工作者线程池的最大值是20.

如果你想要修改工作者线程池的最大,只需要修改系统属性`vertx.pool.worker.size`就好了