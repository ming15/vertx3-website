Route order
By default routes are matched in the order they are added to the router.

When a request arrives the router will step through each route and check if it matches, if it matches then the handler for that route will be called.

If the handler subsequently calls next the handler for the next matching route (if any) will be called. And so on.

Here’s an example to illustrate this:

Route route1 = router.route("/some/path/").handler(routingContext -> {

  HttpServerResponse response = routingContext.response();
  // enable chunked responses because we will be adding data as
  // we execute over other handlers. This is only required once and
  // only if several handlers do output.
  response.setChunked(true);

  response.write("route1\n");

  // Now call the next matching route
  routingContext.next();
});

Route route2 = router.route("/some/path/").handler(routingContext -> {

  HttpServerResponse response = routingContext.response();
  response.write("route2\n");

  // Now call the next matching route
  routingContext.next();
});

Route route3 = router.route("/some/path/").handler(routingContext -> {

  HttpServerResponse response = routingContext.response();
  response.write("route3");

  // Now end the response
  routingContext.response().end();
});
In the above example the response will contain:

route1
route2
route3
As the routes have been called in that order for any request that starts with /some/path.

If you want to override the default ordering for routes, you can do so using order, specifying an integer value.

Routes are assigned an order at creation time corresponding to the order in which they were added to the router, with the first route numbered 0, the second route numbered 1, and so on.

By specifying an order for the route you can override the default ordering. Order can also be negative, e.g. if you want to ensure a route is evaluated before route number 0.

Let’s change the ordering of route2 so it runs before route1:

Route route1 = router.route("/some/path/").handler(routingContext -> {

  HttpServerResponse response = routingContext.response();
  response.write("route1\n");

  // Now call the next matching route
  routingContext.next();
});

Route route2 = router.route("/some/path/").handler(routingContext -> {

  HttpServerResponse response = routingContext.response();
  // enable chunked responses because we will be adding data as
  // we execute over other handlers. This is only required once and
  // only if several handlers do output.
  response.setChunked(true);

  response.write("route2\n");

  // Now call the next matching route
  routingContext.next();
});

Route route3 = router.route("/some/path/").handler(routingContext -> {

  HttpServerResponse response = routingContext.response();
  response.write("route3");

  // Now end the response
  routingContext.response().end();
});

// Change the order of route2 so it runs before route1
route2.order(-1);
then the response will now contain:

route2
route1
route3
If two matching routes have the same value of order, then they will be called in the order they were added.

You can also specify that a route is handled last, with last

