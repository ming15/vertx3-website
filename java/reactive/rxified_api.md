# Rxified API

Embedding Rxfified Vert.x
Just use the Vertx.vertx methods:

Vertx vertx = io.vertx.rxjava.core.Vertx.vertx();
As a Verticle
Extend the AbstractVerticle class, it will wrap it for you:

class MyVerticle extends io.vertx.rxjava.core.AbstractVerticle {
  public void start() {
    // Use Rxified Vertx here
  }
}
Deploying an RxJava verticle is still performed by the Java deployer and does not need a specified deployer.

Api examples
Let’s study now a few examples of using Vert.x with RxJava.

EventBus message stream
The event bus MessageConsumer provides naturally an Observable<Message<T>>:

EventBus eb = vertx.eventBus();
MessageConsumer<String> consumer = eb.<String>consumer("the-address");
Observable<Message<String>> observable = consumer.toObservable();
Subscription sub = observable.subscribe(msg -> {
  // Got message
});

// Unregisters the stream after 10 seconds
vertx.setTimer(10000, id -> {
  sub.unsubscribe();
});
The MessageConsumer provides a stream of Message. The body gives access to a new stream of message bodies if needed:

EventBus eb = vertx.eventBus();
MessageConsumer<String> consumer = eb.<String>consumer("the-address");
Observable<String> observable = consumer.bodyStream().toObservable();
RxJava map/reduce composition style can be then be used:

Observable<Double> observable = vertx.eventBus().
    <Double>consumer("heat-sensor").
    bodyStream().
    toObservable();

observable.
    buffer(1, TimeUnit.SECONDS).
    map(samples -> samples.
        stream().
        collect(Collectors.averagingDouble(d -> d))).
    subscribe(heat -> {
      vertx.eventBus().send("news-feed", "Current heat is " + heat);
    });
Timers
Timer task can be created with timerStream:

vertx.timerStream(1000).
    toObservable().
    subscribe(
        id -> {
          System.out.println("Callback after 1 second");
        }
    );
Periodic task can be created with periodicStream:

vertx.periodicStream(1000).
    toObservable().
    subscribe(
        id -> {
          System.out.println("Callback every second");
        }
    );
The observable can be cancelled with an unsubscription:

vertx.periodicStream(1000).
    toObservable().
    subscribe(new Subscriber<Long>() {
      public void onNext(Long aLong) {
        // Callback
        unsubscribe();
      }
      public void onError(Throwable e) {}
      public void onCompleted() {}
    });
Http client requests
toObservable provides a one shot callback with the HttpClientResponse object. The observable reports a request failure.

HttpClient client = vertx.createHttpClient(new HttpClientOptions());
HttpClientRequest request = client.request(HttpMethod.GET, 8080, "localhost", "/the_uri");
request.toObservable().subscribe(
    response -> {
      // Process the response
    },
    error -> {
      // Could not connect
    }
);
request.end();
The response can be processed as an Observable<Buffer> with the toObservable method:

request.toObservable().
    subscribe(
        response -> {
          Observable<Buffer> observable = response.toObservable();
          observable.forEach(
              buffer -> {
                // Process buffer
              }
          );
        }
    );
The same flow can be achieved with the flatMap operation:

request.toObservable().
    flatMap(HttpClientResponse::toObservable).
    forEach(
        buffer -> {
          // Process buffer
        }
    );
We can also unmarshall the Observable<Buffer> into an object using the RxHelper.unmarshaller static method. This method creates an Rx.Observable.Operator unmarshalling buffers to an object:

request.toObservable().
    flatMap(HttpClientResponse::toObservable).
    lift(io.vertx.rxjava.core.RxHelper.unmarshaller(MyPojo.class)).
    forEach(
        pojo -> {
          // Process pojo
        }
    );
Http server requests
The requestStream provides a callback for each incoming request:

Observable<HttpServerRequest> requestObservable = server.requestStream().toObservable();
requestObservable.subscribe(request -> {
  // Process request
});
The HttpServerRequest can then be adapted to an Observable<Buffer>:

Observable<HttpServerRequest> requestObservable = server.requestStream().toObservable();
requestObservable.subscribe(request -> {
  Observable<Buffer> observable = request.toObservable();
});
The RxHelper.unmarshaller can be used to parse and map a json request to an object:

Observable<HttpServerRequest> requestObservable = server.requestStream().toObservable();
requestObservable.subscribe(request -> {
  Observable<MyPojo> observable = request.
      toObservable().
      lift(io.vertx.rxjava.core.RxHelper.unmarshaller(MyPojo.class));
});
Websocket client
The websocketStream provides a single callback when the websocket connects, otherwise a failure:

HttpClient client = vertx.createHttpClient(new HttpClientOptions());
WebSocketStream stream = client.websocketStream(8080, "localhost", "/the_uri");
stream.toObservable().subscribe(
    ws -> {
      // Use the websocket
    },
    error -> {
      // Could not connect
    }
);
The WebSocket can then be turned into an Observable<Buffer> easily

socketObservable.subscribe(
    socket -> {
      Observable<Buffer> dataObs = socket.toObservable();
      dataObs.subscribe(buffer -> {
        System.out.println("Got message " + buffer.toString("UTF-8"));
      });
    }
);
Websocket server
The {@link io.vertx.rxjava.core.http.HttpServer#websocketStream()` provides a callback for each incoming connection:

Observable<ServerWebSocket> socketObservable = server.websocketStream().toObservable();
socketObservable.subscribe(
    socket -> System.out.println("Web socket connect"),
    failure -> System.out.println("Should never be called"),
    () -> {
      System.out.println("Subscription ended or server closed");
    }
);
The ServerWebSocket can be turned into an Observable<Buffer> easily:

socketObservable.subscribe(
    socket -> {
      Observable<Buffer> dataObs = socket.toObservable();
      dataObs.subscribe(buffer -> {
        System.out.println("Got message " + buffer.toString("UTF-8"));
      });
    }
);