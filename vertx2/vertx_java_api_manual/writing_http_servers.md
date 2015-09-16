# Writing HTTP servers

Vert.x能帮你完成一个全功能的高性能的可扩展的HTTP服务器

#### Creating an HTTP Server

调用`vertx`对象上的`createHttpServer`就可以创建一个HTTP服务器
```java
HttpServer server = vertx.createHttpServer();
```

#### Start the Server Listening

然后我们调用`listen`绑定土匪用于监听要接收处理的请求的端口
```java
HttpServer server = vertx.createHttpServer();

server.listen(8080, "myhost");
```
1. 第一个参数是绑定的端口号
2. 第二个参数是主机域名或者IP地址。如果忽略该参数，则服务器会采取默认值`0.0.0.0`,服务器会在所有可用的网络接口中监听绑定的端口号

实际上绑定操作是异步进行的，也就是当`listen`方法返回之后，并不意味着就绑定成功了。如果你想当绑定真正完成的时候，你可以向`listen`方法传递一个handler，用以接受绑定成功之后的通知
```java
server.listen(8080, "myhost", new AsyncResultHandler<Void>() {
    public void handle(AsyncResult<HttpServer> asyncResult) {
        log.info("Listen succeeded? " + asyncResult.succeeded());
    }
});
```

#### Getting Notified of Incoming Requests

我们还要设置一个request handler,这个handler是为了当请求到来时，我们能收到通知:
```java
HttpServer server = vertx.createHttpServer();

server.requestHandler(new Handler<HttpServerRequest>() {
    public void handle(HttpServerRequest request) {
        log.info("A request has arrived on the server!");
        request.response().end();
    }
});

server.listen(8080, "localhost");
```

每当有请求到来时，该handler都会被调用一次，然后向handler方法传递一个`org.vertx.java.core.http.HttpServerRequest`参数

你可以在verticle中实现一个HTTP 服务器,然后在浏览器里输入`http://localhost:8080`测试一下

和`NetServer`一样,`requestHandler`方法返回的也是它自身,因此我们也可以使用链式调用模式
```java
HttpServer server = vertx.createHttpServer();

server.requestHandler(new Handler<HttpServerRequest>() {
    public void handle(HttpServerRequest request) {
        log.info("A request has arrived on the server!");
        request.response().end();
    }
}).listen(8080, "localhost");
```
Or:
```java
vertx.createHttpServer().requestHandler(new Handler<HttpServerRequest>() {
    public void handle(HttpServerRequest request) {
        log.info("A request has arrived on the server!");
        request.response().end();
    }
}).listen(8080, "localhost");
```

#### Handling HTTP Requests

到目前为止，我们看到了如何创建一个`HttpServer`以及如何捕获通知,下面让我们看一下如何处理接受到的请求：

当捕获到一个请求时，会将请求封装到一个`HttpServerRequest`中，接着request handler会被调用。

The handler is called when the headers of the request have been fully read. If the request contains a body, that body may arrive at the server some time after the request handler has been called.

当请求的header被全部读取完之后，该handler就会被调用. 如果请求中包含body，body也许会在request handler被调用之后才达到。

`HttpServerRequest`包含有`get the URI, path, request headers and request parameters`等功能。我们还可以通过调用该对象的`response()`方法来获得一个表示服务器向客户端进行回应的对象。

##### Request Method

`HttpServerRequest`的`method()`表示的是请求使用的`HTTP method`(该方法的可能返回值有`GET, PUT, POST, DELETE, HEAD, OPTIONS, CONNECT, TRACE, PATCH`).

##### Request Version

`HttpServerRequest`的`version()`方法返回的当前请求使用的`HTTP`版本号

##### Request URI

`HttpServerRequest`的`rui()`方法返回的完整的`URI(Uniform Resource Locator)地址`，例如：
```java
/a/b/c/page.html?param1=abc&param2=xyz
```

`uri()`返回将会返回`/a/b/c/page.html?param1=abc&param2=xyz`

请求使用的`URI`地址可以是绝对的，也可以是相对的，这取决于客户端使用的什么,在大多数情况下使用的都是绝对的

##### Request Path

`HttpServerRequest`的`path()`方法返回的是请求路径，例如：
```java
a/b/c/page.html?param1=abc&param2=xyz
```

`request.path()`将返回`/a/b/c/page.html`

##### Request Query

`HttpServerRequest`的`query()`方法返回的是请求查询内容，例如
```
a/b/c/page.html?param1=abc&param2=xyz
```
`request.query()`将返回`param1=abc&param2=xyz`

##### Request Headers

我们可以在`HttpServerRequest`的对象上通过`headers()`方法获取请求的请求头(`org.vertx.java.core.MultiMap`对象)。`MultiMap`允许一个key有多个值(这让人想起的guava)

下面的例子对`http://localhost:8080`请求输出了请求头
```java
HttpServer server = vertx.createHttpServer();

server.requestHandler(new Handler<HttpServerRequest>() {
    public void handle(HttpServerRequest request) {
        StringBuilder sb = new StringBuilder();
        for (Map.Entry<String, String> header: request.headers().entries()) {
            sb.append(header.getKey()).append(": ").append(header.getValue()).append("\n");
        }
        request.response().putHeader("content-type", "text/plain");
        request.response().end(sb.toString());
    }
}).listen(8080, "localhost");
```

##### Request params

我们通过`HttpServerRequest`的`params()`方法获得请求的请求参数,同样请求参数也是用`org.vertx.java.core.MultiMap`存储.

例如：
```java
/page.html?param1=abc&param2=xyz
```
Then the params multimap would contain the following entries:
```
param1: 'abc'
param2: 'xyz
```

##### Remote Address
`HttpServerRequest`的`remoteAddress()`返回的是HTTP连接另一端的地址（也就是客户端）

##### Absolute URI

`HttpServerRequest`的`absoluteURI()`返回的是请求的相对`URI`地址

##### Reading Data from the Request Body

有时候我们需要向HTTP body中读取数据。像前面介绍的，当请求头被完整读取出来之后，request handler就会被调用，同时封装一个`HttpServerRequest`对象传递给该`handler`，但是该对象并不包含body。这么做是因为，body也许非常大，我们不希望可能因为超过可用内存而引发任何问题。

如果，你想要读取body数据，那么你只需要调用`HttpServerRequest`的`dataHandler`方法,通过该方法设置一个data handler，每当接受一次request body块都会调用一次该handler。
```java
HttpServer server = vertx.createHttpServer();

server.requestHandler(new Handler<HttpServerRequest>() {
    public void handle(HttpServerRequest request) {
        request.dataHandler(new Handler<Buffer>() {
            public void handle(Buffer buffer) {
                log.info('I received ' + buffer.length() + ' bytes');
            }
        });

    }
}).listen(8080, "localhost");
```

`dataHandler`可能不仅仅被调用一次，调用的次数取决于`body`的大小

这和`NetSocket`中去读数据非常像

`HttpServerRequest`实现了`ReadStream`接口,因此你可以将body转接到一个`WriteStream`中。

在大多数情况下，body并不是非常大而且我们想要一次性就接受到整个body数据，那么你可以像下面这样操作：
```java
HttpServer server = vertx.createHttpServer();

server.requestHandler(new Handler<HttpServerRequest>() {
    public void handle(HttpServerRequest request) {

        final Buffer body = new Buffer(0);

        request.dataHandler(new Handler<Buffer>() {
            public void handle(Buffer buffer) {
                body.appendBuffer(buffer);
            }
        });
        request.endHandler(new VoidHandler() {
            public void handle() {
              // The entire body has now been received
              log.info("The total body received was " + body.length() + " bytes");
            }
        });

    }
}).listen(8080, "localhost");
```

和任何`ReadStream`的实现类一样,当`stream`读到尾之后，end handler就会被调用。

如果`HTTP`请求使用了`HTTP chunking`,那么每次接收到body里每个HTTP chunk时都会调用一次data handler。

如果想要接收到完整的body数据再解析它的话，这是一种非常通用的用法，因此`Vert.x`提供了一个`bodyHandler`方法

`bodyHandler`方法设置的handler，只有当整个body数据接受完之后才会被调用

当body数据非常大的时候，vert.x会将整个body数据换存储在内存里
```java
HttpServer server = vertx.createHttpServer();

server.requestHandler(new Handler<HttpServerRequest>() {
    public void handle(HttpServerRequest request) {
        request.bodyHandler(new Handler<Buffer>() {
            public void handle(Buffer body) {
              // The entire body has now been received
              log.info("The total body received was " + body.length() + " bytes");
            }
        });
    }
}).listen(8080, "localhost");
```

##### Handling Multipart Form Uploads

Vert.x能够理解`HTML`表单里的文件上传操作. 为了能处理文件上传，你需要在request对象上设置`uploadHandler`。表单中每上传一个文件，设置的handler都会被调用一次
```java
request.expectMultiPart(true);

request.uploadHandler(new Handler<HttpServerFileUpload>() {
    public void handle(HttpServerFileUpload upload) {
    }
});
```

`HttpServerFileUpload`类实现了`ReadStream`，因此你可以从该类中读取数据,然后将数据再写入任何实现了`WriteStream`的对象,例如前文一直提到的`Pump`

你也可以直接使用`streamToFileSystem()`方法将文件直接输出磁盘上
```java
request.expectMultiPart(true);

request.uploadHandler(new Handler<HttpServerFileUpload>() {
    public void handle(HttpServerFileUpload upload) {
        upload.streamToFileSystem("uploads/" + upload.filename());
    }
});
```

##### Handling Multipart Form Attributes

如果客户端发送过来的请求是一个HTML表单请求，那么你可以使用`formAttributes`读取请求属性列表。我们要确保请求的全部内容(包含header和body)都被读取之后，采取调用`formAttributes`,这是因为表单属性都存储在了body里
```java
request.endHandler(new VoidHandler() {
    public void handle() {
        // The request has been all ready so now we can look at the form attributes
        MultiMap attrs = request.formAttributes();
        // Do something with them
    }
});
```

#### Setting Status Code and Message

我们使用`setStatusCode()`可以设置返回给客户端的HTTP状态码
```java
HttpServer server = vertx.createHttpServer();

server.requestHandler(new Handler<HttpServerRequest>() {
    public void handle(HttpServerRequest request) {
        request.response().setStatusCode(739).setStatusMessage("Too many gerbils").end();
    }
}).listen(8080, "localhost");
```
你还可以使用`setStatusMessage()`设置状态消息,如果你不进行手动设置的话，那就会采取默认值

默认的状态码是200

#### Writing HTTP responses

如果你想要向HTTP response写入数据，你直接调用`write`方法就好了。当response结束之前，你可以多次调用`write`方法。
```java
Buffer myBuffer = ...
request.response().write(myBuffer);
```
向response中写入使用`UTF-8`编码的字符串
```
request.response().write("hello");
```

使用指定的编码对字符串进行编码然后写入到response中
```
request.response().write("hello", "UTF-16");
```

`write`方法同样是异步的，当write放入到socket的写入队列之后就直接返回(但是此时并不意味着数据就已经写出了)

如果你只想向HTTP response写入一个String或者一个Buffer，那么当你调用完`write`方法之后，再调用一次`end`方法就可以了

The first call to write results in the response header being being written to the response.



因此，如果你没使用HTTP chunking，那么当你向response写入数据之前，必须设置`Content-Length header`。

#### Ending HTTP responses

当你已经向HTTP response写完数据之后，必须手动调用`end()`方法

`end`方法有如下几种调用方式：
```
request.response().end();
```

下面这种方式和先调用一个`write(String)`再调用`end()`方法是一样的
```
request.response().end("That's all folks");
```

#### Closing the underlying connection

你可以通过调用`close()`方法关闭掉当前请求的底层TCP连接
```
request.response().close();
```

#### Response headers

我们可以通过调用`headers()`方法获得`response header`（`Multimap`），然后通过`set`方法向其添加`response header`
```
request.response().headers().set("Cheese", "Stilton");
request.response().headers().set("Hat colour", "Mauve");
```

我们还可以通过链式模式调用`putHeader`方法向HTTP response header添加属性
```
request.response().putHeader("Some-Header", "elephants").putHeader("Pants", "Absent");
```

response header必须在写入body动作之前写入

#### Chunked HTTP Responses and Trailers

Vert.x支持`HTTP Chunked Transfer Encoding`, 这种模式会将HTTP response body以chunk的方式写入到socket中，当向clent输出的response body非常大，且其大小未知时，这是非常有用的。

```
req.response().setChunked(true);
```

response的默认值是`non-chunked`,当在chunked模式下，每一次调用`response.write(...)`都会创建一个新的`HTTP chunk`写入到socket流中

在chunked模式下，你还可以向response中写入`HTTP response trailers`,这些数据实际上是被写入到最后一个chunk中。

你可以向下面这样，通过调用`trailers()`方法向`HTTP response trailers`中写入数据。
```
request.response().trailers().add("Philosophy", "Solipsism");
request.response().trailers().add("Favourite-Shakin-Stevens-Song", "Behind the Green Door");
```
Like headers, individual HTTP response trailers can also be written using the putTrailer() method. This allows a fluent API since calls to putTrailer can be chained:
```
request.response().putTrailer("Cat-Food", "Whiskas").putTrailer("Eye-Wear", "Monocle");
```

### Serving files directly from disk

If you were writing a web server, one way to serve a file from disk would be to open it as an AsyncFile and pump it to the HTTP response. Or you could load it it one go using the file system API and write that to the HTTP response.

Alternatively, Vert.x provides a method which allows you to serve a file from disk to an HTTP response in one operation. Where supported by the underlying operating system this may result in the OS directly transferring bytes from the file to the socket without being copied through userspace at all.

Using sendFile is usually more efficient for large files, but may be slower for small files than using readFile to manually read the file as a buffer and write it directly to the response.

To do this use the sendFile function on the HTTP response. Here's a simple HTTP web server that serves static files from the local web directory:
```
HttpServer server = vertx.createHttpServer();

server.requestHandler(new Handler<HttpServerRequest>() {
    public void handle(HttpServerRequest req) {
      String file = "";
      if (req.path().equals("/")) {
        file = "index.html";
      } else if (!req.path().contains("..")) {
        file = req.path();
      }
      req.response().sendFile("web/" + file);
    }
}).listen(8080, "localhost");
```
There's also a version of sendFile which takes the name of a file to serve if the specified file cannot be found:
```
req.response().sendFile("web/" + file, "handler_404.html");
```
Note: If you use sendFile while using HTTPS it will copy through userspace, since if the kernel is copying data directly from disk to socket it doesn't give us an opportunity to apply any encryption.

If you're going to write web servers using Vert.x be careful that users cannot exploit the path to access files outside the directory from which you want to serve them.

### Pumping Responses

Since the HTTP Response implements WriteStream you can pump to it from any ReadStream, e.g. an AsyncFile, NetSocket, WebSocket or HttpServerRequest.

Here's an example which echoes HttpRequest headers and body back in the HttpResponse. It uses a pump for the body, so it will work even if the HTTP request body is much larger than can fit in memory at any one time:
```
HttpServer server = vertx.createHttpServer();

server.requestHandler(new Handler<HttpServerRequest>() {
    public void handle(final HttpServerRequest req) {
      req.response().headers().set(req.headers());
      Pump.createPump(req, req.response()).start();
      req.endHandler(new VoidHandler() {
        public void handle() {
            req.response().end();
        }
      });
    }
}).listen(8080, "localhost");
```

### HTTP Compression

Vert.x comes with support for HTTP Compression out of the box. Which means you are able to automatically compress the body of the responses before they are sent back to the Client. If the client does not support HTTP Compression the responses are sent back without compressing the body. This allows to handle Client that support HTTP Compression and those that not support it at the same time.

To enable compression you only need to do:
```
HttpServer server = vertx.createHttpServer();
server.setCompressionSupported(true);
```
The default is false.

When HTTP Compression is enabled the HttpServer will check if the client did include an 'Accept-Encoding' header which includes the supported compressions. Common used are deflate and gzip. Both are supported by Vert.x. Once such a header is found the HttpServer will automatically compress the body of the response with one of the supported compressions and send it back to the client.

Be aware that compression may be able to reduce network traffic but is more cpu-intensive.
