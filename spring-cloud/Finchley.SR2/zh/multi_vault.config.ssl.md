## 109. Vault客户端SSL配置

可以通过设置各种属性以声明方式配置SSL.您可以设置 `javax.net.ssl.trustStore` 以配置JVM范围的SSL设置，或者 `spring.cloud.vault.ssl.trust-store` 仅为Spring Cloud Vault配置设置SSL设置.

```java
spring.cloud.vault:
ssl:
trust-store: classpath:keystore.jks
trust-store-password: changeit
```

-  `trust-store` 设置信任存储的资源. SSL加密的Vault通信将使用指定的信任库验证Vault SSL证书.

-  `trust-store-password` 设置信任存储密码

请注意，只有在Apache Http组件或OkHttp客户端位于类路径上时，才能应用配置 `spring.cloud.vault.ssl.*` .
