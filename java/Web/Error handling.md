# Error handling

我们可以设置`handler`来处理请求,同样我们还可以设置`handler`处理`route`中的失败情况.

`Failure handlers` 通常是和处理普通`handler`的`route`一起工作.

例如,你可以提供一个`failure handler`只是用来处理某种特定路径下的或者某种特定`HTTP method`的失败.

这种机制就为你向应用程序的不同部分设置不同的`failure handler`了.

下面的例子演示了我们向`/somepath/`开始的路径和`GET`请求才会被调用的`failure handler`.
```java
Route route = router.get("/somepath/*");

route.failureHandler(frc -> {

  // This will be called for failures that occur
  // when routing requests to paths starting with
  // '/somepath/'

});
```
如果`handler`中抛出异常`Failure routing`就会发生作用, 或者`handler`调用失败,向客户端发送一个失败的`HTTP`状态码信号.

如果我们从`handler`中捕获一个异常, 我们将会向客户端返回`500`的状态码.

在处理失败的时候, `failure handler`会被传递给`routing context`, 我们可以在该`context`中检索出当前的错误, 因此`failure handler`可以用于生成`failure response`。

```java
Route route1 = router.get("/somepath/path1/");

route1.handler(routingContext -> {

  // Let's say this throws a RuntimeException
  throw new RuntimeException("something happened!");

});

Route route2 = router.get("/somepath/path2");

route2.handler(routingContext -> {

  // This one deliberately fails the request passing in the status code
  // E.g. 403 - Forbidden
  routingContext.fail(403);

});

// Define a failure handler
// This will get called for any failures in the above handlers
Route route3 = router.get("/somepath/*");

route3.failureHandler(failureRoutingContext -> {

  int statusCode = failureRoutingContext.statusCode();

  // Status code will be 500 for the RuntimeException or 403 for the other failure
  HttpServerResponse response = failureRoutingContext.response();
  response.setStatusCode(statusCode).end("Sorry! Not today");

});
```