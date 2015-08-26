Capturing path parameters
It’s possible to match paths using placeholders for parameters which are then available in the request params.

Here’s an example

Route route = router.route(HttpMethod.POST, "/catalogue/products/:productype/:productid/");

route.handler(routingContext -> {

  String productType = routingContext.request().getParam("producttype");
  String productID = routingContext.request().getParam("productid");

  // Do something with them...
});
The placeholders consist of : followed by the parameter name. Parameter names consist of any alphabetic character, numeric character or underscore.

In the above example, if a POST request is made to path: /catalogue/products/tools/drill123/ then the route will match and productType will receive the value tools and productID will receive the value drill123.

