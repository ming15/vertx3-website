# Accessing Dropwizard Registry
When configuring the metrics service, an optional registry name can be specified for registering the underlying Dropwizard Registry in the the Dropwizard Shared Registry so you can retrieve this registry and use according to your needs.
```java
VertxOptions options = new VertxOptions().setMetricsOptions(
  new MetricsServiceOptions().setEnabled(true).setRegistryName("the_name")
);
Vertx vertx = Vertx.vertxt(options);

// Get the registry
MetricRegistry registry = SharedMetricRegistries.getOrCreate("the_name");

// Do whatever you need with the registry
```