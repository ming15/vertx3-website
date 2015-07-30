# The metrics
下面列出了当前支持的`metric`

## Vert.x metrics

* `vertx.event-loop-size` - `event loop pool`线程的数量。(`Gauge`表示)

* `vertx.worker-pool-size` - `worker pool`线程的数量。(`Gauge`表示)

* `vertx.cluster-host` - `cluster-host`设置。(`Gauge`表示)

* `vertx.cluster-port` - `cluster-port`设置。(`Gauge`表示)

* `vertx.verticles` - 当前被部署的`verticle`的数量。(`Counter`表示)

## Event bus metrics
Base name: vertx.eventbus

* `handlers` - `event bus`里`handler`的数量.(`Counter`表示) 

* `handlers.myaddress` - A Timer representing the rate of which messages are being received for the myaddress handler

* `messages.bytes-read` - A Meter of the number of bytes read when receiving remote messages

* `messages.bytes-written` - A Meter of the number of bytes written when sending remote messages

* `messages.pending` - A Counter of the number of messages received but not yet processed by an handler

* `messages.pending-local` - A Counter of the number of messages locally received but not yet processed by an handler

* `messages.pending-remote` - A Counter of the number of messages remotely received but not yet processed by an handler

* `messages.received` - A ThroughputMeter representing the rate of which messages are being received

* `messages.received-local` - A ThroughputMeter representing the rate of which local messages are being received

* `messages.received-remote` - A ThroughputMeter representing the rate of which remote messages are being received

* `messages.delivered` - A [throughpu_metert] representing the rate of which messages are being delivered to an handler

* `messages.delivered-local` - A ThroughputMeter representing the rate of which local messages are being delivered to an handler

* `messages.delivered-remote` - A ThroughputMeter representing the rate of which remote messages are being delivered to an handler

* `messages.sent` - A [throughput_metert] representing the rate of which messages are being sent

* `messages.sent-local` - A ThroughputMeter representing the rate of which messages are being sent locally

* `messages.sent-remote` - A ThroughputMeter representing the rate of which messages are being sent remotely

* `messages.published` - A ThroughputMeter representing the rate of which messages are being published

* `messages.published-local` - A ThroughputMeter representing the rate of which messages are being published locally

* `messages.published-remote` - A ThroughputMeter representing the rate of which messages are being published remotely

* `messages.reply-failures` - A Meter representing the rate of reply failures

The monitored event bus handlers is configurable via a match performed on the handler registration address. Vert.x can have potentially a huge amount of registered event bus, therefore the only good default for this setting is to monitor zero handlers.

The monitored handlers can be configured in the DropwizardMetricsOptions via a specific address match or a regex match:
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
> WARNING
if you use regex match, a wrong regex can potentially match a lot of handlers.

## Http server metrics
Base name: vertx.http.servers.<host>:<port>

Http server includes all the metrics of a Net Server plus the following:

* `requests` - A Throughput Timer of a request and the rate of it’s occurrence

* `<http-method>-requests` - A Throughput Timer of a specific http method request and the rate of it’s occurrence

    > Examples: get-requests, post-requests

* `<http-method>-requests./<uri>` - A Throughput Timer of a specific http method & URI request and the rate of it’s occurrence

    > Examples: get-requests./some/uri, post-requests./some/uri?foo=bar

* `responses-1xx` - A ThroughputMeter of the 1xx response code

* `responses-2xx` - A ThroughputMeter of the 2xx response code

* `responses-3xx` - A ThroughputMeter of the 3xx response code

* `responses-4xx` - A ThroughputMeter of the 4xx response code

* `responses-5xx` - A ThroughputMeter of the 5xx response code

* `open-websockets` - A Counter of the number of open web socket connections

* `open-websockets.<remote-host>` - A Counter of the number of open web socket connections for a particular remote host

Http URI metrics must be explicitely configured in the options either by exact match or regex match:
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
Base name: vertx.http.clients.@<id>

Http client includes all the metrics of a Http Server plus the following:

* `connections.max-pool-size` - A Gauge of the max connection pool size

* `connections.pool-ratio` - A ratio Gauge of the open connections / max connection pool size
* `responses-1xx` - A Meter of the 1xx response code
* `responses-2xx` - A Meter of the 2xx response code
* `responses-3xx` - A Meter of the 3xx response code
* `responses-4xx` - A Meter of the 4xx response code
* `responses-5xx` - A Meter of the 5xx response code

## Net server metrics
Base name: vertx.net.servers.<host>:<port>

* `open-netsockets` - A Counter of the number of open net socket connections
* `open-netsockets.<remote-host>` - A Counter of the number of open net socket connections for a particular remote host
* `connections` - A Timer of a connection and the rate of it’s occurrence
* `exceptions` - A Counter of the number of exceptions
* `bytes-read` - A Histogram of the number of bytes read.
* `bytes-written` - A Histogram of the number of bytes written.

## Net client metrics
Base name: vertx.net.clients.@<id>

Net client includes all the metrics of a Net Server

## Datagram socket metrics
Base name: vertx.datagram

* `sockets` - A Counter of the number of datagram sockets
* `exceptions` - A Counter of the number of exceptions
* `bytes-written` - A Histogram of the number of bytes written.
* `<host>:<port>.bytes-read` - A Histogram of the number of bytes read.

This metric will only be available if the datagram socket is listening