# Routing based on MIME type of request
You can specify that a route will match against matching request MIME types using consumes.

In this case, the request will contain a content-type header specifying the MIME type of the request body. This will be matched against the value specified in consumes.

Basically, consumes is describing which MIME types the handler can consume.

Matching can be done on exact MIME type matches:
```
router.route().consumes("text/html").handler(routingContext -> {

  // This handler will be called for any request with
  // content-type header set to `text/html`

});
```
Multiple exact matches can also be specified:
```
router.route().consumes("text/html").consumes("text/plain").handler(routingContext -> {

  // This handler will be called for any request with
  // content-type header set to `text/html` or `text/plain`.

});
```
Matching on wildcards for the sub-type is supported:
```
router.route().consumes("text/*").handler(routingContext -> {

  // This handler will be called for any request with top level type `text`
  // e.g. content-type header set to `text/html` or `text/plain` will both match

});
```
And you can also match on the top level type
```
router.route().consumes("*/json").handler(routingContext -> {

  // This handler will be called for any request with sub-type json
  // e.g. content-type header set to `text/json` or `application/json` will both match

});
```
If you don’t specify a / in the consumers, it will assume you meant the sub-type.

