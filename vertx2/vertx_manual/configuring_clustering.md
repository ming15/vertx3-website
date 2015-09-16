# Configuring clustering

在分布式环境中可以使用`conf/cluster.xml`文件配置集群。

如果你想要采集集群配置过程中的信息，可以修改`conf/logging.properties`文件，设置`com.hazelcast.level=INFO`

当运行一个集群时，如果你有多个网络接口选择，那么你可以通过修改`interfaces-enabled`元素，确保`Hazelcast`使用的上古当前网络接口

如果你的网络不支持多播(组播)，那么你可以在配置文件中将`multicast`关闭掉，然后开启`tcp-ip`