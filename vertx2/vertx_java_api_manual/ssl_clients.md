# SSL Clients


Net Clients can also be easily configured to use SSL. They have the exact same API when using SSL as when using standard sockets.

To enable SSL on a NetClient the function setSSL(true) is called.

If the setTrustAll(true) is invoked on the client, then the client will trust all server certificates. The connection will still be encrypted but this mode is vulnerable to 'man in the middle' attacks. I.e. you can't be sure who you are connecting to. Use this with caution. Default value is false.

If setTrustAll(true) has not been invoked then a client trust store must be configured and should contain the certificates of the servers that the client trusts.

The client trust store is just a standard Java key store, the same as the key stores on the server side. The client trust store location is set by using the function setTrustStorePath() on the NetClient. If a server presents a certificate during connection which is not in the client trust store, the connection attempt will not succeed.

If the server requires client authentication then the client must present its own certificate to the server when connecting. This certificate should reside in the client key store. Again it's just a regular Java key store. The client keystore location is set by using the function setKeyStorePath() on the NetClient.

To configure a client to trust all server certificates (dangerous):
```java
NetClient client = vertx.createNetClient()
               .setSSL(true)
               .setTrustAll(true);
```
To configure a client to only trust those certificates it has in its trust store:
```java
NetClient client = vertx.createNetClient()
               .setSSL(true)
               .setTrustStorePath("/path/to/your/client/truststore/client-truststore.jks")
               .setTrustStorePassword("password");
```
To configure a client to only trust those certificates it has in its trust store, and also to supply a client certificate:
```java
NetClient client = vertx.createNetClient()
               .setSSL(true)
               .setTrustStorePath("/path/to/your/client/truststore/client-truststore.jks")
               .setTrustStorePassword("password")
               .setClientAuthRequired(true)
               .setKeyStorePath("/path/to/keystore/holding/client/cert/client-keystore.jks")
               .setKeyStorePassword("password");
```
