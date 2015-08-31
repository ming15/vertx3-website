# Capturing path parameters with regular expressions

当使用正则表达式的时候,你还可以捕获路径参数：

```java
Route route = router.routeWithRegex(".*foo");

// This regular expression matches paths that start with something like:
// "/foo/bar" - where the "foo" is captured into param0 and the "bar" is captured into
// param1
route.pathRegex("\\/([^\\/]+)\\/([^\\/]+)").handler(routingContext -> {

  String productType = routingContext.request().getParam("param0");
  String productID = routingContext.request().getParam("param1");

  // Do something with them...
});
```
在上面的例子中,如果请求路径是`/tools/drill123/`, 那么我们设置的`route`会被匹配到, 然后`productType`会接收到参数值`tools`, `productID`会接收到参数值`drill123`.


Captures are denoted in regular expressions with capture groups (i.e. surrounding the capture with round brackets)

