# Using Vert.x from the command line

`vertx`命令通过命令行与`Vert.x`平台进行交互。它的主要作用是运行`Vert.x``module`和原生`verticle`。

如果你在命令行中仅仅输入`vert`，那么命令行中会输出vertx命令会采用哪些选项。

### Running verticles directly

你可以在命令行中直接通过使用`vertx run`命令直接运行原生`Vert.x` `verticle`实例。

对于快速原型代码(quickly prototyping code)或者简单的应用程序，运行原生`verticle`是非常有用的，但是一般在正式应用程序中，我们建议将你的应用打包成一个`module`来运行。打包成的`module`更加易于运行，封装和复用。

一个最简单的`vertx`示例，就是传递给它一个`verticle`名字选项，然后将该`verticle`运行起来。

如果你的`verticle`是通过`JavaScript, Ruby, Groovy or Python`等脚本语言编写的，那么你只需要将脚本的名字传递给它。(例如`erver.js`, `server.rb`, or `server.groovy`.)

如果`verticle`是通过Java来编写的，那么`verticle`的名字就是应用程序的主类的全限定名，或者是应用程序的主类的Java源文件名，然后由`Vert.x`编译该源文件，再运行它。

###### 下面给出俩种运行verticle方式：
I. 直接运行

```
vertx run app.js

vertx run server.rb

vertx run accounts.py

vertx run MyApp.java

vertx run com.mycompany.widgets.Widget

vertx run SomeScript.groovy
```

II. 在`verticle`名前加上该`verticle`实现语言。例如你的`verticle`是一个编译好的Groovy类，如果你指定好前缀为`groovy`，那么`Vert.x`就知道它是Groovy class而不是Java class。

```
vertx run groovy:com.mycompany.MyGroovyMainClass
```

`vertx run`命令会使用下面几个选项：

* `-conf <config_file>` 对`verticle`提供一些配置。`<config_file>`是一个text文件的名称，该文件包含一个json形式的配置说明，该选项是可选的。

* `-cp <path>` 这个路径指定`verticle`文件和`verticle`中使用到的其他资源的路径。默认值是`.`(当前路径)。如果你的`verticle`引用了其他脚本，类,或者资源(如jar文件)，那么你要确保可以在该路径中找到他们。该路径可以通过`:`分割，包含多个路径。每一个路径都可以是相对或者是绝对路径。例如：`-cp classes:lib/otherscripts:jars/myjar.jar:jars/otherjar.jar`.需要注意的是，不要将这些值放到系统类路径中(`system classpath`),因为这可能会导致在部署应用时，产生不可预期的问题。

* `-instances <instances>` 指定在`Vert.x`中需要实例化该`verticle`的数量。每一个`verticle`实例都必须在单线程中运行，为了能够通过可用核心来扩展你的应用程序，你也许想将同一个`verticle`文件部署多个实例(这样一来,多个线程就可以运行同一种`verticle`了)。如果忽略该选项，那么默认的就只会部署一个实例。

* `-includes <mod_list>` 一个通过`,`分割的`module`名称列表，这些`module`都被包含在`verticle`的`classpath`中。

* `-worker` 该值用于指定`verticle`是否是以工作者的形式启动

* `-cluster` 这个选项用于指定该`Vert.x`实例是否和同一网络下的其他`Vert.x`实例组成一个集群。集群化的`Vert.x`实例会将`Vert.x`和其他节点形成一个分布式事件总线。默认值是`false`

* `-cluster-port` 如果指定了`-cluster`值为`true`，`-cluster-port`指定和其他`Vert.x`实例组成集群的端口号。默认值是0，则随机一个可用端口。一般你不需要指定这个值，除非你真的需要指定一个特殊的端口。

* `-cluster-host` 如果指定了`-cluster`值为true，`-cluster-host`指定和其他`Vert.x`实例组成集群的域名。默认的它会从可用的网络接口中，随机选择一个。如果你的网卡中有多个网络接口，那么你也可以选择一个自己想用的。

下面给出了一些`vertx run`示例：

根据默认配置运行一个JavaScript `verticle`
```
vertx run server.js
```
下面运行10个指定classpath的编译好的Java `verticle`实例
```
vertx run com.acme.Myverticle -cp "classes:lib/myjar.jar" -instances 10
```
用Java源代码运行10个`verticle`实例
```
vertx run Myverticle.java -instances 10
```
运行20个ruby工作者`verticle`实例
```
vertx run order_worker.rb -instances 20 -worker
```
在同一台机器上运行2个JavaScript `verticle`实例，同时将他们和其他服务器组成一个集群。
```
vertx run handler.js -cluster
vertx run sender.js -cluster
```
通过指定配置文件运行一个ruby `verticle`实例
```
vertx run my_vert.rb -conf my_vert.conf
```
my_vert.conf 只包含一些简单配置：
```json
{
    "name": "foo",
    "num_widgets": 46
}
```

#### Forcing language implementation to use

在`Vert.x`的`langs.properties`配置文件中包含了`Vert.x`能够识别的语言，然后`Vert.x`会自动识别出`module`是通过什么语言编写的。有时候也会有一些模棱两可的情况，例如你想要将一个Groovy类作为一个`verticle`，那么你就可以将实现语言作为前缀加在`verticle`前，例如：
```
vertx run groovy:com.mycompany.MyGroovyMain`verticle`
```

### Running modules from the command line

我们高度建议你将任何非实验性的`Vert.x`功能打包成一个`module`。至于如何将你的代码打包成一个`module`，你可以参考[module`s manual]()

想要运行一个`module`，那你就不能再使用`vertx run`命令了，而是要使用`vertx runmod <`module` name>`. 同样，该命令也带有一些选项：

* `-conf <config_file>` - 与`vertx run`意义相同
* `-instances <instances>` - 与`vertx run`意义相同
* `-cluster` - 与`vertx run`意义相同
* `-cluster-host` - 与`vertx run`意义相同
* `-cp` 如果该选项被赋值，那么它将覆盖标准`module classpath`，然后`Vert.x`将会在指定的路径搜索`mod.json`文件和其他`module`资源。这在某些情况下非常有用，例如，你在IDE中开发了一个`module`，你可以在不同的classpath中运行该`module`，然后你就能找到存储项目资源的实际classpath，然后再指定classpath

如果你想要运行一个本地没有安装的`module`，`Vert.x`会自动尝试从仓库(仓库可配置)中安装它。一般`Vert.x`会被配置到从`Maven Central, Sonatype Nexus, Bintray` 以及你本地仓库安装`module`。你也可以在`Vert.x` conf directory中的`repos.txt`中配置其他Maven仓库。

下面是一些直接运行`module`的示例：

运行一个名为`com.acme~my-mod~2.1`的实例
```
vertx runmod com.acme~my-mod~2.1
```
运行一个名为`com.acme~other-mod~1.0.beta1`的`module`，我们指定配置，同时运行10实例
```
vertx runmod com.acme~other-mod~1.0.beta1 -instances 10 -conf other-mod.conf
```

### Running modules directory from .zip files

`vertx runzip`命令能直接从一个`module`zip文件中直接运行`module`. 运行的`module`不要求已经安装在本地或者已经安装在一个`module`仓库里。

```
vertx runzip <zip_file_name>
```
zip`module`运行示例
```
vertx runzip my-mod~2.0.1.zip
```
实际上，`Vert.x`会解压zip文件，将`module`解压到系统临时目录里，然后从该目录里运行该`module`。

### Running modules as executable jars (fat jars)

`Vert.x`还支持装配`fat jars`. `fat jars`都是可运行jar包，它们同时包含着`Vert.x`的二进制文件，这样你就可以直接通过运行`fat jars`来运行里面`module`。
```
java -jar my`module`-1.0-fat.jar
```

这意味着，当你想要通过运行`fat jars`的`module`时，你的机器上可以不安装`Vert.x`，因为jar包中已经包含了所需的`Vert.x`的二进制文件。

你也可以在命令行中向`fat jars`中传递`Vert.x`平台参数
```
java -jar my`module`-1.0-fat.jar -cluster -conf myconf.json
```

你也可以向`fat jars`传递`-cp`参数，该参数同样作用于`Vert.x`平台。下面的例子就在运行`module`时，指定了一个自定制的`cluster.xml`
```
java -jar my`module`-1.0-fat.jar -cluster -conf myconf.json -cp path/to/dir/containiner/cluster_xml
```

可以使用下面的命令创建一个`fat jar`
```
vertx fatjar <module_name>
```
Or you can use the Gradle task in the standard Gradle build or the Maven plugin to build them.

If you want to override any `Vert.x` platform configuration, e.g. langs.properties, cluster.xml or logging configuration, you can add those files to the directory platform_lib inside your `module` that you're making into a fat jar. When executing your fat jar `Vert.x` will recognise this directory and use it to configure `Vert.x` with.

### Displaying version of Vert.x

使用下面的命令查看当前安装的`Vert.x`的版本
```
vertx version
```

### Installing and uninstalling modules

参考 [module`s manual]().