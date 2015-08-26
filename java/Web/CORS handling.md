CORS handling
Cross Origin Resource Sharing is a safe mechanism for allowing resources to be requested from one domain and served from another.

Vert.x-Web includes a handler CorsHandler that handles the CORS protocol for you.

Here’s an example:

router.route().handler(CorsHandler.create("vertx\\.io").allowedMethod(HttpMethod.GET));

router.route().handler(routingContext -> {

  // Your app handlers

});
TODO more CORS docs

