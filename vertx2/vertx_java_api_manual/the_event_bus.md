# The Event Bus

`event bus`充当着Vert.x的"神经系统"

它允许vertivle能够相互通信，不管这些verticle是否是同一种语言实现，或者是否是在同一个`Vert.x`实例里。

It even allows client side JavaScript running in a browser to communicate on the same event bus. (More on that later).

它甚至允许运行在浏览器里的同一个event bus的JavaScript形式的verticle相互交互

`event bus`形成了一个横跨多个服务器节点以及多个浏览器的分布式的端对端的消息系统,

`event bus`的API是相当简单的. 它基本上只涉及了`registering handlers`, `unregistering handlers` 和 `sending/publishing messages`.

## The Theory

### Addressing

我们通过`event bus`向一个地址发送`Message`.

在Vert.x中不需要担心是否会使用到复杂的寻址方案. 在Vert.x中，地址就是一个简单的合法字符串。Vert.x的地址还使用了一些`scheme`,例如使用`.`分割命名空间区间。

一些合法的地址例如：`europe.news.feed1`, `acme.games.pacman`, `sausages`, and `X`

### Handlers

我们使用`handler`从`event bus`中接收消息——向一个地址注册一个`handler`。

无论是否是同一个`verticle`中的`handler`都可以向相同的地址进行注册。`verticle`中的同一个`handler`也可以注册到不同的地址上

### Publish / subscribe messaging

`event bus`也支持消息发布——消息会被发布到某一个地址上.消息发布意味着：将消息发布给在某个地址上注册的全部`handler`。这和`publish/subscribe`消息模式很像。

### Point to point and Request-Response messaging

`event bus`支持点对点消息传送.消息会被发送到一个地址上。Vert.x然后会在该地址上的N个handler中选择一个,然后将消息传递给被选择的handler。如果某个地址上注册了多个handler，Vert.x会根据一个不是很严格的循环算法来选取一个。

在点对点传送消息的情况中，当发送消息时，可以指定一个可选的回复handler。当接受者接受到一个消息后，同时该Message被处理后，接受者可以选择是否回应该消息。如果接受者选择回应该消息，那么reply handler会被调用。

当发送者接收到消息回应后，发送者还可以选择接着回应。这种模式可以永远重复下去，Vert.x还支持在这俩个verticle中创建一个会话。这种通用的消息模式称为`Request-Response`模式。

### Transient

`event bus`消息都具有瞬时性，当`event bus`全部或者部分失败后，那就有可能丢失一部分消息。如果你的应用程序不允许出现消息丢失，那么你应该将你的`handler`编码成`idempotent`，同时当`event bus`恢复后，你的sender再尝试回应消息。

如果你想要持久有你的消息，你可以使用`persistent work queue module`

### Types of messages

在`event bus`上传递的消息可以是一个简单的字符串，一个数字，一个boolean，或者是`Vert.x Buffer` 或者`JSON`消息。

但是我们强烈建议你在不同的verticle中通过JSON消息进行通信。JSON可以在Vert.x支持的语言中轻松地创建和解析。

### Event Bus API

### Registering and Unregistering Handlers

下例展示了如何在`test.address`上注册一个消息`handler`。
```java
EventBus eb = vertx.eventBus();

Handler<Message> myHandler = new Handler<Message>() {
    public void handle(Message message) {
        System.out.println("I received a message " + message.body);
    }
};

eb.registerHandler("test.address", myHandler);
```

`myHandler`会接受到所有发送到`test.address`地址上的消息。

`Message`是一个泛型类，已经指定的消息类型有：`Message<Boolean>, Message<Buffer>, Message<byte[]>, Message<Byte>, Message<Character>, Message<Double>, Message<Float>, Message<Integer>, Message<JsonObject>, Message<JsonArray>, Message<Long>, Message<Short> and Message<String>`

如果你确定接受到的消息都是同一种类型，那么你可以在handler上使用指定类型
```java
Handler<Message<String>> myHandler = new Handler<Message<String>>() {
    public void handle(Message<String> message) {
        String body = message.body;
    }
};
```

`registerHandler`方法返回的是`event bus`自身。我们提供了一个流畅的API，因此你可以将多个调用连接在一起。

当你向某个地址中注册一个handler，同时处于一个集群中，那该注册过程就需要耗费一点时间来在整个集群中的进行传播。如果你想`handler`注册成功后获得通知，那么你可以向`registerHandler`方法的第三个参数中指定另一个handler。当集群中的所有节点都收到向某个地址注册`handler`信息之后，那么第三个参数`handler`就会被调用,然后你就会收到handler注册完成的通知了。
```java
eb.registerHandler("test.address", myHandler, new AsyncResultHandler<Void>() {
    public void handle(AsyncResult<Void> asyncResult) {
        System.out.println("The handler has been registered across the cluster ok? " + asyncResult.succeeded());
    }
});
```

解除`handler`注册也是非常简单的，你只需要向`unregisterHandler`方法传递注册地址和已经注册上的那个`handler`对象就可以了。
```java
eb.unregisterHandler("test.address", myHandler);
```

一个`handler`可以向相同的或者不同的地址上注册多次，因此为了在handler解除注册时，能够确定handler的唯一性，在解除注册时你需要同时指定要被解除的`handler`对象和注册地址

和注册一样，当你在一个集群环境中解除handler注册，这个过程需要耗费一些时间，以便整个集群都会收到该解除注册通知。同样的你如果想要当解除注册完成之后获得通知，`registerHandler`给这个函数增加一个第三个参数就可以了

```java
eb.unregisterHandler("test.address", myHandler, new AsyncResultHandler<Void>() {
    public void handle(AsyncResult<Void> asyncResult) {
        System.out.println("The handler has been unregistered across the cluster ok? " + asyncResult.succeeded());
    }
});
```

如果你想要你的handler存在于整个verticle的生命周期内，那么你就没有必要显式地去对该handler进行解除注册，当verticle停止的时候，Vert.x会自动对其进行解除注册

## Publishing messages

发布一个消息也是非常简单的，你只需要指定一个发布地址，然后在指定发布的内容就可以了
```java
eb.publish("test.address", "hello world");
```

这个消息会发布给在该地址上注册的所有handler。

## Sending messages

通过`send`发送消息，那么目标地址上只有一个handler进行消息接受。这是一种点对点的发送消息模式。选取handler同样采用了一种不是很严格的`round-robin`算法
```java
eb.send("test.address", "hello world");
```

## Replying to messages

当你接受到一个消息后，你可能需要对该消息进行回应，这种模式称为`request-response`

当你`send`一个消息时，你将一个回应`handler`作为第三个参数。当接受者接收到消息后，他们可以调用`Message`的`reply`方法来回应消息。当`reply`方法被调用的时候，它会将回复消息发送者。
```java
Handler<Message<String>> myHandler = new Handler<Message<String>>() {
    public void handle(Message<String> message) {
        System.out.println("I received a message " + message.body);

        // Do some stuff

        // Now reply to it

        message.reply("This is a reply");
    }
};

eb.registerHandler("test.address", myHandler);
```

The sender:
```java
eb.send("test.address", "This is a message", new Handler<Message<String>>() {
    public void handle(Message<String> message) {
        System.out.println("I received a reply " + message.body);
    }
});
```
发送空的`reply`或者`null reply`都是合法的。

The replies themselves can also be replied to so you can create a dialog between two different verticles consisting of multiple rounds.

## Specifying timeouts for replies

如果你在发送消息时指定了一个reply handler, 但是却一直得不到回复响应，那么那么该handler永远都不会被解除注册。

为了解决这个问题，你可以指定一个`Handler<AsyncResult<Message>>`作为`reply handler`，然后再设置一个超时时间。如果在超时之前，你收到了消息的reply，那么该`AsyncResult`的`handler`方法就会被调用。如果超时前一直都得不到`reply`，那么该`handler`就会自动被解除注册，同时`new Handler<AsyncResult<Message<String>>>()`也会被调用，但是`AsyncResult`会包含一个失败的状态，你可以在这种状态下做一些特殊处理:
```java
eb.sendWithTimeout("test.address", "This is a message", 1000, new Handler<AsyncResult<Message<String>>>() {
    public void handle(AsyncResult<Message<String>> result) {
        if (result.succeeded()) {
            System.out.println("I received a reply " + message.body);
        } else {

            System.err.println("No reply was received before the 1 second timeout!");
        }
    }
});
```

当`send`超时之后，我们可以通过`AsyncResult`的`cause()`来获得一个`ReplyException`异常信息。`ReplyException`上的`failureType()`值是`ReplyFailure.TIMEOUT`

你也可以在`event bus`自身上设置一个超时时间. 如果你在`event bus`使用带有reply handler的`send(...)`方法，那这个超时时间就会被使用到。默认的超时时间是`-1`,这意味着reply handler 永远不会超时
```java
eb.setDefaultReplyTimeout(5000);

eb.send("test.address", "This is a message", new Handler<Message<String>>() {
    public void handle(Message<String> message) {
        System.out.println("I received a reply before the timeout of 5 seconds");
    }
});
```

同样，你也可以对reply设置一个超时，然后使用`Handler<AsyncResult<Message>>`在超时时间内获得reply的reply：
```java
message.replyWithTimeout("This is a reply", 1000, new Handler<AsyncResult<Message<String>>>() {
    public void handle(AsyncResult<Message<String>> result) {
        if (result.succeeded()) {
            System.out.println("I received a reply to the reply" + message.body);
        } else {
            System.err.println("No reply to the reply was received before the 1 second timeout!");
        }
    }
});
```

## Getting notified of reply failures

如果你使用超时和一个`result handler`去`send`一个消息，但是没有可用的handler将消息发送出去，那么result handler将会被调用，`AsyncResult`会是一个失败的状态,同样`cause()`会返回一个`ReplyException`. `ReplyException`实例的`failureType()`的返回值是`ReplyFailure.NO_HANDLERS`

如果你使用超时和一个`result handler`去`send`一个消息，但是接受者通过调用`Message.fail(..)`回应该消息, `result handler`会被调用，`AsyncResult`会是一个失败的状态,同样`cause()`会返回一个`ReplyException`. `ReplyException`实例的`failureType()`的返回值是`ReplyFailure.RECIPIENT_FAILURE`

For example:
```java
eb.registerHandler("test.address", new Handler<Message<String>>() {
    public void handle(Message<String> message) {
        message.fail(123, "Not enough aardvarks");
    }
});

eb.sendWithTimeout("test.address", "This is a message", 1000, new Handler<AsyncResult<Message<String>>>() {
    public void handle(AsyncResult<Message<String>> result) {
        if (result.succeeded()) {
            System.out.println("I received a reply " + message.body);
        } else {
            ReplyException ex = (ReplyException)result.cause();
            System.err.println("Failure type: " + ex.failureType();
            System.err.println("Failure code: " + ex.failureCode();
            System.err.println("Failure message: " + ex.message();
        }
    }
});
```

## Message types

你发送的消息类型可以是以下几种(包括部分包装类型)

* `boolean`
* `byte[]`
* `byte`
* `char`
* `double`
* `float`
* `int`
* `long`
* `short`
* `java.lang.String`
* `org.vertx.java.core.json.JsonObject`
* `org.vertx.java.core.json.JsonArray`
* `org.vertx.java.core.buffer.Buffer`

如果`Vert.x buffers` 和 `JSON objects and arrays`是在相同的JVM里进行传递，那么在传递之前，他们会被copy一份，因此不同的verticle不能访问相同的对象实例，相同的对象实例会引发条件竞争。

Send some numbers:
```java
eb.send("test.address", 1234);
eb.send("test.address", 3.14159);
Send a boolean:

eb.send("test.address", true);
```

Send a JSON object:
```java
JsonObject obj = new JsonObject();
obj.putString("foo", "wibble");
eb.send("test.address", obj);
```

Null messages can also be sent:

```java
eb.send("test.address", null);
```

使用JSON作为verticle通信协议是一个不错的约定，这是因为JSON可以被所有Vert.x所支持的语言进行编解码

## Distributed event bus


如果想要在你的特定网络内每一个Vert.x实例都在相同的`event bus`里，你只需要在命令行里启动Vert.x实例时添加`-cluster`参数就好了

一旦你成功启动，集群模式下的Vert.x实例就会合并到一起，组成一个分布式的`event bus`
