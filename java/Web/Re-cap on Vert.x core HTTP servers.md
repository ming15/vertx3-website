Re-cap on Vert.x core HTTP servers
Vert.x-Web uses and exposes the API from Vert.x core, so it’s well worth getting familiar with the basic concepts of writing HTTP servers using Vert.x core, if you’re not already.

The Vert.x core HTTP documentation goes into a lot of detail on this.

Here’s a hello world web server written using Vert.x core. At this point there is no Vert.x-Web involved:

HttpServer server = vertx.createHttpServer();

server.requestHandler(request -> {

  // This handler gets called for each request that arrives on the server
  HttpServerResponse response = request.response();
  response.putHeader("content-type", "text/plain");

  // Write to the response and end it
  response.end("Hello World!");
});

server.listen(8080);
We create an HTTP server instance, and we set a request handler on it. The request handler will be called whenever a request arrives on the server.

When that happens we are just going to set the content type to text/plain, and write Hello World! and end the response.

We then tell the server to listen at port 8080 (default host is localhost).

You can run this, and point your browser at http://localhost:8080 to verify that it works as expected.

