# Are you fluent?

流式API(fluent API)是一种将方法进行链式调用的方式：
```
request.response().putHeader("Content-Type", "text/plain").write("some text").end();
```
在Vert.x APIs中，你都可以使用这种方式

进行链式调用可以避免你的代码看起来罗哩罗嗦的。当然，这并不是强制的，你也可以像下面这样书写你的代码。
```
HttpServerResponse response = request.response();
response.putHeader("Content-Type", "text/plain");
response.write("some text");
response.end();
```
