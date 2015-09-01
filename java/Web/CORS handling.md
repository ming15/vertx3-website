# CORS handling

`Cross Origin Resource Sharing`是一个安全的资源请求途径(AJAX跨域问题的解决方案)

`Vert.x-Web`包含了一个`CorsHandler`, 用于处理`CORS`协议. 例如

```java
router.route().handler(CorsHandler.create("vertx\\.io").allowedMethod(HttpMethod.GET));

router.route().handler(routingContext -> {

  // Your app handlers

});
```
TODO more CORS docs

