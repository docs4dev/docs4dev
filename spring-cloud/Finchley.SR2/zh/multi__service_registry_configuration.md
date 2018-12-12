## 107.服务注册表配置

您可以通过设置spring.cloud.vault.discovery.enabled = true（默认 `false` ）来使用 `DiscoveryClient` （例如来自Spring Cloud Consul）来查找Vault服务器.最终结果是您的应用程序需要具有适当发现配置的bootstrap.yml（或环境变量）.好处是Vault可以更改其坐标，只要发现服务是固定点即可.默认服务标识为 `vault` ，但您可以使用 `spring.cloud.vault.discovery.serviceId` 在客户端上更改该标识.

发现客户端实现都支持某种元数据映射（例如，对于Eureka，我们有eureka.instance.metadataMap）.可能需要在其服务注册元数据中配置服务的某些其他属性，以便客户端可以正确连接.不提供有关传输层安全性详细信息的服务注册表需要提供 `scheme` 元数据条目，以便将其设置为 `https` 或 `http` .如果未配置任何方案且服务未作为安全服务公开，则配置默认为 `spring.cloud.vault.scheme` ，如果未设置，则为 `https` .

```java
spring.cloud.vault.discovery:
enabled: true
service-id: my-vault-service
```

