# Writing Verticles

# Writing Verticles

正如我们在手册里描述的那样,一个verticle是就是一个Vert.x的执行单元

再重复一下，Vert.x是一个Verticle容器，而且Vert.x确保一个verticle实例永远不会被多个线程并发执行。你可以使用Vert.x支持的所有的语言来编写Verticle，同时Vert.x支持并发执行同一个verticle文件实例出多个Verticle实例。

在Vert.x中，你所编写的所有代码其实都是在Verticle实例中运行。

对于一个简单的任务，你可以直接编写原生verticle，然后在命令行中直接运行它们，但是在大部分情况中你都应该将verticle打包成Vert.x module。

> 原生verticle，指的就是一个单独的没有打包进module的文件或者类,例如`verticle1.class, verticle2.java, verticle3.rb, verticle4.groovy`

现在让我们编写一个简单的原生verticle：

我们将编写一个简单的TCP echo服务器。这个服务器仅仅接受网络连接，然后将接收到的数据进行输出。

```java
import org.vertx.java.core.Handler;
import org.vertx.java.core.net.NetSocket;
import org.vertx.java.core.streams.Pump;
import org.vertx.java.platform.Verticle;

public class Server extends Verticle {

  public void start() {
    vertx.createNetServer().connectHandler(new Handler<NetSocket>() {
      public void handle(final NetSocket socket) {
        Pump.createPump(socket, socket).start();
      }
    }).listen(1234);
  }
}
```
现在运行它
```
vertx run Server.java
```
现在服务器运行起来了，然后通过telnet连接它
```
telnet localhost
```
注意，你通过回车发送出去的数据是如何输出的

现在，你已经编写了第一个verticle。

也许你已经注意到了，你并没有手动将`.java`文件编译成`.class`文件。Vert.x知道如何直接"运行"`.java`文件，其实在Vert.x内部会自动编译该源文件。

每一个java vertivle都必须继承`org.vertx.java.deploy.Verticle`,然后必须重载`start`方法，当verticle启动时，Vert.x会自动调用该方法。


## Asynchronous start

假设现在有一个Verticle——`v1`不得不在`start()`方法中，完成一些异步的操作，或者启动一些其他verticle，在这些操作完成之前，`v1`一直都应该是未完成状态。

在这种情况下，你的verticle可以实现`start()`方法的异步版本：
```java
public void start(final Future<Void> startedResult) {
  // For example - deploy some other verticle
  container.deployVerticle("foo.js", new AsyncResultHandler<String>() {
    public void handle(AsyncResult<String> deployResult) {
      if (deployResult.succeeded()) {
        startedResult.setResult(null);
      } else {
        startedResult.setFailure(deployResult.cause());
      }
    }
  });
}
```

## Verticle clean-up

当verticle停止后，其内部的`Servers, clients, event bus handlers and timers`会自动关闭或者取消掉，当某个verticle停止时，你如果想要进行一些其他的清理逻辑，你可以自己实现`stop()`方法，那么当该verticle被解除部署时，该方法就会被自动调用

## The container object

每一个verticle实例都有一个称为`container`的成员变量。`container`表示的是它运行所在的Verticle的一个视图。

`container`对象定义了部署和解除部署verticle和module的方法，同时还允许设置环境变量和一个可访问的logger

## The vertx object

每一个verticle实例都含有一个`vertx`实例变量。该变量提供了访问Vert.x核心API的能力。在Vert.x中，你要使用该核心API完成大部分工作，例如`TCP, HTTP, file system access, event bus, timers`等等。

## Getting Configuration in a Verticle

你可以像下例这样在命令行中通过`-conf`选项向module或者verticle传递配置
```
vertx runmod com.mycompany~my-mod~1.0 -conf myconf.json
```

或者向一个原生vertile传递
```
vertx run foo.js -conf myconf.json
```

`-conf`参数是一个包含JSON对象的文本文件名字。

通过调用verticle成员变量`contailner`的`config()`方法该配置就成功启用了
```java
JsonObject config = container.config();

System.out.println("Config is " + config);
```

`config()`返回一个`org.vertx.java.core.json.JsonObject`实例，该实例代表一个json对象。


无论部署什么语言实现的verticle，对于配置verticle的方式是一致的。

## Logging from a Verticle

每个verticle实例都有一个属于它自己的logger。可以通过调用`container`实例的`logger()`方法获取`logger`对象的引用。

```java
Logger logger = container.logger();

logger.info("I am logging something");
```
The logger is an instance of the class org.vertx.java.core.logging.Logger and has the following methods;

`logger`是`org.vertx.java.core.logging.Logger`的实例,该实例拥有下列方法：

* `trace`
* `debug`
* `info`
* `warn`
* `error`
* `fatal`

`logger`产生的日志存储到系统临时目录的`vertx.log`文件中,在linux中的临时目录是`\tmp`.

更多关于配置logging方法的信息，参考主手册

## Accessing environment variables from a Verticle

你可以通过调用`container`对象的`env()`方法来访问环境变量

## Causing the container to exit

`container`的`exit()`方法会干净地关闭掉Vert.x实例
