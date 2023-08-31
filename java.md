Java was designed to be "write once, run anywhere", which explains why anytime you need to interact with the
underlying operating system it is a massive headache.

# Certificates

Java uses its own certificate store, rather than relying on the parent OS.
This must be managed using the `bin/keytool` utility located inside each Java install.

  ```bash
bin/keytool -list -cacerts
bin/keytool -import -alias <alias> -cacerts -file $path  # Import an X509 PEM certificate by path.
```

Most `cacerts` do not have a password, but the default is often `changeme`.
Beware that there is a difference between using `-cacerts` and using `-keystore cacerts`, the latter
will make a new non-default certificate store that will not be otherwise used.

JVM options can configure how the trust store is used, if the defaults are not appropriate.

```bash
-Djavax.net.ssl.trustStore=$JDKPath$\lib\security\cacerts
-Djavax.net.debug=ssl:handshake
```
