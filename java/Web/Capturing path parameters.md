# Capturing path parameters

我们可以在请求参数上使用通配符进行匹配路径

```java
Route route = router.route(HttpMethod.POST, "/catalogue/products/:productype/:productid/");

route.handler(routingContext -> {

  String productType = routingContext.request().getParam("producttype");
  String productID = routingContext.request().getParam("productid");

  // Do something with them...
});
```

通配符由`:`组成,将其放在参数名后面. 参数名由字母和数字下划线组成.

在上面的例子中,如果一个`POST`请求地址是`/catalogue/products/tools/drill123/`, 那么上面的`route`会被匹配到, `productType`接收到`tools`值, `productID`会接收到`drill123`值.

