# Don’t block me!

With very few exceptions (i.e. some file system operations ending in 'Sync'), Vert.x中的API方法都不会阻塞该该方法的执行线程。

如果结果能够直接返回的话，那么你可以直接获得该结果进行后续处理，否则你需要提供一个handler，等结果正式产生之后再处理该结果。

因为Vert.x API不会阻塞任何线程，因此你可以使用少量的线程来处理非常大的并发量

在传统的阻塞API中，下面的操作会阻塞住当前的调用线程
* 从socket中读取数据
* 向磁盘中写入数据
* 向远端发送一条消息，同时等待消息返回
* … Many other situations

在上面的例子中，当你的线程等待一个结果时，这个线程就不能再做其他的任何事，这是非常低效的。

>`译者注：` 也许你会说这个线程被阻塞了，但是还有其他的线程可以工作啊，但是首先我们的目标是用少量地线程做大量的工作，当我们引入更多线程首先与我们的目标不符合，再有更多的线程会消耗更多的内存和更多的线程上下文切换操作

这意味着，如果你想要使用阻塞API进行大量并发操作，你需要非常多的线程以避免你的应用程序慢慢停止掉。

For the levels of concurrency required in many modern applications, a blocking approach just doesn’t scale.
