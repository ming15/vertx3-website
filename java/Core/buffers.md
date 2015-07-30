# Buffers

在Vert.x中进行数据传播的大多是org.vertx.java.core.buffer.Buffer实例

`Buffer`表示的是一个字节序列(size >= 0), 可以向Buffer写入或者读取数据, 当写入数据时，超过其容量最大值时，会自动拓容。

## Creating buffers

我们可以直接使用一系列`Buffer.buffer`开头的静态方法来创建一个`Buffer`.

`Buffer`可以从`String`或者`byte arrays`进行初始化,当然我们也可以直接创建出一个空`Buffer`.

下面给出了一些创建`Buffer`的示例：

创建一个内容为空的`Buffer`
```java
Buffer buff = Buffer.buffer();
```

创建一个`Buffer`,并使用`String`进行初始化,在`Buffer`内部该字符串会使用`UTF-8`进行编码
```java
Buffer buff = Buffer.buffer("some string");
```

创建一个`Buffer`,并使用`String`进行初始化,在`Buffer`内部该字符串会使用指定的编码方法进行编码
```java
Buffer buff = Buffer.buffer("some string", "UTF-16");
```

创建一个`Buffer`,并使用`byte[]`进行初始化
```java
byte[] bytes = new byte[] {1, 3, 5};
Buffer buff = Buffer.buffer(bytes);
```

我们还可以在创建`Buffer`时指定其初始化大小。如果你能确定向`Buffer`写入数据的大小，那么你可以在创建`Buffer`指定其初始化大小。当`Buffer`创建成功之后，`Buffer`就会被分配出所指定的内存，一般来说这种方式适用于你的`Buffer`在不断地自动拓容的情况下。

需要注意的是，使用指定大小的方式创建一个`Buffer`，它本身是空的，只是分配了那么多内存而已。它并不会使用0来填充整个`Buffer`。
```java
Buffer buff = Buffer.buffer(10000);
```

## Writing to a Buffer
有俩种方式向`Buffer`中添加数据：`appending`和`random access`. 不管使用哪种方式,`Buffer`都会当容量不足时进行自动拓容。

### Appending to a Buffer

`Buffer`提供了多种`append`方法，向`Buffer`中追加不同类型的数据。而且`append`方法返回的都是`Buffer`自身，因此我们可以使用链式调用`append`方法

```java
Buffer buff = Buffer.buffer();

buff.appendInt(123).appendString("hello\n");

socket.write(buff);
```

### Random access buffer writes

你也可以通过一系列`set`方法在某个合法的索引位置上写入数据。在`set`方法中第一个参数是要开始写入数据的索引位置,第二个参数是要写入的数据

```java
Buffer buff = Buffer.buffer();

buff.setInt(1000, 123);
buff.setString(0, "hello");
```

## Reading from a Buffer

我们通过一系列`get`方法从`Buffer`中读取数据,第一个参数是要开始读取的索引位置。
```java
Buffer buff = Buffer.buffer();
for (int i = 0; i < buff.length(); i += 4) {
  System.out.println("int value at " + i + " is " + buff.getInt(i));
}
```
## Buffer length
使用`length`方法来获得`Buffer`的长度, `length`的取值方式是最大的索引值+1

## Copying buffers
使用`copy`可以直接对`Buffer`进行数据拷贝

> 拷贝之后的俩个`Buffer`是否使用同一个缓冲区

## Slicing buffers
我们使用`slice`方法创建一个`sliced buffer`,它与原`Buffer`共享一个数据缓冲区。

##Buffer re-use
当`Buffer`被写入到`socket`中，或者其他的一些类似的地方，他们就不能被复用了
