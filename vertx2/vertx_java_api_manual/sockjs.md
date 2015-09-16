# SockJS

WebSockets are a new technology, and many users are still using browsers that do not support them, or which support older, pre-final, versions.

Moreover, WebSockets do not work well with many corporate proxies. This means that's it's not possible to guarantee a WebSockets connection is going to succeed for every user.

Enter SockJS.

SockJS is a client side JavaScript library and protocol which provides a simple WebSocket-like interface to the client side JavaScript developer irrespective of whether the actual browser or network will allow real WebSockets.

It does this by supporting various different transports between browser and server, and choosing one at runtime according to browser and network capabilities. All this is transparent to you - you are simply presented with the WebSocket-like interface which just works.

Please see the SockJS website for more information.

SockJS Server

Vert.x provides a complete server side SockJS implementation.

This enables Vert.x to be used for modern, so-called real-time (this is the modern meaning of real-time, not to be confused by the more formal pre-existing definitions of soft and hard real-time systems) web applications that push data to and from rich client-side JavaScript applications, without having to worry about the details of the transport.

To create a SockJS server you simply create a HTTP server as normal and then call the createSockJSServer method of your vertx instance passing in the Http server:

HttpServer httpServer = vertx.createHttpServer();

SockJSServer sockJSServer = vertx.createSockJSServer(httpServer);
Each SockJS server can host multiple applications.

Each application is defined by some configuration, and provides a handler which gets called when incoming SockJS connections arrive at the server.

For example, to create a SockJS echo application:

HttpServer httpServer = vertx.createHttpServer();

SockJSServer sockJSServer = vertx.createSockJSServer(httpServer);

JsonObject config = new JsonObject().putString("prefix", "/echo");

sockJSServer.installApp(config, new Handler<SockJSSocket>() {
    public void handle(SockJSSocket sock) {
        Pump.createPump(sock, sock).start();
    }
});

httpServer.listen(8080);
The configuration is an instance of org.vertx.java.core.json.JsonObject, which takes the following fields:

prefix: A url prefix for the application. All http requests whose paths begins with selected prefix will be handled by the application. This property is mandatory.
insert_JSESSIONID: Some hosting providers enable sticky sessions only to requests that have JSESSIONID cookie set. This setting controls if the server should set this cookie to a dummy value. By default setting JSESSIONID cookie is enabled. More sophisticated beaviour can be achieved by supplying a function.
session_timeout: The server sends a close event when a client receiving connection have not been seen for a while. This delay is configured by this setting. By default the close event will be emitted when a receiving connection wasn't seen for 5 seconds.
heartbeat_period: In order to keep proxies and load balancers from closing long running http requests we need to pretend that the connecion is active and send a heartbeat packet once in a while. This setting controlls how often this is done. By default a heartbeat packet is sent every 5 seconds.
max_bytes_streaming: Most streaming transports save responses on the client side and don't free memory used by delivered messages. Such transports need to be garbage-collected once in a while. max_bytes_streaming sets a minimum number of bytes that can be send over a single http streaming request before it will be closed. After that client needs to open new request. Setting this value to one effectively disables streaming and will make streaming transports to behave like polling transports. The default value is 128K.
library_url: Transports which don't support cross-domain communication natively ('eventsource' to name one) use an iframe trick. A simple page is served from the SockJS server (using its foreign domain) and is placed in an invisible iframe. Code run from this iframe doesn't need to worry about cross-domain issues, as it's being run from domain local to the SockJS server. This iframe also does need to load SockJS javascript client library, and this option lets you specify its url (if you're unsure, point it to the latest minified SockJS client release, this is the default). The default value is http://cdn.sockjs.org/sockjs-0.3.4.min.js
Reading and writing data from a SockJS server

The SockJSSocket object passed into the SockJS handler implements ReadStream and WriteStream much like NetSocket or WebSocket. You can therefore use the standard API for reading and writing to the SockJS socket or using it in pumps.

See the chapter on Streams and Pumps for more information.

SockJS client

For full information on using the SockJS client library please see the SockJS website. A simple example:

<script>
   var sock = new SockJS('http://mydomain.com/my_prefix');

   sock.onopen = function() {
       console.log('open');
   };

   sock.onmessage = function(e) {
       console.log('message', e.data);
   };

   sock.onclose = function() {
       console.log('close');
   };
</script>
As you can see the API is very similar to the WebSockets API.
