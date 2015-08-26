Routing by exact path
A route can be set-up to match the path from the request URI. In this case it will match any request which has a path that’s the same as the specified path.

In the following example the handler will be called for a request /some/path/. We also ignore trailing slashes so it will be called for paths /some/path and /some/path// too:

Route route = router.route().path("/some/path/");

route.handler(routingContext -> {
  // This handler will be called for the following request paths:

  // `/some/path`
  // `/some/path/`
  // `/some/path//`
  //
  // but not:
  // `/some/path/subdir`
});
