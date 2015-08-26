Handling cookies
Vert.x-Web has cookies support using the CookieHandler.

You should make sure a cookie handler is on a matching route for any requests that require this functionality.

router.route().handler(CookieHandler.create());
Manipulating cookies
You use getCookie to retrieve a cookie by name, or use cookies to retrieve the entire set.

To remove a cookie, use removeCookie.

To add a cookie use addCookie.

The set of cookies will be written back in the response automatically when the response headers are written so the browser can store them.

Cookies are described by instances of Cookie. This allows you to retrieve the name, value, domain, path and other normal cookie properties.

Here’s an example of querying and adding cookies:

router.route().handler(CookieHandler.create());

router.route("some/path/").handler(routingContext -> {

  Cookie someCookie = routingContext.getCookie("mycookie");
  String cookieValue = someCookie.getValue();

  // Do something with cookie...

  // Add a cookie - this will get written back in the response automatically
  routingContext.addCookie(Cookie.cookie("othercookie", "somevalue"));
});
