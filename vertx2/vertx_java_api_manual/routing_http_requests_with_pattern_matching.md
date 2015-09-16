# Routing HTTP requests with Pattern Matching

Vert.x lets you route HTTP requests to different handlers based on pattern matching on the request path. It also enables you to extract values from the path and use them as parameters in the request.

This is particularly useful when developing REST-style web applications.

To do this you simply create an instance of org.vertx.java.core.http.RouteMatcher and use it as handler in an HTTP server. See the chapter on HTTP servers for more information on setting HTTP handlers. Here's an example:

HttpServer server = vertx.createHttpServer();

RouteMatcher routeMatcher = new RouteMatcher();

server.requestHandler(routeMatcher).listen(8080, "localhost");
Specifying matches.

You can then add different matches to the route matcher. For example, to send all GET requests with path /animals/dogs to one handler and all GET requests with path /animals/cats to another handler you would do:

HttpServer server = vertx.createHttpServer();

RouteMatcher routeMatcher = new RouteMatcher();

routeMatcher.get("/animals/dogs", new Handler<HttpServerRequest>() {
    public void handle(HttpServerRequest req) {
        req.response().end("You requested dogs");
    }
});
routeMatcher.get("/animals/cats", new Handler<HttpServerRequest>() {
    public void handle(HttpServerRequest req) {
        req.response().end("You requested cats");
    }
});

server.requestHandler(routeMatcher).listen(8080, "localhost");
Corresponding methods exist for each HTTP method - get, post, put, delete, head, options, trace, connect and patch.

There's also an all method which applies the match to any HTTP request method.

The handler specified to the method is just a normal HTTP server request handler, the same as you would supply to the requestHandler method of the HTTP server.

You can provide as many matches as you like and they are evaluated in the order you added them, the first matching one will receive the request.

A request is sent to at most one handler.

Extracting parameters from the path

If you want to extract parameters from the path, you can do this too, by using the : (colon) character to denote the name of a parameter. For example:

HttpServer server = vertx.createHttpServer();

RouteMatcher routeMatcher = new RouteMatcher();

routeMatcher.put("/:blogname/:post", new Handler<HttpServerRequest>() {
    public void handle(HttpServerRequest req) {
        String blogName = req.params().get("blogname");
        String post = req.params().get("post");
        req.response().end("blogname is " + blogName + ", post is " + post);
    }
});

server.requestHandler(routeMatcher).listen(8080, "localhost");
Any params extracted by pattern matching are added to the map of request parameters.

In the above example, a PUT request to /myblog/post1 would result in the variable blogName getting the value myblog and the variable post getting the value post1.

Valid parameter names must start with a letter of the alphabet and be followed by any letters of the alphabet or digits or the underscore character.

Extracting params using Regular Expressions

Regular Expressions can be used to extract more complex matches. In this case capture groups are used to capture any parameters.

Since the capture groups are not named they are added to the request with names param0, param1, param2, etc.

Corresponding methods exist for each HTTP method - getWithRegEx, postWithRegEx, putWithRegEx, deleteWithRegEx, headWithRegEx, optionsWithRegEx, traceWithRegEx, connectWithRegEx and patchWithRegEx.

There's also an allWithRegEx method which applies the match to any HTTP request method.

For example:

HttpServer server = vertx.createHttpServer();

RouteMatcher routeMatcher = new RouteMatcher();

routeMatcher.allWithRegEx("\\/([^\\/]+)\\/([^\\/]+)", new Handler<HttpServerRequest>() {
    public void handle(HttpServerRequest req) {
        String first = req.params().get("param0");
        String second = req.params().get("param1");
        req.response.end("first is " + first + " and second is " + second);
    }
});

server.requestHandler(routeMatcher).listen(8080, "localhost");
Run the above and point your browser at http://localhost:8080/animals/cats.

Handling requests where nothing matches

You can use the noMatch method to specify a handler that will be called if nothing matches. If you don't specify a no match handler and nothing matches, a 404 will be returned.

routeMatcher.noMatch(new Handler<HttpServerRequest>() {
    public void handle(HttpServerRequest req) {
        req.response().end("Nothing matched");'
    }
});
