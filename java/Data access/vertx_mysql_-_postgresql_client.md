# Vert.x MySQL - PostgreSQL client

`MySQL / PostgreSQL`客户端为Vert.x应用提供了一个与`MySQL / PostgreSQL`数据库交互的接口.

它使用`Mauricio Linhares`开源驱动与`MySQL / PostgreSQL`数据库进行非阻塞交互.

## Creating a the client
下面给出了几种创建方式：

#### Using default shared pool
在大多数情况下,我们需要在多个客户端实例中共享同一个连接池

例如你通过部署多个`verticle`实例的方式进行程序拓展,但是可以每个`verticle`可以共享同一个连接池.

```java
JsonObject mySQLClientConfig = new JsonObject().put("host", "mymysqldb.mycompany");
AsyncSQLClient mySQLClient = MySQLClient.createShared(vertx, mySQLClientConfig);

// To create a PostgreSQL client:

JsonObject postgreSQLClientConfig = new JsonObject().put("host", "mypostgresqldb.mycompany");
AsyncSQLClient postgreSQLClient = PostgreSQLClient.createShared(vertx, postgreSQLClientConfig);
```
`MySQLClient.createShared`或者`PostgreSQLClient.createShared`会根据指定的配置创建一个连接池. 随后再调用这俩个方式时会使用同一个连接池,同时新的配置不会被采用.

#### Specifying a pool name
你也可以像下面这样指定一个连接池的名字.
```java
JsonObject mySQLClientConfig = new JsonObject().put("host", "mymysqldb.mycompany");
AsyncSQLClient mySQLClient = MySQLClient.createShared(vertx, mySQLClientConfig, "MySQLPool1");

// To create a PostgreSQL client:

JsonObject postgreSQLClientConfig = new JsonObject().put("host", "mypostgresqldb.mycompany");
AsyncSQLClient postgreSQLClient = PostgreSQLClient.createShared(vertx, postgreSQLClientConfig, "PostgreSQLPool1");
```
如果不同的客户端使用相同的`Vertx`实例和相同的连接池名字,那么他们将使用同一个连接池.

使用这种创建方式你可以在不同的客户端上使用不同的`datasource`, 例如和不同的数据库进行交互.

#### Creating a client with a non shared data source
虽然在大部分情况下不同的客户端实例需要共享相同的`data source`,但是有时候也可能客户端需要一个非共享的`data source`, 你可以使用`MySQLClient.createNonShared`或者`PostgreSQLClient.createNonShared`方法.

```java
JsonObject mySQLClientConfig = new JsonObject().put("host", "mymysqldb.mycompany");
AsyncSQLClient mySQLClient = MySQLClient.createNonShared(vertx, mySQLClientConfig);

// To create a PostgreSQL client:

JsonObject postgreSQLClientConfig = new JsonObject().put("host", "mypostgresqldb.mycompany");
AsyncSQLClient postgreSQLClient = PostgreSQLClient.createNonShared(vertx, postgreSQLClientConfig);
```
这种方式和调用`MySQLClient.createNonShared`或者`PostgreSQLClient.createNonShared`时传递一个唯一的`data source`名字达到相同的效果.

## Closing the client
你可以在整个`verticle`生命周期之内都持有客户端的引用,但是一旦你使用完该客户端你就应该调用`close`关闭它.

## Getting a connection

当你成功创建出客户端之后,你可以使用`getConnection`方法来获得一个连接.

当连接池中有了可用连接之后,`handler`会获得一个可用连接
```java
client.getConnection(res -> {
  if (res.succeeded()) {

    SQLConnection connection = res.result();

    // Got a connection

  } else {
    // Failed to get connection - deal with it
  }
});
```
连接是`SQLConnection`的一个实例, `SQLConnection`是一个被`Sql`客户端使用的公共接口.

> 需要注意的是`date`和`timestamps`类型. 无论何时从数据库中获取`date`时, 客户端会将它转换成`ISO 8601`形式的字符串(`yyyy-MM-ddTHH:mm:ss.SSS`).`Mysql`会忽略毫秒数.

## Configuration
`PostgreSql`和`MySql`使用了下面相同的配置：
```json
{
  "host" : <your-host>,
  "port" : <your-port>,
  "maxPoolSize" : <maximum-number-of-open-connections>,
  "username" : <your-username>,
  "password" : <your-password>,
  "database" : <name-of-your-database>
}
```
* `host` : 数据库主机地址(默认是`localhost`)
* `port` : 数据库端口(`PostgreSQL`默认是5432. `MySQL`默认是3306)
* `maxPoolSize` : 连接池保持开启的最大数量,默认是10.
* `username` : 连接数据库使用的用户名.(`PostgreSQL`的默认值是`postgres`, `MySQL`的默认值是`root`)
* `password` : 连接数据库使用的密码(默认没有设置密码).
* `database` : 连接的数据库名称.(默认值是`test`)
