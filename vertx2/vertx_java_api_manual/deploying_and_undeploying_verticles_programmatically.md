## Deploying and Undeploying Verticles Programmatically

你可以在一个verticle中通过编程方式对其他verticle进行部署和解除部署。任何通过该方式部署的verticle都有能力看见主verticle的资源(classes, scripts 或者其他文件)

## Deploying a simple verticle

如果想要通过程序的方式部署一个verticle，只需要调用`container`变量里的`deployVerticle`方法。

下面的例子就部署了一个verticle实例
```java
container.deployVerticle(main);
```

`main`是被部署的Verticle的名字(java源文件名称或者类的FQCN)

具体参考主手册的[running Vert.x]()章节

## Deploying Worker Verticles

`deployVerticle`方法部署的是标准verticle,如果你想要部署工作者verticle,你可以使用`deployWorkerVerticle`方法，这俩个方法的参数一致。

## Deploying a module programmatically

你可以采用下面的方式部署一个`module`：
```java
container.deployModule("io.vertx~mod-mailer~2.0.0-beta1", config);
```

程序会根据指定的配置部署一个`io.vertx~mod-mailer~2.0.0-beta1`的`module`实例。

## Passing configuration to a verticle programmatically

我们也可以将JSON配置传递给通过程序部署的verticle。在部署的verticle内部，配置可以被`config()`方法访问。
```java
JsonObject config = new JsonObject();
config.putString("foo", "wibble");
config.putBoolean("bar", false);
container.deployVerticle("foo.ChildVerticle", config);
```

然后，在`ChildVerticle`中，你能通过`config()`方法访问刚才的配置

## Using a Verticle to co-ordinate loading of an application

如果你的应用程序是由多个verticle组成，并且希望都当应用程序启动的时候，所有的verticle都能启动起来，那么你可以使用一个单独的verticle来管理应用程序的配置，而且由该verticle启动剩余的全部verticle。

下例中，我们创建了一个`AppStarter`verticle.
```java
// Application config
JsonObject appConfig = container.config();

JsonObject verticle1Config = appConfig.getObject("verticle1_conf");
JsonObject verticle2Config = appConfig.getObject("verticle2_conf");
JsonObject verticle3Config = appConfig.getObject("verticle3_conf");
JsonObject verticle4Config = appConfig.getObject("verticle4_conf");
JsonObject verticle5Config = appConfig.getObject("verticle5_conf");

// Start the verticles that make up the app

container.deployVerticle("verticle1.js", verticle1Config);
container.deployVerticle("verticle2.rb", verticle2Config);
container.deployVerticle("foo.Verticle3", verticle3Config);
container.deployWorkerVerticle("foo.Verticle4", verticle4Config);
container.deployWorkerVerticle("verticle5.js", verticle5Config, 10
```

然后我们创建一个`config.json`配置文件
```java
{
    "verticle1_conf": {
        "foo": "wibble"
    },
    "verticle2_conf": {
        "age": 1234,
        "shoe_size": 12,
        "pi": 3.14159
    },
    "verticle3_conf": {
        "strange": true
    },
    "verticle4_conf": {
        "name": "george"
    },
    "verticle5_conf": {
        "tel_no": "123123123"
    }
}
```

然后将`AppStarter`设置为module里的主要verticle, 接着你就可以通过下面的例子来启动整个应用程序
```
vertx runmod com.mycompany~my-mod~1.0 -conf config.json
```

如果你的应用程序是非常庞大的，而且是由多个module组成，那么你仍然可以使用相同的技术来实现。

通常，你也许会选择一种脚本语言(`JavaScript, Groovy, Ruby or Python`)作为你的启动verticle实现语言，那些语言通常会比java更好地支持JSON，因此你可以在启动verticle中非常友好地持有整个JSON配置。

## Specifying number of instances

当你部署一个verticle时，默认地会只部署一个verticle实例。由于verticle实例是单线程执行的，因此这意味着，这种方式只会用到一个服务器核心。

Vert.x通过部署多个verticle实例来达到拓展（并发运行）

如果你想在程序中部署多个verticle或者module，你可以像下面这样，指定部署实例的数量：
```java
container.deployVerticle("foo.ChildVerticle", 10);
```
或者使用下面这种方式
```
container.deployModule("io.vertx~some-mod~1.0", 10);
```


## Getting Notified when Deployment is complete

verticle的部署实际上是以异步方式运行的，也许是在`deployVerticle`或者`deployModule`方法返回之后才完成部署.如果你想当部署完成之后获得通知，那么你可以向 `deployVerticle`或者`deployModule`方法传递一个handler，以便当部署完成时获得通知。
```java
container.deployVerticle("foo.ChildVerticle", new AsyncResultHandler<String>() {
    public void handle(AsyncResult<String> asyncResult) {
        if (asyncResult.succeeded()) {
            System.out.println("The verticle has been deployed, deployment ID is " + asyncResult.result());
        } else {
            asyncResult.cause().printStackTrace();
        }
    }
});
```

当部署完成时，`handler`会获得一个`AsyncResult`实例. 你可以通过调用`AsyncResult`对象的`succeeded()` 和 `failed()`方法来观察部署是否正确完成了。

`result()`方法提供异步操作的结果,在这个例子中，部署的结果是部署ID,如果你以后要接触部署verticle或者module的话，你就需要这个部署ID了。

`cause()`方法提供了失败原因

## Undeploying a Verticle or Module

如果verticle被解除部署后，那么通过该verticle部署的verticle或者module，以及它们所有子代，都会被自动解除部署，所以在大多数情况下，你不需要手动地去解除部署一个verticle。然而，当你真的需要手动去解除部署verticle或者module时，你可以通过调用`undeployVerticle`或者`undeployModule`的方法来实现(这俩个方法需要传递部署ID)。
```
container.undeployVerticle(deploymentID);
```

你也可以向这俩个方法中传递一个handler，那么当解除部署完成后，你就会得到一个通知

## Scaling your application

一个verticle实例总是单线程的(工作者verticle除外),这意味着一个verticle实例最多使用一个服务器核心

为了能够利用多核优势，你需要部署多个verticle实例。需要部署的具体数量就取决于你的应用程序了，例如有多少个verticle(不是verticle实例)以及verticle的类型都是什么。

你可以通过程序方式部署多个verticle实例，或者在命令行上通过`-instances`选项指定部署的数量
