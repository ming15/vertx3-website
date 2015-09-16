# Buffers

在Vert.x中进行数据传播的大多是`org.vertx.java.core.buffer.Buffer`实例

`Buffer`表示的是一个字节序列(size >= 0), 可以向`Buffer`写入或者读取数据, 当写入数据时，超过其容量最大值时，会自动拓容。

#### Creating Buffers

创建一个空`Buffer`
```java
Buffer buff = new Buffer();
```

使用`String`类型创建一个buffer。这个`String`在`Buffer`内部以`UTF-8`进行编码
```java
Buffer buff = new Buffer("some-string");
```

指定`String`编码格式创建一个`Buffer`实例。
```java
Buffer buff = new Buffer("some-string", "UTF-16");
```

使用`byte[]`创建一个`Buffer`
```java
byte[] bytes = new byte[] { ... };
new Buffer(bytes);
```

在创建`Buffer`实例时，我们也可以指定其大小。当你确定写入buffer的数据大小时，你可以创建一个指定大小的buffer。当buffer创建成功之后，就会分配出指定大小的内存，这种方式比buffer容量不足时，自动拓容要高效的多，但是要慎用，因为它一开始就可能会非常大的内存。

注意，通过指定大小的方式创建出的`Buffer`实例，给它分配的内存是空的，并不会用0去填充它。
```java
Buffer buff = new Buffer(100000);
```

#### Writing to a Buffer

有俩种方式向一个buffer中写入数据：
1. appending
2. random access

buffer会随着写入的数据的不断增加自动拓容，因此，`Buffer`实例的写数据操作不可能产生`IndexOutOfBoundsException`异常

##### Appending to a Buffer

想要使用`append`方式向buffer中写入数据,你只需要调用`appendXXX`方法. `Append`方法支持追加`buffers, byte[], String and all primitive types`

`appendXXX`方法会返回`Buffer`实例自身，所以在也可以直接使用`chain`模式
```java
Buffer buff = new Buffer();

buff.appendInt(123).appendString("hello\n");

socket.write(buff);
```

##### Random access buffer writes

你也可以通过`setXXX`方法在一个指定位置上写入数据。 `setXXX`方法支持`buffers, byte[], String and all primitive types`.所有的`setXXX`方法的第一个参数都是个写入位置的索引值。

无论采用什么写数据的方式,`Buffer`总会当内存不足时，进行自动拓容
```java
Buffer buff = new Buffer();

buff.setInt(1000, 123);
buff.setBytes(0, "hello");
```

#### Reading from a Buffer

我们通过`getXXX`方法从`Buffer`里读数据. `getXXX`方法支持`byte[], String and all primitive types`. `getXXX`方法的第一个值是开始读取的位置索引值
```java
Buffer buff = ...;
for (int i = 0; i < buff.length(); i += 4) {
    System.out.println("int value at " + i + " is " + buff.getInt(i));
}
```

#### Other buffer methods:

* length(). 获得buffer的大小。buffer的length值是buffer的最大索引值 + 1
* copy(). 拷贝整个buffer

更多方法参考javadoc手册。