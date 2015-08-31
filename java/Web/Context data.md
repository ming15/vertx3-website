# Context data


You can use the context data in the RoutingContext to maintain any data that you want to share between handlers for the lifetime of the request.

Here’s an example where one handler sets some data in the context data and a subsequent handler retrieves it:

You can use the put to put any object, and get to retrieve any object from the context data.

A request sent to path /some/path/other will match both routes.
```java
router.get("/some/path").handler(routingContext -> {

  routingContext.put("foo", "bar");
  routingContext.next();

});

router.get("/some/path/other").handler(routingContext -> {

  String bar = routingContext.get("foo");
  // Do something with bar
  routingContext.response().end();

});
```
Alternatively you can access the entire context data map with data.

