# In the beginning there was Vert.x

> NOTE
Much of this is Java specific - need someway of swapping in language specific parts

在Vert.x里，如果你不使用`Vertx`对象，你几乎是寸步难行。

`Vertx`对象扮演着Vert.x控制中心的角色，同时它也提供了大量的功能，例如：
* 创建客户端和服务器
* 获得`event bus`引用
* 设置定时器
* ...

如果你将Vert.x嵌入到你的应用程序中，你可以向下面这样获得一个`Vertx`对象的引用
```
Vertx vertx = Vertx.vertx();
```
If you’re using Verticles

> 注意： 在绝大多数应用程序中，你只需要一个Vert.x实例，但是如果你想要创建多个Vert.x实例，这也是可以的，例如你想要将`event bus`或者服务器与客户端进行隔离

## Specifying options when creating a Vertx object
当你实例化`Vertx`对象时，如果你感觉默认的参数不符合你的需求，你可以指定实例化时的参数：
```
Vertx vertx = Vertx.vertx(new VertxOptions().setWorkerPoolSize(40));
```

`VertxOptions`对象拥有N多设置，例如配置集群，高可用设置，线程池大小以及等等其他参数

