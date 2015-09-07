# Verticles

`Vert.x`引入了一个简单的可扩展的类`actor`的部署和并发模型.

###### 这个模型是可选的, 如果你不想要采取该模型也可以不实现它,vertx并不强制要求你实现它.

这个模型并不是`actor-model`的严格实现, 但是该模型在并发处理,拓展模式和开发模式上确实和`actor-model`非常像。

其实当我们在`verticle`中开始实现逻辑代码时，就已经开始使用这个开发模型了。

> `verticle`简而言之就是一个代码块,然后你通过Vert.x部署和运行它. 我们可以使用Vert.x支持的不同语言实现`verticle`,而且一个单独的应用程序中可以包含多种语言实现的`verticle`.
> 
> 你可以把`verticle`理解成`Actor`模型中的`actor`.

一般来说,一个Vert.x应用应该只是由`verticle`实例构成.不同的 `verticle`可以在`event bus`上通过发送消息进行交互.

## Writing Verticles
Java`verticle`必须实现`Verticle`接口。

当然如果你不想实现这个接口，还有一种其他定义方法，那就是继承`AbstractVerticle`抽象类
```java
public class MyVerticle extends AbstractVerticle {

  // Called when verticle is deployed
  public void start() {
  }

  // Optional - called when verticle is undeployed
  public void stop() {
  }

}
```

`verticle`在被Vert.x部署的过程中,`verticle`的`start`方法会被调用,当`start`方法被调用完之后,`verticle`就被认为部署完成.

`verticle`实现中像例子中`start`方法是必须要实现的,但是`stop`方法可以选择不实现. `stop`方法是当`verticle`被`undeployed`进行调用的,当`stop`方法调用完成之后,`verticle`就被认为停止运行了

## Asynchronous Verticle start and stop

有时你想要在`verticle`开始部署阶段(`start`方法中)完成一些耗时的操作,但是当这些操作完成之前,你不希望当前`verticle`处于部署完成状态.例如你想要在`start`方法中部署其他`verticle`，但是由于部署是异步进行的,因此可能主`verticle`都已经返回了,但是其他`verticle`的部署工作还没有完成,那么你就不想让主`verticle`处于完成状态.

但是你也不能在`start`方法中进行阻塞等待其他`verticle`部署完成,在`event loop`无论何时你都不应该把它阻塞掉。

那么该怎么办呢？我们的办法是你实现一个`asynchronous`的`start`方法,这个方法会传入一个`Future`对象作为参数.即使当`start`方法返回了,该`verticle`也不会被认为已经完成部署了。

当你在start方法里所有的工作都完成之后,通过调用`Future`对象的`complete`方法,在外部获得一个通知,部署工作真正完成了.

```java
public class MyVerticle extends AbstractVerticle {

  public void start(Future<Void> startFuture) {
    // Now deploy some other verticle:

    vertx.deployVerticle("com.foo.OtherVerticle", res -> {
      if (res.succeeded()) {
        startFuture.complete();
      } else {
        startFuture.fail();
      }
    });
  }
}
```
同样的,`stop`方法也有一个`asynchronous`版本的.
```java
public class MyVerticle extends AbstractVerticle {

  public void start() {
    // Do something
  }

  public void stop(Future<Void> startFuture) {
    obj.doSomethingThatTakesTime(res -> {
      if (res.succeeded()) {
        startFuture.complete();
      } else {
        startFuture.fail();
      }
    });
  }
}
```
> 注意,通过`verticle`部署的`vertcle`，这俩种`vertcle`会构成一种"父子"关系，当父`verticle`被`undeploy`后,子`verticle`会自动被Vert.x进行`undeploy`

## Verticle Types
在Vert.x中有三种不同类型的`verticle`

* `Standard Verticles` : 这是最常用的一种. 这种`verticle`通过`event loop`线程执行.接下来我们会详细讨论这种`verticle`.
* `Worker Verticles` : 这种`verticle`通过`worker pool`中的线程执行。该`verticle`在同一时刻永远不会被多个线程并发执行
* `Multi-threaded worker verticles` : 该`verticle`同样通过`worker pool`中的线程执行.但是这种`verticle`可能会被多个线程并发执行.


### Standard verticles

`Standard Verticles`当被创建的时候会被分配到一个`event loop`上, 同时`Standard Verticles`的`start()`会被该`event loop`进行调用. 当你在`event loop`中,通过核心API以及带有`handler`参数的的方式调用其他方法时,`Vert.x`确保那些`handler`回调时是被刚才那个`event loop`进行调用的.

这意味着,我们能保证`vertcle`实例里全部代码总是能被相同的`event loop`进行调用(当然这是在你不故意自己创建线程调用他们的前提下).

这意味着,当你基于Vert.x开发应用程序时,vert.x会帮你完成那些并发操作,你自己完全不需要考虑多线程和并发情况,只需要像在单线程中那样写代码就好了. 从此你的生活就远离了`synchronized`, `volatile`, `条件竞争`, `死锁`等等..

### Worker verticles

`worker verticle`和`standard verticle`相比,`worker verticle`并不是运行在`event loop`中,而是在`worker thread pool`中的某个线程中运行.

`worker verticle`是被设计成专门用来调用阻塞代码的,他们不会阻塞掉任何的`event loop`.

如果你不想在`worker verticle`中运行阻塞代码, 你也可以在`event loop`中执行运行内联的阻塞代码.

如果你想要部署`worker verticle`时, 你可以使用`setWorker()`.
```java
DeploymentOptions options = new DeploymentOptions().setWorker(true);
vertx.deployVerticle("com.mycompany.MyOrderProcessorVerticle", options);
```
`Worker verticle`实例永远不会被多线程同一时间并发执行,但是却可以在不同的时间被不同的线程执行.

###Multi-threaded worker verticles

`multi-threaded worker verticle`和`worker verticle`很像,只不过这种`multi-threaded worker verticle`可以被多个不同线程并发执行.

> 注意:`multi-threaded worker verticle`是一个非常高级的特性,而且大部分的应用程序并不会需要使用到它. 因为在这种`verticle`中的并发操作,你需要非常小心通过使用传统的多线程编程技术保持`verticle`的状态一致性.

## Deploying verticles programmatically
你可以通过`deployVerticle()`方法部署一个`verticle`, 使用这种方式你需要指定该`verticle`的名字或者传递一个你已经创建好的该`verticle`的实例.

> 注意: 只有在java中才可以部署Verticle实例

```java
Verticle myVerticle = new MyVerticle();
vertx.deployVerticle(myVerticle);
```

`verticle`名字用来查找特定的`VerticleFactory`, 我们使用`VerticleFactory`来实例化出实际的`verticle`实例.

不同的`VerticleFactory`用于在不同的语言实现中对`verticle`进行实例化, 除此之外不同的`VerticleFactory`也用于加载`service`或者在`Maven`运行时获得`verticle`.

这种特性可以让你在某种语言中部署其他语言实现的`verticle`.

下面的例子演示了在Java中部署不同语言类型的`verticle`
```java
vertx.deployVerticle("com.mycompany.MyOrderProcessorVerticle");

// Deploy a JavaScript verticle
vertx.deployVerticle("verticles/myverticle.js");

// Deploy a Ruby verticle verticle
vertx.deployVerticle("verticles/my_verticle.rb");
```

## Rules for mapping a verticle name to a verticle factory

当使用`verticle`名称部署`verticle`时,这个名字被用来找到实际的`VerticleFactory`,对`verticle`进行实例化

`verticle`名称还可以有一个前缀(随后跟一个冒号),当该前缀被指定后,会使用该前缀来查找`VerticleFactory`.
```java
js:foo.js // Use the JavaScript verticle factory
groovy:com.mycompany.SomeGroovyCompiledVerticle // Use the Groovy verticle factory
service:com.mycompany:myorderservice // Uses the service verticle factory
```

如果我们并没有指定前缀,那么Vert.x会去名称中找到后缀(文件类型),然后使用该后缀找到`VerticleFactory`.
```
foo.js // Will also use the JavaScript verticle factory
SomeScript.groovy // Will use the Groovy verticle factory
```
但是如果既没有前缀，也没有后缀被找到,那么Vert.x会认为这个名称是个Java类的全限定名,使用使用这个全限定名进行实例化

## How are Verticle Factories located?

在Vert.x启动时,它会在`classpath`中对大多数的`VerticleFactory`加载和注册.

当然如果你想在程序中通过编程的方式对`VerticleFactory`进行注册和解除注册到话,你可以使用`registerVerticleFactory`和`unregisterVerticleFactory`方法

## Waiting for deployment to complete

同样`verticle`的部署也是异步进行的, 当调用部署方法进行返回的时候也许部署操作并没有真正的,可能要等到过一段时间才能真正完成,

如果你想当部署操作真正完成的时候捕获一个通知,你可以在部署方法里添加一个`completion handler`,用于处理完成时候你想进行的特定操作.
```java
vertx.deployVerticle("com.mycompany.MyOrderProcessorVerticle", res -> {
  if (res.succeeded()) {
    System.out.println("Deployment id is: " + res.result());
  } else {
    System.out.println("Deployment failed!");
  }
});
```

如果部署成功了,handler会捕获一个result(内含一个字符串形式的部署ID).

当你后期想要对`verticle`进行`undeploy`操作时,你就需要使用刚才的那个部署成功时获得的字符串形式的部署ID了.

## Undeploying verticle deployments

当`verticle`被部署成功之后,我们也可以通过调用`undeploy`方法对其进行`undeploy`操作.

当然`undeploy`一样是异步进行的,如果你想当`undeploy`操作完成时同样捕获通知,你也可以对`undeploy`方法设置一个`completion handler`.
```java
vertx.undeploy(deploymentID, res -> {
  if (res.succeeded()) {
    System.out.println("Undeployed ok");
  } else {
    System.out.println("Undeploy failed!");
  }
});
```

## Specifying number of verticle instances

当你使用`verticle`名字对某个`verticle`进行部署时,你也可以指定该verticle部署成功后的实例数量:
```java
DeploymentOptions options = new DeploymentOptions().setInstances(16);
vertx.deployVerticle("com.mycompany.MyOrderProcessorVerticle", options);
```

当我们想在多核主机上对应用进行拓展时,通过这种方式就可以轻松实现了. 假设,你现在要在一个多核主机上部署一个`web-server verticle`,因此你想要对该`verticle`部署多个实例以便能使用上所有核心.

## Passing configuration to a verticle

当`verticle`被部署时,我们还可以向其指定一个JSON形式的配置:
```java
JsonObject config = new JsonObject().put("name", "tim").put("directory", "/blah");
DeploymentOptions options = new DeploymentOptions().setConfig(config);
vertx.deployVerticle("com.mycompany.MyOrderProcessorVerticle", options);
```
稍后我们就可以通过`Context`对象来操作`Configuration`配置了

TODO

## Accessing environment variables in a Verticle
TODO

## Verticle Isolation Groups
By default, Vert.x has a flat classpath. I.e, it does everything, including deploying verticles without messing with class-loaders. In the majority of cases this is the simplest, clearest and sanest thing to do.

Vert.x默认有一个`flat classpath`,它会实现N多功能,包括在部署`verticle`时不会干扰类加载的工作. 在大多数情况下,这是最简单，清晰，明智的事情。

However, in some cases you may want to deploy a verticle so the classes of that verticle are isolated from others in your application.

This might be the case, for example, if you want to deploy two different versions of a verticle with the same class name in the same Vert.x instance, or if you have two different verticles which use different versions of the same jar library.

> WARNING
Use this feature with caution. Class-loaders can be a can of worms, and can make debugging difficult, amongst other things.
Here’s an example of using an isolation group to isolate a verticle deployment.
```java
DeploymentOptions options = new DeploymentOptions().setIsolationGroup("mygroup");
options.setExtraClasspath(Arrays.asList("lib/jars/some-library.jar"));
vertx.deployVerticle("com.mycompany.MyIsolatedVerticle", options);
```
Isolation groups are identified by a name, and the name can be used between different deployments if you want them to share an isolated class-loader.

Extra classpath entries can also be provided with setExtraClasspath so they can locate resources that are isolated to them.

## High Availability
Verticles can be deployed with High Availability (HA) enabled.



TODO

## Running Verticles from the command line

通常做法是你可以在`Maven`或者`Gradle`项目中添加一个`Vert.x core library`引用,你就可以直接运行Vert.x了.

然而,你如果不习惯那种做法,你还可以直接在命令行中执行运行`Vert.x verticle`.

在命令行中运行Vert.x你需要下载`Vert.x`的分发版本,然后将安装好的bin目录添加到`Path`环境变量中,同时要确保在`PATH`中你也添加上了JDK8.

下例演示了如何直接在命令中运行`verticle`
```java
# Run a JavaScript verticle
vertx run my_verticle.js

# Run a Ruby verticle
vertx run a_n_other_verticle.rb

# Run a Groovy script verticle, clustered

vertx run FooVerticle.groovy -cluster
```

令人惊喜的是,你可以在命令行中直接运行java源文件.
```java
vertx run SomeJavaSourceFile.java
```
`Vert.x`会在运行该java源文件之前自己去编译它. 这对于`quickly prototyping verticle`和写`verticle demo`是非常有用的.

## Causing Vert.x to exit
Threads maintained by Vert.x instances are not daemon threads so they will prevent the JVM from exiting.



如果你将Vert.x嵌入在了你的应用程序中,而且当你的应用程序已经使用完了Vert.x的功能,你需要关闭掉Vert.x的时候,你可以直接调用`close`将其关闭掉.

这个操作会关闭Vert.x内部所有的线程池和其他的资源,但是并不会让JVM也跟着关闭掉.

## The Context object
TODO

## Executing periodic and delayed actions
It’s very common in Vert.x to want to perform an action after a delay, or periodically.

在`standard verticle`中,你不能因为想要得到一个延迟的效果就将线程`sleep`掉,因为这个操作会将`event loop`线程阻塞掉.

取而代之的是,你可以使用`Vert.x timers`,`Vert.x timers`既可以执行一次也可以周期性执行.

### One-shot Timers

`one shot timer`会在一个特定的延迟后(单位毫秒)调用一个`event handler`.

设置`one shot timer`是非常简单的,调用`setTimer`方法然后设置一个延迟和一个`handler`就ok了.
```java
long timerID = vertx.setTimer(1000, id -> {
  System.out.println("And one second later this is printed");
});

System.out.println("First this is printed");
```
这个返回的返回值是一个唯一的`timer id`,如果你想在后期取消掉这个`timer`,就需要使用这个id了.

### Periodic Timers

你可以通过调用`setPeriodic`方法设置一个周期性的定时器.

There will be an initial delay equal to the period.



The return value of setPeriodic is a unique timer id (long). This can be later used if the timer needs to be cancelled.

The argument passed into the timer event handler is also the unique timer id:
```java
long timerID = vertx.setPeriodic(1000, id -> {
  System.out.println("And every second this is printed");
});

System.out.println("First this is printed");
```

### Cancelling timers

To cancel a periodic timer, call cancelTimer specifying the timer id. For example:
```java
vertx.cancelTimer(timerID);
```

### Automatic clean-up in verticles

If you’re creating timers from inside verticles, those timers will be automatically closed when the verticle is undeployed.
























