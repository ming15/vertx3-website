# SSL TLS
Configuring servers and clients to work with SSL/TLS
TCP clients and servers can be configured to use Transport Layer Security - earlier versions of TLS were known as SSL.

The APIs of the servers and clients are identical whether or not SSL/TLS is used, and it’s enabled by configuring the NetClientOptions or NetServerOptions instances used to create the servers or clients.

Enabling SSL/TLS on the server

SSL/TLS is enabled with ssl.

By default it is disabled.

Specifying key/certificate for the server

SSL/TLS servers usually provide certificates to clients in order verify their identity to clients.

Certificates/keys can be configured for servers in several ways:

The first method is by specifying the location of a Java key-store which contains the certificate and private key.

Java key stores can be managed with the keytool utility which ships with the JDK.

The password for the key store should also be provided:

NetServerOptions options = new NetServerOptions().setSsl(true).setKeyStoreOptions(
    new JksOptions().
        setPath("/path/to/your/server-keystore.jks").
        setPassword("password-of-your-keystore")
);
NetServer server = vertx.createNetServer(options);
Alternatively you can read the key store yourself as a buffer and provide that directly:

Buffer myKeyStoreAsABuffer = vertx.fileSystem().readFileBlocking("/path/to/your/server-keystore.jks");
JksOptions jksOptions = new JksOptions().
    setValue(myKeyStoreAsABuffer).
    setPassword("password-of-your-keystore");
NetServerOptions options = new NetServerOptions().
    setSsl(true).
    setKeyStoreOptions(jksOptions);
NetServer server = vertx.createNetServer(options);
Key/certificate in PKCS#12 format (http://en.wikipedia.org/wiki/PKCS_12), usually with the .pfx or the .p12 extension can also be loaded in a similar fashion than JKS key stores:

NetServerOptions options = new NetServerOptions().setSsl(true).setPfxKeyCertOptions(
    new PfxOptions().
        setPath("/path/to/your/server-keystore.pfx").
        setPassword("password-of-your-keystore")
);
NetServer server = vertx.createNetServer(options);
Buffer configuration is also supported:

Buffer myKeyStoreAsABuffer = vertx.fileSystem().readFileBlocking("/path/to/your/server-keystore.pfx");
PfxOptions pfxOptions = new PfxOptions().
    setValue(myKeyStoreAsABuffer).
    setPassword("password-of-your-keystore");
NetServerOptions options = new NetServerOptions().
    setSsl(true).
    setPfxKeyCertOptions(pfxOptions);
NetServer server = vertx.createNetServer(options);
Another way of providing server private key and certificate separately using .pem files.

NetServerOptions options = new NetServerOptions().setSsl(true).setPemKeyCertOptions(
    new PemKeyCertOptions().
        setKeyPath("/path/to/your/server-key.pem").
        setCertPath("/path/to/your/server-cert.pem")
);
NetServer server = vertx.createNetServer(options);
Buffer configuration is also supported:

Buffer myKeyAsABuffer = vertx.fileSystem().readFileBlocking("/path/to/your/server-key.pem");
Buffer myCertAsABuffer = vertx.fileSystem().readFileBlocking("/path/to/your/server-cert.pem");
PemKeyCertOptions pemOptions = new PemKeyCertOptions().
    setKeyValue(myKeyAsABuffer).
    setCertValue(myCertAsABuffer);
NetServerOptions options = new NetServerOptions().
    setSsl(true).
    setPemKeyCertOptions(pemOptions);
NetServer server = vertx.createNetServer(options);
Keep in mind that pem configuration, the private key is not crypted.

Specifying trust for the server

SSL/TLS servers can use a certificate authority in order to verify the identity of the clients.

Certificate authorities can be configured for servers in several ways:

Java trust stores can be managed with the keytool utility which ships with the JDK.

The password for the trust store should also be provided:

NetServerOptions options = new NetServerOptions().
    setSsl(true).
    setClientAuthRequired(true).
    setTrustStoreOptions(
        new JksOptions().
            setPath("/path/to/your/truststore.jks").
            setPassword("password-of-your-truststore")
    );
NetServer server = vertx.createNetServer(options);
Alternatively you can read the trust store yourself as a buffer and provide that directly:

Buffer myTrustStoreAsABuffer = vertx.fileSystem().readFileBlocking("/path/to/your/truststore.jks");
NetServerOptions options = new NetServerOptions().
    setSsl(true).
    setClientAuthRequired(true).
    setTrustStoreOptions(
        new JksOptions().
            setValue(myTrustStoreAsABuffer).
            setPassword("password-of-your-truststore")
    );
NetServer server = vertx.createNetServer(options);
Certificate authority in PKCS#12 format (http://en.wikipedia.org/wiki/PKCS_12), usually with the .pfx or the .p12 extension can also be loaded in a similar fashion than JKS trust stores:

NetServerOptions options = new NetServerOptions().
    setSsl(true).
    setClientAuthRequired(true).
    setPfxTrustOptions(
        new PfxOptions().
            setPath("/path/to/your/truststore.pfx").
            setPassword("password-of-your-truststore")
    );
NetServer server = vertx.createNetServer(options);
Buffer configuration is also supported:

Buffer myTrustStoreAsABuffer = vertx.fileSystem().readFileBlocking("/path/to/your/truststore.pfx");
NetServerOptions options = new NetServerOptions().
    setSsl(true).
    setClientAuthRequired(true).
    setPfxTrustOptions(
        new PfxOptions().
            setValue(myTrustStoreAsABuffer).
            setPassword("password-of-your-truststore")
    );
NetServer server = vertx.createNetServer(options);
Another way of providing server certificate authority using a list .pem files.

NetServerOptions options = new NetServerOptions().
    setSsl(true).
    setClientAuthRequired(true).
    setPemTrustOptions(
        new PemTrustOptions().
            addCertPath("/path/to/your/server-ca.pem")
    );
NetServer server = vertx.createNetServer(options);
Buffer configuration is also supported:

Buffer myCaAsABuffer = vertx.fileSystem().readFileBlocking("/path/to/your/server-ca.pfx");
NetServerOptions options = new NetServerOptions().
    setSsl(true).
    setClientAuthRequired(true).
    setPemTrustOptions(
        new PemTrustOptions().
            addCertValue(myCaAsABuffer)
    );
NetServer server = vertx.createNetServer(options);
Enabling SSL/TLS on the client

Net Clients can also be easily configured to use SSL. They have the exact same API when using SSL as when using standard sockets.

To enable SSL on a NetClient the function setSSL(true) is called.

Client trust configuration

If the trustALl is set to true on the client, then the client will trust all server certificates. The connection will still be encrypted but this mode is vulnerable to 'man in the middle' attacks. I.e. you can’t be sure who you are connecting to. Use this with caution. Default value is false.

NetClientOptions options = new NetClientOptions().
    setSsl(true).
    setTrustAll(true);
NetClient client = vertx.createNetClient(options);
If trustAll is not set then a client trust store must be configured and should contain the certificates of the servers that the client trusts.

Likewise server configuration, the client trust can be configured in several ways:

The first method is by specifying the location of a Java trust-store which contains the certificate authority.

It is just a standard Java key store, the same as the key stores on the server side. The client trust store location is set by using the function path on the jks options. If a server presents a certificate during connection which is not in the client trust store, the connection attempt will not succeed.

NetClientOptions options = new NetClientOptions().
    setSsl(true).
    setTrustStoreOptions(
        new JksOptions().
            setPath("/path/to/your/truststore.jks").
            setPassword("password-of-your-truststore")
    );
NetClient client = vertx.createNetClient(options);
Buffer configuration is also supported:

Buffer myTrustStoreAsABuffer = vertx.fileSystem().readFileBlocking("/path/to/your/truststore.jks");
NetClientOptions options = new NetClientOptions().
    setSsl(true).
    setTrustStoreOptions(
        new JksOptions().
            setValue(myTrustStoreAsABuffer).
            setPassword("password-of-your-truststore")
    );
NetClient client = vertx.createNetClient(options);
Certificate authority in PKCS#12 format (http://en.wikipedia.org/wiki/PKCS_12), usually with the .pfx or the .p12 extension can also be loaded in a similar fashion than JKS trust stores:

NetClientOptions options = new NetClientOptions().
    setSsl(true).
    setPfxTrustOptions(
        new PfxOptions().
            setPath("/path/to/your/truststore.pfx").
            setPassword("password-of-your-truststore")
    );
NetClient client = vertx.createNetClient(options);
Buffer configuration is also supported:

Buffer myTrustStoreAsABuffer = vertx.fileSystem().readFileBlocking("/path/to/your/truststore.pfx");
NetClientOptions options = new NetClientOptions().
    setSsl(true).
    setPfxTrustOptions(
        new PfxOptions().
            setValue(myTrustStoreAsABuffer).
            setPassword("password-of-your-truststore")
    );
NetClient client = vertx.createNetClient(options);
Another way of providing server certificate authority using a list .pem files.

NetClientOptions options = new NetClientOptions().
    setSsl(true).
    setPemTrustOptions(
        new PemTrustOptions().
            addCertPath("/path/to/your/ca-cert.pem")
    );
NetClient client = vertx.createNetClient(options);
Buffer configuration is also supported:

Buffer myTrustStoreAsABuffer = vertx.fileSystem().readFileBlocking("/path/to/your/ca-cert.pem");
NetClientOptions options = new NetClientOptions().
    setSsl(true).
    setPemTrustOptions(
        new PemTrustOptions().
            addCertValue(myTrustStoreAsABuffer)
    );
NetClient client = vertx.createNetClient(options);
Specifying key/certificate for the client

If the server requires client authentication then the client must present its own certificate to the server when connecting. The client can be configured in several ways:

The first method is by specifying the location of a Java key-store which contains the key and certificate. Again it’s just a regular Java key store. The client keystore location is set by using the function path on the jks options.

NetClientOptions options = new NetClientOptions().setSsl(true).setKeyStoreOptions(
    new JksOptions().
        setPath("/path/to/your/client-keystore.jks").
        setPassword("password-of-your-keystore")
);
NetClient client = vertx.createNetClient(options);
Buffer configuration is also supported:

Buffer myKeyStoreAsABuffer = vertx.fileSystem().readFileBlocking("/path/to/your/client-keystore.jks");
JksOptions jksOptions = new JksOptions().
    setValue(myKeyStoreAsABuffer).
    setPassword("password-of-your-keystore");
NetClientOptions options = new NetClientOptions().
    setSsl(true).
    setKeyStoreOptions(jksOptions);
NetClient client = vertx.createNetClient(options);
Key/certificate in PKCS#12 format (http://en.wikipedia.org/wiki/PKCS_12), usually with the .pfx or the .p12 extension can also be loaded in a similar fashion than JKS key stores:

NetClientOptions options = new NetClientOptions().setSsl(true).setPfxKeyCertOptions(
    new PfxOptions().
        setPath("/path/to/your/client-keystore.pfx").
        setPassword("password-of-your-keystore")
);
NetClient client = vertx.createNetClient(options);
Buffer configuration is also supported:

Buffer myKeyStoreAsABuffer = vertx.fileSystem().readFileBlocking("/path/to/your/client-keystore.pfx");
PfxOptions pfxOptions = new PfxOptions().
    setValue(myKeyStoreAsABuffer).
    setPassword("password-of-your-keystore");
NetClientOptions options = new NetClientOptions().
    setSsl(true).
    setPfxKeyCertOptions(pfxOptions);
NetClient client = vertx.createNetClient(options);
Another way of providing server private key and certificate separately using .pem files.

NetClientOptions options = new NetClientOptions().setSsl(true).setPemKeyCertOptions(
    new PemKeyCertOptions().
        setKeyPath("/path/to/your/client-key.pem").
        setCertPath("/path/to/your/client-cert.pem")
);
NetClient client = vertx.createNetClient(options);
Buffer configuration is also supported:

Buffer myKeyAsABuffer = vertx.fileSystem().readFileBlocking("/path/to/your/client-key.pem");
Buffer myCertAsABuffer = vertx.fileSystem().readFileBlocking("/path/to/your/client-cert.pem");
PemKeyCertOptions pemOptions = new PemKeyCertOptions().
    setKeyValue(myKeyAsABuffer).
    setCertValue(myCertAsABuffer);
NetClientOptions options = new NetClientOptions().
    setSsl(true).
    setPemKeyCertOptions(pemOptions);
NetClient client = vertx.createNetClient(options);
Keep in mind that pem configuration, the private key is not crypted.

Revoking certificate authorities

Trust can be configured to use a certificate revocation list (CRL) for revoked certificates that should no longer be trusted. The crlPath configures the crl list to use:

NetClientOptions options = new NetClientOptions().
    setSsl(true).
    setTrustStoreOptions(trustOptions).
    addCrlPath("/path/to/your/crl.pem");
NetClient client = vertx.createNetClient(options);
Buffer configuration is also supported:

Buffer myCrlAsABuffer = vertx.fileSystem().readFileBlocking("/path/to/your/crl.pem");
NetClientOptions options = new NetClientOptions().
    setSsl(true).
    setTrustStoreOptions(trustOptions).
    addCrlValue(myCrlAsABuffer);
NetClient client = vertx.createNetClient(options);
