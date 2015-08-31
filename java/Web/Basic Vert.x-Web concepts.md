# Basic Vert.x-Web concepts
Here’s the 10000 foot view:

`Router`是`Vert.x-Web`最核心的概念. `Router`是一个持有零到多个`Routes`的对象

`Router`会将`HTTP request`发送到第一个匹配该请求的`route`身上.

`route`持有一个与`HTTP request`相匹配的`handler`, 然后该`handler`接受该请求. 然后执行具体任务, 当执行完任务之后你可以选择结束该请求或者将它传递给下一个匹配的`handler`.

下面是一个简单的示例：
```java
HttpServer server = vertx.createHttpServer();

Router router = Router.router(vertx);

router.route().handler(routingContext -> {

  // This handler will be called for every request
  HttpServerResponse response = routingContext.response();
  response.putHeader("content-type", "text/plain");

  // Write to the response and end it
  response.end("Hello World from Vert.x-Web!");
});

server.requestHandler(router::accept).listen(8080);
```

我们创建了一个`HTTP Server`服务器, 接着创建了一个`router`. 我们没有对这个`route`指定匹配规则,因此它会匹配所有的`HTTP request`.

然后我们在该`route`上设置了一个`handler`, 这个`handler`会处理该服务器上所有的`HTTP request`.

传递给`handler`的是一个`RoutingContext`对象, 该对象包含一个一个标准的`Vert.x HttpServerRequest`和`Vert.x HttpServerResponse`,但是还包含了很多其他的`Vert.x-Web`里的特性.

对于每一个`HTTP request`都会生成一个唯一的`RoutingContext`实例, 但是给实例会传递给所有匹配该请求的`handler`.


