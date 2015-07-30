# Using Shared Data with Vert.x

Shared data contains functionality that allows you to safely share data between different parts of your application, or different applications in the same Vert.x instance or across a cluster of Vert.x instances.

`Shared data`可以让你在同一个`Vertx`应用中安全地共享数据,或者在`Vertx`组成的集群中进行安全的数据共享.


Shared data includes local shared maps, distributed, cluster-wide maps, asynchronous cluster-wide locks and asynchronous cluster-wide counters.

## Local shared maps

Local shared maps allow you to share data safely between different event loops (e.g. different verticles) in the same Vert.x instance.

`Local shared maps`可以让你在不同的`event loop`(例如不同的`verticle`)中安全的共享数据.

`Local shared maps`只允许你

Local shared maps only allow certain data types to be used as keys and values. Those types must either be immutable, or certain other types that can be copied like Buffer. In the latter case the key/value will be copied before putting it in the map.

This way we can ensure there is no shared access to mutable state between different threads in your Vert.x application so you don’t have to worry about protecting that state by synchronising access to it.

Here’s an example of using a shared local map:
```
SharedData sd = vertx.sharedData();

LocalMap<String, String> map1 = sd.getLocalMap("mymap1");

map1.put("foo", "bar"); // Strings are immutable so no need to copy

LocalMap<String, Buffer> map2 = sd.getLocalMap("mymap2");

map2.put("eek", Buffer.buffer().appendInt(123)); // This buffer will be copied before adding to map

// Then... in another part of your application:

map1 = sd.getLocalMap("mymap1");

String val = map1.get("foo");

map2 = sd.getLocalMap("mymap2");

Buffer buff = map2.get("eek");
```

## Cluster-wide asynchronous maps
Cluster-wide asynchronous maps allow data to be put in the map from any node of the cluster and retrieved from any other node.

This makes them really useful for things like storing session state in a farm of servers hosting a Vert.x web application.

You get an instance of AsyncMap with getClusterWideMap.

Getting the map is asynchronous and the result is returned to you in the handler that you specify. Here’s an example:
```
SharedData sd = vertx.sharedData();

sd.<String, String>getClusterWideMap("mymap", res -> {
  if (res.succeeded()) {
    AsyncMap<String, String> map = res.result();
  } else {
    // Something went wrong!
  }
});
```
### Putting data in a map

You put data in a map with put.

The actual put is asynchronous and the handler is notified once it is complete:
```
map.put("foo", "bar", resPut -> {
  if (resPut.succeeded()) {
    // Successfully put the value
  } else {
    // Something went wrong!
  }
});
```
### Getting data from a map

You get data from a map with get.

The actual get is asynchronous and the handler is notified with the result some time later
```
map.get("foo", resGet -> {
  if (resGet.succeeded()) {
    // Successfully got the value
    Object val = resGet.result();
  } else {
    // Something went wrong!
  }
});
```
### Other map operations

You can also remove entries from an asynchronous map, clear them and get the size.

See the API docs for more information.

## Cluster-wide locks
Cluster wide locks allow you to obtain exclusive locks across the cluster - this is useful when you want to do something or access a resource on only one node of a cluster at any one time.

Cluster wide locks have an asynchronous API unlike most lock APIs which block the calling thread until the lock is obtained.

To obtain a lock use getLock.

This won’t block, but when the lock is available, the handler will be called with an instance of Lock, signifying that you now own the lock.

While you own the lock no other caller, anywhere on the cluster will be able to obtain the lock.

When you’ve finished with the lock, you call release to release it, so another caller can obtain it.
```
sd.getLock("mylock", res -> {
  if (res.succeeded()) {
    // Got the lock!
    Lock lock = res.result();

    // 5 seconds later we release the lock so someone else can get it

    vertx.setTimer(5000, tid -> lock.release());

  } else {
    // Something went wrong
  }
});
```
You can also get a lock with a timeout. If it fails to obtain the lock within the timeout the handler will be called with a failure:
```
sd.getLockWithTimeout("mylock", 10000, res -> {
  if (res.succeeded()) {
    // Got the lock!
    Lock lock = res.result();

  } else {
    // Failed to get lock
  }
});
```

## Cluster-wide counters
It’s often useful to maintain an atomic counter across the different nodes of your application.

You can do this with Counter.

You obtain an instance with getCounter:
```
sd.getCounter("mycounter", res -> {
  if (res.succeeded()) {
    Counter counter = res.result();
  } else {
    // Something went wrong!
  }
});
```
Once you have an instance you can retrieve the current count, atomically increment it, decrement and add a value to it using the various methods.

See the API docs for more information.
