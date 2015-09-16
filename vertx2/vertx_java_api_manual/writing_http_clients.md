# Writing HTTP Clients

### Creating an HTTP Client

To create an HTTP client you call the createHttpClient method on your vertx instance:

HttpClient client = vertx.createHttpClient();
You set the port and hostname (or ip address) that the client will connect to using the setHost and setPort functions:
```
HttpClient client = vertx.createHttpClient();
client.setPort(8181);
client.setHost("foo.com");
```
This, of course, can be chained:
```
HttpClient client = vertx.createHttpClient()
    .setPort(8181)
    .setHost("foo.com");
```
A single HTTPClient always connects to the same host and port. If you want to connect to different servers, create more instances.

The default port is 80 and the default host is localhost. So if you don't explicitly set these values that's what the client will attempt to connect to.

### Pooling and Keep Alive

By default the HTTPClient pools HTTP connections. As you make requests a connection is borrowed from the pool and returned when the HTTP response has ended.

If you do not want connections to be pooled you can call setKeepAlive with false:
```
HttpClient client = vertx.createHttpClient()
               .setPort(8181)
               .setHost("foo.com").
               .setKeepAlive(false);
```
In this case a new connection will be created for each HTTP request and closed once the response has ended.

You can set the maximum number of connections that the client will pool as follows:

```
HttpClient client = vertx.createHttpClient()
               .setPort(8181)
               .setHost("foo.com").
               .setMaxPoolSize(10);
```
The default value is 1.

### Closing the client

Any HTTP clients created in a verticle are automatically closed for you when the verticle is stopped, however if you want to close it explicitly you can:
```
client.close();
```

### Making Requests

To make a request using the client you invoke one the methods named after the HTTP method that you want to invoke.

For example, to make a POST request:
```
HttpClient client = vertx.createHttpClient().setHost("foo.com");

HttpClientRequest request = client.post("/some-path/", new Handler<HttpClientResponse>() {
    public void handle(HttpClientResponse resp) {
        log.info("Got a response: " + resp.statusCode());
    }
});

request.end();
```
To make a PUT request use the put method, to make a GET request use the get method, etc.

Legal request methods are: get, put, post, delete, head, options, connect, trace and patch.

The general modus operandi is you invoke the appropriate method passing in the request URI as the first parameter, the second parameter is an event handler which will get called when the corresponding response arrives. The response handler is passed the client response object as an argument.

The value specified in the request URI corresponds to the Request-URI as specified in Section 5.1.2 of the HTTP specification. In most cases it will be a relative URI.

Please note that the domain/port that the client connects to is determined by setPort and setHost, and is not parsed from the uri.

The return value from the appropriate request method is an instance of org.vertx.java.core.http.HTTPClientRequest. You can use this to add headers to the request, and to write to the request body. The request object implements WriteStream.

Once you have finished with the request you must call the end() method.

If you don't know the name of the request method in advance there is a general request method which takes the HTTP method as a parameter:
```
HttpClient client = vertx.createHttpClient().setHost("foo.com");

HttpClientRequest request = client.request("POST", "/some-path/",
    new Handler<HttpClientResponse>() {
        public void handle(HttpClientResponse resp) {
            log.info("Got a response: " + resp.statusCode());
        }
    });

request.end();
```
There is also a method called getNow which does the same as get, but automatically ends the request. This is useful for simple GETs which don't have a request body:
```
HttpClient client = vertx.createHttpClient().setHost("foo.com");

client.getNow("/some-path/", new Handler<HttpClientResponse>() {
    public void handle(HttpClientResponse resp) {
        log.info("Got a response: " + resp.statusCode());
    }
});
```

#### Handling exceptions

You can set an exception handler on the HttpClient class and it will receive all exceptions for the client unless a specific exception handler has been set on a specific HttpClientRequest object.

#### Writing to the request body

Writing to the client request body has a very similar API to writing to the server response body.

To write data to an HttpClientRequest object, you invoke the write function. This function can be called multiple times before the request has ended. It can be invoked in a few ways:

With a single buffer:
```
Buffer myBuffer = ...
request.write(myBuffer);
```
A string. In this case the string will encoded using UTF-8 and the result written to the wire.
```
request.write("hello");
```
A string and an encoding. In this case the string will encoded using the specified encoding and the result written to the wire.
```
request.write("hello", "UTF-16");
```
The write function is asynchronous and always returns immediately after the write has been queued. The actual write might complete some time later.

If you are just writing a single string or Buffer to the HTTP request you can write it and end the request in a single call to the end function.

The first call to write will result in the request headers being written to the request. Consequently, if you are not using HTTP chunking then you must set the Content-Length header before writing to the request, since it will be too late otherwise. If you are using HTTP chunking you do not have to worry.

#### Ending HTTP requests

Once you have finished with the HTTP request you must call the end function on it.

This function can be invoked in several ways:

With no arguments, the request is simply ended.
```
request.end();
```
The function can also be called with a string or Buffer in the same way write is called. In this case it's just the same as calling write with a string or Buffer followed by calling end with no arguments.

#### Writing Request Headers

To write headers to the request, add them to the multi-map returned from the headers() method:
```
HttpClient client = vertx.createHttpClient().setHost("foo.com");

HttpClientRequest request = client.post("/some-path/", new Handler<HttpClientResponse>() {
    public void handle(HttpClientResponse resp) {
        log.info("Got a response: " + resp.statusCode());
    }
});

request.headers().set("Some-Header", "Some-Value");
request.end();
```
You can also adds them using the putHeader method. This enables a more fluent API since calls can be chained, for example:
```
request.putHeader("Some-Header", "Some-Value").putHeader("Some-Other", "Blah");
```
These can all be chained together as per the common Vert.x API pattern:
```
client.setHost("foo.com").post("/some-path/", new Handler<HttpClientResponse>() {
    public void handle(HttpClientResponse resp) {
        log.info("Got a response: " + resp.statusCode());
    }
}).putHeader("Some-Header", "Some-Value").end();
```

#### Request timeouts

You can set a timeout for specific Http Request using the setTimeout() method. If the request does not return any data within the timeout period an exception will be passed to the exception handler (if provided) and the request will be closed.

#### HTTP chunked requests

Vert.x supports HTTP Chunked Transfer Encoding for requests. This allows the HTTP request body to be written in chunks, and is normally used when a large request body is being streamed to the server, whose size is not known in advance.

You put the HTTP request into chunked mode as follows:
```
request.setChunked(true);
```
Default is non-chunked. When in chunked mode, each call to request.write(...) will result in a new HTTP chunk being written out.

### HTTP Client Responses

Client responses are received as an argument to the response handler that is passed into one of the request methods on the HTTP client.

The response object implements ReadStream, so it can be pumped to a WriteStream like any other ReadStream.

To query the status code of the response use the statusCode() method. The statusMessage() method contains the status message. For example:
```
HttpClient client = vertx.createHttpClient().setHost("foo.com");

client.getNow("/some-path/", new Handler<HttpClientResponse>() {
    public void handle(HttpClientResponse resp) {
        log.info('server returned status code: ' + resp.statusCode());
        log.info('server returned status message: ' + resp.statusMessage());
    }
});
```
#### Reading Data from the Response Body

The API for reading an HTTP client response body is very similar to the API for reading a HTTP server request body.

Sometimes an HTTP response contains a body that we want to read. Like an HTTP request, the client response handler is called when all the response headers have arrived, not when the entire response body has arrived.

To receive the response body, you set a dataHandler on the response object which gets called as parts of the HTTP response arrive. Here's an example:
```
HttpClient client = vertx.createHttpClient().setHost("foo.com");

client.getNow("/some-path/", new Handler<HttpClientResponse>() {
    public void handle(HttpClientResponse resp) {
        resp.dataHandler(new Handler<Buffer>() {
            public void handle(Buffer data) {
                log.info('I received ' + buffer.length() + ' bytes');
            }
        });
    }
});
```
The response object implements the ReadStream interface so you can pump the response body to a WriteStream. See the chapter on streams and pump for a detailed explanation.

The dataHandler can be called multiple times for a single HTTP response.

As with a server request, if you wanted to read the entire response body before doing something with it you could do something like the following:
```
HttpClient client = vertx.createHttpClient().setHost("foo.com");

client.getNow("/some-path/", new Handler<HttpClientResponse>() {
    public void handle(HttpClientResponse resp) {

        final Buffer body = new Buffer(0);

        resp.dataHandler(new Handler<Buffer>() {
            public void handle(Buffer data) {
                body.appendBuffer(data);
            }
        });
        resp.endHandler(new VoidHandler() {
            public void handle() {
               // The entire response body has been received
               log.info('The total body received was ' + body.length() + ' bytes');
            }
        });
    }
});
```
Like any ReadStream the end handler is invoked when the end of stream is reached - in this case at the end of the response.

If the HTTP response is using HTTP chunking, then each chunk of the response body will correspond to a single call to the dataHandler.

It's a very common use case to want to read the entire body in one go, so Vert.x allows a bodyHandler to be set on the response object.

The body handler is called only once when the entire response body has been read.

Beware of doing this with very large responses since the entire response body will be stored in memory.

Here's an example using bodyHandler:
```
HttpClient client = vertx.createHttpClient().setHost("foo.com");

client.getNow("/some-path/", new Handler<HttpClientResponse>() {
    public void handle(HttpClientResponse resp) {
        resp.bodyHandler(new Handler<Buffer>() {
            public void handle(Buffer body) {
               // The entire response body has been received
               log.info("The total body received was " + body.length() + " bytes");
            }
        });
    }
});
```

#### Reading cookies

You can read the list of cookies from the response using the method cookies().

### 100-Continue Handling

According to the HTTP 1.1 specification a client can set a header Expect: 100-Continue and send the request header before sending the rest of the request body.

The server can then respond with an interim response status Status: 100 (Continue) to signify the client is ok to send the rest of the body.

The idea here is it allows the server to authorise and accept/reject the request before large amounts of data is sent. Sending large amounts of data if the request might not be accepted is a waste of bandwidth and ties up the server in reading data that it will just discard.

Vert.x allows you to set a continueHandler on the client request object. This will be called if the server sends back a Status: 100 (Continue) response to signify it is ok to send the rest of the request.

This is used in conjunction with the sendHead function to send the head of the request.

An example will illustrate this:
```
HttpClient client = vertx.createHttpClient().setHost("foo.com");

final HttpClientRequest request = client.put("/some-path/", new Handler<HttpClientResponse>() {
    public void handle(HttpClientResponse resp) {
        log.info("Got a response " + resp.statusCode());
    }
});

request.putHeader("Expect", "100-Continue");

request.continueHandler(new VoidHandler() {
    public void handle() {

        // OK to send rest of body
        request.write("Some data").end();
    }
});

request.sendHead();
```

### HTTP Compression

Vert.x comes with support for HTTP Compression out of the box. Which means the HTTPClient can let the remote Http server know that it supports compression, and so will be able to handle compressed response bodies. A Http server is free to either compress with one of the supported compression algorithm or send the body back without compress it at all. So this is only a hint for the Http server which it may ignore at all.

To tell the Http server which compression is supported by the HttpClient it will include a 'Accept-Encoding' header with the supported compression algorithm as value. Multiple compression algorithms are supported. In case of Vert.x this will result in have the following header added:
```
Accept-Encoding: gzip, deflate
```
The Http Server will choose then from one of these. You can detect if a HttpServer did compress the body by checking for the 'Content-Encoding' header in the response sent back from it.

If the body of the response was compressed via gzip it will include for example the following header:
```
Content-Encoding: gzip
```
To enable compression you only need to do:
```
HttpClient client = vertx.createHttpClient();
client.setTryUseCompression(true);
```

The default is false.

## Pumping Requests and Responses

The HTTP client and server requests and responses all implement either ReadStream or WriteStream. This means you can pump between them and any other read and write streams.

## HTTPS Servers

HTTPS servers are very easy to write using Vert.x.

An HTTPS server has an identical API to a standard HTTP server. Getting the server to use HTTPS is just a matter of configuring the HTTP Server before listen is called.

Configuration of an HTTPS server is done in exactly the same way as configuring a NetServer for SSL. Please see SSL server chapter for detailed instructions.

## HTTPS Clients

HTTPS clients can also be very easily written with Vert.x

Configuring an HTTP client for HTTPS is done in exactly the same way as configuring a NetClient for SSL. Please see SSL client chapter for detailed instructions.

## Scaling HTTP servers

Scaling an HTTP or HTTPS server over multiple cores is as simple as deploying more instances of the verticle. For example:
```
vertx runmod com.mycompany~my-mod~1.0 -instance 20
```
Or, for a raw verticle:
```
vertx run foo.MyServer -instances 20
```
The scaling works in the same way as scaling a NetServer. Please see the chapter on scaling Net Servers for a detailed explanation of how this works.