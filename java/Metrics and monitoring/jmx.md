# JMX
JMX is disabled by default.

`JMX`是被默认不开启的.

如果你想要开启`JMX`,你需要像下面那样开启它.
```java
Vertx vertx = Vertx.vertx(new VertxOptions().setMetricsOptions(
    new DropwizardMetricsOptions().setJmxEnabled(true)
));
```

如果你是从命令行中运行`Vert.x`想要开启`JMX`, 你可以在`vertx`或者`vertx.bat`脚本中将`JMX_OPTS`那一行取消掉注释.
```
JMX_OPTS="-Dcom.sun.management.jmxremote -Dvertx.options.jmxEnabled=true"
```

你可以配置`MBeans`创建时是处于哪个域名下的:
```java
Vertx vertx = Vertx.vertx(new VertxOptions().setMetricsOptions(
    new DropwizardMetricsOptions().
        setJmxEnabled(true).
        setJmxDomain("mydomain")
));
```