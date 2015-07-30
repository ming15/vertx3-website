# Retrieving metrics

一旦开启了`metrics`功能, `MetricsService`可以从任意`Measured`对象中检索出`metrics`快照, `Measured`对象提供了一个`meteric name`和`meteric data`映射关系的map,这个map通过`JsonObject`存储。

下面的例子打印出`Vertx`实例中所有的`metrics`.
```java
MetricsService metricsService = MetricsService.create(vertx);
JsonObject metrics = metricsService.getMetricsSnapshot(vertx);
System.out.println(metrics);
```

由于`HttpServer`实现了`Measured`,你可以轻松地抓取到该http服务器上所有的`metrics`
```java
MetricsService metricsService = MetricsService.create(vertx);
HttpServer server = vertx.createHttpServer();
// set up server
JsonObject metrics = metricsService.getMetricsSnapshot(server);
```