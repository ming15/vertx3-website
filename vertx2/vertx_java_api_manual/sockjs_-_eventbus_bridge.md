# SockJS - EventBus Bridge


Setting up the Bridge

By connecting up SockJS and the Vert.x event bus we create a distributed event bus which not only spans multiple Vert.x instances on the server side, but can also include client side JavaScript running in browsers.

We can therefore create a huge distributed bus encompassing many browsers and servers. The browsers don't have to be connected to the same server as long as the servers are connected.

On the server side we have already discussed the event bus API.

We also provide a client side JavaScript library called vertxbus.js which provides the same event bus API, but on the client side.

This library internally uses SockJS to send and receive data to a SockJS Vert.x server called the SockJS bridge. It's the bridge's responsibility to bridge data between SockJS sockets and the event bus on the server side.

Creating a Sock JS bridge is simple. You just call the bridge method on the SockJS server.

You will also need to secure the bridge (see below).

The following example bridges the event bus to client side JavaScript:

HttpServer server = vertx.createHttpServer();

JsonObject config = new JsonObject().putString("prefix", "/eventbus");

JsonArray noPermitted = new JsonArray();
noPermitted.add(new JsonObject());

vertx.createSockJSServer(server).bridge(config, noPermitted, noPermitted);

server.listen(8080);
To let all messages through you can specify two JSON array with a single empty JSON object which will match all messages.

Be very careful!

Using the Event Bus from client side JavaScript

Once you've set up a bridge, you can use the event bus from the client side as follows:

In your web page, you need to load the script vertxbus.js, then you can access the Vert.x event bus API. Here's a rough idea of how to use it. For a full working examples, please consult the vert.x examples.

<script src="http://cdn.sockjs.org/sockjs-0.3.4.min.js"></script>
<script src='vertxbus.js'></script>

<script>

    var eb = new vertx.EventBus('http://localhost:8080/eventbus');

    eb.onopen = function() {

      eb.registerHandler('some-address', function(message) {

        console.log('received a message: ' + JSON.stringify(message);

      });

      eb.send('some-address', {name: 'tim', age: 587});

    }

</script>
You can find vertxbus.js in the client directory of the Vert.x distribution.

The first thing the example does is to create a instance of the event bus

var eb = new vertx.EventBus('http://localhost:8080/eventbus');
The parameter to the constructor is the URI where to connect to the event bus. Since we create our bridge with the prefix eventbus we will connect there.

You can't actually do anything with the bridge until it is opened. When it is open the onopen handler will be called.

The client side event bus API for registering and unregistering handlers and for sending messages is the same as the server side one. Please consult the chapter on the event bus for full information.

There is one more thing to do before getting this working, please read the following section....

Securing the Bridge

If you started a bridge like in the above example without securing it, and attempted to send messages through it you'd find that the messages mysteriously disappeared. What happened to them?

For most applications you probably don't want client side JavaScript being able to send just any message to any verticle on the server side or to all other browsers.

For example, you may have a persistor verticle on the event bus which allows data to be accessed or deleted. We don't want badly behaved or malicious clients being able to delete all the data in your database! Also, we don't necessarily want any client to be able to listen in on any topic.

To deal with this, a SockJS bridge will, by default refuse to let through any messages. It's up to you to tell the bridge what messages are ok for it to pass through. (There is an exception for reply messages which are always allowed through).

In other words the bridge acts like a kind of firewall which has a default deny-all policy.

Configuring the bridge to tell it what messages it should pass through is easy. You pass in two Json arrays that represent matches, as arguments to bridge.

The first array is the inbound list and represents the messages that you want to allow through from the client to the server. The second array is the outbound list and represents the messages that you want to allow through from the server to the client.

Each match can have up to three fields:

address: This represents the exact address the message is being sent to. If you want to filter messages based on an exact address you use this field.
address_re: This is a regular expression that will be matched against the address. If you want to filter messages based on a regular expression you use this field. If the address field is specified this field will be ignored.
match: This allows you to filter messages based on their structure. Any fields in the match must exist in the message with the same values for them to be passed. This currently only works with JSON messages.
When a message arrives at the bridge, it will look through the available permitted entries.

If an address field has been specified then the address must match exactly with the address of the message for it to be considered matched.

If an address field has not been specified and an address_re field has been specified then the regular expression in address_re must match with the address of the message for it to be considered matched.

If a match field has been specified, then also the structure of the message must match.

Here is an example:

HttpServer server = vertx.createHttpServer();

JsonObject config = new JsonObject().putString("prefix", "/echo");

JsonArray inboundPermitted = new JsonArray();

// Let through any messages sent to 'demo.orderMgr'
JsonObject inboundPermitted1 = new JsonObject().putString("address", "demo.orderMgr");
inboundPermitted.add(inboundPermitted1);

// Allow calls to the address 'demo.persistor' as long as the messages
// have an action field with value 'find' and a collection field with value
// 'albums'
JsonObject inboundPermitted2 = new JsonObject().putString("address", "demo.persistor")
    .putObject("match", new JsonObject().putString("action", "find")
                                        .putString("collection", "albums"));
inboundPermitted.add(inboundPermitted2);

// Allow through any message with a field `wibble` with value `foo`.
JsonObject inboundPermitted3 = new JsonObject().putObject("match", new JsonObject().putString("wibble", "foo"));
inboundPermitted.add(inboundPermitted3);

JsonArray outboundPermitted = new JsonArray();

// Let through any messages coming from address 'ticker.mystock'
JsonObject outboundPermitted1 = new JsonObject().putString("address", "ticker.mystock");
outboundPermitted.add(outboundPermitted1);

// Let through any messages from addresses starting with "news." (e.g. news.europe, news.usa, etc)
JsonObject outboundPermitted2 = new JsonObject().putString("address_re", "news\\..+");
outboundPermitted.add(outboundPermitted2);

vertx.createSockJSBridge(server).bridge(config, inboundPermitted, outboundPermitted);

server.listen(8080);
Messages that require authorisation

The bridge can also refuse to let certain messages through if the user is not authorised.

To enable this you need to make sure an instance of the vertx.auth-mgr module is available on the event bus. (Please see the modules manual for a full description of modules).

To tell the bridge that certain messages require authorisation before being passed, you add the field requires_auth with the value of true in the match. The default value is false. For example, the following match:

{
  address : 'demo.persistor',
  match : {
    action : 'find',
    collection : 'albums'
  },
  requires_auth: true
}
This tells the bridge that any messages to save orders in the orders collection, will only be passed if the user is successful authenticated (i.e. logged in ok) first.
