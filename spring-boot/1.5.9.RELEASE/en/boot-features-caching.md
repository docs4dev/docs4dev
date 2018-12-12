## 32. Caching

The Spring Framework provides support for transparently adding caching to an application. At its core, the abstraction applies caching to methods, thus reducing the number of executions based on the information available in the cache. The caching logic is applied transparently, without any interference to the invoker. Spring Boot auto-configures the cache infrastructure as long as caching support is enabled via the  `@EnableCaching`  annotation.

> Check the [relevant section](https://docs.spring.io/spring/docs/5.1.2.RELEASE/spring-framework-reference/integration.html#cache) of the Spring Framework reference for more details.

In a nutshell, adding caching to an operation of your service is as easy as adding the relevant annotation to its method, as shown in the following example:

```java
import org.springframework.cache.annotation.Cacheable;
import org.springframework.stereotype.Component;

@Component
public class MathService {

	@Cacheable("piDecimals")
	public int computePiDecimal(int i) {
		// ...
	}

}
```

This example demonstrates the use of caching on a potentially costly operation. Before invoking  `computePiDecimal` , the abstraction looks for an entry in the  `piDecimals`  cache that matches the  `i`  argument. If an entry is found, the content in the cache is immediately returned to the caller, and the method is not invoked. Otherwise, the method is invoked, and the cache is updated before returning the value.

> You can also use the standard JSR-107 (JCache) annotations (such as  `@CacheResult` ) transparently. However, we strongly advise you to not mix and match the Spring Cache and JCache annotations.

If you do not add any specific cache library, Spring Boot auto-configures a [simple provider](boot-features-caching.html#boot-features-caching-provider-simple) that uses concurrent maps in memory. When a cache is required (such as  `piDecimals`  in the preceding example), this provider creates it for you. The simple provider is not really recommended for production usage, but it is great for getting started and making sure that you understand the features. When you have made up your mind about the cache provider to use, please make sure to read its documentation to figure out how to configure the caches that your application uses. Nearly all providers require you to explicitly configure every cache that you use in the application. Some offer a way to customize the default caches defined by the  `spring.cache.cache-names`  property.

> It is also possible to transparently [update](https://docs.spring.io/spring/docs/5.1.2.RELEASE/spring-framework-reference/integration.html#cache-annotations-put) or [evict](https://docs.spring.io/spring/docs/5.1.2.RELEASE/spring-framework-reference/integration.html#cache-annotations-evict) data from the cache.

## 32.1 Supported Cache Providers

The cache abstraction does not provide an actual store and relies on abstraction materialized by the  `org.springframework.cache.Cache`  and  `org.springframework.cache.CacheManager`  interfaces.

If you have not defined a bean of type  `CacheManager`  or a  `CacheResolver`  named  `cacheResolver`  (see [CachingConfigurer](https://docs.spring.io/spring/docs/5.1.2.RELEASE/javadoc-api/org/springframework/cache/annotation/CachingConfigurer.html)), Spring Boot tries to detect the following providers (in the indicated order):

Generic JCache (JSR-107) (EhCache 3, Hazelcast, Infinispan, and others) EhCache 2.x Hazelcast Infinispan Couchbase Redis Caffeine Simple

> It is also possible to force a particular cache provider by setting the  `spring.cache.type`  property. Use this property if you need to [disable caching altogether](boot-features-caching.html#boot-features-caching-provider-none) in certain environment (such as tests).

> Use the  `spring-boot-starter-cache`  “Starter” to quickly add basic caching dependencies. The starter brings in  `spring-context-support` . If you add dependencies manually, you must include  `spring-context-support`  in order to use the JCache, EhCache 2.x, or Guava support.

If the  `CacheManager`  is auto-configured by Spring Boot, you can further tune its configuration before it is fully initialized by exposing a bean that implements the  `CacheManagerCustomizer`  interface. The following example sets a flag to say that null values should be passed down to the underlying map:

```java
@Bean
public CacheManagerCustomizer<ConcurrentMapCacheManager> cacheManagerCustomizer() {
	return new CacheManagerCustomizer<ConcurrentMapCacheManager>() {
		@Override
		public void customize(ConcurrentMapCacheManager cacheManager) {
			cacheManager.setAllowNullValues(false);
		}
	};
}
```

> In the preceding example, an auto-configured  `ConcurrentMapCacheManager`  is expected. If that is not the case (either you provided your own config or a different cache provider was auto-configured), the customizer is not invoked at all. You can have as many customizers as you want, and you can also order them by using  `@Order`  or  `Ordered` .

### 32.1.1 Generic

Generic caching is used if the context defines at least one  `org.springframework.cache.Cache`  bean. A  `CacheManager`  wrapping all beans of that type is created.

### 32.1.2 JCache (JSR-107)

[JCache](https://jcp.org/en/jsr/detail?id=107) is bootstrapped through the presence of a  `javax.cache.spi.CachingProvider`  on the classpath (that is, a JSR-107 compliant caching library exists on the classpath), and the  `JCacheCacheManager`  is provided by the  `spring-boot-starter-cache`  “Starter”. Various compliant libraries are available, and Spring Boot provides dependency management for Ehcache 3, Hazelcast, and Infinispan. Any other compliant library can be added as well.

It might happen that more than one provider is present, in which case the provider must be explicitly specified. Even if the JSR-107 standard does not enforce a standardized way to define the location of the configuration file, Spring Boot does its best to accommodate setting a cache with implementation details, as shown in the following example:

```java
# Only necessary if more than one provider is present
spring.cache.jcache.provider=com.acme.MyCachingProvider
spring.cache.jcache.config=classpath:acme.xml
```

> When a cache library offers both a native implementation and JSR-107 support, Spring Boot prefers the JSR-107 support, so that the same features are available if you switch to a different JSR-107 implementation.

> Spring Boot has [general support for Hazelcast](boot-features-hazelcast.html). If a single  `HazelcastInstance`  is available, it is automatically reused for the  `CacheManager`  as well, unless the  `spring.cache.jcache.config`  property is specified.

There are two ways to customize the underlying  `javax.cache.cacheManager` :

- Caches can be created on startup by setting the  `spring.cache.cache-names`  property. If a custom  `javax.cache.configuration.Configuration`  bean is defined, it is used to customize them.

-  `org.springframework.boot.autoconfigure.cache.JCacheManagerCustomizer`  beans are invoked with the reference of the  `CacheManager`  for full customization.

> If a standard  `javax.cache.CacheManager`  bean is defined, it is wrapped automatically in an  `org.springframework.cache.CacheManager`  implementation that the abstraction expects. No further customization is applied to it.

### 32.1.3 EhCache 2.x

[EhCache](http://www.ehcache.org/) 2.x is used if a file named  `ehcache.xml`  can be found at the root of the classpath. If EhCache 2.x is found, the  `EhCacheCacheManager`  provided by the  `spring-boot-starter-cache`  “Starter” is used to bootstrap the cache manager. An alternate configuration file can be provided as well, as shown in the following example:

```java
spring.cache.ehcache.config=classpath:config/another-config.xml
```

### 32.1.4 Hazelcast

Spring Boot has [general support for Hazelcast](boot-features-hazelcast.html). If a  `HazelcastInstance`  has been auto-configured, it is automatically wrapped in a  `CacheManager` .

### 32.1.5 Infinispan

[Infinispan](http://infinispan.org/) has no default configuration file location, so it must be specified explicitly. Otherwise, the default bootstrap is used.

```java
spring.cache.infinispan.config=infinispan.xml
```

Caches can be created on startup by setting the  `spring.cache.cache-names`  property. If a custom  `ConfigurationBuilder`  bean is defined, it is used to customize the caches.

> The support of Infinispan in Spring Boot is restricted to the embedded mode and is quite basic. If you want more options, you should use the official Infinispan Spring Boot starter instead. See [Infinispan’s documentation](https://github.com/infinispan/infinispan-spring-boot) for more details.

### 32.1.6 Couchbase

If the [Couchbase](https://www.couchbase.com/) Java client and the  `couchbase-spring-cache`  implementation are available and Couchbase is [configured](boot-features-nosql.html#boot-features-couchbase), a  `CouchbaseCacheManager`  is auto-configured. It is also possible to create additional caches on startup by setting the  `spring.cache.cache-names`  property. These caches operate on the  `Bucket`  that was auto-configured. You can also create additional caches on another  `Bucket`  by using the customizer. Assume you need two caches ( `cache1`  and  `cache2` ) on the "main"  `Bucket`  and one ( `cache3` ) cache with a custom time to live of 2 seconds on the “another”  `Bucket` . You can create the first two caches through configuration, as follows:

```java
spring.cache.cache-names=cache1,cache2
```

Then you can define a  `@Configuration`  class to configure the extra  `Bucket`  and the  `cache3`  cache, as follows:

```java
@Configuration
public class CouchbaseCacheConfiguration {

	private final Cluster cluster;

	public CouchbaseCacheConfiguration(Cluster cluster) {
		this.cluster = cluster;
	}

	@Bean
	public Bucket anotherBucket() {
		return this.cluster.openBucket("another", "secret");
	}

	@Bean
	public CacheManagerCustomizer<CouchbaseCacheManager> cacheManagerCustomizer() {
		return c -> {
			c.prepareCache("cache3", CacheBuilder.newInstance(anotherBucket())
					.withExpiration(2));
		};
	}

}
```

This sample configuration reuses the  `Cluster`  that was created through auto-configuration.

### 32.1.7 Redis

If [Redis](http://redis.io/) is available and configured, a  `RedisCacheManager`  is auto-configured. It is possible to create additional caches on startup by setting the  `spring.cache.cache-names`  property and cache defaults can be configured by using  `spring.cache.redis.*`  properties. For instance, the following configuration creates  `cache1`  and  `cache2`  caches with a time to live of 10 minutes:

```java
spring.cache.cache-names=cache1,cache2
spring.cache.redis.time-to-live=600000
```

> By default, a key prefix is added so that, if two separate caches use the same key, Redis does not have overlapping keys and cannot return invalid values. We strongly recommend keeping this setting enabled if you create your own  `RedisCacheManager` .

> You can take full control of the configuration by adding a  `RedisCacheConfiguration`   `@Bean`  of your own. This can be useful if you’re looking for customizing the serialization strategy.

### 32.1.8 Caffeine

[Caffeine](https://github.com/ben-manes/caffeine) is a Java 8 rewrite of Guava’s cache that supersedes support for Guava. If Caffeine is present, a  `CaffeineCacheManager`  (provided by the  `spring-boot-starter-cache`  “Starter”) is auto-configured. Caches can be created on startup by setting the  `spring.cache.cache-names`  property and can be customized by one of the following (in the indicated order):

A cache spec defined by spring.cache.caffeine.spec A com.github.benmanes.caffeine.cache.CaffeineSpec bean is defined A com.github.benmanes.caffeine.cache.Caffeine bean is defined

For instance, the following configuration creates  `cache1`  and  `cache2`  caches with a maximum size of 500 and a time to live of 10 minutes

```java
spring.cache.cache-names=cache1,cache2
spring.cache.caffeine.spec=maximumSize=500,expireAfterAccess=600s
```

If a  `com.github.benmanes.caffeine.cache.CacheLoader`  bean is defined, it is automatically associated to the  `CaffeineCacheManager` . Since the  `CacheLoader`  is going to be associated with all caches managed by the cache manager, it must be defined as  `CacheLoader<Object, Object>` . The auto-configuration ignores any other generic type.

### 32.1.9 Simple

If none of the other providers can be found, a simple implementation using a  `ConcurrentHashMap`  as the cache store is configured. This is the default if no caching library is present in your application. By default, caches are created as needed, but you can restrict the list of available caches by setting the  `cache-names`  property. For instance, if you want only  `cache1`  and  `cache2`  caches, set the  `cache-names`  property as follows:

```java
spring.cache.cache-names=cache1,cache2
```

If you do so and your application uses a cache not listed, then it fails at runtime when the cache is needed, but not on startup. This is similar to the way the "real" cache providers behave if you use an undeclared cache.

### 32.1.10 None

When  `@EnableCaching`  is present in your configuration, a suitable cache configuration is expected as well. If you need to disable caching altogether in certain environments, force the cache type to  `none`  to use a no-op implementation, as shown in the following example:

```java
spring.cache.type=none
```

