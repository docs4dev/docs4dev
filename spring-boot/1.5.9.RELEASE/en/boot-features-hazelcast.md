## 39. Hazelcast

If [Hazelcast](https://hazelcast.com/) is on the classpath and a suitable configuration is found, Spring Boot auto-configures a  `HazelcastInstance`  that you can inject in your application.

If you define a  `com.hazelcast.config.Config`  bean, Spring Boot uses that. If your configuration defines an instance name, Spring Boot tries to locate an existing instance rather than creating a new one.

You could also specify the  `hazelcast.xml`  configuration file to use through configuration, as shown in the following example:

```java
spring.hazelcast.config=classpath:config/my-hazelcast.xml
```

Otherwise, Spring Boot tries to find the Hazelcast configuration from the default locations:  `hazelcast.xml`  in the working directory or at the root of the classpath. We also check if the  `hazelcast.config`  system property is set. See the [Hazelcast documentation](http://docs.hazelcast.org/docs/latest/manual/html-single/) for more details.

If  `hazelcast-client`  is present on the classpath, Spring Boot first attempts to create a client by checking the following configuration options:

- The presence of a  `com.hazelcast.client.config.ClientConfig`  bean.

- A configuration file defined by the  `spring.hazelcast.config`  property.

- The presence of the  `hazelcast.client.config`  system property.

- A  `hazelcast-client.xml`  in the working directory or at the root of the classpath.

> Spring Boot also has [explicit caching support for Hazelcast](boot-features-caching.html#boot-features-caching-provider-hazelcast). If caching is enabled, the  `HazelcastInstance`  is automatically wrapped in a  `CacheManager`  implementation.

