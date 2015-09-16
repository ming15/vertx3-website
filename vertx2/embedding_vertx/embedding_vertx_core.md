# Embedding Vert.x core

###### Please note this feature is intended for power users only.

你也可以直接在应用程序中内嵌Vert.x core代码，而不用引入整个Vert.x平台

如果你想要在应用程序直接使用Vert.x部分，你需要注意以下几点：
* 在你的应用程序中，你只能使用Vert.x 的Java部分，不能使用Vert.x支持的其他语言
* 在你的应用程序中，你没办法访问Vert.x的module系统，因此由社区提供的各种功能模块你就没办法使用了
* 在你的应用程序中，你没办法使用Java运行期的Java源码verticle的自动编译功能，因此你不得不手动编译你所有的源代码。
* 在你的应用程序中，你必须自己通过程序的方式创建多个服务器实例来扩展应用程序。你没办法像使用`vertx -instances`命令或者通过Vert.x平台API那样让vert.x自己拓展服务器处理能力
* 你必须要自己确保，在你的应用程序中你不会并发访问那些非线程安全的vert.x core对象(Ver.x平台会对此进行保证)。

如果你确保以上问题都okay，那么你可以使用下面这种方式：
```java
public class Embedded {
  public static void main(String[] args) throws Exception {

    Vertx vertx = VertxFactory.newVertx();

    // Create an echo server
    vertx.createNetServer().connectHandler(new Handler<NetSocket>() {
      public void handle(final NetSocket socket) {
        Pump.createPump(socket, socket).start();
      }
    }).listen(1234);

    // Prevent the JVM from exiting
    System.in.read();

  }
}
```
You first get a reference to the Vertx object using the VertxFactory class, then you use the core classes as you wish.

Note that all Vert.x threads are daemon threads and they will not prevent the JVM for exiting. Therefore you must ensure that the main() method does not run to completion, e.g. in this example we have used System.in.read() to block the main thread waiting for IO from the console.

### Required jars

To embed the Vert.x platform you will need all the jars from the Vert.x install lib directory apart from vertx-platform-*.jar on your classpath. You won't need the hazelcast jar if you aren't using clustering.

### Core thread safety

Many of the Vert.x core classes are not thread-safe. When running Vert.x in the Vert.x platform you don't have to worry about that as Vert.x guarantees that your verticle code is never executed by more than one thread concurrently.

When running core embedded you have to be more careful as there is no container to make such guarantees. Therefore it's up to you the developer to guard against concurrent access to non thread-safe classes.

Please consult the JavaDoc to see which classes are thread-safe and which are not.

#### Event loops and scaling Vert.x embedded

When running your Vert.x code in a standard verticle, Vert.x ensures it's all executed by the exact same thread (event loop).

When running Vert.x embedded you will start off on one of your own application threads which won't be a Vert.x event loop. When Vert.x calls the handlers of any of the core objects that you create, they will be called on an event loop.

Vert.x determines which event loops to use according to the following rules:

If the calling thread is not an event loop then:

* Instances of NetServer, NetClient, HttpServer and HttpClient will be assigned an event loop when they are created. When any of their handlers are subsequently called they will be called using that event loop.
* When a handler is registered on the event bus an event loop will be assigned to it. This event loop will then be used every time the handler is called.
* File system operation handlers are called using an event loop determined by Vert.x
If the calling thread is an event loop, then that current event loop will be used.

Since a particular instance of NetServer, HttpServer will have all their handlers executed on the same event loop, if you want to scale your servers across multiple cores you need to create multiple instances of them, from a thread that is not an event loop.