# TCP Client

## Creating a TCP client
最简单的创建TCP客户端的方式是使用默认的`NetClientOptions`：
```java
NetClient client = vertx.createNetClient();
```

## Configuring a TCP client
如果你不想要使用默认的`NetClientOptions`配置,那么你可以创建一个`NetClientOptions`实例进行TCP客户端创建：
```java
NetClientOptions options = new NetClientOptions().setConnectTimeout(10000);
NetClient client = vertx.createNetClient(options);
```

## Making connections
为了和服务器创建一个连接,你需要使用`connect`方法,在该方法中需要指定`hsot`和`port`,同时需要设置一个`handler`,当连接成功或者失败之后,`handler`会获得一个`NetSocket`的参数.
```java
NetClientOptions options = new NetClientOptions().setConnectTimeout(10000);
NetClient client = vertx.createNetClient(options);
client.connect(4321, "localhost", res -> {
  if (res.succeeded()) {
    System.out.println("Connected!");
    NetSocket socket = res.result();
  } else {
    System.out.println("Failed to connect: " + res.cause().getMessage());
  }
});
```

## Configuring connection attempts
A client can be configured to automatically retry connecting to the server in the event that it cannot connect. This is configured with setReconnectInterval and setReconnectAttempts.
客户端可以被配制成当连接不成功的时候在`event`里自动响应服务器的应答. 通过`setReconnectInterval`和`setReconnectAttempts`来设置这种机制.
> NOTE
> Currently Vert.x will not attempt to reconnect if a connection fails, reconnect attempts and interval only apply to creating initial connections.
> 注意：当连接失败之后,Vertx不会尝试自动重连,

```java
NetClientOptions options = new NetClientOptions();
options.setReconnectAttempts(10).setReconnectInterval(500);

NetClient client = vertx.createNetClient(options);
```

在默认情况下,创建多个连接是会失败的.
