# Context data

在一个请求的生命周期之内, 你可以将想要在`handler`之间共享的数据放在`RoutingContext`中进行传递.

下面的例子中,使用`put`添加数据, 使用`get`检索数据.

```java
router.get("/some/path").handler(routingContext -> {

  routingContext.put("foo", "bar");
  routingContext.next();

});

router.get("/some/path/other").handler(routingContext -> {

  String bar = routingContext.get("foo");
  // Do something with bar
  routingContext.response().end();

});
```

