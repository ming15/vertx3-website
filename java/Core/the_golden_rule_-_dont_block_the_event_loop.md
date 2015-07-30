# The Golden Rule - Don’t Block the Event Loop

我们已经知道`Vert.x API`不会有任何阻塞操作,也不会阻塞住`event loop`. 但是如果在你自己处理handler时阻塞住了当前线程(工作者vertile除外)，那么同样会影响Vert.x性能

如果你确实在handler处理时，阻塞住了`event loop`,那么`event loop`会一直等待你的操作完成，在等待的时候它只是傻傻地干等着。如果你讲`Vertx`对象中的所有`event loop`都阻塞住了话，那么你的应用程序不久就会挂掉了。

我们下面举出一些常见的阻塞操作：
* Thread.sleep()
* 等待锁
* Waiting on a mutex or monitor (e.g. synchronized section)
* 执行一个长时间的数据库操作，并同步等待结果的返回
* 执行耗时较长的复杂计算
* 在循环中进行自旋

如果某个操作耗费大量时间引发了上述问题,从而阻塞了`event loop`,那么你应该直接跳过当前操作.

那么多长的时长才能被称为大量时间呢? 这完全取决于你的应用的并发数量。


如果只在一个`event loop`中，你想要每秒处理10000个http请求，那么处理每个请求不能超过0.1ms，因此在`event loop`中每次处理过程中的阻塞时间不能超过0.1ms

******The maths is not hard and shall be left as an exercise for the reader******.

如果你的应用程序没有应答，也许是因为在某些地方阻塞住了`event loop`.为了能够帮你确定那个问题，当在一定时间内`event loop`都没有返回的话，Vert.x会自动对此产生警告日志。如果你在日志中见到了像那样的警告，你就需要好好研究一下问题出在哪里了。

例如`vertx-eventloop-thread-3`线程已经被阻塞了20458 ms，Vert.x会提供堆栈信息帮你找到阻塞发生的具体位置。

如果你想要关闭那么警告信息或者改变一些其他设置，那么你可以在创建`Vertx`对象之前，在`VertxOptions`中进行设置