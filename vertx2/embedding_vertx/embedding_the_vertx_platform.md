# Embedding the Vert.x platform


The Vert.x platform is the container which knows how to run Vert.x modules and verticles, it also contains the Vert.x module system.



You use an instance of the interface org.vertx.java.platform.PlatformManager to control the platform.

To get an instance of PlatformManager you use the org.vertx.java.platform.PlatformLocator class:
```java
PlatformManager pm = PlatformLocator.factory.createPlatformManager();
```
Once you have an instance of PlatformManager you can deploy and undeploy modules and verticles, install modules and various other actions. See the JavaDoc for more information on the available methods.

Here's an example:

Deploy 10 instances of a module passing in some config
```java
JsonObject conf = new JsonObject().putString("foo", "wibble");

pm.deployModule("com.mycompany~my-module~1.0", conf, 10, new AsyncResultHandler<String>() {
    public void handle(AsyncResult<String> asyncResult) {
        if (asyncResult.succeeded()) {
            System.out.println("Deployment ID is " + asyncResult.result());
        } else {
            asyncResult.cause().printStackTrace();
        }
    }
});
```
The methods available on PlatformManager roughly map to the actions performed by the vertx command at the command line.

### Required jars

To embed the Vert.x platform you will need all the jars from the Vert.x install lib directory on your classpath. You won't need the hazelcast jar if you aren't using clustering.

### System Properties

When using the platform manager the following system properties can be set:

* `vertx.home` - When installing system modules, vert.x will install them in a directory sys-mods which is in the directory given by this property.
* `vertx.mods` - When looking for or installing non system modules Vert.x will look in the directory mods in the current working directory. If this property is set this will tell Vert.x to instead look in the provided directory.
* `vertx.clusterManagerFactory` - If you're using clustering this property should contain the fully qualified class name of the cluster manager factory that you want to use. If you are just using he default Hazelcast cluster manager factory the value should be org.vertx.java.spi.cluster.impl.hazelcast.HazelcastClusterManagerFactory.

### Config files

The Vert.x platform will look for various config files on the classpath. The vert.x platform .jar contains the default files inside it, but if you want to override any settings you can provide your own versions - just make sure you put them on the classpath ahead of the vert.x platform jar.

#### File `langs.properties`

This config file tells Vert.x which modules contain the language implementations for particular languages. It's described in the Support a New Language Guide

#### File `cluster.xml`

This configures Hazelcast clustering. It's described in the main manual.

#### File `repos.txt`

This configures which repositories the Vert.x module system will look in for modules. It's described in the modules manual.