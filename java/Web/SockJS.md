SockJS
SockJS is a client side JavaScript library and protocol which provides a simple WebSocket-like interface allowing you to make connections to SockJS servers irrespective of whether the actual browser or network will allow real WebSockets.

It does this by supporting various different transports between browser and server, and choosing one at run-time according to browser and network capabilities.

All this is transparent to you - you are simply presented with the WebSocket-like interface which just works.

Please see the SockJS website for more information on SockJS.

SockJS handler
Vert.x provides an out of the box handler called SockJSHandler for using SockJS in your Vert.x-Web applications.

You should create one handler per SockJS application using SockJSHandler.create. You can also specify configuration options when creating the instance. The configuration options are described with an instance of SockJSHandlerOptions.

Router router = Router.router(vertx);

SockJSHandlerOptions options = new SockJSHandlerOptions().setHeartbeatInterval(2000);

SockJSHandler sockJSHandler = SockJSHandler.create(vertx, options);

router.route("/myapp").handler(sockJSHandler);
Handling SockJS sockets
On the server-side you set a handler on the SockJS handler, and this will be called every time a SockJS connection is made from a client:

The object passed into the handler is a SockJSSocket. This has a familiar socket-like interface which you can read and write to similarly to a NetSocket or a WebSocket. It also implements ReadStream and WriteStream so you can pump it to and from other read and write streams.

Here’s an example of a simple SockJS handler that simply echoes back any back any data that it reads:

Router router = Router.router(vertx);

SockJSHandlerOptions options = new SockJSHandlerOptions().setHeartbeatInterval(2000);

SockJSHandler sockJSHandler = SockJSHandler.create(vertx, options);

sockJSHandler.socketHandler(sockJSSocket -> {

  // Just echo the data back
  sockJSSocket.handler(sockJSSocket::write);
});

router.route("/myapp").handler(sockJSHandler);
The client side
In client side JavaScript you use the SockJS client side library to make connections.

You can find that here. The minified version is here.

Full details for using the SockJS JavaScript client are on the SockJS website, but in summary you use it something like this:

var sock = new SockJS('http://mydomain.com/myapp');

sock.onopen = function() {
  console.log('open');
};

sock.onmessage = function(e) {
  console.log('message', e.data);
};

sock.onclose = function() {
  console.log('close');
};

sock.send('test');

sock.close();
Configuring the SockJS handler
The handler can be configured with various options using SockJSHandlerOptions.

insertJSESSIONID
Insert a JSESSIONID cookie so load-balancers ensure requests for a specific SockJS session are always routed to the correct server. Default is true.

sessionTimeout
The server sends a close event when a client receiving connection have not been seen for a while. This delay is configured by this setting. By default the close event will be emitted when a receiving connection wasn’t seen for 5 seconds.

heartbeatInterval
In order to keep proxies and load balancers from closing long running http requests we need to pretend that the connection is active and send a heartbeat packet once in a while. This setting controls how often this is done. By default a heartbeat packet is sent every 25 seconds.

maxBytesStreaming
Most streaming transports save responses on the client side and don’t free memory used by delivered messages. Such transports need to be garbage-collected once in a while. max_bytes_streaming sets a minimum number of bytes that can be send over a single http streaming request before it will be closed. After that client needs to open new request. Setting this value to one effectively disables streaming and will make streaming transports to behave like polling transports. The default value is 128K.

libraryURL
Transports which don’t support cross-domain communication natively ('eventsource' to name one) use an iframe trick. A simple page is served from the SockJS server (using its foreign domain) and is placed in an invisible iframe. Code run from this iframe doesn’t need to worry about cross-domain issues, as it’s being run from domain local to the SockJS server. This iframe also does need to load SockJS javascript client library, and this option lets you specify its url (if you’re unsure, point it to the latest minified SockJS client release, this is the default). The default value is http://cdn.sockjs.org/sockjs-0.3.4.min.js

disabledTransports
This is a list of transports that you want to disable. Possible values are WEBSOCKET, EVENT_SOURCE, HTML_FILE, JSON_P, XHR.
