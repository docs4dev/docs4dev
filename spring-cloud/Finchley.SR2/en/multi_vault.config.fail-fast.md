## 108. Vault Client Fail Fast

In some cases, it may be desirable to fail startup of a service if it cannot connect to the Vault Server. If this is the desired behavior, set the bootstrap configuration property  `spring.cloud.vault.fail-fast=true`  and the client will halt with an Exception.

```java
spring.cloud.vault:
fail-fast: true
```

