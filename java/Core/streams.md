# Streams

There are several objects in Vert.x that allow items to be read from and written.

In previous versions the streams.adoc package was manipulating Buffer objects exclusively. From now, streams are not anymore coupled to buffers and work with any kind of objects.

In Vert.x, calls to write item return immediately and writes are internally queued.

It’s not hard to see that if you write to an object faster than it can actually write the data to its underlying resource then the write queue could grow without bound - eventually resulting in exhausting available memory.

To solve this problem a simple flow control capability is provided by some objects in the Vert.x API.

Any flow control aware object that can be written-to implements ReadStream, and any flow control object that can be read-from is said to implement WriteStream.

Let’s take an example where we want to read from a ReadStream and write the data to a WriteStream.

A very simple example would be reading from a NetSocket on a server and writing back to the same NetSocket - since NetSocket implements both ReadStream and WriteStream, but you can do this between any ReadStream and any WriteStream, including HTTP requests and response, async files, WebSockets, etc.

A naive way to do this would be to directly take the data that’s been read and immediately write it to the NetSocket, for example:

NetServer server = vertx.createNetServer(
    new NetServerOptions().setPort(1234).setHost("localhost")
);
server.connectHandler(sock -> {
  sock.handler(buffer -> {
    // Write the data straight back
    sock.write(buffer);
  });
}).listen();
There’s a problem with the above example: If data is read from the socket faster than it can be written back to the socket, it will build up in the write queue of the NetSocket, eventually running out of RAM. This might happen, for example if the client at the other end of the socket wasn’t reading very fast, effectively putting back-pressure on the connection.

Since NetSocket implements WriteStream, we can check if the WriteStream is full before writing to it:

NetServer server = vertx.createNetServer(
    new NetServerOptions().setPort(1234).setHost("localhost")
);
server.connectHandler(sock -> {
  sock.handler(buffer -> {
    if (!sock.writeQueueFull()) {
      sock.write(buffer);
    }
  });

}).listen();
This example won’t run out of RAM but we’ll end up losing data if the write queue gets full. What we really want to do is pause the NetSocket when the write queue is full. Let’s do that:

NetServer server = vertx.createNetServer(
    new NetServerOptions().setPort(1234).setHost("localhost")
);
server.connectHandler(sock -> {
  sock.handler(buffer -> {
    sock.write(buffer);
    if (sock.writeQueueFull()) {
      sock.pause();
    }
  });
}).listen();
We’re almost there, but not quite. The NetSocket now gets paused when the file is full, but we also need to unpause it when the write queue has processed its backlog:

NetServer server = vertx.createNetServer(
    new NetServerOptions().setPort(1234).setHost("localhost")
);
server.connectHandler(sock -> {
  sock.handler(buffer -> {
    sock.write(buffer);
    if (sock.writeQueueFull()) {
      sock.pause();
      sock.drainHandler(done -> {
        sock.resume();
      });
    }
  });
}).listen();
And there we have it. The drainHandler event handler will get called when the write queue is ready to accept more data, this resumes the NetSocket which allows it to read more data.

It’s very common to want to do this when writing Vert.x applications, so we provide a helper class called Pump which does all this hard work for you. You just feed it the ReadStream and the WriteStream and it tell it to start:

NetServer server = vertx.createNetServer(
    new NetServerOptions().setPort(1234).setHost("localhost")
);
server.connectHandler(sock -> {
  Pump.pump(sock, sock).start();
}).listen();
Which does exactly the same thing as the more verbose example.

Let’s look at the methods on ReadStream and WriteStream in more detail:

ReadStream
ReadStream is implemented by HttpClientResponse, DatagramSocket, HttpClientRequest, HttpServerFileUpload, HttpServerRequest, HttpServerRequestStream, MessageConsumer, NetSocket, NetSocketStream, WebSocket, WebSocketStream, TimeoutStream, AsyncFile.

Functions:

handler: set a handler which will receive items from the ReadStream.

pause: pause the handler. When paused no items will be received in the handler.

resume: resume the handler. The handler will be called if any item arrives.

exceptionHandler: Will be called if an exception occurs on the ReadStream.

endHandler: Will be called when end of stream is reached. This might be when EOF is reached if the ReadStream represents a file, or when end of request is reached if it’s an HTTP request, or when the connection is closed if it’s a TCP socket.

WriteStream
WriteStream is implemented by HttpClientRequest, HttpServerResponse WebSocket, NetSocket, AsyncFile, PacketWritestream and MessageProducer

Functions:

write: write an object to the WriteStream. This method will never block. Writes are queued internally and asynchronously written to the underlying resource.

setWriteQueueMaxSize: set the number of object at which the write queue is considered full, and the method writeQueueFull returns true. Note that, when the write queue is considered full, if write is called the data will still be accepted and queued. The actual number depends on the stream implementation, for Buffer the size represents the actual number of bytes written and not the number of buffers.

writeQueueFull: returns true if the write queue is considered full.

exceptionHandler: Will be called if an exception occurs on the WriteStream.

drainHandler: The handler will be called if the WriteStream is considered no longer full.

Pump
Instances of Pump have the following methods:

start: Start the pump.

stop: Stops the pump. When the pump starts it is in stopped mode.

setWriteQueueMaxSize: This has the same meaning as setWriteQueueMaxSize on the WriteStream.

A pump can be started and stopped multiple times.

When a pump is first created it is not started. You need to call the start() method to start it.
