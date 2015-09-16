# NetClient

`NetClient`常常是用来与服务器进行TCP连接

### Creating a Net Client

你只需要通过调用`vertx`的`createNetClient`方法就可以创建一个TCP客户端
```java
NetClient client = vertx.createNetClient();
```

### Making a Connection

然后调用`connect`方法就可以连接到服务器
```java
NetClient client = vertx.createNetClient();

client.connect(1234, "localhost", new AsyncResultHandler<NetSocket>() {
    public void handle(AsyncResult<NetSocket> asyncResult) {
        if (asyncResult.succeeded()) {
          log.info("We have connected! Socket is " + asyncResult.result());
    } else {
          asyncResult.cause().printStackTrace();
        }
    }
});
```

`connetc`方法第一个参数是服务器的端口，第二个参数是服务器绑定的域名或者IP地址。第三个参数是一个connect handler，当连接建立成功之后，这个handler就会被调用

`connect handler`泛型参数是`AsyncResult<NetSocket>`,我们可以从这个对象的`result()`方法中获取`NetSocket`对象。你可以像在服务器端那样，在socket上进行读写数据。

当然你也可以像在服务器端那样执行`close , set the closed handler, set the exception handler`操作

### Configuring Reconnection

`NetClient`可以被设置成自动重连或者当它无法连接到服务器/与服务器断开连接后进行断线重连。你可以通过调用`setReconnectAttempts`和`setReconnectInterval`方法来实现这样的功能
```java
NetClient client = vertx.createNetClient();

client.setReconnectAttempts(1000);

client.setReconnectInterval(500);
```

* `ReconnectAttempts`:该值设定重连服务器的次数。`-1`表示无限次。默认值是0
* `ReconnectInterval`:该值设定重连服务器的间隔。单位是毫秒。默认值是1000

### NetClient Properties

`NetClient`也有一套TCP Properties，这套属性值的含义和`NetServer`一样，具体使用参考`NetServer`就好了。

