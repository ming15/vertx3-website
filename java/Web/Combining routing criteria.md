# Combining routing criteria

你可以将很多`route`规则组合到一起, 例如：

```java
Route route = router.route(HttpMethod.PUT, "myapi/orders")
                    .consumes("application/json")
                    .produces("application/json");

route.handler(routingContext -> {

  // This would be match for any PUT method to paths starting with "myapi/orders" with a
  // content-type of "application/json"
  // and an accept header matching "application/json"

});
```