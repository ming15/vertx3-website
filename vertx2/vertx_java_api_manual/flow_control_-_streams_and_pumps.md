# Flow Control - Streams and Pumps

Vert.x提供了几个对象用于从`Buffer`中读取和写入数据。

在Vert.x中，调用写入数据的方法会直接返回，但是这个写入操作会在Vert.x内部入列(Vert.x内部有一个写入队列)。

如果你向一个对象中写入数据的速度快于这个对象向底层资源写入数据的速度的话，那么这个写入队列会无限制增长下去，直到最后将全部的可用内存都消耗掉。

为了解决这种问题，Vert.x API中的某些对象提供了`flow control`功能

我们可以向`org.vertx.java.core.streams.ReadStream`的实现类写入任何带有`flow control`功能对象, 我们可以从`org.vertx.java.core.streams.WriteStream`的实现类中读取出任何带有`flow control`功能的对象。

下面我们给出一个向`ReadStream`中读取数据,向`WriteStream`中写入数据的例子。

A very simple example would be reading from a NetSocket on a server and writing back to the same NetSocket - since NetSocket implements both ReadStream and WriteStream, but you can do this between any ReadStream and any WriteStream, including HTTP requests and response, async files, WebSockets, etc.

一个非常简单的例子是在服务器中从`NetSocket`中读取数据，然后将数据再写回到相同的`NetSocket`中,能这样做是因为`NetSocket`实现了`ReadStream`和`WriteStream`接口, 但是你可以在任何实现了`ReadStream`和`WriteStream`接口的类之间进行这样的操作,包括`HTTP requests and response`, `async files`, `WebSockets`, 等等.

对于刚才提到的情况，我们可以可以将接受的数据再直接写回到`NetSocket`中
```java
NetServer server = vertx.createNetServer();

server.connectHandler(new Handler<NetSocket>() {
    public void handle(final NetSocket sock) {

        sock.dataHandler(new Handler<Buffer>() {
            public void handle(Buffer buffer) {
                // Write the data straight back
                sock.write(buffer);
            }
        });

    }
}).listen(1234, "localhost");
```

在上述的例子中有一个问题：如果从socket中读取数据的速度快于向socket中写入数据的速度，它会慢慢地增长`NetSocket`中的写入队列，最终会引发内存溢出。例如，如果socket客户端读取数据不是很快，那么慢慢地该连接会阻塞掉。

由于`NetSocket`实现了`WriteStream`, 在写入数据之前我们可以检查`WriteStream`是否已经满了
```java
NetServer server = vertx.createNetServer();

server.connectHandler(new Handler<NetSocket>() {
    public void handle(final NetSocket sock) {

        sock.dataHandler(new Handler<Buffer>() {
            public void handle(Buffer buffer) {
                if (!sock.writeQueueFull()) {
                    sock.write(buffer);
                }
            }
        });

    }
}).listen(1234, "localhost");
```

上面的例子中不会引发内存溢出，但是当写入队列写满之后，就会发生丢消息的问题了。我们真的想做的是，当`NetSocket`的写入队列满了之后，就将`NetSocket`暂停掉：
```java
NetServer server = vertx.createNetServer();

server.connectHandler(new Handler<NetSocket>() {
    public void handle(final NetSocket sock) {

        sock.dataHandler(new Handler<Buffer>() {
            public void handle(Buffer buffer) {
                sock.write(buffer);
                if (sock.writeQueueFull()) {
                    sock.pause();
                }
            }
        });

    }
}).listen(1234, "localhost");
```

貌似我们已经完成了需求，但其实不然。当socket句柄满了之后，`NetSocket`被暂停了，但是当写入队列缓解之后，我们希望还能唤起暂停的`NetSocket`
```java
NetServer server = vertx.createNetServer();

server.connectHandler(new Handler<NetSocket>() {
    public void handle(final NetSocket sock) {

        sock.dataHandler(new Handler<Buffer>() {
            public void handle(Buffer buffer) {
                sock.write(buffer);
                if (sock.writeQueueFull()) {
                    sock.pause();
                    sock.drainHandler(new VoidHandler() {
                        public void handle() {
                            sock.resume();
                        }
                    });
                }
            }
        });

    }
}).listen(1234, "localhost");
```

当写入队列能够接受新的数据时,`drainHandler`会被调用, 这个操作会让`NetSocket`重新读取数据。

在我们开发Vert.x应用程序时，这是一种非常普遍的操作，因此我们提供了一个辅助类`Pump`, 这个类会完成刚才我们写的那一段代码。你可以将`Pump`看成`ReadStream`和`WriteStream`，`Pump`会自己知道何时重新读写数据
```java
NetServer server = vertx.createNetServer();

server.connectHandler(new Handler<NetSocket>() {
    public void handle(NetSocket sock) {
        Pump.create(sock, sock).start();
    }
}).listen(1234, "localhost");
```

## ReadStream

`HttpClientResponse, HttpServerRequest, WebSocket, NetSocket, SockJSSocket and AsyncFile`等类都实现了`ReadStream`接口

`ReadStream`接口定义如下方法：

* `dataHandler(handler)`: 设置一个从`ReadStream`读取数据的handler，当有数据到来时，handler会接受到一个`buffer`对象.
* `pause()`: 暂停dataHandler. 调用该方法之后，dataHandler不会再接受新的数据
* `resume()`: 激活dataHandler. 如果有数据来临时，dataHandler会被调用.
* `exceptionHandler(handler)`: `ReadStream`中发生异常时，exceptionHandler会被调用.
* `endHandler(handler)`: 当流读到结尾时，endHandler会被调用. This might be when EOF is reached if the ReadStream represents a file, or when end of request is reached if it's an HTTP request, or when the connection is closed if it's a TCP socket.

## WriteStream

` HttpClientRequest, HttpServerResponse, WebSocket, NetSocket, SockJSSocket and AsyncFile`实现了`WriteStream`接口

`WriteStream`接口定义如下方法：

* `write(buffer)`: 将`Buffer`中写入`WriteStream`.这个方法不会发生阻塞.写入操作在Vert.x内部会向写入队列中入列，写入队列将数据异步地写入底层资源。
* `setWriteQueueMaxSize(size)`: set the number of bytes at which the write queue is considered full, and the method * * writeQueueFull() returns true. Note that, even if the write queue is considered full, if write is called the data will still be accepted and queued.
* `writeQueueFull()`: 获取write queue是否满了，如果满了，返回true
* `exceptionHandler(handler)`: 当`WriteStream`发生异常时，将会调用这个handler
* `drainHandler(handler)`: The handler will be called if the WriteStream is considered no longer full.当`WriteStream`

## Pump

`Pump`实例拥有下列方法

* `start()`: 启动pump.
* `stop()`: 停止pump. When the pump starts it is in stopped mode.
* `setWriteQueueMaxSize()`: 与`WriteStream`的`setWriteQueueMaxSize`意义相同.
* `bytesPumped()`: 返回pumped的总的字节数

`Pump`可以多次启动和停止

当`Pump`第一次创建出来后，并不是`started`状态，你需要调用`start()`方法来启动它