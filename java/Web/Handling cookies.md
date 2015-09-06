# Handling cookies

`Vert.x-Web`使用`CookieHandler`来支持`cookies`.

你必须确保当请求需要`cookies`支持的时候,你已经设置上了`cookie handler`。

```java
router.route().handler(CookieHandler.create());
```

### Manipulating cookies
你可以向`getCookie()`方法, 或者通过`cookies()`方法检索出`cookie`集合.

* `getCookie()`, 传递一个`cookie name`的参数来检索出一个`cookie`
* `cookies()`, 检索出`cookie`集合
* `removeCookie`, 删除一个`cookie`
* `addCookie`, 添加一个`cookie`

当`response headers`被写回的时候, `cookies`集合会自动的被写入到`response`中.

`Cookies`是通过`Cookie`实例进行描述的. 你可以通过该实例检索出`cookie`中的`name`, `value`, `domain`, `path` 或者其他的`cookie`属性.

下面的例子演示了如何检索和添加`cookie`
```java
router.route().handler(CookieHandler.create());

router.route("some/path/").handler(routingContext -> {

  Cookie someCookie = routingContext.getCookie("mycookie");
  String cookieValue = someCookie.getValue();

  // Do something with cookie...

  // Add a cookie - this will get written back in the response automatically
  routingContext.addCookie(Cookie.cookie("othercookie", "somevalue"));
});
```