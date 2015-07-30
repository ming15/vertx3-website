# HTTP client
Creating an HTTP client
You create an HttpClient instance with default options as follows:

HttpClient client = vertx.createHttpClient();
If you want to configure options for the client, you create it as follows:

HttpClientOptions options = new HttpClientOptions().setKeepAlive(false);
HttpClient client = vertx.createHttpClient();
Making requests
The http client is very flexible and there are various ways you can make requests with it.

Often you want to make many requests to the same host/port with an http client. To avoid you repeating the host/port every time you make a request you can configure the client with a default host/port:

HttpClientOptions options = new HttpClientOptions().setDefaultHost("wibble.com");
// Can also set default port if you want...
HttpClient client = vertx.createHttpClient(options);
client.getNow("/some-uri", response -> {
  System.out.println("Received response with status code " + response.statusCode());
});
Alternatively if you find yourself making lots of requests to different host/ports with the same client you can simply specify the host/port when doing the request.

HttpClient client = vertx.createHttpClient();

// Specify both port and host name
client.getNow(8080, "myserver.mycompany.com", "/some-uri", response -> {
  System.out.println("Received response with status code " + response.statusCode());
});

// This time use the default port 80 but specify the host name
client.getNow("foo.othercompany.com", "/other-uri", response -> {
  System.out.println("Received response with status code " + response.statusCode());
});
Both methods of specifying host/port are supported for all the different ways of making requests with the client.

Simple requests with no request body

Often, you’ll want to make HTTP requests with no request body. This is usually the case with HTTP GET, OPTIONS and HEAD requests.

The simplest way to do this with the Vert.x http client is using the methods prefixed with Now. For example getNow.

These methods create the http request and send it in a single method call and allow you to provide a handler that will be called with the http response when it comes back.

HttpClient client = vertx.createHttpClient();

// Send a GET request
client.getNow("/some-uri", response -> {
  System.out.println("Received response with status code " + response.statusCode());
});

// Send a GET request
client.headNow("/other-uri", response -> {
  System.out.println("Received response with status code " + response.statusCode());
});
Writing general requests

At other times you don’t know the request method you want to send until run-time. For that use case we provide general purpose request methods such as request which allow you to specify the HTTP method at run-time:

HttpClient client = vertx.createHttpClient();

client.request(HttpMethod.GET, "some-uri", response -> {
  System.out.println("Received response with status code " + response.statusCode());
}).end();

client.request(HttpMethod.POST, "foo-uri", response -> {
  System.out.println("Received response with status code " + response.statusCode());
}).end("some-data");
Writing request bodies

Sometimes you’ll want to write requests which have a body, or perhaps you want to write headers to a request before sending it.

To do this you can call one of the specific request methods such as post or one of the general purpose request methods such as request.

These methods don’t send the request immediately, but instead return an instance of HttpClientRequest which can be used to write to the request body or write headers.

Here are some examples of writing a POST request with a body:

HttpClient client = vertx.createHttpClient();

HttpClientRequest request = client.post("some-uri", response -> {
  System.out.println("Received response with status code " + response.statusCode());
});

// Now do stuff with the request
request.putHeader("content-length", "1000");
request.putHeader("content-type", "text/plain");
request.write(body);

// Make sure the request is ended when you're done with it
request.end();

// Or fluently:

client.post("some-uri", response -> {
  System.out.println("Received response with status code " + response.statusCode());
}).putHeader("content-length", "1000").putHeader("content-type", "text/plain").write(body).end();

// Or event more simply:

client.post("some-uri", response -> {
  System.out.println("Received response with status code " + response.statusCode());
}).putHeader("content-type", "text/plain").end(body);
Methods exist to write strings in UTF-8 encoding and in any specific encoding and to write buffers:

request.write("some data");

// Write string encoded in specific encoding
request.write("some other data", "UTF-16");

// Write a buffer
Buffer buffer = Buffer.buffer();
buffer.appendInt(123).appendLong(245l);
request.write(buffer);
If you are just writing a single string or buffer to the HTTP request you can write it and end the request in a single call to the end function.

request.end("some simple data");

// Write buffer and end the request (send it) in a single call
Buffer buffer = Buffer.buffer().appendDouble(12.34d).appendLong(432l);
request.end(buffer);
When you’re writing to a request, the first call to write will result in the request headers being written out to the wire.

The actual write is asychronous and might not occur until some time after the call has returned.

Non-chunked HTTP requests with a request body require a Content-Length header to be provided.

Consequently, if you are not using chunked HTTP then you must set the Content-Length header before writing to the request, as it will be too late otherwise.

If you are calling one of the end methods that take a string or buffer then Vert.x will automatically calculate and set the Content-Length header before writing the request body.

If you are using HTTP chunking a a Content-Length header is not required, so you do not have to calculate the size up-front.

Writing request headers

You can write headers to a request using the headers multi-map as follows:

MultiMap headers = request.headers();
headers.set("content-type", "application/json").set("other-header", "foo");
The headers are an instance of MultiMap which provides operations for adding, setting and removing entries. Http headers allow more than one value for a specific key.

You can also write headers using putHeader

request.putHeader("content-type", "application/json").putHeader("other-header", "foo");
If you wish to write headers to the request you must do so before any part of the request body is written.

Ending HTTP requests

Once you have finished with the HTTP request you must end it with one of the end operations.

Ending a request causes any headers to be written, if they have not already been written and the request to be marked as complete.

Requests can be ended in several ways. With no arguments the request is simply ended:

request.end();
Or a string or buffer can be provided in the call to end. This is like calling write with the string or buffer before calling end with no arguments

request.end("some-data");

// End it with a buffer
Buffer buffer = Buffer.buffer().appendFloat(12.3f).appendInt(321);
request.end(buffer);
Chunked HTTP requests

Vert.x supports HTTP Chunked Transfer Encoding for requests.

This allows the HTTP request body to be written in chunks, and is normally used when a large request body is being streamed to the server, whose size is not known in advance.

You put the HTTP request into chunked mode using setChunked.

In chunked mode each call to write will cause a new chunk to be written to the wire. In chunked mode there is no need to set the Content-Length of the request up-front.

request.setChunked(true);

// Write some chunks
for (int i = 0; i < 10; i++) {
  request.write("this-is-chunk-" + i);
}

request.end();
Request timeouts

You can set a timeout for a specific http request using setTimeout.

If the request does not return any data within the timeout period an exception will be passed to the exception handler (if provided) and the request will be closed.

Handling exceptions

You can handle exceptions corresponding to a request by setting an exception handler on the HttpClientRequest instance:

HttpClientRequest request = client.post("some-uri", response -> {
  System.out.println("Received response with status code " + response.statusCode());
});
request.exceptionHandler(e -> {
  System.out.println("Received exception: " + e.getMessage());
  e.printStackTrace();
});
TODO - what about exceptions in the getNow methods where no exception handler can be provided??

Maybe need a catch all exception handler??

Specifying a handler on the client request

Instead of providing a response handler in the call to create the client request object, alternatively, you can not provide a handler when the request is created and set it later on the request object itself, using handler, for example:

HttpClientRequest request = client.post("some-uri");
request.handler(response -> {
  System.out.println("Received response with status code " + response.statusCode());
});
Using the request as a stream

The HttpClientRequest instance is also a WriteStream which means you can pump to it from any ReadStream instance.

For, example, you could pump a file on disk to a http request body as follows:

request.setChunked(true);
Pump pump = Pump.pump(file, request);
file.endHandler(v -> request.end());
pump.start();
Handling http responses
You receive an instance of HttpClientResponse into the handler that you specify in of the request methods or by setting a handler directly on the HttpClientRequest object.

You can query the status code and the status message of the response with statusCode and statusMessage.

client.getNow("some-uri", response -> {
  // the status code - e.g. 200 or 404
  System.out.println("Status code is " + response.statusCode());

  // the status message e.g. "OK" or "Not Found".
  System.out.println("Status message is " + response.statusMessage());
});
Using the response as a stream

The HttpClientResponse instance is also a ReadStream which means you can pump it to any WriteStream instance.

Response headers and trailers

Http responses can contain headers. Use headers to get the headers.

The object returned is a MultiMap as HTTP headers can contain multiple values for single keys.

String contentType = response.headers().get("content-type");
String contentLength = response.headers().get("content-lengh");
Chunked HTTP responses can also contain trailers - these are sent in the last chunk of the response body.

You use trailers to get the trailers. Trailers are also a MultiMap.

Reading the request body

The response handler is called when the headers of the response have been read from the wire.

If the response has a body this might arrive in several pieces some time after the headers have been read. We don’t wait for all the body to arrive before calling the response handler as the response could be very large and we might be waiting a long time, or run out of memory for large responses.

As parts of the response body arrive, the handler is called with a Buffer representing the piece of the body:

client.getNow("some-uri", response -> {

  response.handler(buffer -> {
    System.out.println("Received a part of the response body: " + buffer);
  });
});
If you know the response body is not very large and want to aggregate it all in memory before handling it, you can either aggregate it yourself:

client.getNow("some-uri", response -> {

  // Create an empty buffer
  Buffer totalBuffer = Buffer.buffer();

  response.handler(buffer -> {
    System.out.println("Received a part of the response body: " + buffer.length());

    totalBuffer.appendBuffer(buffer);
  });

  response.endHandler(v -> {
    // Now all the body has been read
    System.out.println("Total response body length is " + totalBuffer.length());
  });
});
Or you can use the convenience bodyHandler which is called with the entire body when the response has been fully read:

client.getNow("some-uri", response -> {

  response.bodyHandler(totalBuffer -> {
    // Now all the body has been read
    System.out.println("Total response body length is " + totalBuffer.length());
  });
});
Response end handler

The response endHandler is called when the entire response body has been read or immediately after the headers have been read and the response handler has been called if there is no body.

Reading cookies from the response

You can retrieve the list of cookies from a response using cookies.

Alternatively you can just parse the Set-Cookie headers yourself in the response.

100-Continue handling

According to the HTTP 1.1 specification a client can set a header Expect: 100-Continue and send the request header before sending the rest of the request body.

The server can then respond with an interim response status Status: 100 (Continue) to signify to the client that it is ok to send the rest of the body.

The idea here is it allows the server to authorise and accept/reject the request before large amounts of data are sent. Sending large amounts of data if the request might not be accepted is a waste of bandwidth and ties up the server in reading data that it will just discard.

Vert.x allows you to set a continueHandler on the client request object

This will be called if the server sends back a Status: 100 (Continue) response to signify that it is ok to send the rest of the request.

This is used in conjunction with `sendHead`to send the head of the request.

Here’s an example:

HttpClientRequest request = client.put("some-uri", response -> {
  System.out.println("Received response with status code " + response.statusCode());
});

request.putHeader("Expect", "100-Continue");

request.continueHandler(v -> {
  // OK to send rest of body
  request.write("Some data");
  request.write("Some more data");
  request.end();
});
Enabling compression on the client
The http client comes with support for HTTP Compression out of the box.

This means the client can let the remote http server know that it supports compression, and will be able to handle compressed response bodies.

An http server is free to either compress with one of the supported compression algorithms or to send the body back without compressing it at all. So this is only a hint for the Http server which it may ignore at will.

To tell the http server which compression is supported by the client it will include an Accept-Encoding header with the supported compression algorithm as value. Multiple compression algorithms are supported. In case of Vert.x this will result in the following header added:

Accept-Encoding: gzip, deflate
The server will choose then from one of these. You can detect if a server ompressed the body by checking for the Content-Encoding header in the response sent back from it.

If the body of the response was compressed via gzip it will include for example the following header:

Content-Encoding: gzip
To enable compression set setTryUseCompression on the options used when creating the client.

By default compression is disabled.

Pooling and keep alive
Http keep alive allows http connections to be used for more than one request. This can be a more efficient use of connections when you’re making multiple requests to the same server.

The http client supports pooling of connections, allowing you to reuse connections between requests.

For pooling to work, keep alive must be true using setKeepAlive on the options used when configuring the client. The default value is true.

When keep alive is enabled. Vert.x will add a Connection: Keep-Alive header to each HTTP request sent.

The maximum number of connections to pool for each server is configured using setMaxPoolSize

When making a request with pooling enabled, Vert.x will create a new connection if there are less than the maximum number of connections already created for that server, otherwise it will add the request to a queue.

When a response returns, if there are pending requests for the server, then the connection will be reused, otherwise it will be closed.

This gives the benefits of keep alive when the client is loaded but means we don’t keep connections hanging around unnecessarily when there would be no benefits anyway.

Pipe-lining
The client also supports pipe-lining of requests on a connection.

Pipe-lining means another request is sent on the same connection before the response from the preceding one has returned. Pipe-lining is not appropriate for all requests.

To enable pipe-lining, it must be enabled using setPipelining. By default pipe-lining is disabled.

When pipe-lining is enabled requests will be written to connections without waiting for previous responses to return.

When pipe-line responses return at the client, the connection will be automatically closed when all in-flight responses have returned and there are no outstanding pending requests to write.

Server sharing
TODO round robin requests etc

Using HTTPS with Vert.x
Vert.x http servers and clients can be configured to use HTTPS in exactly the same way as net servers.

Please see configuring net servers to use SSL for more information.

WebSockets
WebSockets are a web technology that allows a full duplex socket-like connection between HTTP servers and HTTP clients (typically browsers).

Vert.x supports WebSockets on both the client and server-side.

WebSockets on the server

There are two ways of handling WebSockets on the server side.

WebSocket handler

The first way involves providing a websocketHandler on the server instance.

When a WebSocket connection is made to the server, the handler will be called, passing in an instance of ServerWebSocket.

server.websocketHandler(websocket -> {
  System.out.println("Connected!");
});
You can choose to reject the WebSocket by calling reject.

server.websocketHandler(websocket -> {
  if (websocket.path().equals("/myapi")) {
    websocket.reject();
  } else {
    // Do something
  }
});
Upgrading to WebSocket

The second way of handling WebSockets is to handle the HTTP Upgrade request that was sent from the client, and call upgrade on the server request.

server.requestHandler(request -> {
  if (request.path().equals("/myapi")) {

    ServerWebSocket websocket = request.upgrade();
    // Do something

  } else {
    // Reject
    request.response().setStatusCode(400).end();
  }
});
The server WebSocket

The ServerWebSocket instance enables you to retrieve the headers, path path}, query and uri URI} of the HTTP request of the WebSocket handshake.

WebSockets on the client

The Vert.x HttpClient supports WebSockets.

You can connect a WebSocket to a server using one of the websocket operations and providing a handler.

The handler will be called with an instance of WebSocket when the connection has been made:

client.websocket("/some-uri", websocket -> {
  System.out.println("Connected!");
});
Writing messages to WebSockets

If you wish to write a single binary WebSocket message containing a single WebSocket frame to the WebSocket (a common case) the simplest way to do this is to use writeMessage:

Buffer buffer = Buffer.buffer().appendInt(123).appendFloat(1.23f);

websocket.writeMessage(buffer);
If the websocket message is larger than the maximum websocket frame size as configured with setMaxWebsocketFrameSize then Vert.x will split it into multiple WebSocket frames before sending it on the wire.

Writing frames to WebSockets

A WebSocket message can be composed of multiple frames. In this case the first frame is either a binary or text frame followed by one or more continuation frames.

The last frame in the message is marked as final.

To send a message consisting of multiple frames you create frames using WebSocketFrame.binaryFrame , WebSocketFrame.textFrame or WebSocketFrame.continuationFrame and write them to the WebSocket using writeFrame.

Here’s an example for binary frames:

WebSocketFrame frame1 = WebSocketFrame.binaryFrame(buffer1, false);
websocket.writeFrame(frame1);

WebSocketFrame frame2 = WebSocketFrame.continuationFrame(buffer2, false);
websocket.writeFrame(frame2);

// Write the final frame
WebSocketFrame frame3 = WebSocketFrame.continuationFrame(buffer2, true);
websocket.writeFrame(frame3);
Reading frames from WebSockets

To read frames from a WebSocket you use the frameHandler.

The frame handler will be called with instances of WebSocketFrame when a frame arrives, for example:

websocket.frameHandler(frame -> {
  System.out.println("Received a frame of size!");
});
Closing WebSockets

Use close to close the WebSocket connection when you have finished with it.

Streaming WebSockets

The WebSocket instance is also a ReadStream and a WriteStream so it can be used with pumps.

When using a WebSocket as a write stream or a read stream it can only be used with WebSockets connections that are used with binary frames that are no split over multiple frames.

Automatic clean-up in verticles
If you’re creating http servers and clients from inside verticles, those servers and clients will be automatically closed when the verticle is undeployed.
