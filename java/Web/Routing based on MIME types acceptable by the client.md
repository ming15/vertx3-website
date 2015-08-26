Routing based on MIME types acceptable by the client
The HTTP accept header is used to signify which MIME types of the response are acceptable to the client.

An accept header can have multiple MIME types separated by ‘,’.

MIME types can also have a q value appended to them* which signifies a weighting to apply if more than one response MIME type is available matching the accept header. The q value is a number between 0 and 1.0. If omitted it defaults to 1.0.

For example, the following accept header signifies the client will accept a MIME type of only text/plain:

Accept: text/plain
With the following the client will accept text/plain or text/html with no preference.

Accept: text/plain, text/html
With the following the client will accept text/plain or text/html but prefers text/html as it has a higher q value (the default value is q=1.0)

Accept: text/plain; q=0.9, text/html
If the server can provide both text/plain and text/html it should provide the text/html in this case.

By using produces you define which MIME type(s) the route produces, e.g. the following handler produces a response with MIME type application/json.

router.route().produces("application/json").handler(routingContext -> {

  HttpServerResponse response = routingContext.response();
  response.putHeader("content-type", "application/json");
  response.write(someJSON).end();

});
In this case the route will match with any request with an accept header that matches application/json.

Here are some examples of accept headers that will match:

Accept: application/json
Accept: application/*
Accept: application/json, text/html
Accept: application/json;q=0.7, text/html;q=0.8, text/plain
You can also mark your route as producing more than one MIME type. If this is the case, then you use getAcceptableContentType to find out the actual MIME type that was accepted.

router.route().produces("application/json").produces("text/html").handler(routingContext -> {

  HttpServerResponse response = routingContext.response();

  // Get the actual MIME type acceptable
  String acceptableContentType = routingContext.getAcceptableContentType();

  response.putHeader("content-type", acceptableContentType);
  response.write(whatever).end();
});
In the above example, if you sent a request with the following accept header:

Accept: application/json; q=0.7, text/html
Then the route would match and acceptableContentType would contain text/html as both are acceptable but that has a higher q value.

