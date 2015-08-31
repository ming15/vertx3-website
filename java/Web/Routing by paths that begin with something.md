# Routing by paths that begin with something

通常情况下,你会想设置一个通用的前置路径. 如果是这种情况你可以使用正则表达式, 但是一个比较简单的实现方式是在`route path`的末尾加上一个`*`.

在下面的例子中,当请求路径前缀为`/some/path/`的时候,我们设置的`handler`都会被执行. (例如`/some/path/foo.html`和`/some/path/otherdir/blah.css`都是匹配的)

```java
Route route = router.route().path("/some/path/*");

route.handler(routingContext -> {
  // This handler will be called for any path that starts with
  // `/some/path/`, e.g.

  // `/some/path`
  // `/some/path/`
  // `/some/path/subdir`
  // `/some/path/subdir/blah.html`
  //
  // but not:
  // `/some/bath`
});
```
你还可以将路径参数放在`route()`方法里
```java
Route route = router.route("/some/path/*");

route.handler(routingContext -> {
  // This handler will be called same as previous example
});
```