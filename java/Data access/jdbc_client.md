# JDBC client

这个客户端允许你在Vertx应用中使用异步API与JDBC数据库相交互.

`JDBCClient`接口定义了异步方式的JDBC API

## Creating a the client

下面的几种方式介绍了如何创建一个客户端：

### Using default shared data source

在大多数情景中,你可能需要在不同的客户端实例(client instances)中共享同一个`data source`.

例如,你想要通过部署多个`verticle`实例来拓展应用的规模,然而每个`verticle`都共享相同的`datasource`,这样你就避免了多个`datasource pool`了。

如下：
```java
JDBCClient client = JDBCClient.createShared(vertx, config);
```
当第一次调用`JDBCClient.createShared`的时候,确实会根据传进的配置创建出`data source`. 但是当接下来再次调用这个方法的时候,也会创建一个新的客户端,但是它却没有创建新的`data source`,因此第二次传进的配置也没有生效.

#### Specifying a data source name

在创建`JDBCClient`实例的时候也可以指定`data source`的名字
```java
JDBCClient client = JDBCClient.createShared(vertx, config, "MyDataSource");
```
如果使用相同的`Vertx`实例和相同的`data source`名字创建出不同的客户端,那这些客户端会使用相同的`data source`.

使用这种创建方式你可以在不同的客户端上使用不同的`datasource`, 例如和不同的数据库进行交互.

#### Creating a client with a non shared data source

虽然在大部分情况下不同的客户端实例需要共享相同的`data source`,但是有时候也可能客户端需要一个非共享的`data source`, 你可以使用`JDBCClient.createNonShared`方法.

```java
JDBCClient client = JDBCClient.createNonShared(vertx, config);
```
这种方式和调用`JDBCClient.createShared`时传递一个唯一的`data source`名字达到相同的效果.

#### Specifying a data source

如果你想使用一个已经存在的`data source`, 那么你也可以直接使用那个`data source`创建一个客户端.
```java
JDBCClient client = JDBCClient.create(vertx, dataSource);
```

## Closing the client
你可以在整个`verticle`生命周期之内都持有客户端的引用,但是一旦你使用完该客户端你就应该主动关闭它.

由于`data source`内部持有一个引用计数器,每当客户端关闭一次,`data source`内部的技术器就会减1,当计数器为0的时候,`data source`就会关闭自己.

## Getting a connection

当你成功创建出客户端之后,你可以使用`getConnection`方法来获得一个连接.

当连接池中有了可用连接之后,`handler`会获得一个可用连接
```java
client.getConnection(res -> {
  if (res.succeeded()) {

    SQLConnection connection = res.result();

    connection.query("SELECT * FROM some_table", res2 -> {
      if (res2.succeeded()) {

        ResultSet rs = res2.result();
        // Do something with results
      }
    });
  } else {
    // Failed to get connection - deal with it
  }
});
```
获得的连接是一个`SQLConnection`实例, `SQLConnection`接口更多的是被`Vert.x sql service`使用.

## Configuration
当我们创建客户端时,向其传递了一个配置,该配置包含下面属性：

* `provider_class` : 用于管理数据库连接的类名. 默认的类名是`io.vertx.ext.jdbc.spi.impl.C3P0DataSourceProvider`, 但是如果你想要使用其他连接池,那么你可以使用你自己的实现覆盖该属性.

###### 因为我们使用了`C3P0`的连接池,因此我们还可以使用下列属性
* `url` : 数据库的JDBC连接URL地址
* `driver_class` : JDBC`dirver`名称
* `user` : 数据库名称
* `password` : 数据库密码
* `max_pool_size` : 连接池最大数量(默认是15)
* `initial_pool_size` : 连接池初始大小(默认是3)
* `min_pool_size` : 连接池最小值
* `max_statements` : 缓存`prepared statements`的最大值(默认是0)
* `max_statements_per_connection` : 每个连接缓存`prepared statements`的最大值(默认是0)
* `max_idle_time` : 该值表示一个闲置连接多少秒后会被关闭(默认是0, 从不关闭).

如果你还想要配置其他`C3P0`的配置,那么你可以在`classpath`上添加一个`c3p0.properties`文件


下面给出了一个配置示例：
```java
JsonObject config = new JsonObject()
  .put("url", "jdbc:hsqldb:mem:test?shutdown=true")
  .put("driver_class", "org.hsqldb.jdbcDriver")
  .put("max_pool_size", 30);

JDBCClient client = JDBCClient.createShared(vertx, config);
```