# Routing by HTTP method

在默认的情况下`route`会匹配所有的`HTTP methods`.

如果你想要某个`route`只匹配特定的`HTTP method`,你可以像下面这样做：
```java
Route route = router.route().method(HttpMethod.POST);

route.handler(routingContext -> {

  // This handler will be called for any POST request

});
```
或者你在创建`route`时直接指定：
```
Route route = router.route(HttpMethod.POST, "/some/path/");

route.handler(routingContext -> {

  // This handler will be called for any POST request to a URI path starting with /some/path/

});
```
当然还有其他方式可用,你可以直接调用`get()`, `post`, `put`等方法调用
```
router.get().handler(routingContext -> {

  // Will be called for any GET request

});

router.get("/some/path/").handler(routingContext -> {

  // Will be called for any GET request to a path
  // starting with /some/path

});

router.getWithRegex(".*foo").handler(routingContext -> {

  // Will be called for any GET request to a path
  // ending with `foo`

});
```
如果你想要对某个`route`指定多个`HTTP method`,你可以像下面这样做：
```
Route route = router.route().method(HttpMethod.POST).method(HttpMethod.PUT);

route.handler(routingContext -> {

  // This handler will be called for any POST or PUT request

});
```