# Sub-routers

有时候你也许会有很多`handler`来处理请求, 在这种情况下将这些`handler`分配到多个`route`中是非常有好处的. 另外如果你想在不同的应用程序中通过不同的`root`路径来复用这些`handler`, 这种做法同样是很有帮助的.

为了能达到这种效果, 你可以

Sometimes if you have a lot of handlers it can make sense to split them up into multiple routers. This is also useful if you want to reuse a set of handlers in a different application, rooted at a different path root.

To do this you can mount a router at a mount point in another router. The router that is mounted is called a sub-router. Sub routers can mount other sub routers so you can have several levels of sub-routers if you like.

Let’s look at a simple example of a sub-router mounted with another router.

This sub-router will maintain the set of handlers that corresponds to a simple fictional REST API. We will mount that on another router. The full implementation of the REST API is not shown.

Here’s the sub-router:
```java
Router restAPI = Router.router(vertx);

restAPI.get("/products/:productID").handler(rc -> {

  // TODO Handle the lookup of the product....
  rc.response().write(productJSON);

});

restAPI.put("/products/:productID").handler(rc -> {

  // TODO Add a new product...
  rc.response().end();

});

restAPI.delete("/products/:productID").handler(rc -> {

  // TODO delete the product...
  rc.response().end();

});
```
If this router was used as a top level router, then GET/PUT/DELETE requests to urls like /products/product1234 would invoke the API.

However, let’s say we already have a web-site as described by another router:
```java
Router mainRouter = Router.router(vertx);

// Handle static resources
mainRouter.route("/static/*").handler(myStaticHandler);

mainRouter.route(".*\\.templ").handler(myTemplateHandler);
```
We can now mount the sub router on the main router, against a mount point, in this case /productsAPI

mainRouter.mountSubRouter("/productsAPI", restAPI);
This means the REST API is now accessible via paths like: /productsAPI/products/product1234

