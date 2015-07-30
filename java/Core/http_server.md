# HTTP Server
## Creating an HTTP Server
我们使用全部默认选项创建一个非常简单的HTTP服务器：
```java
HttpServer server = vertx.createHttpServer();
```

## Configuring an HTTP server

如果想创建一个自配置的HTTP服务器也很简单,你只需要在创建的时候,创建一个`HttpServerOptions`参数就可以了：
```java
HttpServerOptions options = new HttpServerOptions().setMaxWebsocketFrameSize(1000000);

HttpServer server = vertx.createHttpServer(options);
```

## Start the Server Listening

接下来我们使用`listen()`方法,让服务器开始监听客户端的请求.

```java
HttpServer server = vertx.createHttpServer();
server.listen();
```
或者我们指定要监听的端口和主机地址(这种方式会忽略掉在`HttpServerOptions`中配置的端口和主机地址)
```java
HttpServer server = vertx.createHttpServer();
server.listen(8080, "myhost.com");
```
如果不指定主机和端口的话,默认监听的主机地址是`0.0.0.0`(这意味着在所有可用的主机地址上进行绑定),默认的端口是80

The actual bind is asynchronous so the server might not actually be listening until some time after the call to listen has returned.

实际上这个绑定操作(`listen()`)是异步进行着,这意味着可能要等到

If you want to be notified when the server is actually listening you can provide a handler to the listen call. For example:
```java
HttpServer server = vertx.createHttpServer();
server.listen(8080, "myhost.com", res -> {
  if (res.succeeded()) {
    System.out.println("Server is now listening!");
  } else {
    System.out.println("Failed to bind!");
  }
});
```

## Getting notified of incoming requests
To be notified when a request arrives you need to set a requestHandler:
```java
HttpServer server = vertx.createHttpServer();
server.requestHandler(request -> {
  // Handle the request in here
});
```

## Handling requests
When a request arrives, the request handler is called passing in an instance of HttpServerRequest. This object represents the server side HTTP request.

The handler is called when the headers of the request have been fully read.

If the request contains a body, that body will arrive at the server some time after the request handler has been called.

The server request object allows you to retrieve the uri, path, params and headers, amongst other things.

Each server request object is associated with one server response object. You use response to get a reference to the HttpServerResponse object.

Here’s a simple example of a server handling a request and replying with "hello world" to it.
```java
vertx.createHttpServer().requestHandler(request -> {
  request.response().end("Hello world");
}).listen(8080);
```

#### Request version

The version of HTTP specified in the request can be retrieved with version

#### Request method

Use method to retrieve the HTTP method of the request. (i.e. whether it’s GET, POST, PUT, DELETE, HEAD, OPTIONS, etc).

#### Request URI

Use uri to retrieve the URI of the request.

Note that this is the actual URI as passed in the HTTP request, and it’s almost always a relative URI.

The URI is as defined in Section 5.1.2 of the HTTP specification - Request-URI

#### Request path

Use path to return the path part of the URI

For example, if the request URI was:

a/b/c/page.html?param1=abc&param2=xyz
Then the path would be

/a/b/c/page.html

#### Request query

Use query to return the query part of the URI

For example, if the request URI was:

a/b/c/page.html?param1=abc&param2=xyz
Then the query would be

param1=abc&param2=xyz

#### Request headers

Use headers to return the headers of the HTTP request.

This returns an instance of MultiMap - which is like a normal Map or Hash but allows multiple values for the same key - this is because HTTP allows multiple header values with the same key.

It also has case-insensitive keys, that means you can do the following:

MultiMap headers = request.headers();

// Get the User-Agent:
System.out.println("User agent is " + headers.get("user-agent"));

// You can also do this and get the same result:
System.out.println("User agent is " + headers.get("User-Agent"));

#### Request parameters

Use params to return the parameters of the HTTP request.

Just like headers this returns an instance of MultiMap as there can be more than one parameter with the same name.

Request parameters are sent on the request URI, after the path. For example if the URI was:

/page.html?param1=abc&param2=xyz
Then the parameters would contain the following:

param1: 'abc'
param2: 'xyz
Note that these request parameters are retrieved from the URL of the request. If you have form attributes that have been sent as part of the submission of an HTML form submitted in the body of a multi-part/form-data request then they will not appear in the params here.

#### Remote address

The address of the sender of the request can be retrieved with remoteAddress.

#### Absolute URI

The URI passed in an HTTP request is usually relative. If you wish to retrieve the absolute URI corresponding to the request, you can get it with absoluteURI

#### End handler

The endHandler of the request is invoked when the entire request, including any body has been fully read.

#### Reading Data from the Request Body

Often an HTTP request contains a body that we want to read. As previously mentioned the request handler is called when just the headers of the request have arrived so the request object does not have a body at that point.

This is because the body may be very large (e.g. a file upload) and we don’t generally want to buffer the entire body in memory before handing it to you, as that could cause the server to exhaust available memory.

To receive the body, you can use the handler on the request, this will get called every time a chunk of the request body arrives. Here’s an example:
```java
request.handler(buffer -> {
  System.out.println("I have received a chunk of the body of length " + buffer.length());
});
```
The object passed into the handler is a Buffer, and the handler can be called multiple times as data arrives from the network, depending on the size of the body.

In some cases (e.g. if the body is small) you will want to aggregate the entire body in memory, so you could do the aggregation yourself as follows:
```java
Buffer totalBuffer = Buffer.buffer();

request.handler(buffer -> {
  System.out.println("I have received a chunk of the body of length " + buffer.length());
  totalBuffer.appendBuffer(buffer);
});

request.endHandler(v -> {
  System.out.println("Full body received, length = " + totalBuffer.length());
});
```
This is such a common case, that Vert.x provides a bodyHandler to do this for you. The body handler is called once when all the body has been received:
```java
request.bodyHandler(totalBuffer -> {
  System.out.println("Full body received, length = " + totalBuffer.length());
});
```
#### Pumping requests

The request object is a ReadStream so you can pump the request body to any WriteStream instance.

See the chapter on streams and pumps for a detailed explanation.

#### Handling HTML forms

HTML forms can be submitted with either a content type of application/x-www-form-urlencoded or multipart/form-data.

For url encoded forms, the form attributes are encoded in the url, just like normal query parameters.

For multi-part forms they are encoded in the request body, and as such are not available until the entire body has been read from the wire.

Multi-part forms can also contain file uploads.

If you want to retrieve the attributes of a multi-part form you should tell Vert.x that you expect to receive such a form before any of the body is read by calling setExpectMultipart with true, and then you should retrieve the actual attributes using formAttributes once the entire body has been read:
```java
server.requestHandler(request -> {
  request.setExpectMultipart(true);
  request.endHandler(v -> {
    // The body has now been fully read, so retrieve the form attributes
    MultiMap formAttributes = request.formAttributes();
  });
});
```

#### Handling form file uploads

Vert.x can also handle file uploads which are encoded in a multi-part request body.

To receive file uploads you tell Vert.x to expect a multi-part form and set an uploadHandler on the request.

This handler will be called once for every upload that arrives on the server.

The object passed into the handler is a HttpServerFileUpload instance.
```java
server.requestHandler(request -> {
  request.setExpectMultipart(true);
  request.uploadHandler(upload -> {
    System.out.println("Got a file upload " +  upload.name());
  });
});
```

File uploads can be large we don’t provide the entire upload in a single buffer as that might result in memory exhaustion, instead, the upload data is received in chunks:
```java
request.uploadHandler(upload -> {
  upload.handler(chunk -> {
    System.out.println("Received a chunk of the upload of length " + chunk.length());
  });
});
```
The upload object is a ReadStream so you can pump the request body to any WriteStream instance. See the chapter on streams and pumps for a detailed explanation.

If you just want to upload the file to disk somewhere you can use streamToFileSystem:
```java
request.uploadHandler(upload -> {
  upload.streamToFileSystem("myuploads_directory/" + upload.filename());
});
```java

WARNING
Make sure you check the filename in a production system to avoid malicious clients uploading files to arbitrary places on your filesystem. See security notes for more information.

## Sending back responses
The server response object is an instance of HttpServerResponse and is obtained from the request with response.

You use the response object to write a response back to the HTTP client.

#### Setting status code and message

The default HTTP status code for a response is 200, representing OK.

Use setStatusCode to set a different code.

You can also specify a custom status message with setStatusMessage.

If you don’t specify a status message, the default one corresponding to the status code will be used.

#### Writing HTTP responses

To write data to an HTTP response, you use one the write operations.

These can be invoked multiple times before the response is ended. They can be invoked in a few ways:

With a single buffer:
```java
HttpServerResponse response = request.response();
response.write(buffer);
```
With a string. In this case the string will encoded using UTF-8 and the result written to the wire.

HttpServerResponse response = request.response();
response.write("hello world!");
With a string and an encoding. In this case the string will encoded using the specified encoding and the result written to the wire.
```java
HttpServerResponse response = request.response();
response.write("hello world!", "UTF-16");
```
Writing to a response is asynchronous and always returns immediately after the write has been queued.

If you are just writing a single string or buffer to the HTTP response you can write it and end the response in a single call to the end

The first call to write results in the response header being being written to the response. Consequently, if you are not using HTTP chunking then you must set the Content-Length header before writing to the response, since it will be too late otherwise. If you are using HTTP chunking you do not have to worry.

#### Ending HTTP responses

Once you have finished with the HTTP response you should end it.

This can be done in several ways:

With no arguments, the response is simply ended.
```java
HttpServerResponse response = request.response();
response.write("hello world!");
response.end();
```

It can also be called with a string or buffer in the same way write is called. In this case it’s just the same as calling write with a string or buffer followed by calling end with no arguments. For example:
```java
HttpServerResponse response = request.response();
response.end("hello world!");
```java

#### Closing the underlying connection

You can close the underlying TCP connection with close.

Non keep-alive connections will be automatically closed by Vert.x when the response is ended.

Keep-alive connections are not automatically closed by Vert.x by default. If you want keep-alive connections to be closed after an idle time, then you configure setIdleTimeout.

#### Setting response headers

HTTP response headers can be added to the response by adding them directly to the headers:
```java
HttpServerResponse response = request.response();
MultiMap headers = response.headers();
headers.set("content-type", "text/html");
headers.set("other-header", "wibble");
```
Or you can use putHeader
```java
HttpServerResponse response = request.response();
response.putHeader("content-type", "text/html").putHeader("other-header", "wibble");
```
Headers must all be added before any parts of the response body are written.

#### Chunked HTTP responses and trailers

Vert.x supports HTTP Chunked Transfer Encoding.

This allows the HTTP response body to be written in chunks, and is normally used when a large response body is being streamed to a client and the total size is not known in advance.

You put the HTTP response into chunked mode as follows:
```java
HttpServerResponse response = request.response();
response.setChunked(true);
```
Default is non-chunked. When in chunked mode, each call to one of the write methods will result in a new HTTP chunk being written out.

When in chunked mode you can also write HTTP response trailers to the response. These are actually written in the final chunk of the response.

To add trailers to the response, add them directly to the trailers.
```java
HttpServerResponse response = request.response();
response.setChunked(true);
MultiMap trailers = response.trailers();
trailers.set("X-wibble", "woobble").set("X-quux", "flooble");
```
Or use putTrailer.
```java
HttpServerResponse response = request.response();
response.setChunked(true);
response.putTrailer("X-wibble", "woobble").putTrailer("X-quux", "flooble");
```

#### Serving files directly from disk

If you were writing a web server, one way to serve a file from disk would be to open it as an AsyncFile and pump it to the HTTP response.

Or you could load it it one go using readFile and write it straight to the response.

Alternatively, Vert.x provides a method which allows you to serve a file from disk to an HTTP response in one operation. Where supported by the underlying operating system this may result in the OS directly transferring bytes from the file to the socket without being copied through user-space at all.

This is done by using sendFile, and is usually more efficient for large files, but may be slower for small files.

Here’s a very simple web server that serves files from the file system using sendFile:
```java
vertx.createHttpServer().requestHandler(request -> {
  String file = "";
  if (request.path().equals("/")) {
    file = "index.html";
  } else if (!request.path().contains("..")) {
    file = request.path();
  }
  request.response().sendFile("web/" + file);
}).listen(8080);
```
Sending a file is asynchronous and may not complete until some time after the call has returned. If you want to be notified when the file has been writen you can use sendFile

>NOTE
If you use sendFile while using HTTPS it will copy through user-space, since if the kernel is copying data directly from disk to socket it doesn’t give us an opportunity to apply any encryption.

> WARNING
If you’re going to write web servers directly using Vert.x be careful that users cannot exploit the path to access files outside the directory from which you want to serve them. It may be safer instead to use Vert.x Apex.

#### Pumping responses

The server response is a WriteStream instance so you can pump to it from any ReadStream, e.g. AsyncFile, NetSocket, WebSocket or HttpServerRequest.

Here’s an example which echoes the request body back in the response for any PUT methods. It uses a pump for the body, so it will work even if the HTTP request body is much larger than can fit in memory at any one time:
```java
vertx.createHttpServer().requestHandler(request -> {
  HttpServerResponse response = request.response();
  if (request.method() == HttpMethod.PUT) {
    response.setChunked(true);
    Pump.pump(request, response).start();
    request.endHandler(v -> response.end());
  } else {
    response.setStatusCode(400).end();
  }
}).listen(8080);
```

## HTTP Compression
Vert.x comes with support for HTTP Compression out of the box.

This means you are able to automatically compress the body of the responses before they are sent back to the client.

If the client does not support HTTP compression the responses are sent back without compressing the body.

This allows to handle Client that support HTTP Compression and those that not support it at the same time.

To enable compression use can configure it with setCompressionSupported.

By default compression is not enabled.

When HTTP compression is enabled the server will check if the client incldes an Accept-Encoding header which includes the supported compressions. Commonly used are deflate and gzip. Both are supported by Vert.x.

If such a header is found the server will automatically compress the body of the response with one of the supported compressions and send it back to the client.

Be aware that compression may be able to reduce network traffic but is more CPU-intensive.
