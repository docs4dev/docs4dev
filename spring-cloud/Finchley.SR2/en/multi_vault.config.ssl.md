## 109. Vault Client SSL configuration

SSL can be configured declaratively by setting various properties. You can set either  `javax.net.ssl.trustStore`  to configure JVM-wide SSL settings or  `spring.cloud.vault.ssl.trust-store`  to set SSL settings only for Spring Cloud Vault Config.

```java
spring.cloud.vault:
ssl:
trust-store: classpath:keystore.jks
trust-store-password: changeit
```

-  `trust-store`  sets the resource for the trust-store. SSL-secured Vault communication will validate the Vault SSL certificate with the specified trust-store.

-  `trust-store-password`  sets the trust-store password

Please note that configuring  `spring.cloud.vault.ssl.*`  can be only applied when either Apache Http Components or the OkHttp client is on your class-path.
