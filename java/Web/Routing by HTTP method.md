Routing by HTTP method
By default a route will match all HTTP methods.

If you want a route to only match for a specific HTTP method you can use method

Route route = router.route().method(HttpMethod.POST);

route.handler(routingContext -> {

  // This handler will be called for any POST request

});
Or you can specify this with a path when creating the route:

Route route = router.route(HttpMethod.POST, "/some/path/");

route.handler(routingContext -> {

  // This handler will be called for any POST request to a URI path starting with /some/path/

});
If you want to route for a specific HTTP method you can also use the methods such as get, post and put named after the HTTP method name. For example:

router.get().handler(routingContext -> {

  // Will be called for any GET request

});

router.get("/some/path/").handler(routingContext -> {

  // Will be called for any GET request to a path
  // starting with /some/path

});

router.getWithRegex(".*foo").handler(routingContext -> {

  // Will be called for any GET request to a path
  // ending with `foo`

});
If you want to specify a route will match for more than HTTP method you can call method multiple times:

Route route = router.route().method(HttpMethod.POST).method(HttpMethod.PUT);

route.handler(routingContext -> {

  // This handler will be called for any POST or PUT request

});
