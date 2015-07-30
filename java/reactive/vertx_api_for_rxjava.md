# Vert.x API for RxJava

RxJava is a popular library for composing asynchronous and event based programs using observable sequences for the Java VM.

Vert.x integrates naturally with RxJava, allowing to use observable wherever you can use streams or asynchronous results.

There are two ways for using the RxJava API with Vert.x:

* via the original Vert.x API with the RxHelper helper class that provides static methods for converting objects between Vert.x core API and RxJava API.

* via the Rxified Vert.x API enhancing the core Vert.x API.

## Read stream support
RxJava observable is a perfect match for Vert.x ReadStream class : both provides provides a flow of items.

The RxHelper.toObservable static methods converts a Vert.x read stream to an rx.Observable:
```java
FileSystem fileSystem = vertx.fileSystem();
fileSystem.open("/data.txt", new OpenOptions(), result -> {
  AsyncFile file = result.result();
  Observable<Buffer> observable = RxHelper.toObservable(file);
  observable.forEach(data -> System.out.println("Read data: " + data.toString("UTF-8")));
});
```
The Rxified Vert.x API provides a toObservable method on ReadStream:
```java
FileSystem fs = vertx.fileSystem();
fs.open("/data.txt", new OpenOptions(), result -> {
  AsyncFile file = result.result();
  Observable<Buffer> observable = file.toObservable();
  observable.forEach(data -> System.out.println("Read data: " + data.toString("UTF-8")));
});
```
Such observables are hot observables, i.e they will produce notifications regardless of subscriptions.

## Handler support
The RxHelper can create an ObservableHandler: an Observable with a toHandler method returning an Handler<T> implementation:
```java
ObservableHandler<Long> observable = RxHelper.observableHandler();
observable.subscribe(id -> {
  // Fired
});
vertx.setTimer(1000, observable.toHandler());
```
The Rxified Vert.x API does not provide a specific API for handler.

## Async result support
The Vert.x Handler<AsyncResult<T>> construct occuring as last parameter of an asynchronous methods can be mapped to an observable of a single element:

* when the callback is a success, the observer onNext method is called with the item and the onComplete method is immediatly invoked after

* when the callback is a failure, the observer onError method is called

The RxHelper.observableFuture method creates an ObservableFuture: an Observable with a toHandler method returning a Handler<AsyncResult<T>> implementation:
```java
ObservableFuture<HttpServer> observable = RxHelper.observableFuture();
observable.subscribe(
    server -> {
      // Server is listening
    },
    failure -> {
      // Server could not start
    }
);
vertx.createHttpServer(new HttpServerOptions().
    setPort(1234).
    setHost("localhost")
).listen(observable.toHandler());
```
The ObservableFuture<Server> will get a single HttpServer object, if the listen operation fails, the subscriber will be notified with the failure.

The RxHelper.toHandler method adapts an existing Observer into an handler:
```java
Observer<HttpServer> observer = new Observer<HttpServer>() {
  @Override
  public void onNext(HttpServer o) {
  }
  @Override
  public void onError(Throwable e) {
  }
  @Override
  public void onCompleted() {
  }
};
Handler<AsyncResult<HttpServer>> handler = RxHelper.toFuture(observer);
```
It also works with just actions:
```java
Action1<HttpServer> onNext = httpServer -> {};
Action1<Throwable> onError = httpServer -> {};
Action0 onComplete = () -> {};

Handler<AsyncResult<HttpServer>> handler1 = RxHelper.toFuture(onNext);
Handler<AsyncResult<HttpServer>> handler2 = RxHelper.toFuture(onNext, onError);
Handler<AsyncResult<HttpServer>> handler3 = RxHelper.toFuture(onNext, onError, onComplete);
```
The Rxified Vert.x API duplicates each such method with the Observable suffix that returns an observable:
```java
vertx.createHttpServer(
    new HttpServerOptions().setPort(1234).setHost("localhost")
).listenObservable().
    subscribe(
        server -> {
          // Server is listening
        },
        failure -> {
          // Server could not start
        }
    );
```
Such observables are cold observables, i.e they will produce notifications on request.

## Scheduler support
The reactive extension sometimes needs to schedule actions, for instance Observable#timer creates and returns a timer that emit periodic events. By default, scheduled actions are managed by RxJava, it means that the timer thread are not Vert.x threads and therefore not executing in a Vert.x event loop.

When an RxJava method deals with a scheduler, it accepts an overloaded method accepting an extra rx.Scheduler, the RxHelper.scheduler method will return a scheduler that can be used in such places.
```java
Scheduler scheduler = RxHelper.scheduler(vertx);
Observable<Long> timer = Observable.timer(100, 100, TimeUnit.MILLISECONDS, scheduler);
```
For blocking scheduled actions, a scheduler can be created with the RxHelper.blockingScheduler method:
```java
Scheduler scheduler = RxHelper.blockingScheduler(vertx);
Observable<Integer> obs = blockingObservable.observeOn(scheduler);
```
RxJava can also be reconfigured to use the Vert.x scheduler, thanks to the scheduler hook created with RxHelper.schedulerHook, the returned scheduler hook uses a blocking scheduler for IO actions:
```java
RxJavaSchedulersHook hook = RxHelper.schedulerHook(vertx);
rx.plugins.RxJavaPlugins.getInstance().registerSchedulersHook(hook);
```
The Rxified Vert.x API provides also similar method on the RxHelper class:
```java
Scheduler scheduler = io.vertx.rxjava.core.RxHelper.scheduler(vertx);
Observable<Long> timer = Observable.timer(100, 100, TimeUnit.MILLISECONDS, scheduler);
RxJavaSchedulersHook hook = io.vertx.rxjava.core.RxHelper.schedulerHook(vertx);
rx.plugins.RxJavaPlugins.getInstance().registerSchedulersHook(hook);
```

## Json unmashalling
The RxHelper.unmarshaller creates an rx.Observable.Operator that transforms an Observable<Buffer> in json format into an object observable:
```java
fileSystem.open("/data.txt", new OpenOptions(), result -> {
  AsyncFile file = result.result();
  Observable<Buffer> observable = RxHelper.toObservable(file);
  observable.lift(RxHelper.unmarshaller(MyPojo.class)).subscribe(
      mypojo -> {
        // Process the object
      }
  );
});
```
The same can be done with the Rxified helper:
```java
fileSystem.open("/data.txt", new OpenOptions(), result -> {
  AsyncFile file = result.result();
  Observable<Buffer> observable = file.toObservable();
  observable.lift(io.vertx.rxjava.core.RxHelper.unmarshaller(MyPojo.class)).subscribe(
      mypojo -> {
        // Process the object
      }
  );
});
```