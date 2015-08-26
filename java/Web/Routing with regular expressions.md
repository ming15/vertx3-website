Routing with regular expressions
Regular expressions can also be used to match URI paths in routes.

Route route = router.route().pathRegex(".*foo");

route.handler(routingContext -> {

  // This handler will be called for:

  // /some/path/foo
  // /foo
  // /foo/bar/wibble/foo
  // /foo/bar

  // But not:
  // /bar/wibble
});
Alternatively the regex can be specified when creating the route:

Route route = router.routeWithRegex(".*foo");

route.handler(routingContext -> {

  // This handler will be called same as previous example

});
