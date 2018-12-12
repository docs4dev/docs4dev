## 108. Vault Client快速失败

在某些情况下，如果服务无法连接到Vault服务器，则可能需要失败启动服务.如果这是所需的行为，请设置引导程序配置属性 `spring.cloud.vault.fail-fast=true` ，客户端将以异常停止.

```java
spring.cloud.vault:
fail-fast: true
```

