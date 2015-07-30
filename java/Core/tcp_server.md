# TCP Server

## Creating a TCP server
我们使用默认的选项(`HttpServerOptions`)来创建一个最简单的`TCP`服务器.
```java
NetServer server = vertx.createNetServer();
```

## Configuring a TCP server
如果想要对创建的服务器进行特殊配置,可以使用`HttpServerOptions`来创建服务器。
```java
NetServerOptions options = new NetServerOptions().setPort(4321);
NetServer server = vertx.createNetServer(options);
```

## Start the Server Listening

我们提供了众多的`listen`方法,下面我们选择一个不带参数的`listen`方法(端口和主机地址已经在刚才的`HttpServerOptions`中指定了)
```java
NetServer server = vertx.createNetServer();
server.listen();
```

下面我们在`listen`方法中显式地指定监听的端口和网卡地址(这时候会忽略掉`HttpServerOptions`中设置的端口号)
```java
NetServer server = vertx.createNetServer();
server.listen(1234, "localhost");
```

`listen`方法默认监听的地址是`0.0.0.0`(所有可用地址),默认的端口是0(这种情况下会随机选择一个可用的端口). 需要注意的是绑定操作(`listen`)是异步进行的,所以当`listen`方法返回之后并不保证绑定操作已经成功.
在示例中我们添加了一个`handler`用于接受绑定成功之后的通知.
```java
NetServer server = vertx.createNetServer();
server.listen(1234, "localhost", res -> {
  if (res.succeeded()) {
    System.out.println("Server is now listening!");
  } else {
    System.out.println("Failed to bind!");
  }
});
```

## Listening on a random port

当我们监听端口号为0时,系统会自动随机选择一个实际可用的端口进行监听,如果想要获取真实监听端口可以调用`actualPort`方法.
```java
NetServer server = vertx.createNetServer();
server.listen(0, "localhost", res -> {
  if (res.succeeded()) {
    System.out.println("Server is now listening on actual port: " + server.actualPort());
  } else {
    System.out.println("Failed to bind!");
  }
});
```

## Getting notified of incoming connections

下例中我们设置了一个`connectHandler`,用于处理服务器接受到的网络连接
```java
NetServer server = vertx.createNetServer();
server.connectHandler(socket -> {
  // Handle the connection in here
});
```

当网络连接建立成功之后,`handler`就会自动被调用(同时会带有一个`NetSocket`对象作为参数)

`NetSocket`是对实际网络连接的一个`类Socket`接口抽象(`socket-like interface`),你可以在这个接口进行读写数据,或者直接关闭Socket等操作.

## Reading data from the socket
To read data from the socket you set the handler on the socket.

想要读取Socket中的数据,那你就需要调用`NetSocket`的`handler`方法,设置一个`handler`,用于处理数据.

当Socket流中有数据到达时,服务器就会将接受到的数据封装成一个`Buffer`对象,然后刚刚在`NetSocket`上设置的那个`handler`就会被调用.
> 考虑半包处理
```java
NetServer server = vertx.createNetServer();
server.connectHandler(socket -> {
  socket.handler(buffer -> {
    System.out.println("I received some bytes: " + buffer.length());
  });
});
```

## Writing data to a socket

你可以直接调用`NetSocket`的`write`方法进行写回数据
```java
Buffer buffer = Buffer.buffer().appendFloat(12.34f).appendInt(123);
socket.write(buffer);

// Write a string in UTF-8 encoding
socket.write("some data");

// Write a string using the specified encoding
socket.write("some data", "UTF-16");
```
> 注意，`write`方法一样是异步进行的,当`write`方法返回后,并不保证数据已经完全写入到Socket流中,也不保证数据能够写入成功

## Closed handler

下例中我们设置了一个`closeHandler`用于当Socket关闭时,获得一些通知
```java
socket.closeHandler(v -> {
  System.out.println("The socket has been closed");
});
```

## Handling exceptions
如果你想当socket操作发生异常时获得通知,你可以设置一个`exceptionHandler`

## Event bus write handler

每一个Socket都会在`event bus`上自动注册一个`handler`,一旦该`handler`接受到`Buffer`, `handler`会将`Buffer`写到Socket上.

利用这种特性你可以在不同的`verticle`甚至不同的`Vertx`实例里对同一个socket写数据. 这种功能的实现方式是`handler`身上有一个`writeHandlerID`,这个ID是`handler`在`event bus`上的注册地址,不同的`verticle`甚至不同的`Vertx`实例就可以通过该地址向Socket写入数据。


## Local and remote addresses
我们可以通过`localAddress`获得`NetSocket`的本地地址. 通过`remoteAddress`获得网络对等端地址.

## Sending files

我们可以通过`sendFile`直接向Socket中写入一个文件. 这是一种非常高效发送文件的方式,如果操作操作系统支持的话,这还可以被OS内核支持
```java
socket.sendFile("myfile.dat");
```

## Streaming sockets
Instances of NetSocket are also ReadStream and WriteStream instances so they can be used to pump data to or from other read and write streams.

See the chapter on streams and pumps for more information.

`NetSocket`实例还是`ReadStream`和`WriteStream`的实例,因此

## Upgrading connections to SSL/TLS

我们可以使用`upgradeToSsl`方法将一个不支持`SSL/TLS`的连接改为支持`SSL/TLS`的连接,具体参考相关章节

## Closing a TCP Server

我们可以调用`close`方法关闭服务器,`close`方法会关闭所有打开的连接和所有的服务器资源.

一样一样的,关闭操作同样是异步的,你懂得,想要关闭完成时进行某些操作,设置handler吧.
```java
server.close(res -> {
  if (res.succeeded()) {
    System.out.println("Server is now closed");
  } else {
    System.out.println("close failed");
  }
});
```

## Automatic clean-up in verticles
如果你是在`verticle`中创建的`TCP`服务器和客户端,那么当宿主`verticle`被`undeployed`时,宿主身上的服务器和客户端也会被自动的关闭掉

## Scaling - sharing TCP servers

`TCP`服务器上的所有`handler`都只会被相同的`event loop`执行.

这意味着如果你的服务器是运行在一个多核心的主机上,但是你在该主机上只部署了一个服务器实例,那么你最多也就是利用了主机上的一个核心.

为了能使用更多的核心,你需要在该主机上部署多个服务器实例. 下面的示例演示了如何通过编程的方式部署多个服务器实例：

```java
for (int i = 0; i < 10; i++) {
  NetServer server = vertx.createNetServer();
  server.connectHandler(socket -> {
    socket.handler(buffer -> {
      // Just echo back the data
      socket.write(buffer);
    });
  });
  server.listen(1234, "localhost");
}
```
或者如果你的服务器是在`verticle`内实现的,那么你也可以在命令行中通过`-instances`部署多个服务器实例.
```
vertx run com.mycompany.MyVerticle -instances 10
```
以及通过编程的方式部署多个服务器`verticle`实例

```java
DeploymentOptions options = new DeploymentOptions().setInstances(10);
vertx.deployVerticle("com.mycompany.MyVerticle", options);
```
Once you do this you will find the echo server works functionally identically to before, but all your cores on your server can be utilised and more work can be handled.

At this point you might be asking yourself 'How can you have more than one server listening on the same host and port? Surely you will get port conflicts as soon as you try and deploy more than one instance?'

Vert.x does a little magic here.*

When you deploy another server on the same host and port as an existing server it doesn’t actually try and create a new server listening on the same host/port.

Instead it internally maintains just a single server, and, as incoming connections arrive it distributes them in a round-robin fashion to any of the connect handlers.

Consequently Vert.x TCP servers can scale over available cores while each instance remains single threaded.
