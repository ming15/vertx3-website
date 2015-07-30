# Enabling remote JMX

如果你想`metrics`远程被`JMX`暴露，最少你需要设置下面这个系统属性:
```
com.sun.management.jmxremote
```
如果你是通过命令行运行Vert.x，那么你需要在`vertx or vertx.bat`文件中将`JMX_OPTS`的注释去掉。

如果你在公共服务器上运行Vert.x，那么你需要小心对`JMX`的远程访问了。

Please see the Oracle JMX documentation for more information on configuring JMX