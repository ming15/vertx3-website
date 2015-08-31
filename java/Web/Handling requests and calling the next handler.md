# Handling requests and calling the next handler

当`Vert.x-Web``route`一个`HTTP Request`到一个与之匹配的`route`，它会向该`route`的`handler`传递一个`RoutingContext`实例.

如果在当前`handler`里,你不想结束`response`, 那么你应该调用下一个相匹配的`route`继续处理该请求.

你没有必要在当前`handler`执行完之前调用下一个`route`, 你可以稍后再做这件事.

```java
Route route1 = router.route("/some/path/").handler(routingContext -> {

  HttpServerResponse response = routingContext.response();
  // enable chunked responses because we will be adding data as
  // we execute over other handlers. This is only required once and
  // only if several handlers do output.
  response.setChunked(true);

  response.write("route1\n");

  // Call the next matching route after a 5 second delay
  routingContext.vertx().setTimer(5000, tid -> routingContext.next());
});

Route route2 = router.route("/some/path/").handler(routingContext -> {

  HttpServerResponse response = routingContext.response();
  response.write("route2\n");

  // Call the next matching route after a 5 second delay
  routingContext.vertx().setTimer(5000, tid ->  routingContext.next());
});

Route route3 = router.route("/some/path/").handler(routingContext -> {

  HttpServerResponse response = routingContext.response();
  response.write("route3");

  // Now end the response
  routingContext.response().end();
});
```

在上面的例子中, `route1`被写到`response`的5秒钟之后,`route2`被写到`response`,在过了5秒钟之后,`route3`被写到`response`,然后结束掉`response`.

注意这一切的发生都是非阻塞的,
