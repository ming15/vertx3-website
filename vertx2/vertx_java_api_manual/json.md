# JSON

在javascript中有一等类支持`JSON`, RUBY中有哈希字面量非常好的支持JSON,但是java并不支持这俩点.

因此,如果你想要在java `verticle`中使用JSON,我们提供了一些简单的JSON类,这些JSON类可以表示JSON对象或者JSON数组.那些类提供了从一个JSON对象或者数组中set/get JSON支持的所有类型。

JSON对象是`org.vertx.java.core.json.JsonObject`的实例.JSON数组是`org.vertx.java.core.json.JsonArray`的实例

下面的例子给出了在java verticle中在从`event bus`中收发JSON消息
```java
EventBus eb = vertx.eventBus();

JsonObject obj = new JsonObject().putString("foo", "wibble")
                                 .putNumber("age", 1000);

eb.send("some-address", obj);


// ....
// And in a handler somewhere:

public void handle(Message<JsonObject> message) {
    System.out.println("foo is " + message.body.getString("foo");
    System.out.println("age is " + message.body.getNumber("age");
}
```

我们还提供了对象和JSON格式之间的转化方法

Please see the JavaDoc for the full Java Json API.
