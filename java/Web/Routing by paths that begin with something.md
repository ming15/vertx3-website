Routing by paths that begin with something
Often you want to route all requests that begin with a certain path. You could use a regex to do this, but a simply way is to use an asterisk * at the end of the path when declaring the route path.

In the following example the handler will be called for any request with a URI path that starts with /some/path/.

For example /some/path/foo.html and /some/path/otherdir/blah.css would both match.

Route route = router.route().path("/some/path/*");

route.handler(routingContext -> {
  // This handler will be called for any path that starts with
  // `/some/path/`, e.g.

  // `/some/path`
  // `/some/path/`
  // `/some/path/subdir`
  // `/some/path/subdir/blah.html`
  //
  // but not:
  // `/some/bath`
});
With any path it can also be specified when creating the route:

Route route = router.route("/some/path/*");

route.handler(routingContext -> {
  // This handler will be called same as previous example
});
