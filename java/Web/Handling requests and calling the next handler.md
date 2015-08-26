Handling requests and calling the next handler
When Vert.x-Web decides to route a request to a matching route, it calls the handler of the route passing in an instance of RoutingContext.

If you don’t end the response in your handler, you should call next so another matching route can handle the request (if any).

You don’t have to call next before the handler has finished executing. You can do this some time later, if you want:

Route route1 = router.route("/some/path/").handler(routingContext -> {

  HttpServerResponse response = routingContext.response();
  // enable chunked responses because we will be adding data as
  // we execute over other handlers. This is only required once and
  // only if several handlers do output.
  response.setChunked(true);

  response.write("route1\n");

  // Call the next matching route after a 5 second delay
  routingContext.vertx().setTimer(5000, tid -> routingContext.next());
});

Route route2 = router.route("/some/path/").handler(routingContext -> {

  HttpServerResponse response = routingContext.response();
  response.write("route2\n");

  // Call the next matching route after a 5 second delay
  routingContext.vertx().setTimer(5000, tid ->  routingContext.next());
});

Route route3 = router.route("/some/path/").handler(routingContext -> {

  HttpServerResponse response = routingContext.response();
  response.write("route3");

  // Now end the response
  routingContext.response().end();
});
In the above example route1 is written to the response, then 5 seconds later route2 is written to the response, then 5 seconds later route3 is written to the response and the response is ended.

Note, all this happens without any thread blocking.

