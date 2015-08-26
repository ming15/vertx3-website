Authentication - authorisation
Vert.x comes with some out-of-the-box handlers for handling both authentication and authorisation.

Creating an auth handler
To create an auth handler you need an instance of AuthProvider. Auth provider is used for authentication and authorisation of users. Vert.x provides several auth provider instances out of the box in the vertx-auth project. For full information on auth providers and how to use and configure them please consult the auth documentation.

Here’s a simple example of creating a basic auth handler given an auth provider.

router.route().handler(CookieHandler.create());
router.route().handler(SessionHandler.create(LocalSessionStore.create(vertx)));

AuthHandler basicAuthHandler = BasicAuthHandler.create(authProvider);
Handling auth in your application
Let’s say you want all requests to paths that start with /private/ to be subject to auth. To do that you make sure your auth handler is before your application handlers on those paths:

router.route().handler(CookieHandler.create());
router.route().handler(SessionHandler.create(LocalSessionStore.create(vertx)));
router.route().handler(UserSessionHandler.create(authProvider));

AuthHandler basicAuthHandler = BasicAuthHandler.create(authProvider);

// All requests to paths starting with '/private/' will be protected
router.route("/private/*").handler(basicAuthHandler);

router.route("/someotherpath").handler(routingContext -> {

  // This will be public access - no login required

});

router.route("/private/somepath").handler(routingContext -> {

  // This will require a login

  // This will have the value true
  boolean isAuthenticated = routingContext.user() != null;

});
If the auth handler has successfully authenticated and authorised the user it will inject a User object into the RoutingContext so it’s available in your handlers with: user.

If you want your User object to be stored in the session so it’s available between requests so you don’t have to authenticate on each request, then you should make sure you have a session handler and a user session handler on matching routes before the auth handler.

Once you have your user object you can also programmatically use the methods on it to authorise the user.

If you want to cause the user to be logged out you can call clearUser on the routing context.

HTTP Basic Authentication
HTTP Basic Authentication is a simple means of authentication that can be appropriate for simple applications.

With basic auth, credentials are sent unencrypted across the wire in HTTP headers so it’s essential that you serve your application using HTTPS not HTTP.

With basic auth, if a user requests a resource that requires authorisation, the basic auth handler will send back a 401 response with the header WWW-Authenticate set. This prompts the browser to show a log-in dialogue and prompt the user to enter their username and password.

The request is made to the resource again, this time with the Authorization header set, containing the username and password encoded in Base64.

When the basic auth handler receives this information, it calls the configured AuthProvider with the username and password to authenticate the user. If the authentication is successful the handler attempts to authorise the user. If that is successful then the routing of the request is allowed to continue to the application handlers, otherwise a 403 response is returned to signify that access is denied.

The auth handler can be set-up with a set of authorities that are required for access to the resources to be granted.

Redirect auth handler
With redirect auth handling the user is redirected to towards a login page in the case they are trying to access a protected resource and they are not logged in.

The user then fills in the login form and submits it. This is handled by the server which authenticates the user and, if authenticated redirects the user back to the original resource.

To use redirect auth you configure an instance of RedirectAuthHandler instead of a basic auth handler.

You will also need to setup handlers to serve your actual login page, and a handler to handle the actual login itself. To handle the login we provide a prebuilt handler FormLoginHandler for the purpose.

Here’s an example of a simple app, using a redirect auth handler on the default redirect url /loginpage.

router.route().handler(CookieHandler.create());
router.route().handler(SessionHandler.create(LocalSessionStore.create(vertx)));
router.route().handler(UserSessionHandler.create(authProvider));

AuthHandler redirectAuthHandler = RedirectAuthHandler.create(authProvider);

// All requests to paths starting with '/private/' will be protected
router.route("/private/*").handler(redirectAuthHandler);

// Handle the actual login
router.route("/login").handler(FormLoginHandler.create(authProvider));

// Set a static server to serve static resources, e.g. the login page
router.route().handler(StaticHandler.create());

router.route("/someotherpath").handler(routingContext -> {
  // This will be public access - no login required
});

router.route("/private/somepath").handler(routingContext -> {

  // This will require a login

  // This will have the value true
  boolean isAuthenticated = routingContext.user() != null;

});
JWT authorisation
With JWT authorisation resources can be protected by means of permissions and users without enough rights are denied access.

To use this handler there are 2 steps involved:

Setup an handler to issue tokens (or rely on a 3rd party)

Setup the handler to filter the requests

Please note that these 2 handlers should be only available on HTTPS, not doing so allows sniffing the tokens in transit which leads to session hijacking attacks.

Here’s an example on how to issue tokens:

Router router = Router.router(vertx);

JsonObject authConfig = new JsonObject().put("keyStore", new JsonObject()
    .put("type", "jceks")
    .put("path", "keystore.jceks")
    .put("password", "secret"));

JWTAuth authProvider = JWTAuth.create(vertx, authConfig);

router.route("/login").handler(ctx -> {
  // this is an example, authentication should be done with another provider...
  if ("paulo".equals(ctx.request().getParam("username")) && "secret".equals(ctx.request().getParam("password"))) {
    ctx.response().end(authProvider.generateToken(new JsonObject().put("sub", "paulo"), new JWTOptions()));
  } else {
    ctx.fail(401);
  }
});
Now that your client has a token all it is required is that for all consequent request the HTTP header Authorization is filled with: Bearer <token> e.g.:

Router router = Router.router(vertx);

JsonObject authConfig = new JsonObject().put("keyStore", new JsonObject()
    .put("type", "jceks")
    .put("path", "keystore.jceks")
    .put("password", "secret"));

JWTAuth authProvider = JWTAuth.create(vertx, authConfig);

router.route("/protected/*").handler(JWTAuthHandler.create(authProvider));

router.route("/protected/somepage").handler(ctx -> {
  // some handle code...
});
JWT allows you to add any information you like to the token itself. By doing this there is no state in the server which allows you to scale your applications without need for clustered session data. In order to add data to the token, during the creation of the token just add data to the JsonObject parameter:

JsonObject authConfig = new JsonObject().put("keyStore", new JsonObject()
    .put("type", "jceks")
    .put("path", "keystore.jceks")
    .put("password", "secret"));

JWTAuth authProvider = JWTAuth.create(vertx, authConfig);

authProvider.generateToken(new JsonObject().put("sub", "paulo").put("someKey", "some value"), new JWTOptions());
And the same when consuming:

Handler<RoutingContext> handler = rc -> {
  String theSubject = rc.user().principal().getString("sub");
  String someKey = rc.user().principal().getString("someKey");
};
Configuring required authorities
With any auth handler you can also configure required authorities to access the resource.

By default, if no authorities are configured then it is sufficient to be logged in to access the resource, otherwise the user must be both logged in (authenticated) and have the required authorities.

Here’s an example of configuring an app so that different authorities are required for different parts of the app. Note that the meaning of the authorities is determined by the underlying auth provider that you use. E.g. some may support a role/permission based model but others might use another model.

AuthHandler listProductsAuthHandler = RedirectAuthHandler.create(authProvider);
listProductsAuthHandler.addAuthority("list_products");

// Need "list_products" authority to list products
router.route("/listproducts/*").handler(listProductsAuthHandler);

AuthHandler settingsAuthHandler = RedirectAuthHandler.create(authProvider);
settingsAuthHandler.addAuthority("role:admin");

// Only "admin" has access to /private/settings
router.route("/private/settings/*").handler(settingsAuthHandler);
