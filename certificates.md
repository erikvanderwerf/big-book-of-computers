> Every developer knows the Pareto Principle. 80% of your development time will configuring 20 different certificates.

Setting up certificates is essential to configuring all modern environments, from mutual authentication to zero-trust.

```bash
openssl
keytool  # Java, $jre/bin
```

## Mutual Authentication from [PEM](https://en.wikipedia.org/wiki/Privacy-Enhanced_Mail) in Java/Kotlin

Some internet services (e.g. MongoDB Atlas) can use mutual authentication but they only provide PEM files.
This is fine when your client library supports PEM files directly, but Java does not natively have support
for this.

To natively use the private key from the PEM in Java it must be converted into a 
[PKCS12](https://en.wikipedia.org/wiki/PKCS_12)
store.


```bash
openssl pks12 -export -in ${PEM_PATH}.pem -out ${OUT_PATH}.pfx
```

I have not had success using the default `SSLContext` to load this `pfx` store, so it may be required to
manually load the store and configure a new `SSLContext` using this explicitly.

```kotlin
val keyStoreLocation = ...
val keyStorePassword = ...
val trustStoreLocation = ...
val trustStorePassword = ...

// Keys Used for authenticating the client.
val keyStore = KeyStore.getInstance(KeyStore.getDefaultType())
File(keyStoreLocation).inputStream().use {
  keyStore.load(it, keyStorePassword.toCharArray());
}
val keyManagerFactory = KeyManagerFactory.getInstance(KeyManagerFactory.getDefaultAlgorithm())
keyManagerFactory.init(keyStore, keyStorePassword.toCharArray())

// Trust used to authenticate the server.
val trustStore = KeyStore.getInstance(KeyStore.getDefaultType())
File(trustStoreLocation).inputStream().use {
  trustStore.load(it, trustStorePassword.toCharArray())
}
val trustManagerFactory =
  TrustManagerFactory.getInstance(TrustManagerFactory.getDefaultAlgorithm())
trustManagerFactory.init(trustStore)

val sslContext = SSLContext.getInstance("TLS")
sslContext.init(keyManagerFactory.keyManagers, trustManagerFactory.trustManagers, SecureRandom())
```

Code adapted from [this blog post by Damian Terlecki](https://blog.termian.dev/posts/spring-mongodb-x509-ssl-tls/).
MongoDB allowed connections once this was plugged in.
