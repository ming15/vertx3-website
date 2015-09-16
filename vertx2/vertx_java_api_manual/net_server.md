# Net Server

### Creating a Net Server

我们可以通过`vertx`实例的`createNetServer`方法轻松创建一个TCP服务器
```
NetServer server = vertx.createNetServer();
```

### Start the Server Listening

接下来我们告诉服务器要监听入站连接的端口号
```java
NetServer server = vertx.createNetServer();

server.listen(1234, "myhost");
```

第一个参数是要监听的端口号。如果将要监听的端口号设置为0的话，那服务器会随机出一个可用的端口号。一旦服务器完成监听动作，你可以调用`port()`方法查看服务器真实监听的端口号。

第二个参数是域名或者IP地址。如果该参数省略不填的话，那么才采取默认值`0.0.0.0`,这意味着它会监听所有可用的网络接口

实际上的绑定动作是异步的，这意味着，可能你的`listen`方法已经返回了，但是绑定动作还没有完成。如果你想要开始正式监听时获取一个通知的话，那么你可以在第三个参数上指定一个handler。
```java
server.listen(1234, "myhost", new AsyncResultHandler<Void>() {
    public void handle(AsyncResult<NetServer> asyncResult) {
        log.info("Listen succeeded? " + asyncResult.succeeded());
    }
});
```

### Getting Notified of Incoming Connections

我们需要调用`connectHandler`来处理到来的网络连接.
```java
NetServer server = vertx.createNetServer();

server.connectHandler(new Handler<NetSocket>() {
    public void handle(NetSocket sock) {
        log.info("A client has connected!");
    }
});

server.listen(1234, "localhost");
```

`connectHandler`方法返回值就是服务器自身，因此我们将多个方法调用链式地组合在一起：
```java
NetServer server = vertx.createNetServer();

server.connectHandler(new Handler<NetSocket>() {
    public void handle(NetSocket sock) {
        log.info("A client has connected!");
    }
}).listen(1234, "localhost");
```
or
```java
vertx.createNetServer().connectHandler(new Handler<NetSocket>() {
    public void handle(NetSocket sock) {
        log.info("A client has connected!");
    }
}).listen(1234, "localhost");
```
Vert.x API大多数都采用这种模式思想

### Closing a Net Server

如果想要结束一个net server，我们只需要调用`close`方法就好了
```java
server.close();
```

`close`方法同样是异步的，因此它也有可能close方法已经返回了，但是close操作其实还没完成。当然你如果想要当close完成时获得通知的话，你也可以选择向`close`方法指定一个handler

```java
server.close(new AsyncResultHandler<Void>() {
    public void handle(AsyncResult<Void> asyncResult) {
        log.info("Close succeeded? " + asyncResult.succeeded());
    }
});
```

如果你想要你的net server的生命周期和verticle保持一致，那么你就没必要显式的调用`close`方法了，当verticle解除部署时，Vert.x container会自动帮你关闭掉服务器

### NetServer Properties

`NetServer`有一套属性可以设置，属性可以影响`NetServer`的行为。首先，这套属性调整的是TCP参数，在大多数情况下，你不需要设置他们。

* `setTCPNoDelay(tcpNoDelay)` If true then Nagle's Algorithm is disabled. If false then it is enabled.
* `setSendBufferSize(size)` Sets the TCP send buffer size in bytes.
* `setReceiveBufferSize(size)` Sets the TCP receive buffer size in bytes.
* `setTCPKeepAlive(keepAlive)` if keepAlive is true then TCP keep alive is enabled, if false it is disabled.
* `setReuseAddress(reuse)` if reuse is true then addresses in TIME_WAIT state can be reused after they have been closed.
* `setSoLinger(linger)`
* `setTrafficClass(trafficClass)`


### Handling Data

当服务器接受到一个连接，connect handler对象的`handler`方法会被调用，同时向该方法中传递一个`NetSocket`对象。`NetSocket`是一个类Socket接口，该类允许你进行读写数据，甚至还允许你关闭该Socket。

#### Reading Data from the Socket

如果想要从`NetSocket`读取数据，你需要在`NetSocket`上调用`dataHandler`方法设置一个dataHandler。每当在socket上接受到数据后，dataHandler都会被调用,同时向`dataHandler`方法传递一个`org.vertx.java.core.buffer.Buffer`对象。你可以使用下面的例子开启一个服务器：
```java
NetServer server = vertx.createNetServer();

server.connectHandler(new Handler<NetSocket>() {
    public void handle(NetSocket sock) {
        sock.dataHandler(new Handler<Buffer>() {
            public void handle(Buffer buffer) {
                log.info("I received " + buffer.length() + " bytes of data");
            }
        });
    }
}).listen(1234, "localhost");
```

#### Writing Data to a Socket

如果想要向scoket中写入数据的话，你可以调用`write`方法，这个方法可以通过下面几种方式进行调用：

With a single buffer:
```java
Buffer myBuffer = new Buffer(...);
sock.write(myBuffer);
```

下面我们使用`UTF-8`编码的字符串，写入到socket中。
```java
sock.write("hello");
```
下面我们将指定编码格式化一个字符串，然后写入到socket中
```java
sock.write("hello", "UTF-16");
```

`write`方法同样是异步的，当该`write`方法入栈之后就会立即返回

下面给出了一个TCP 服务器，它将接受到的数据直接返回回去。
```java
NetServer server = vertx.createNetServer();

server.connectHandler(new Handler<NetSocket>() {
    public void handle(final NetSocket sock) {
        sock.dataHandler(new Handler<Buffer>() {
            public void handle(Buffer buffer) {
                sock.write(buffer);
            }
        });
    }
}).listen(1234, "localhost");
```

### Socket Remote Address

通过`remoteAddress()`方法你可以获得socket对等端的地址

### Socket Local Address

通过`localAddress()`方法你可以获得socket的本地地址

### Closing a socket

`close`方法会关闭一个socket，它会直接关闭底层的TCP连接

### Closed Handler

如果你想当socket关闭时获得通知，你可以设置一个`closedHandler`
```java
NetServer server = vertx.createNetServer();

server.connectHandler(new Handler<NetSocket>() {
    public void handle(final NetSocket sock) {
        sock.closedHandler(new VoidHandler() {
            public void handle() {
                log.info("The socket is now closed");
            }
        });
    }
}).listen(1234, "localhost");
```

不管是服务器还是客户端，任何一方关闭连接，close handler都会被调用

### Exception handler

如果担心通信过程中连接发生异常，你可以设置一个exception handler
```java
NetServer server = vertx.createNetServer();

server.connectHandler(new Handler<NetSocket>() {
    public void handle(final NetSocket sock) {
        sock.exceptionHandler(new Handler<Throwable>() {
            public void handle(Throwable t) {
                log.info("Oops, something went wrong", t);
            }
        });
    }
}).listen(1234, "localhost");
```

### Event Bus Write Handler

每一个`NetSocket`都会自动向`event bus`上注册一个handler， 当该handler接受到任何buffer之后，它会将buffer写入到`NetSocket`上。这样一来，由于我们可以通过从不同的verticle里，甚至从不同的Vert.x实例中向向`NetSocket`注册的`event bus`地址上写数据，那么我们实现了不单单从传统socket通信的方式向`NetSocket`写入数据的方式。

我们可以从`netSocket`上通过`writeHandlerID()`方法获取注册在`event bus`上的地址


下面的例子给出了从不同的verticle中向一个`NetSocket`中写入数据
```java
String writeHandlerID = ... // E.g. retrieve the ID from shared data

vertx.eventBus().send(writeHandlerID, buffer);
```

### Read and Write Streams

NetSocket also implements org.vertx.java.core.streams.ReadStream and org.vertx.java.core.streams.WriteStream. This allows flow control to occur on the connection and the connection data to be pumped to and from other object such as HTTP requests and responses, WebSockets and asynchronous files.

`NetSocket`实现了`org.vertx.java.core.streams.ReadStream`和`org.vertx.java.core.streams.WriteStream`接口。


## Scaling TCP Servers

每一个verticle实例都是纯单线程的。

如果你在一个verticle上创建了一个TCPserver，而且对该verticle只部署了一个实例，那么在该verticle上所有的handler就总是在同一个event loop上执行。

这意味着，如果你在一个多核心主机上上运行一个服务器，同时你只部署了一个服务器verticle实例，那么你的服务器只会使用该机器的一个核心

为了解决这个情况，你可以在同一个机器上部署多个服务器上module实例
```java
vertx runmod com.mycompany~my-mod~1.0 -instances 20
```

或者部署原生的verticle
```java
vertx run foo.MyApp -instances 20
```

上面的代码在同一个Vert.x实例上运行了20个module/verticle实例

当你这样部署之后，你会发现，服务器和之前运行的一样，但是，令人惊奇的是，你机器上的所有核心都处于使用状态，而且处理任务的能力也大大增强了

这时，你也许会问自己"等等，你怎么能让多个服务器同时监听相同的IP和端口呢？当你部署运行多个实例的时候不会造成端口冲突吗"

当主机上已经存在一个服务器监听某个`host/port`的时候，当你再部署一个服务器,监听相同的`host/port`时，Vert.x并不会新建一个server，再在相同的`host/port`上进行监听

> Vert.x内部是这样做的，当你尝试部署多个服务器时监听相同的主机和端口时，Vert.x并不会再创建新的服务器对象对相同的主机和端口号进行监听，但是它会在处理网络连接的地方上再注册一个connetc handler(每个)，这么一来就相当于实现了一个处理网络的"集群"

Vert.x内部只会持有一个服务器，当有连接到来时，Vert.x会根据`round-robin`算法，在众多的connet handler中选择一个，然后将到来的连接转发到被选择出的那个connet handler上

Consequently Vert.x TCP servers can scale over available cores while each Vert.x verticle instance remains strictly single threaded, and you don't have to do any special tricks like writing load-balancers in order to scale your server on your multi-core machine.

因此，Vert.x TCP server就可以在非常方便地在可用核心上进行水平拓展，每个verticle实例都被分配到一个单线程上，因此你就不需要自己在多核主机上去实现服务器的负载均衡了。