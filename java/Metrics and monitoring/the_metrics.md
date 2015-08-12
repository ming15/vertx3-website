# The metrics
下面列出了当前支持的`metric`

## Vert.x metrics

* `vertx.event-loop-size` - `event loop pool`线程的数量。(`Gauge`表示)

* `vertx.worker-pool-size` - `worker pool`线程的数量。(`Gauge`表示)

* `vertx.cluster-host` - `cluster-host`设置。(`Gauge`表示)

* `vertx.cluster-port` - `cluster-port`设置。(`Gauge`表示)

* `vertx.verticles` - 当前被部署的`verticle`的数量。(`Counter`表示)

## Event bus metrics
Base name: `vertx.eventbus`

* `handlers` - `event bus`里`handler`的数量.(`Counter`表示) 

* `handlers.myaddress` - Timer,表示`myaddress`的`handler`接收到消息的比例

* `messages.bytes-read` - Meter, 表示接收到远端消息时读到的字节数的`Meter` 

* `messages.bytes-written` - 向远端发送消息时,写入字节的`Meter`

* `messages.pending` - 接收到消息但并没有传递给`handller`的统计消息数的`Counter`

* `messages.pending-local` - 接收到`locally`消息但并没有传递给`handller`的统计消息数的`Counter`

* `messages.pending-remote` - Counter, 表示接收到远端但是并没有传递给handler的消息数量. 

* `messages.received` - ThroughputMeter,表示接收到消息的比例 

* `messages.received-local` - ThroughputMeter,表示接收到本地消息的比例 

* `messages.received-remote` - ThroughputMeter, 表示接收到远程消息接收到的比例 

* `messages.delivered` - [throughpu_metert], 表示消息传递给handler的比例. 

* `messages.delivered-local` - ThroughputMeter,表示local消息传递给handler的比例.

* `messages.delivered-remote` - ThroughputMeter,表示remote消息传递给handler的比例.

* `messages.sent` - [throughput_metert], 表示发送出去的消息的比例. 

* `messages.sent-local` -ThroughputMeter, 表示本地`send`出去的消息的比例. 

* `messages.sent-remote` - ThroughputMeter, 表示远端`send`出去的消息的比例. 

* `messages.published` - ThroughputMeter, 表示`publish`出去的消息的比例. 

* `messages.published-local` - ThroughputMeter, 表示向本地`publish`出去的消息的比例.

* `messages.published-remote` - ThroughputMeter, 表示给远端`publish`出去的消息的比例. 

* `messages.reply-failures` - Meter, 表示`reply`失败的比例

`monitored event bus handlers` 通过向`handler`注册地址上的`match`来完成配置. 由于Vert.x可以在`event bus`进行海量的注册,因此比较好的配置是在默认的情况下我们不对任何`handler`进行监听.

`monitored handlers`可以通过一个指定的`address match`或者`regex match`在`DropwizardMetricsOptions`中完成配置.
```java
Vertx vertx = Vertx.vertx(new VertxOptions().setMetricsOptions(
    new DropwizardMetricsOptions().
        setEnabled(true).
        addMonitoredEventBusHandler(
            new Match().setValue("some-address")).
        addMonitoredEventBusHandler(
            new Match().setValue("business-.*").setType(MatchType.REGEX))
));
```
> 警告：如果你使用`regex match`, 当出现错误的`regex`,那么可能会`match`出大量的`handler`

## Http server metrics
Base name: `vertx.http.servers.<host>:<port>`

`Http server`除了包含`Net Server`的`metrics`之外还包含下面这些：

* `requests` - 请求的`Throughput Timer`和该请求出现的比例

* `<http-method>-requests` - 指定的`http method`请求的`Throughput Timer`和该`http method`请求的出现的比例

    > Examples: get-requests, post-requests

* `<http-method>-requests./<uri>` - 指定的`http method & URI`请求的`Throughput Timer`和该请求的出现的比例

    > Examples: get-requests./some/uri, post-requests./some/uri?foo=bar

* `responses-1xx` - 回应状态码为`1xx`的`ThroughputMeter` 

* `responses-2xx` - 回应状态码为`2xx`的`ThroughputMeter`

* `responses-3xx` - 回应状态码为`3xx`的`ThroughputMeter`

* `responses-4xx` - 回应状态码为`4xx`的`ThroughputMeter`

* `responses-5xx` - 回应状态码为`5xx`的`ThroughputMeter`

* `open-websockets` - 统计开启的`web socket`连接数的`Counter`

* `open-websockets.<remote-host>` - 统计对某个指定的`remote host`开启的`web socket`连接数的`Counter`

不管是`exact match`还是`regex match`,`Http URI metrics`必须在`DropwizardMetricsOptions`中显式地配置.
```java
Vertx vertx = Vertx.vertx(new VertxOptions().setMetricsOptions(
    new DropwizardMetricsOptions().
        setEnabled(true).
        addMonitoredHttpServerUri(
            new Match().setValue("/")).
        addMonitoredHttpServerUri(
            new Match().setValue("/foo/.*").setType(MatchType.REGEX))
));
```
For bytes-read and bytes-written the bytes represent the body of the request/response, so headers, etc are ignored.

## Http client metrics
Base name: `vertx.http.clients.@<id>`

`Http client`除了包含`Http Server`全部的`metrics`之外,还包含下面这些.

* `connections.max-pool-size` - 表示最大连接池大小的`Gauge`

* `connections.pool-ratio` - 表示`open connections / max connection pool size`的比例`Gauge`
* `responses-1xx` - 回应状态码为`1xx`的`Meter` 
* `responses-2xx` - 回应状态码为`2xx`的`Meter`
* `responses-3xx` - 回应状态码为`3xx`的`Meter`
* `responses-4xx` - 回应状态码为`4xx`的`Meter`
* `responses-5xx` - 回应状态码为`5xx`的`Meter`

## Net server metrics
Base name: `vertx.net.servers.<host>:<port>`

* `open-netsockets` - 开启的`socket`连接数的`Counter`
* `open-netsockets.<remote-host>` - 统计对于某个指定`remote host`开启的`socket`连接数的`Counter`
* `connections` - 某个连接的`Timer`和该连接出现的比例
* `exceptions` - 出现异常次数的`Counter`
* `bytes-read` - 已读字节数的`Histogram`.
* `bytes-written` - 写出字节数的`Histogram`.

## Net client metrics
Base name: `vertx.net.clients.@<id>`

`Net client`包含全部的`Net Server`的`metrics`

## Datagram socket metrics
Base name: `vertx.datagram`

* `sockets` - 统计`datagram sockets`数的`Counter`
* `exceptions` - 统计异常出现次数的`Counter`
* `bytes-written` - 写出字节数的`Histogram`.
* `<host>:<port>.bytes-read` - 已读字节数的`Histogram`.

只有当`datagram socket`被监听的时候上面的统计才有效。