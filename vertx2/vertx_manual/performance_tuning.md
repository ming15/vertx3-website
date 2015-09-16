# Performance Tuning

### Improving connection time

If you're creating a lot of connections to a `Vert.x` server in a short period of time, you may need to tweak some settings in order to avoid the TCP accept queue getting full. This can result in connections being refused or packets being dropped during the handshake which can then cause the client to retry.

如果短时间内`Vert.x`服务器接受到了大量的连接，你可以通过修改一些配置来避免TCP接收队列饱和。这些设置可以作用于连接复用或者握手期间的丢包情况

出现这种情况的典型例子就是，你的客户端出现了有超过`3000ms`的长连接

How to tune this is operating system specific but in Linux you need to increase a couple of settings in the TCP / Net config (10000 is an arbitrarily large number)

一般是操作系统来调整TCP接收，但是在Linux中，你可能需要将`TCP / Net`设置进行翻倍。
```
sudo sysctl -w net.core.somaxconn=10000
sudo sysctl -w net.ipv4.tcp_max_syn_backlog=10000
```

对于其他的系统，你就要参考具体系统的配置手册了。

同时，你还要对服务器端的`accept backlog`参数进行设置
```
HttpServer server = vertx.createHttpServer();
server.setAcceptBacklog(10000);
```

###Handling large numbers of connections

#### Increase number of available file handles

为了使服务器能够处理大量的网络链接，你也许需要提升你系统的最大文件句柄数(每一个socket都需要一个文件句柄)，至于如何进行设置就看具体操作操作系统了。

#### Tune TCP buffer size

每一个TCP连接的都会为它自己的buffer分配一块内存，因此为了能够支持在固定大小的内存限制下，结构更多的网络连接，我们就需要削减每个TCP buffer的大小了。
```java
HttpServer server = vertx.createHttpServer();
server.setSendBufferSize(4 * 1024);
server.setReceiveBufferSize(4 * 1024);
```