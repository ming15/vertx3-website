# Don’t call us, we’ll call you.

Vert.x APIs大多数是基于事件驱动的。这意味着，在Vert.x中，当你关注的事件发生时，Vert.x会自动通知你。

例如当下面这些事件发生时，Vert.x就会自动通知你
* 定时器被触发
* socket中接收到数据
* 从磁盘中读取数据已经就绪
* 异常发生
* HTTP服务器接受到一个请求

我们需要通过向Vert.x APIs提供`handler`来处理Vert.x通知给我们的事件，例如下例中演示了我们每秒从定时器中接受一个事件
```java
vertx.setPeriodic(1000, id -> {
  // This handler will get called every second
  System.out.println("timer fired!");
});
```
或者接受一个HTTP请求
```java
server.requestHandler(request -> {
  // This handler will be called every time an HTTP request is received at the server
  request.response().end("hello world!");
});
```
当Vert.x中产生一个事件之后，它会异步地将这个事件传递到你设置的`handler`中

This leads us to some important concepts in Vert.x:

TODO
