## 110.租赁生命周期管理（续订和撤销）

对于每个秘密，Vault都会创建租约：包含持续时间，可续订性等信息的元数据.

Vault承诺数据在给定的持续时间或生存时间（TTL）内有效.一旦租约到期，Vault就可以撤销数据，并且秘密的消费者不再能够确定它是否有效.

Spring Cloud Vault维护一个超出创建登录令牌和机密的租约生命周期.也就是说，登记令牌和与租约相关的秘密计划在租约到期之前续约，直到终端到期.应用程序关闭撤销获取的登录令牌和可续订租约.

特勤服务和数据库后端（例如MongoDB或MySQL）通常会生成可更新的租约，因此在应用程序关闭时将禁用生成的凭据.

> Static令牌未续订或撤销.

默认情况下启用租赁续订和撤销，可以通过将 `spring.cloud.vault.config.lifecycle.enabled` 设置为 `false` 来禁用.建议不要这样做，因为租约可能会过期，并且Spring Cloud Vault无法再使用生成的凭据访问Vault或服务，并且在应用程序关闭后，有效凭据仍保持活动状态.

```java
spring.cloud.vault:
config.lifecycle.enabled: true
```

另见：[Vault Documentation: Lease, Renew, and Revoke](https://www.vaultproject.io/docs/concepts/lease.html)
