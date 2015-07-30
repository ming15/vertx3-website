# Using the file system with Vert.x

Vert.x `FileSystem`对象对多个文件系统都提供了很多操作.

每一个`Vert.x`实例都有一个文件系统对象,你可以通过`fileSystem`方法获得它.

每一个操作都提供了一个阻塞和一个非阻塞版本.

非阻塞版本会带有一个`handler`参数,当非阻塞操作完成之后或者错误发生的时候,这个handler就会被调用.

下面的例子演示了一个异步拷贝文件的操作.
```
FileSystem fs = vertx.fileSystem();

// Copy file from foo.txt to bar.txt
fs.copy("foo.txt", "bar.txt", res -> {
  if (res.succeeded()) {
    // Copied ok!
  } else {
    // Something went wrong
  }
});
```
那些阻塞版本操作正如其名,会一直进行阻塞操作直到结果返回或者异常发生.

在许多情况下,基于不同的操作系统和文件系统,那些阻塞操作也可以非常快的返回,这也是我们提供阻塞版本的原因,但是我们还是强烈建议你,在`event loop`中当你使用一个阻塞操作时,你应该测试一下,它究竟会耗时多少.

下面演示了如何使用阻塞API
```
FileSystem fs = vertx.fileSystem();

// Copy file from foo.txt to bar.txt synchronously
fs.copyBlocking("foo.txt", "bar.txt");
```
还有很多的其他文件操作(`copy, move, truncate, chmod`),我们就不在此一一列出的,具体的你可以去查看相关API.

## Asynchronous files
Vert.x提供了一种异步文件概念,你可以使用这种方式在文件系统中操作文件.下面的是一种演示.
```
OpenOptions options = new OpenOptions();
fileSystem.open("myfile.txt", options, res -> {
  if (res.succeeded()) {
    AsyncFile file = res.result();
  } else {
    // Something went wrong!
  }
});
```


TODO
