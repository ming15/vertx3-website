# Features
`Measured`接口提供一套简单的API来检索`metrics`,很多`Vert.x`组件都实现了这个接口,例如` HttpServer, NetServer`,甚至`Vertx`实例自身.

我们基于`Dropwizard`接口配置`JMX`报告,这个接口将`Vert.x`像`JMX MBeans`那样暴露出来.