## 39. Hazelcast

如果[Hazelcast](https://hazelcast.com/)位于类路径上并找到合适的配置，则Spring Boot会自动配置您可以在应用程序中注入的 `HazelcastInstance` .

如果定义 `com.hazelcast.config.Config`  bean，Spring Boot会使用它.如果您的配置定义了实例名称，Spring Boot会尝试查找现有实例而不是创造一个新的.

您还可以指定要通过配置使用的 `hazelcast.xml` 配置文件，如以下示例所示：

```java
spring.hazelcast.config=classpath:config/my-hazelcast.xml
```

否则，Spring Boot会尝试从默认位置找到Hazelcast配置：工作目录中的 `hazelcast.xml` 或类路径的根目录.我们还检查是否设置了 `hazelcast.config` 系统属性.有关详细信息，请参阅[Hazelcast documentation](http://docs.hazelcast.org/docs/latest/manual/html-single/).

如果类路径中存在 `hazelcast-client` ，则Spring Boot首先尝试通过检查以下配置选项来创建客户端：

- 存在 `com.hazelcast.client.config.ClientConfig`  bean.

- 由 `spring.hazelcast.config` 属性定义的配置文件.

-   `hazelcast.client.config` 系统属性的存在.

- A  `hazelcast-client.xml` 在工作目录中或类路径的根目录下.

> Spring Boot也有[explicit caching support for Hazelcast](boot-features-caching.html#boot-features-caching-provider-hazelcast).如果启用了缓存， `HazelcastInstance` 将自动包装在 `CacheManager` 实现中.

