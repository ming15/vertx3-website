# Routing by exact path

`route`可以被设定为匹配特定的`URI`. 在这种情况下,它就可以指定路径的请求了.

在下面的例子中,`handler`会被路径为`/some/path/`的请求调用. 但是我们会忽略末尾斜杠,因此当路径为`/some/path`和`/some/path//`该handler都会被调用.

```java
Route route = router.route().path("/some/path/");

route.handler(routingContext -> {
  // This handler will be called for the following request paths:

  // `/some/path`
  // `/some/path/`
  // `/some/path//`
  //
  // but not:
  // `/some/path/subdir`
});
```