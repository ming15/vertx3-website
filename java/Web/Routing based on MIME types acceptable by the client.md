# Routing based on MIME types acceptable by the client

` HTTP accept header`常常用于表示客户端接收到的服务器响应的`MIME`类型.

`accept header`可以带有多个`MIME`类型,他们之间通过`','`分割.

`MIME`类型还可以有一个`q`值, 
MIME types can also have a q value appended to them* which signifies a weighting to apply if more than one response MIME type is available matching the accept header. The q value is a number between 0 and 1.0. If omitted it defaults to 1.0.

例如,下面的`accept header`表示只会接受`text/plain`的`MIME`类型.
```
Accept: text/plain
```
下面的`accept header`会接受`text/plain`和`text/html`的`MIME`类型,这俩者直接并没有优先级.
```
Accept: text/plain, text/html
```
但是下面的客户端会会接受`text/plain`和`text/html`的`MIME`类型,但是`text/html`的优先级高于`text/plain`, 因为`text/html`有一个更高的`q`值. (默认情况下`q=1`)
```
Accept: text/plain; q=0.9, text/html
```
如果服务器能够同时提供`text/plain`和`text/html`, 那么在这个例子中,他就应该提供`text/html`.

通过使用`produces`方法设置了`route`产生的`MIME`类型, 例如下面的`handler`设置了一个`MIME`类型为`application/json`的响应
```
router.route().produces("application/json").handler(routingContext -> {

  HttpServerResponse response = routingContext.response();
  response.putHeader("content-type", "application/json");
  response.write(someJSON).end();

});
```
在这个例子中,`route`会匹配到所有的` accept header`为`application/json`的请求.

下面是该`accept headers`的匹配值:
```
Accept: application/json
Accept: application/*
Accept: application/json, text/html
Accept: application/json;q=0.7, text/html;q=0.8, text/plain
```

你还可以设置你的`route`produce多个`MIME`类型. 在这种情况中, 你可以使用`getAcceptableContentType()`方法找到实际接受到的`MIME`类型
```
router.route().produces("application/json").produces("text/html").handler(routingContext -> {

  HttpServerResponse response = routingContext.response();

  // Get the actual MIME type acceptable
  String acceptableContentType = routingContext.getAcceptableContentType();

  response.putHeader("content-type", acceptableContentType);
  response.write(whatever).end();
});
```
在上面的例子中，如果你发送下面的`accept header`
```java
Accept: application/json; q=0.7, text/html
```
Then the route would match and acceptableContentType would contain text/html as both are acceptable but that has a higher q value.

