# The Event Bus

`event bus`是Vert.x的神经系统。

每一个`Vertx`对象内部都有一个唯一的`event bus`实例，我们可以通过`eventBus`这个方法获取它的引用。

`event bus`可以让你的应用程序的不同组件进行交互, 但是强大的是进行交互的组件可以自由选择实现语言，而且并不局限于仅仅只有在相同的`Vertx`实例内的组件才能交互。

`event bus`构成了一个在多个服务器节点和多个浏览器间的分布式端对端消息系统。

`event bus`还支持以下三种消息模式：`publish/subscribe`, `point to point`, `request-response messaging`

`event bus`API是非常简单的,你基本只需要调用`registering handlers`, `unregistering handlers` 以及`sending messages`, `publishing messages`


## The Theory
### Addressing

我们通过`event bus`向一个地址发送`Message`.

在Vert.x中不需要担心是否会使用到复杂的寻址方案. 在Vert.x中，地址就是一个简单的合法字符串。Vert.x的地址还使用了一些`scheme`,例如使用`.`分割命名空间区间。

一些合法的地址例如：`europe.news.feed1`, `acme.games.pacman`, `sausages`, and `X`

### Handlers

我们使用`handler`从`event bus`中接收消息,因此你只需向一个`address`注册一个`handler`。

`handler`和`address`是一种多对多的关系,这意味着,一个`handler`可以向很多个`address`注册,同时多个`handler`可以向同一个`address`注册

### Publish / subscribe messaging

`event bus`也支持`publishing messages`:
消息会被发布到某一个地址上.这意味着：某一消息会发布给在某个地址上注册的全部`handler`。这和`publish/subscribe`消息模式很像。

### Point to point and Request-Response messaging

`event bus`支持点对点消息传送.

这种模式下消息会被发送到一个地址上。Vert.x然后会在该地址上的N个handler中选择一个,然后将消息传递给被选择的handler。

如果某个地址上注册了多个handler，Vert.x会根据`non-strict round-robin`算法来选取一个。

在点对点传送消息的情况中，当发送消息时，可以指定一个可选的回复handler。当接受者接受到一个消息后，同时该Message被处理后，接受者可以选择是否回应该消息。如果接受者选择回应该消息，那么reply handler会被调用。

当发送者接收到消息回应后，发送者还可以选择接着回应。这种模式可以永远重复下去，Vert.x还支持在这俩个verticle中创建一个会话。

这种通用的消息模式称为`Request-Response`模式。

### Best-effort delivery

Vert.x会尽自己的全力进行消息分发,而且Vert.x保证不会主动抛弃消息,这种模式称为`best-effort delivery`.

然而,当`event bus`失效时,可能会发生消息丢失情况.如果你的应用程序不允许出现消息丢失,那么你应该将你的`handler`编码成`idempotent`(code your handlers to be idempotent),当`event bus`恢复正常后,你的消息发送者再次尝试发送消息.

### Types of messages

Vert.x 消息支持所有的原生类型, `String`, `Buffer`. 但是在Vert.x中一般是使用`JSON`作为消息数据格式. 这是因为在Vert.x所支持的所有语言中，都很容易创建,读取和解析`JSON`。

当然，Vert.x并不强制你必须使用`JSON`作为消息数据传输格式。

`event bus`本身是非常灵活的,而且支持发送任意的对象数据,只要你能进行编解码就可以

## The Event Bus API
Let’s jump into the API

### Getting the event bus

下例我们演示一下如何获得`EventBus`引用：
```java
EventBus eb = vertx.eventBus();
```
每一个`Vertx`实例中都有一个`event bus`实例。

### Registering Handlers

下例演示了如何在`event bus`上注册一个`handler`
```java
EventBus eb = vertx.eventBus();

eb.consumer("news.uk.sport", message -> {
  System.out.println("I have received a message: " + message.body());
});
```
当你的`handler`收到一条`message`时, `handler`会自动被调用.

调用`consumer(.., ..)`方法的返回值是一个`MessageConsumer`实例.

我们可以通过`MessageConsumer`实例来`unregister handler`,也可以像流一样使用那个`handler`

或者你可以不向`consumer`方法中设置`handler`,那么你同样会获得一个`MessageConsumer`实例，你可以在`MessageConsumer`实例上再设置`handler`
```java
EventBus eb = vertx.eventBus();

MessageConsumer<String> consumer = eb.consumer("news.uk.sport");
consumer.handler(message -> {
  System.out.println("I have received a message: " + message.body());
});
```
当向一个集群的`event bus`上注册一个`handler`时,那么就需要向集群中的每一个节点上都要注册一个该`handler`，那这就需要消耗一些时间了。

如果你需要当向集群中所有的节点都注册完成时，捕获一个通知，那么你可以再在`MessageConsumer`上注册一个`"completion" handler`.
```java
consumer.completionHandler(res -> {
  if (res.succeeded()) {
    System.out.println("The handler registration has reached all nodes");
  } else {
    System.out.println("Registration failed!");
  }
});
```
### Un-registering Handlers

想要`unregister`一个`handler`只需要调用`unregister`方法就可以了

如果你当前的环境是一个集群环境, 那么就需要向整个集群中的所有节点都执行`unregister`操作，这同样需要一些时间等待,当然你也可以注册一个`"completion" handler`
```java
consumer.unregister(res -> {
  if (res.succeeded()) {
    System.out.println("The handler un-registration has reached all nodes");
  } else {
    System.out.println("Un-registration failed!");
  }
});
```

### Publishing messages

`publish`消息同样是非常简单的,你只需要向目标`address`上调用`publish`方法就可以了
```
eventBus.publish("news.uk.sport", "Yay! Someone kicked a ball");
```
这个消息会被分发到在目标地址上注册所有的`handler`上.

### Sending messages

`Sending`出来的消息则只会在目的地址上注册的某个`handler`接受.这是一种`point to point`消息模式.`handler`的选择同样采用的是`non-strict round-robin`算法

下例演示了如何`send message`
```java
eventBus.send("news.uk.sport", "Yay! Someone kicked a ball");
```

### Setting headers on messages

在`event bus`上传送的消息同样可以带有消息头. 在`sending`和`publishing`这俩种模式下,可以通过`DeliveryOptions`对象指定消息头
```java
DeliveryOptions options = new DeliveryOptions();
options.addHeader("some-header", "some-value");
eventBus.send("news.uk.sport", "Yay! Someone kicked a ball", options);
```

### The Message object

在消息`handler`上你接受的对象是一个`Message`实例

`Message`实例中的`body`就相当于被`sent`或者`publish`的对象.

我们还可以通过`headers`方法获得`message`的`header`.

### Replying to messages

有时候当你`send`出一个消息之后,你可能期待某些答复. 这种消息模式被称为`request-response pattern`

想要达到这种效果,你可以在`send`消息时设置一个`reply handler`.

当消息接收者收到消息后,可以通过调用消息上的`reply`方法进行应答

当接收者通过消息的`reply`方法进行应答时，那么发送者在`send`时设置的`reply handler`将会被调用,下面给出了这种应答模式的演示：

The receiver:
```java
MessageConsumer<String> consumer = eventBus.consumer("news.uk.sport");
consumer.handler(message -> {
  System.out.println("I have received a message: " + message.body());
  message.reply("how interesting!");
});
```
The sender:
```java
eventBus.send("news.uk.sport", "Yay! Someone kicked a ball across a patch of grass", ar -> {
  if (ar.succeeded()) {
    System.out.println("Received reply: " + ar.result().body());
  }
});
```
这种应答可以形成往复的应答模式从而生成一个会话

### Sending with timeouts

在`send`发送消息时，如果指定了一个`reply handler`,那么你还可以通过`DeliveryOptions`设置一个超时时间(默认是30s)。

当在指定的时间内没有收到对方应答时，`reply handler`将会以一种失败的状态被调用

### Send Failures

在消息发送时可能会在下面几种情况下引发失败：
* There are no handlers available to send the message to
* The recipient has explicitly failed the message using fail

In all cases the reply handler will be called with the specific failure.

### Message Codecs

如果你对在`event bus`上传送的对象指定一个消息编码器并且在`event bus`上注册了该消息编码器, 那么无论该对象是何类型，你都可以在`event bus`上对其进行传递.

当你`sending`或者`publishing`一个对象时, 你需要在`DeliveryOptions`对象里指定该对象所对应的编码器名称.
```java
eventBus.registerCodec(myCodec);

DeliveryOptions options = new DeliveryOptions().setCodecName(myCodec.name());

eventBus.send("orders", new MyPOJO(), options);
```

你也可以在`eventBus`上指定一个默认的编码器，这样一来，当你再`send`消息时，就不用每次都手动的设置编码器了
```java
eventBus.registerDefaultCodec(MyPOJO.class, myCodec);

eventBus.send("orders", new MyPOJO());
```
如果你想要解除一个消息编码器，你只需要使用`unregisterCodec`就好了

Message codecs don’t always have to encode and decode as the same type. For example you can write a codec that allows a MyPOJO class to be sent, but when that message is sent to a handler it arrives as a MyOtherPOJO class.



### Clustered Event Bus

`event bus`的作用域并不是单单的在一个单独的`Vertx`实例里。在集群里，你的局域网中的不同的`Vertx`实例可以聚合在一起，而每一个`Vertx`实例里的`event bus`可以相互聚集形成一个单独的分布式的`event bus`。

### Clustering programmatically

如果你通过编程的方式使用集群方法创建`Vertx`实例,在这种方式下你就得到了一个集群`event bus`
```java
VertxOptions options = new VertxOptions();
Vertx.clusteredVertx(options, res -> {
  if (res.succeeded()) {
    Vertx vertx = res.result();
    EventBus eventBus = vertx.eventBus();
    System.out.println("We now have a clustered event bus: " + eventBus);
  } else {
    System.out.println("Failed: " + res.cause());
  }
});
```
你必须确保你已经在`classpath`上实现了`ClusterManager`, 例如你也可以使用Vertx的`ClusterManager`实现

### Clustering on the command line

你可以通过下面的方式进行命令行的集群配置
```java
vertx run MyVerticle -cluster
```
##Automatic clean-up in verticles
If you’re registering event bus handlers from inside verticles, those handlers will be automatically unregistered when the verticle is undeployed.


## Examples

#### Codec
```java
class ClientCodec implements MessageCodec<ClientSource, ClientTarget> {

	/*
	 * 当把对象s传输网络中时,该方法会被调用. 
	 * 会将s写入buffer中
	 */
	@Override
	public void encodeToWire(Buffer buffer, ClientSource s) {
		
	}

	/*
	 * pos表示从buffer哪里开始读
	 */
	@Override
	public ClientTarget decodeFromWire(int pos, Buffer buffer) {
		return null;
	}

	/*
	 * 如果message是在本地event bus上传递上传输时, 该方法会被调用, 将ClientSource类型对象改变为ClientTarget
	 */
	@Override
	public ClientTarget transform(ClientSource s) {
		return null;
	}

	/*
	 * 该编码器的名称, 每个编码器都必须有一个唯一的名字. 当发送message或者从event bus上解除编码器的时候,需要使用到该编码器
	 */
	@Override
	public String name() {
		return null;
	}

	@Override
	public byte systemCodecID() {
		return -1;
	}

}

class ClientSource {
	
}

class ClientTarget {
	
}
```