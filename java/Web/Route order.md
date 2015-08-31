# Route order

在默认情况下`routes`的排序是按照添加添加进`router`的顺序进行排序的.

当`router`接受到一个请求时, `router`会遍历自身的每一个`route`查看是否与请求匹配,如果匹配的话,该route的`handler`就会被调用.

如果`handler`随后调用了下一个`handler`, 那么下一个与之匹配的`route`也会被调用.
```java
Route route1 = router.route("/some/path/").handler(routingContext -> {

  HttpServerResponse response = routingContext.response();
  // enable chunked responses because we will be adding data as
  // we execute over other handlers. This is only required once and
  // only if several handlers do output.
  response.setChunked(true);

  response.write("route1\n");

  // Now call the next matching route
  routingContext.next();
});

Route route2 = router.route("/some/path/").handler(routingContext -> {

  HttpServerResponse response = routingContext.response();
  response.write("route2\n");

  // Now call the next matching route
  routingContext.next();
});

Route route3 = router.route("/some/path/").handler(routingContext -> {

  HttpServerResponse response = routingContext.response();
  response.write("route3");

  // Now end the response
  routingContext.response().end();
});
```
在上面的例子中，输出结果为：
```
route1
route2
route3
```

`/some/path`开头的请求都是按照刚才的那个顺序进行`route`调用的.

如果你想要改变`route`的调用顺序,你可以使用`order()`方法,向其指定一个指定值.


Routes are assigned an order at creation time corresponding to the order in which they were added to the router, with the first route numbered 0, the second route numbered 1, and so on.

By specifying an order for the route you can override the default ordering. Order can also be negative, e.g. if you want to ensure a route is evaluated before route number 0.

Let’s change the ordering of route2 so it runs before route1:
```java
Route route1 = router.route("/some/path/").handler(routingContext -> {

  HttpServerResponse response = routingContext.response();
  response.write("route1\n");

  // Now call the next matching route
  routingContext.next();
});

Route route2 = router.route("/some/path/").handler(routingContext -> {

  HttpServerResponse response = routingContext.response();
  // enable chunked responses because we will be adding data as
  // we execute over other handlers. This is only required once and
  // only if several handlers do output.
  response.setChunked(true);

  response.write("route2\n");

  // Now call the next matching route
  routingContext.next();
});

Route route3 = router.route("/some/path/").handler(routingContext -> {

  HttpServerResponse response = routingContext.response();
  response.write("route3");

  // Now end the response
  routingContext.response().end();
});

// Change the order of route2 so it runs before route1
route2.order(-1);
```
then the response will now contain:
```
route2
route1
route3
```
If two matching routes have the same value of order, then they will be called in the order they were added.

You can also specify that a route is handled last, with last

