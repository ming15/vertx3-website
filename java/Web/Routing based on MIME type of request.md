# Routing based on MIME type of request

你可以通过`consumes`方法指定`route`需要匹配请求的`MIME`类型.

这下面的例子中, 请求包含了一个`content-type`请求头,该值指定了请求体的`MINE`类型. 这个值会和`consumes`方法里的值进行匹配.

`MINE`类型的匹配可以做到精确匹配.
```
router.route().consumes("text/html").handler(routingContext -> {

  // This handler will be called for any request with
  // content-type header set to `text/html`

});
```
同样我们还可以进行多个`MINE`类型的匹配
```
router.route().consumes("text/html").consumes("text/plain").handler(routingContext -> {

  // This handler will be called for any request with
  // content-type header set to `text/html` or `text/plain`.

});
```
我们还可以通过通配符对子类型进行匹配
```
router.route().consumes("text/*").handler(routingContext -> {

  // This handler will be called for any request with top level type `text`
  // e.g. content-type header set to `text/html` or `text/plain` will both match

});
```
我们还可以通过通配符对父类型进行匹配
```
router.route().consumes("*/json").handler(routingContext -> {

  // This handler will be called for any request with sub-type json
  // e.g. content-type header set to `text/json` or `application/json` will both match

});
```

如果你在`consumers`不指定`/`, 它会假定你指的是子类型.