Basic Vert.x-Web concepts
Here’s the 10000 foot view:

A Router is one of the core concepts of Vert.x-Web. It’s an object which maintains zero or more Routes .

A router takes an HTTP request and finds the first matching route for that request, and passes the request to that route.

The route can have a handler associated with it, which then receives the request. You then do something with the request, and then, either end it or pass it to the next matching handler.

Here’s a simple router example:

HttpServer server = vertx.createHttpServer();

Router router = Router.router(vertx);

router.route().handler(routingContext -> {

  // This handler will be called for every request
  HttpServerResponse response = routingContext.response();
  response.putHeader("content-type", "text/plain");

  // Write to the response and end it
  response.end("Hello World from Vert.x-Web!");
});

server.requestHandler(router::accept).listen(8080);
It basically does the same thing as the Vert.x Core HTTP server hello world example from the previous section, but this time using Vert.x-Web.

We create an HTTP server as before, then we create a router. Once we’ve done that we create a simple route with no matching criteria so it will match all requests that arrive on the server.

We then specify a handler for that route. That handler will be called for all requests that arrive on the server.

The object that gets passed into the handler is a RoutingContext - this contains the standard Vert.x HttpServerRequest and HttpServerResponse but also various other useful stuff that makes working with Vert.x-Web simpler.

For every request that is routed there is a unique routing context instance, and the same instance is passed to all handlers for that request.

Once we’ve set up the handler, we set the request handler of the HTTP server to pass all incoming requests to accept.

So, that’s the basics. Now we’ll look at things in more detail:

