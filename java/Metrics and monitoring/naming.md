# Naming

下文中列出的每一个测量组件都会被分配一个名字. 每个`metric`都可以通过全限定名`baseName + . + metricName`从`Vertx`实例中获得.
```java
JsonObject metrics = metricsService.getMetricsSnapshot(vertx);
metrics.getJsonObject("vertx.eventbus.handlers");
```

或者使用`metric`名字获得该自身.
```java
EventBus eventBus = vertx.eventBus();
JsonObject metrics = metricsService.getMetricsSnapshot(eventBus);
metrics.getJsonObject("handlers");
```