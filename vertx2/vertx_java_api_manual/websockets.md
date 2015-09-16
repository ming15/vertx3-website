# WebSockets

WebSockets are a web technology that allows a full duplex socket-like connection between HTTP servers and HTTP clients (typically browsers).

WebSockets on the server

To use WebSockets on the server you create an HTTP server as normal, but instead of setting a requestHandler you set a websocketHandler on the server.

HttpServer server = vertx.createHttpServer();

server.websocketHandler(new Handler<ServerWebSocket>() {
    public void handle(ServerWebSocket ws) {
        // A WebSocket has connected!
    }
}).listen(8080, "localhost");
Reading from and Writing to WebSockets

The websocket instance passed into the handler implements both ReadStream and WriteStream, so you can read and write data to it in the normal ways. I.e by setting a dataHandler and calling the write method.

See the chapter on streams and pumps for more information.

For example, to echo all data received on a WebSocket:

HttpServer server = vertx.createHttpServer();

server.websocketHandler(new Handler<ServerWebSocket>() {
    public void handle(ServerWebSocket ws) {
        Pump.createPump(ws, ws).start();
    }
}).listen(8080, "localhost");
The websocket instance also has method writeBinaryFrame for writing binary data. This has the same effect as calling write.

Another method writeTextFrame also exists for writing text data. This is equivalent to calling

websocket.write(new Buffer("some-string"));
Rejecting WebSockets

Sometimes you may only want to accept WebSockets which connect at a specific path.

To check the path, you can query the path() method of the websocket. You can then call the reject() method to reject the websocket.

HttpServer server = vertx.createHttpServer();

server.websocketHandler(new Handler<ServerWebSocket>() {
    public void handle(ServerWebSocket ws) {
        if (ws.path().equals("/services/echo")) {
            Pump.createPump(ws, ws).start();
        } else {
            ws.reject();
        }
    }
}).listen(8080, "localhost");
Headers on the websocket

You can use the headers() method to retrieve the headers passed in the Http Request from the client that caused the upgrade to websockets.

WebSockets on the HTTP client

To use WebSockets from the HTTP client, you create the HTTP client as normal, then call the connectWebsocket function, passing in the URI that you wish to connect to at the server, and a handler.

The handler will then get called if the WebSocket successfully connects. If the WebSocket does not connect - perhaps the server rejects it - then any exception handler on the HTTP client will be called.

Here's an example of WebSocket connection;

HttpClient client = vertx.createHttpClient().setHost("foo.com");

client.connectWebsocket("/some-uri", new Handler<WebSocket>() {
    public void handle(WebSocket ws) {
        // Connected!
    }
});
Note that the host (and port) is set on the HttpClient instance, and the uri passed in the connect is typically a relative URI.

Again, the client side WebSocket implements ReadStream and WriteStream, so you can read and write to it in the same way as any other stream object.

WebSockets in the browser

To use WebSockets from a compliant browser, you use the standard WebSocket API. Here's some example client side JavaScript which uses a WebSocket.

<script>

    var socket = new WebSocket("ws://foo.com/services/echo");

    socket.onmessage = function(event) {
        alert("Received data from websocket: " + event.data);
    }

    socket.onopen = function(event) {
        alert("Web Socket opened");
        socket.send("Hello World");
    };

    socket.onclose = function(event) {
        alert("Web Socket closed");
    };

</script>
For more information see the WebSocket API documentation
