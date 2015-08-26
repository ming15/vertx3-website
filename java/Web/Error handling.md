Error handling
As well as setting handlers to handle requests you can also set handlers to handle failures in routing.

Failure handlers are used with the exact same route matching criteria that you use with normal handlers.

For example you can provide a failure handler that will only handle failures on certain paths, or for certain HTTP methods.

This allows you to set different failure handlers for different parts of your application.

Here’s an example failure handler that will only be called for failure that occur when routing to GET requests to paths that start with /somepath/:

Route route = router.get("/somepath/*");

route.failureHandler(frc -> {

  // This will be called for failures that occur
  // when routing requests to paths starting with
  // '/somepath/'

});
Failure routing will occur if a handler throws an exception, or if a handler calls fail specifying an HTTP status code to deliberately signal a failure.

If an exception is caught from a handler this will result in a failure with status code 500 being signalled.

When handling the failure, the failure handler is passed the routing context which also allows the failure or failure code to be retrieved so the failure handler can use that to generate a failure response.

Route route1 = router.get("/somepath/path1/");

route1.handler(routingContext -> {

  // Let's say this throws a RuntimeException
  throw new RuntimeException("something happened!");

});

Route route2 = router.get("/somepath/path2");

route2.handler(routingContext -> {

  // This one deliberately fails the request passing in the status code
  // E.g. 403 - Forbidden
  routingContext.fail(403);

});

// Define a failure handler
// This will get called for any failures in the above handlers
Route route3 = router.get("/somepath/*");

route3.failureHandler(failureRoutingContext -> {

  int statusCode = failureRoutingContext.statusCode();

  // Status code will be 500 for the RuntimeException or 403 for the other failure
  HttpServerResponse response = failureRoutingContext.response();
  response.setStatusCode(statusCode).end("Sorry! Not today");

});
