# High availability with Vert.x

`Vert.x`同样支持对`module`的高可用(`high availability (HA)`)运行.

### Automatic failover

假设一个`module`与`HA`一起运行，如果`module`所在的`Vert.x`实例运行失败了，那么该`module`会自动在集群中其他的节点上重新启动，那我们称该`module`为`fail-over`

想要`module`与`HA`一起运行，那么只需要在命令行上加上`-ha`参数
```
vertx runmod com.acme~my-mod~2.1 -ha
```

但是想要`HA`正常工作，你就需要在集群中添加多个`Vert.x`实例，也就是说必须存在一个已经启动的`Vert.x`实例

> (该实例是否也需要加-ha参数呢？应该是不需要的，要不然就自我矛盾了)
```
vertx runmod com.acme~my-other-mod~1.1 -ha
```

如果现在运行着`com.acme~my-mod~2.1`的`module`的`Vert.x`实例挂掉了(你可以通过`kill -9`这个命令进行测试)，那么运行着`com.acme~my-mod~1.1`的`Vert.x`实例会自动开始部署`com.acme~my-mod~2.1``module`，因此，运行着`com.acme~my-mod~1.1`的`Vert.x`实例接下来会运行那俩个`module`。

> 注意：干净地关闭一个`Vert.x`实例并不会引起`failover`,例如使用`CTRL-C`或者`kill -SIGINT`

你也可以以`bare`模式运行`Vert.x`实例。这种模式下运行的`Vert.x`实例启动时并不会自动运行任何`module`，they will also failover for nodes in the cluster. 可以通过下面的命令开启一个bare 实例
```
vertx -ha
```

当你使用`ha`选项时，你就不需要指定`-cluster`选项了，因为当`Vert.x`实例与`HA`一起运行时，cluster就默认开启了。

### HA groups

当你与`HA`运行一个`Vert.x`实例时，你也可以选择指定一个`HA group`。 `HA group`是在逻辑上将集群中一组节点进行分组。只有同一个`HA group`组内的节点才能相互`failover`,也就是说`failover`是不同横跨`HA group`的. 如果你不显式指定`HA group`，那么就会将其分配进默认的`__DEFAULT__`组里.

当运行`module`时，通过`-hgroup`选项指定`HA group`
```
vertx runmod com.acme~my-mod~2.1 -ha -hagroup somegroup
```
下面展示了一些示例：

In console 1:
```
vertx runmod com.mycompany~my-mod1~1.0 -ha -hagroup g1
```
In console 2:
```
vertx runmod com.mycompany~my-mod2~1.0 -ha -hagroup g1
```
In console 3:
```
vertx runmod com.mycompany~my-mod3~1.0 -ha -hagroup g2
```

如果我们把console 1中的实例`kill`掉，那么该实例会向console 2中的实例进行`fail over`，但不会向console 3中的实例进行fail over，因为他们没有在一个共同的`HA group`。

如果我们将console 3中的实例`kill`掉之后，就不会发生`fail over`了，因为该组中只有一个`Vert.x`实例

### Dealing with network partitions - Quora

`HA`实现还支持`quora`

When starting a `Vert.x` instance you can instruct it that it requires a "quorum" before any HA deployments will be deployed. A quorum is a minimum number of nodes for a particular group in the cluster. Typically you chose your quorum size to Q = 1 + N/2 where N is the number of nodes in the group.

当你开启一个`Vert.x`实例时，你可以在它进行`HA`部署之前，引导它进行`quorum`。`quorum`指的是集群中某个`HA group`中的最小节点数。一般你可将`qurom`设定为`Q = 1 + N / 2`，其中N是`HA group`中节点的数量。

If there are less than Q nodes in the cluster the HA deployments will undeploy. They will redeploy again if/when a quorum is re-attained. By doing this you can prevent against network partitions, a.k.a. split brain.

如果集群中的节点数小于Q，那么`HA`部署会undeploy。 如果quorum重新获得后，那么HA会进行重新部署。By doing this you can prevent against network partitions, a.k.a. split brain.

更多信息参考[quora]()

如果想要使用`quorum`运行`Vert.x`实例，你只需要在命令行中指定`-quorum`选项就好：

In console 1:
```
vertx runmod com.mycompany~my-mod1~1.0 -ha -quorum 3
```

在console 1中`Vert.x`实例会启动成功但是，它并不会部署`module`，因为在集群中只有一个节点

In console 2:
```
vertx runmod com.mycompany~my-mod2~1.0 -ha -quorum 3
```
在console 1中`Vert.x`实例会启动成功但是，它并不会部署`module`，因为在集群中只有俩个节点

In console 3:
```
vertx runmod com.mycompany~my-mod3~1.0 -ha -quorum 3
```

现在，我们有三个节点了，`quorum`条件达到了。现在`module`就会自动地部署到他们所在的`Vert.x`实例上

如果我们close或者kill掉三个节点中的一个，其他节点上所有的`module`都会自动的解除部署，因为现在`quorum`不再满足条件。

`Quora`也可以和`HA group`一起结合使用。

### Logging

每一个`verticle`实例都可以从它内部检索出属于它自己的logger，至于如何检索就要参考具体语言实现的API了。

默认的log日志是存储在系统临时目录的`vertx.log`文件中，在Linux中是`\tmp`.

默认实现使用`JUL`纪录日志。但是我们可以通过`$VERTX_HOME\conf\logging.properties`这个属性文件进行修改。

如果你想要使用其他的日志框架，例如`log4j`，你可以在启动`Vert.x`的时候，通过系统参数的方式进行指定。
```
-Dorg.vertx.logger-delegate-factory-class-name=org.vertx.java.core.logging.impl.Log4jLogDelegateFactory
```
or
```
-Dorg.vertx.logger-delegate-factory-class-name=org.vertx.java.core.logging.impl.SLF4JLogDelegateFactory
```

如果你不想要使用`Vert.x`提供的日志功能也可以，你只需要像平常那样使用你喜欢的日志框架，同时引用相关日志jar包，同时你还要在`module`里进行配置。