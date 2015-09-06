# Handling sessions

`Vert.x-Web` 同样提供了对于`session`的支持.


Sessions last between HTTP requests for the length of a browser session and give you a place where you can add session-scope information, such as a shopping basket.

Vert.x-Web uses session cookies to identify a session. The session cookie is temporary and will be deleted by your browser when it’s closed.

We don’t put the actual data of your session in the session cookie - the cookie simply uses an identifier to look-up the actual session on the server. The identifier is a random UUID generated using a secure random, so it should be effectively unguessable.

Cookies are passed across the wire in HTTP requests and responses so it’s always wise to make sure you are using HTTPS when sessions are being used. Vert.x will warn you if you attempt to use sessions over straight HTTP.

To enable sessions in your application you must have a SessionHandler on a matching route before your application logic.

The session handler handles the creation of session cookies and the lookup of the session so you don’t have to do that yourself.

## Session stores
To create a session handler you need to have a session store instance. The session store is the object that holds the actual sessions for your application.

Vert.x-Web comes with two session store implementations out of the box, and you can also write your own if you prefer.

#### Local session store

With this store, sessions are stored locally in memory and only available in this instance.

This store is appropriate if you have just a single Vert.x instance of you are using sticky sessions in your application and have configured your load balancer to always route HTTP requests to the same Vert.x instance.

If you can’t ensure your requests will all terminate on the same server then don’t use this store as your requests might end up on a server which doesn’t know about your session.

Local session stores are implemented by using a shared local map, and have a reaper which clears out expired sessions.

The reaper interval can be configured with LocalSessionStore.create.

Here are some examples of creating a LocalSessionStore

SessionStore store1 = LocalSessionStore.create(vertx);

// Create a local session store specifying the local shared map name to use
// This might be useful if you have more than one application in the same
// Vert.x instance and want to use different maps for different applications
SessionStore store2 = LocalSessionStore.create(vertx, "myapp3.sessionmap");

// Create a local session store specifying the local shared map name to use and
// setting the reaper interval for expired sessions to 10 seconds
SessionStore store3 = LocalSessionStore.create(vertx, "myapp3.sessionmap", 10000);

#### Clustered session store

With this store, sessions are stored in a distributed map which is accessible across the Vert.x cluster.

This store is appropriate if you’re not using sticky sessions, i.e. your load balancer is distributing different requests from the same browser to different servers.

Your session is accessible from any node in the cluster using this store.

To you use a clustered session store you should make sure your Vert.x instance is clustered.

Here are some examples of creating a ClusteredSessionStore

Vertx.clusteredVertx(new VertxOptions().setClustered(true), res -> {

  Vertx vertx = res.result();

  // Create a clustered session store using defaults
  SessionStore store1 = ClusteredSessionStore.create(vertx);

  // Create a clustered session store specifying the distributed map name to use
  // This might be useful if you have more than one application in the cluster
  // and want to use different maps for different applications
  SessionStore store2 = ClusteredSessionStore.create(vertx, "myclusteredapp3.sessionmap");
});
## Creating the session handler
Once you’ve created a session store you can create a session handler, and add it to a route. You should make sure your session handler is routed to before your application handlers.

You’ll also need to include a CookieHandler as the session handler uses cookies to lookup the session. The cookie handler should be before the session handler when routing.

Here’s an example:

Router router = Router.router(vertx);

// We need a cookie handler first
router.route().handler(CookieHandler.create());

// Create a clustered session store using defaults
SessionStore store = ClusteredSessionStore.create(vertx);

SessionHandler sessionHandler = SessionHandler.create(store);

// Make sure all requests are routed through the session handler too
router.route().handler(sessionHandler);

// Now your application handlers
router.route("/somepath/blah/").handler(routingContext -> {

  Session session = routingContext.session();
  session.put("foo", "bar");
  // etc

});
The session handler will ensure that your session is automatically looked up (or created if no session exists) from the session store and set on the routing context before it gets to your application handlers.

## Using the session
In your handlers you an access the session instance with session.

You put data into the session with put, you get data from the session with get, and you remove data from the session with remove.

The keys for items in the session are always strings. The values can be any type for a local session store, and for a clustered session store they can be any basic type, or Buffer, JsonObject, JsonArray or a serializable object, as the values have to serialized across the cluster.

Here’s an example of manipulating session data:

router.route().handler(CookieHandler.create());
router.route().handler(sessionHandler);

// Now your application handlers
router.route("/somepath/blah").handler(routingContext -> {

  Session session = routingContext.session();

  // Put some data from the session
  session.put("foo", "bar");

  // Retrieve some data from a session
  int age = session.get("age");

  // Remove some data from a session
  JsonObject obj = session.remove("myobj");

});
Sessions are automatically written back to the store after after responses are complete.

You can manually destroy a session using destroy. This will remove the session from the context and the session store. Note that if there is no session a new one will be automatically created for the next request from the browser that’s routed through the session handler.

## Session timeout
Sessions will be automatically timed out if they are not accessed for a time greater than the timeout period. When a session is timed out, it is removed from the store.

Sessions are automatically marked as accessed when a request arrives and the session is looked up and and when the response is complete and the session is stored back in the store.

You can also use setAccessed to manually mark a session as accessed.

The session timeout can be configured when creating the session handler. Default timeout is 30 minutes.

