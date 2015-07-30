# Vert.x 3.0 文档翻译

关于Vert.x 2.0 的文档翻译参考[Ver.x 2.0](http://vertx.gnim.wang)

引用一篇文章[关于Java框架Vert.x的几点思考](http://www.csdn.net/article/2015-05-20/2824733-Java),来阐述一下Vert.x 2.0和3.0之间的区别：

Vert.x3.0是对Vert.x2.x的重大升级,不仅仅是package从org.vertx到io.vertx的全面替换,一些重要的核心类也都做了破坏式的重构,几乎很难从vert.x2程序升级到vert.x3.0程序.建议新项目直接从Vert.x3.0开始.以下是Vert.x3的一些功能升级：

1. Vert.x2.x中的模块体系去掉了.目前Vert.x3.0推荐用Maven的模块体系,当然不仅限于Maven;支持其他语言在Vert.x上的代码生成;
2. Vert.x3.0项目构建,从Gradle改为Maven;为了更好地利用Java8的Lambdas表达式,只支持Java8;默认采用扁平的classpath结构;
3. Verticle工厂方式简化;支持用编程的方式实例化Verticle、以及部署Verticle实例;当你阻塞Eventloop主线程时警告,阻塞Reactor主线程是一种错误的使用方式;移除了PlatformManager模块;集群管理可以用编程的方式调用支持集群节点之间的共享数据;完全重写了HTTPclient,更完善;
4. WebSocketAPI改善;
5. SSL/TLS的改善;
6. Eventbus的API改善;
7. 支持Eventbus代理;增加了扩展项目集'ext'stack;
8. 增加了MongoService,支持MongoDB的纯异步驱动;
9. 实现ReactiveStreams;
10. 对reactive-streams的实现;
11. 支持Options类的使用,可以构造函数带参数进去;

对本文翻译有任何异议的朋友可以到QQ群`219655467`上找我(lol_vertx)进行讨论,,,
