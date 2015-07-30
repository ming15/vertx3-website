# Getting started

想要使用`metrics`,首先我们在pom文件里依赖下面的仓库：
```xml
<dependency>
  <groupId>io.vertx</groupId>
  <artifactId>vertx-metrics</artifactId>
  <version>${vertx.metrics.version}</version>
</dependency>
```

然后我们使用`DropwizardMetricsOptions`对`Vertx`实例开启`metrics`.
```java
Vertx vertx = Vertx.vertx(new VertxOptions().setMetricsOptions(
    new DropwizardMetricsOptions().setEnabled(true)
));
```
你也可以开启`JMX`:
```java
Vertx vertx = Vertx.vertx(new VertxOptions().setMetricsOptions(
    new DropwizardMetricsOptions().setJmxEnabled(true)
));
```
