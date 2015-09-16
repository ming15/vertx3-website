# Shared Data

我们可能需要一种安全的方式在不同的verticle间共享数据。 Vert.x允许`java.util.concurrent.ConcurrentMap`和`java.util.Set`这俩个数据结构在verticle间共享。

> 注意：为了避免可变数据带来的问题，Vert.x只允许简单的不可变类型，例如`number, boolean and string or Buffer`等数据类型用于做数据共享。当共享一个buffer时， 当我们从共享数据获取`Buffer`数据时，其实我们只是从共享数据里copy了一个`Buffer`，因此不同的verticle永远不会访问同一个对象。

并发数据只能在同一个Vert.x实例中的verticle实例中进行共享。在以后的版本中，Vert.x会允许数据可以在集群中的所有Vert.x实例间进行共享。

### Shared Maps

如果想要在不同的verticle中共享一个map。首先我们获得这个map的引用，然后就可以使用`java.util.concurrent.ConcurrentMap`的共享实例了。

```java
ConcurrentMap<String, Integer> map = vertx.sharedData().getMap("demo.mymap");

map.put("some-key", 123);
```

当然你也可以在其他的verticle中访问它
```java
ConcurrentMap<String, Integer> map = vertx.sharedData().getMap("demo.mymap");

// etc
```

### Shared Sets

在不同的verticle中使用一个共享的set和使用一个共享的map，方式基本相同
```
Set<String> set = vertx.sharedData().getSet("demo.myset");

set.add("some-value");
```

然后在不同的verticle中使用它
```
Set<String> set = vertx.sharedData().getSet("demo.myset");

// etc
```
