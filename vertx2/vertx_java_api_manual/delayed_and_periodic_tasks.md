# Delayed and Periodic Tasks

在Vert.x中有一种常用操作就是经过一段时间的延迟后执行某种操作

chap.c在标准verticle中,你不能通过让线程sleep的方式来达到延迟的效果,因为这会阻塞`event loop`线程

你可以使用Vert.x定时器.定时器可以是one-shot 或者 periodic

### One-shot Timers

`one shot`定时器当延迟时间一到就会调用一个`event handler`.延迟单位是毫秒

你只需要调用`setTimer`方法,然后向该方法传递需延迟的时间和一个handler.
```java
long timerID = vertx.setTimer(1000, new Handler<Long>() {
    public void handle(Long timerID) {
        log.info("And one second later this is printed");
    }
});

log.info("First this is printed");
```

该方法的返回值是一个唯一的定时器ID,我们可以使用该ID取消该定时器

### Periodic Timers

你还可以使用`setPeriodic`方法设置一个阶段定时器.这个定时器每隔一段时间就会执行一次.同样该方法的返回值是一个唯一的定时器ID,我们同样可以使用该ID取消定时器.
```java
long timerID = vertx.setPeriodic(1000, new Handler<Long>() {
    public void handle(Long timerID) {
        log.info("And every second this is printed");
    }
});

log.info("First this is printed");
```

### Cancelling timers

我们调用`cancelTimer`方法可以取消掉`periodic timer`
```java
long timerID = vertx.setPeriodic(1000, new Handler<Long>() {
    public void handle(Long timerID) {
    }
});

// And immediately cancel it

vertx.cancelTimer(timerID);
```

或者你可以在event handler里取消它.下面的例子就是在10秒后取消掉了该定时器
```java
long timerID = vertx.setPeriodic(1000, new Handler<Long>() {
    int count;
    public void handle(Long timerID) {
        log.info("In event handler " + count);
        if (++count == 10) {
            vertx.cancelTimer(timerID);
        }
    }
});
```
