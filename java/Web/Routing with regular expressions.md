# Routing with regular expressions

正则还可以被使用在`URI`路径的匹配上：
```java
Route route = router.route().pathRegex(".*foo");

route.handler(routingContext -> {

  // This handler will be called for:

  // /some/path/foo
  // /foo
  // /foo/bar/wibble/foo
  // /foo/bar

  // But not:
  // /bar/wibble
});
```
还有一种做法是,正则可以在创建`route`时进行指定.
```java
Route route = router.routeWithRegex(".*foo");

route.handler(routingContext -> {

  // This handler will be called same as previous example

});
```