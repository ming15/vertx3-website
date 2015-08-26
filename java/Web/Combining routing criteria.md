Combining routing criteria
You can combine all the above routing criteria in many different ways, for example:

Route route = router.route(HttpMethod.PUT, "myapi/orders")
                    .consumes("application/json")
                    .produces("application/json");

route.handler(routingContext -> {

  // This would be match for any PUT method to paths starting with "myapi/orders" with a
  // content-type of "application/json"
  // and an accept header matching "application/json"

});
