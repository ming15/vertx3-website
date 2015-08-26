Templates
Vert.x-Web includes dynamic page generation capabilities by including out of the box support for several popular template engines. You can also easily add your own.

Template engines are described by TemplateEngine. In order to render a template render is used.

The simplest way to use templates is not to call the template engine directly but to use the TemplateHandler. This handler calls the template engine for you based on the path in the HTTP request.

By default the template handler will look for templates in a directory called templates. This can be configured.

The handler will return the results of rendering with a content type of text/html by default. This can also be configured.

When you create the template handler you pass in an instance of the template engine you want.

Here are some examples

TemplateEngine engine = HandlebarsTemplateEngine.create();
TemplateHandler handler = TemplateHandler.create(engine);

// This will route all GET requests starting with /dynamic/ to the template handler
// E.g. /dynamic/graph.hbs will look for a template in /templates/dynamic/graph.hbs
router.get("/dynamic/").handler(handler);

// Route all GET requests for resource ending in .hbs to the template handler
router.getWithRegex(".+\\.hbs").handler(handler);
MVEL template engine
When using the MVEL template engine, it will by default look for templates with the .templ extension if no extension is specified in the file name.

The routing context RoutingContext is available in the MVEL template as the context variable, this means you can render the template based on anything in the context including the request, response, session or context data.

Here are some examples:

The request path is @{context.request().path()}

The variable 'foo' from the session is @{context.session().get('foo')}

The value 'bar' from the context data is @{context.get('bar')}
Please consult the MVEL templates documentation for how to write MVEL templates.

Jade template engine
When using the Jade template engine, it will by default look for templates with the .jade extension if no extension is specified in the file name.

The routing context RoutingContext is available in the Jade template as the context variable, this means you can render the template based on anything in the context including the request, response, session or context data.

Here are some examples:

!!! 5
html
  head
    title= context.get('foo') + context.request().path()
  body
Please consult the Jade4j documentation for how to write Jade templates.

Handlebars template engine
When using the Handlebars template engine, it will by default look for templates with the .hbs extension if no extension is specified in the file name.

Handlebars templates are not able to call arbitrary methods in objects so we can’t just pass the routing context into the template and let the template introspect it like we can with other template engines.

Instead, the context data is available in the template.

If you want to have access to other data like the request path, request params or session data you should add it the context data in a handler before the template handler. For example:

TemplateEngine engine = HandlebarsTemplateEngine.create();
TemplateHandler handler = TemplateHandler.create(engine);

router.get("/dynamic").handler(routingContext -> {

  routingContext.put("request_path", routingContext.request().path());
  routingContext.put("session_data", routingContext.session().data());

  routingContext.next();
});

router.get("/dynamic/").handler(handler);
Please consult the Handlebars Java port documentation for how to write handlebars templates.

Thymeleaf template engine
When using the Thymeleaf template engine, it will by default look for templates with the .html extension if no extension is specified in the file name.

The routing context RoutingContext is available in the Thymeleaf template as the context variable, this means you can render the template based on anything in the context including the request, response, session or context data.

Here are some examples:

[snip]
&lt;p th:text="${context.get('foo')}"&gt;&lt;/p&gt;
&lt;p th:text="${context.get('bar')}"&gt;&lt;/p&gt;
&lt;p th:text="${context.normalisedPath()}"&gt;&lt;/p&gt;
&lt;p th:text="${context.request().params().get('param1')}"&gt;&lt;/p&gt;
&lt;p th:text="${context.request().params().get('param2')}"&gt;&lt;/p&gt;
[snip]
Please consult the Thymeleaf documentation for how to write Thymeleaf templates.

