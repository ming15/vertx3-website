# SSL Servers

Net servers can also be configured to work with Transport Layer Security (previously known as SSL).

Net servers仍然可以被配置成以`Transport Layer Security`模式工作

When a NetServer is working as an SSL Server the API of the NetServer and NetSocket is identical compared to when it working with standard sockets. Getting the server to use SSL is just a matter of configuring the NetServer before listen is called.

SSL Server和标准server的API用法是一样的。想要Server以SSL方式运行只需要在调用`listen`方法之前进行一些特殊配置。

To enabled SSL the function setSSL(true) must be called on the Net Server.

* 调用NetServer的`setSSL(true)`,这样就开启了`SSL`功能

The server must also be configured with a key store and an optional trust store.

* 服务器还需要配置一个`key store`和一个可选的`trust store`

These are both Java keystores which can be managed using the keytool utility which ships with the JDK.

*

The keytool command allows you to create keystores, and import and export certificates from them.

The key store should contain the server certificate. This is mandatory - the client will not be able to connect to the server over SSL if the server does not have a certificate.

The key store is configured on the server using the setKeyStorePath() and setKeyStorePassword() methods.

The trust store is optional and contains the certificates of any clients it should trust. This is only used if client authentication is required.

To configure a server to use server certificates only:
```java
NetServer server = vertx.createNetServer()
               .setSSL(true)
               .setKeyStorePath("/path/to/your/keystore/server-keystore.jks")
               .setKeyStorePassword("password");
```
Making sure that server-keystore.jks contains the server certificate.

To configure a server to also require client certificates:
```java
NetServer server = vertx.createNetServer()
               .setSSL(true)
               .setKeyStorePath("/path/to/your/keystore/server-keystore.jks")
               .setKeyStorePassword("password")
               .setTrustStorePath("/path/to/your/truststore/server-truststore.jks")
               .setTrustStorePassword("password")
               .setClientAuthRequired(true);
```
Making sure that server-truststore.jks contains the certificates of any clients who the server trusts.

If clientAuthRequired is set to true and the client cannot provide a certificate, or it provides a certificate that the server does not trust then the connection attempt will not succeed.

