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

`Routes`在`router`中的位置是按照他们添加进去的时间顺序进行排序的, 而且他们的位置是从0开始的.

当然像上文所说的,你还可以调用`order()`方法改变这个排序. 需要注意的是序号可以是负数, 例如你想要某个`route`在序号0的`route`之前执行,你就可以将某个`route`序号指定为-1.

下例中我们改变了`route2`的序号,确保他在`route1`之前执行.
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
接下来我们就看到了我们所期望的结果
```
route2
route1
route3
```

如果俩个`route`有相同的序号,那么他们会按照添加的顺序进行执行. 你还可以通过调用`last()`方法将某个`route`放到最后一个.