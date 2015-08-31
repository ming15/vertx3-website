# Timeout handler

`Vert.x-Web`包含一个超时`handler`,你可以使用它应付某些操作执行时间太长的请求.

我们通过一个`TimeoutHandler`实例来配置它.

如果一个请求在写回之前超时了，那么4.8响应将会写回个给客户端.

下面的例子使用了一个超时`handler`:
```java
router.route("/foo/").handler(TimeoutHandler.create(5000));
```