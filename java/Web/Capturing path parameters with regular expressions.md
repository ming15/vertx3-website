Capturing path parameters with regular expressions
You can also capture path parameters when using regular expressions, here’s an example:

Route route = router.routeWithRegex(".*foo");

// This regular expression matches paths that start with something like:
// "/foo/bar" - where the "foo" is captured into param0 and the "bar" is captured into
// param1
route.pathRegex("\\/([^\\/]+)\\/([^\\/]+)").handler(routingContext -> {

  String productType = routingContext.request().getParam("param0");
  String productID = routingContext.request().getParam("param1");

  // Do something with them...
});
In the above example, if a request is made to path: /tools/drill123/ then the route will match and productType will receive the value tools and productID will receive the value drill123.

Captures are denoted in regular expressions with capture groups (i.e. surrounding the capture with round brackets)

