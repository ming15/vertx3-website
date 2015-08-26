SockJS event bus bridge
Vert.x-Web comes with a built-in SockJS socket handler called the event bus bridge which effectively extends the server-side Vert.x event bus into client side JavaScript.

This creates a distributed event bus which not only spans multiple Vert.x instances on the server side, but includes client side JavaScript running in browsers.

We can therefore create a huge distributed bus encompassing many browsers and servers. The browsers don’t have to be connected to the same server as long as the servers are connected.

This is done by providing a simple client side JavaScript library called vertxbus.js which provides an API very similar to the server-side Vert.x event-bus API, which allows you to send and publish messages to the event bus and register handlers to receive messages.

This JavaScript library uses the JavaScript SockJS client to tunnel the event bus traffic over SockJS connections terminating at at a SockJSHandler on the server-side.

A special SockJS socket handler is then installed on the SockJSHandler which handles the SockJS data and bridges it to and from the server side event bus.

To activate the bridge you simply call bridge on the SockJS handler.

Router router = Router.router(vertx);

SockJSHandler sockJSHandler = SockJSHandler.create(vertx);
BridgeOptions options = new BridgeOptions();
sockJSHandler.bridge(options);

router.route("/eventbus/*").handler(sockJSHandler);
In client side JavaScript you use the 'vertxbus.js` library to create connections to the event bus and to send and receive messages:

<script src="http://cdn.sockjs.org/sockjs-0.3.4.min.js"></script>
<script src='vertxbus.js'></script>

<script>

var eb = new vertx.EventBus('http://localhost:8080/eventbus');

eb.onopen = function() {

  // set a handler to receive a message
  eb.registerHandler('some-address', function(message) {
    console.log('received a message: ' + JSON.stringify(message);
  });

  // send a message
  eb.send('some-address', {name: 'tim', age: 587});

}

</script>
The first thing the example does is to create a instance of the event bus

var eb = new vertx.EventBus('http://localhost:8080/eventbus');
The parameter to the constructor is the URI where to connect to the event bus. Since we create our bridge with the prefix eventbus we will connect there.

You can’t actually do anything with the connection until it is opened. When it is open the onopen handler will be called.

Securing the Bridge
If you started a bridge like in the above example without securing it, and attempted to send messages through it you’d find that the messages mysteriously disappeared. What happened to them?

For most applications you probably don’t want client side JavaScript being able to send just any message to any handlers on the server side or to all other browsers.

For example, you may have a service on the event bus which allows data to be accessed or deleted. We don’t want badly behaved or malicious clients being able to delete all the data in your database!

Also, we don’t necessarily want any client to be able to listen in on any event bus address.

To deal with this, a SockJS bridge will by default refuse to let through any messages. It’s up to you to tell the bridge what messages are ok for it to pass through. (There is an exception for reply messages which are always allowed through).

In other words the bridge acts like a kind of firewall which has a default deny-all policy.

Configuring the bridge to tell it what messages it should pass through is easy.

You can specify which matches you want to allow for inbound and outbound traffic using the BridgeOptions that you pass in when calling bridge.

Each match is a PermittedOptions object:

setAddress
This represents the exact address the message is being sent to. If you want to allow messages based on an exact address you use this field.

setAddressRegex
This is a regular expression that will be matched against the address. If you want to allow messages based on a regular expression you use this field. If the address field is specified this field will be ignored.

setMatch
This allows you to allow messages based on their structure. Any fields in the match must exist in the message with the same values for them to be allowed. This currently only works with JSON messages.
If a message is in-bound (i.e. being sent from client side JavaScript to the server) when it is received Vert.x-Web will look through any inbound permitted matches. If any match, it will be allowed through.

If a message is out-bound (i.e. being sent from the server to client side JavaScript) before it is sent to the client Vert.x-Web will look through any inbound permitted matches. If any match, it will be allowed through.

The actual matching works as follows:

If an address field has been specified then the address must match exactly with the address of the message for it to be considered matched.

If an address field has not been specified and an addressRegex field has been specified then the regular expression in address_re must match with the address of the message for it to be considered matched.

If a match field has been specified, then also the structure of the message must match. Structuring matching works by looking at all the fields and values in the match object and checking they all exist in the actual message body.

Here’s an example:

Router router = Router.router(vertx);

SockJSHandler sockJSHandler = SockJSHandler.create(vertx);


// Let through any messages sent to 'demo.orderMgr' from the client
PermittedOptions inboundPermitted1 = new PermittedOptions().setAddress("demo.orderMgr");

// Allow calls to the address 'demo.persistor' from the client as long as the messages
// have an action field with value 'find' and a collection field with value
// 'albums'
PermittedOptions inboundPermitted2 = new PermittedOptions().setAddress("demo.persistor")
    .setMatch(new JsonObject().put("action", "find")
        .put("collection", "albums"));

// Allow through any message with a field `wibble` with value `foo`.
PermittedOptions inboundPermitted3 = new PermittedOptions().setMatch(new JsonObject().put("wibble", "foo"));

// First let's define what we're going to allow from server -> client

// Let through any messages coming from address 'ticker.mystock'
PermittedOptions outboundPermitted1 = new PermittedOptions().setAddress("ticker.mystock");

// Let through any messages from addresses starting with "news." (e.g. news.europe, news.usa, etc)
PermittedOptions outboundPermitted2 = new PermittedOptions().setAddressRegex("news\\..+");

// Let's define what we're going to allow from client -> server
BridgeOptions options = new BridgeOptions().
    addInboundPermitted(inboundPermitted1).
    addInboundPermitted(inboundPermitted1).
    addInboundPermitted(inboundPermitted3).
    addOutboundPermitted(outboundPermitted1).
    addOutboundPermitted(outboundPermitted2);

sockJSHandler.bridge(options);

router.route("/eventbus/*").handler(sockJSHandler);
Requiring authorisation for messages
The event bus bridge can also be configured to use the Vert.x-Web authorisation functionality to require authorisation for messages, either in-bound or out-bound on the bridge.

To do this, you can add extra fields to the match described in the previous section that determine what authority is required for the match.

To declare that a specific authority for the logged-in user is required in order to access allow the messages you use the setRequiredAuthority field.

Here’s an example:

PermittedOptions inboundPermitted = new PermittedOptions().setAddress("demo.orderService");

// But only if the user is logged in and has the authority "place_orders"
inboundPermitted.setRequiredAuthority("place_orders");

BridgeOptions options = new BridgeOptions().addInboundPermitted(inboundPermitted);
For the user to be authorised they must be first logged in and secondly have the required authority.

To handle the login and actually auth you can configure the normal Vert.x auth handlers. For example:

Router router = Router.router(vertx);

// Let through any messages sent to 'demo.orderService' from the client
PermittedOptions inboundPermitted = new PermittedOptions().setAddress("demo.orderService");

// But only if the user is logged in and has the authority "place_orders"
inboundPermitted.setRequiredAuthority("place_orders");

SockJSHandler sockJSHandler = SockJSHandler.create(vertx);
sockJSHandler.bridge(new BridgeOptions().
        addInboundPermitted(inboundPermitted));

// Now set up some basic auth handling:

router.route().handler(CookieHandler.create());
router.route().handler(SessionHandler.create(LocalSessionStore.create(vertx)));

AuthHandler basicAuthHandler = BasicAuthHandler.create(authProvider);

router.route("/eventbus/*").handler(basicAuthHandler);


router.route("/eventbus/*").handler(sockJSHandler);
Handling event bus bridge events
If you want to be notified when an event occurs on the bridge you can provide a handler when calling bridge.

Whenever an event occurs on the bridge it will be passed to the handler. The event is described by an instance of BridgeEvent.

The event can be one of the following types:

SOCKET_CREATED
This event will occur when a new SockJS socket is created.

SOCKET_CLOSED
This event will occur when a SockJS socket is closed.

SEND
This event will occur when a message is attempted to be sent from the client to the server.

PUBLISH
This event will occur when a message is attempted to be published from the client to the server.

RECEIVE
This event will occur when a message is attempted to be delivered from the server to the client. REGISTER. This event will occur when a client attempts to register a handler. UNREGISTER. This event will occur when a client attempts to unregister a handler.
The event enables you to retrieve the type using type and inspect the raw message of the event using rawMessage.

The raw message is a JSON object with the following structure:

{
  "type": "send"|"publish"|"receive"|"register"|"unregister",
  "address": the event bus address being sent/published/registered/unregistered
  "body": the body of the message
}
The event is also an instance of Future. When you are finished handling the event you can complete the future with true to enable further processing.

If you don’t want the event to be processed you can complete the future with false. This is a useful feature that enables you to do your own filtering on messages passing through the bridge, or perhaps apply some fine grained authorisation or metrics.

Here’s an example where we reject all messages flowing through the bridge if they contain the word "Armadillos".

Router router = Router.router(vertx);

// Let through any messages sent to 'demo.orderMgr' from the client
PermittedOptions inboundPermitted = new PermittedOptions().setAddress("demo.someService");

SockJSHandler sockJSHandler = SockJSHandler.create(vertx);
BridgeOptions options = new BridgeOptions().addInboundPermitted(inboundPermitted);

sockJSHandler.bridge(options, be -> {
  if (be.type() == BridgeEvent.Type.PUBLISH || be.type() == BridgeEvent.Type.RECEIVE) {
    if (be.rawMessage().getString("body").equals("armadillos")) {
      // Reject it
      be.complete(false);
      return;
    }
  }
  be.complete(true);
});

router.route("/eventbus").handler(sockJSHandler);
You can also amend the raw message, e.g. change the body. For messages that are flowing in from the client you can also add headers to the message, here’s an example:

Router router = Router.router(vertx);

// Let through any messages sent to 'demo.orderService' from the client
PermittedOptions inboundPermitted = new PermittedOptions().setAddress("demo.orderService");

SockJSHandler sockJSHandler = SockJSHandler.create(vertx);
BridgeOptions options = new BridgeOptions().addInboundPermitted(inboundPermitted);

sockJSHandler.bridge(options, be -> {
  if (be.type() == BridgeEvent.Type.PUBLISH || be.type() == BridgeEvent.Type.SEND) {
    // Add some headers
    JsonObject headers = new JsonObject().put("header1", "val").put("header2", "val2");
    be.rawMessage().put("headers", headers);
  }
  be.complete(true);
});

router.route("/eventbus").handler(sockJSHandler);
