## 32.缓存

Spring Framework支持透明地向应用程序添加缓存.从本质上讲，抽象将缓存应用于方法，从而根据缓存中可用的信息减少执行次数.缓存逻辑应用透明，不会对调用者造成任何干扰.只要通过 `@EnableCaching` 注释启用了缓存支持，Spring Boot就会自动配置缓存基础结构.

> 查看Spring Framework参考的[relevant section](https://docs.spring.io/spring/docs/5.1.2.RELEASE/spring-framework-reference/integration.html#cache)以获取更多详细信息.

简而言之，将缓存添加到服务操作就像在其方法中添加相关注释一样简单，如以下示例所示：

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

此示例演示了如何在可能代价高昂的操作上使用缓存.在调用 `computePiDecimal` 之前，抽象会在 `piDecimals` 缓存中查找与 `i` 参数匹配的条目.如果找到条目，则缓存中的内容会立即返回给调用者，并且不会调用该方法.否则，将调用该方法，并在返回值之前更新缓存.

> 您还可以透明地使用标准JSR-107（JCache）注释（例如 `@CacheResult` ）.但是，我们强烈建议您不要混合使用Spring Cache和JCache注释.

如果您不添加任何特定的缓存库，Spring Boot会自动配置在内存中使用并发映射的[simple provider](boot-features-caching.html#boot-features-caching-provider-simple).当需要缓存时（例如前面示例中的 `piDecimals` ），此提供程序会为您创建缓存.简单的提供程序并不是真正推荐用于生产环境用途，但它非常适合入门并确保您了解这些功能.当您决定使用缓存提供程序时，请务必阅读其文档以了解如何配置应用程序使用的缓存.几乎所有提供程序都要求您显式配置在应用程序中使用的每个缓存.有些提供了一种自定义 `spring.cache.cache-names` 属性定义的默认缓存的方法.

> 也可以透明地从缓存中[update](https://docs.spring.io/spring/docs/5.1.2.RELEASE/spring-framework-reference/integration.html#cache-annotations-put)或[evict](https://docs.spring.io/spring/docs/5.1.2.RELEASE/spring-framework-reference/integration.html#cache-annotations-evict)数据.

## 32.1支持的缓存提供程序

缓存抽象不提供实际存储，并依赖于 `org.springframework.cache.Cache` 和 `org.springframework.cache.CacheManager` 接口实现的抽象.

如果您尚未定义 `CacheManager` 类型的bean或名为 `cacheResolver` 的 `CacheResolver` （请参阅[CachingConfigurer](https://docs.spring.io/spring/docs/5.1.2.RELEASE/javadoc-api/org/springframework/cache/annotation/CachingConfigurer.html)），则Spring Boot会尝试检测以下提供程序（按指示的顺序）：

Generic JCache（JSR-107）（EhCache 3，Hazelcast，Infinispan等）EhCache 2.x Hazelcast Infinispan Couchbase RedisCaffeine简单

> 也可以通过设置 `spring.cache.type` 属性强制特定缓存提供程序.如果在特定环境（例如测试）中需要[disable caching altogether](boot-features-caching.html#boot-features-caching-provider-none)，请使用此属性.

> 使用 `spring-boot-starter-cache` “Starter”快速添加基本缓存依赖项.起动器引入 `spring-context-support` .如果手动添加依赖项，则必须包含 `spring-context-support` 才能使用JCache，EhCache 2.x或Guava支持.

如果Spring引导自动配置 `CacheManager` ，则可以通过公开实现 `CacheManagerCustomizer` 接口的bean，在完全初始化之前进一步调整其配置.以下示例设置一个标志，表示应将null值传递给底层映射：

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

> 在前面的示例中，预计会自动配置 `ConcurrentMapCacheManager` .如果不是这种情况（您提供了自己的配置或自动配置了不同的缓存提供程序），则根本不会调用自定义程序.您可以拥有任意数量的自定义程序，也可以使用 `@Order` 或 `Ordered` 订购它们.

### 32.1.1通用

如果上下文定义至少一个 `org.springframework.cache.Cache`  bean，则使用通用缓存.将创建包含该类型所有bean的 `CacheManager` .

### 32.1.2 JCache（JSR-107）

[JCache](https://jcp.org/en/jsr/detail?id=107)通过类路径上的 `javax.cache.spi.CachingProvider` （即类路径上存在符合JSR-107的缓存库）进行引导， `JCacheCacheManager` 由 `spring-boot-starter-cache` “Starter”提供.可以使用各种兼容库，Spring Boot为Ehcache 3，Hazelcast和Infinispan提供依赖管理.还可以添加任何其他兼容库.

可能会发生多个提供商的存在，在这种情况下必须明确指定提供者.即使JSR-107标准没有强制执行定义配置文件位置的标准化方法，Spring Boot也会尽力满足设置缓存的实现细节，如下例所示：

```java
# Only necessary if more than one provider is present
spring.cache.jcache.provider=com.acme.MyCachingProvider
spring.cache.jcache.config=classpath:acme.xml
```

> 当缓存库同时提供本机实现和JSR-107支持时，Spring Boot更喜欢JSR-107支持，因此如果切换到不同的JSR-107实现，则可以使用相同的功能.

> Spring Boot有[general support for Hazelcast](boot-features-hazelcast.html).如果单个 `HazelcastInstance` 可用，它也会自动重用于 `CacheManager` ，除非指定了 `spring.cache.jcache.config` 属性.

有两种方法可以自定义底层 `javax.cache.cacheManager` ：

通过设置 `spring.cache.cache-names` 属性，可以在启动时创建
- Caches.如果定义了自定义 `javax.cache.configuration.Configuration`  bean，则用于自定义它们.
使用 `CacheManager` 的引用调用
-  `org.springframework.boot.autoconfigure.cache.JCacheManagerCustomizer`  beans以进行完全自定义.

> 如果定义了标准 `javax.cache.CacheManager`  bean，它将自动包装在抽象所需的 `org.springframework.cache.CacheManager` 实现中.没有进一步的自定义.

### 32.1.3 EhCache 2.x

如果可以在类路径的根目录中找到名为 `ehcache.xml` 的文件，则使用[EhCache](http://www.ehcache.org/) 2.x.如果找到EhCache 2.x， `spring-boot-starter-cache` “Starter”提供的 `EhCacheCacheManager` 用于引导缓存管理器.还可以提供备用配置文件，如以下示例所示：

```java
spring.cache.ehcache.config=classpath:config/another-config.xml
```

### 32.1.4 Hazelcast

Spring Boot有[general support for Hazelcast](boot-features-hazelcast.html).如果已自动配置 `HazelcastInstance` ，则会自动将其包装在 `CacheManager` 中.

### 32.1.5 Infinispan

[Infinispan](http://infinispan.org/)没有默认配置文件位置，因此必须明确指定.否则，使用默认引导程序.

```java
spring.cache.infinispan.config=infinispan.xml
```

可以通过设置 `spring.cache.cache-names` 属性在启动时创建缓存.如果定义了自定义 `ConfigurationBuilder`  bean，则它用于自定义缓存.

>  Infinispan在Spring Boot中的支持仅限于嵌入式模式，非常基础.如果你想要更多选项，你应该使用官方的Infinispan Spring Boot启动器.有关详细信息，请参阅[Infinispan’s documentation](https://github.com/infinispan/infinispan-spring-boot).

### 32.1.6 Couchbase

如果[Couchbase](https://www.couchbase.com/) Java客户端和 `couchbase-spring-cache` 实现可用且Couchbase为[configured](boot-features-nosql.html#boot-features-couchbase)，则自动配置 `CouchbaseCacheManager` .也可以通过设置 `spring.cache.cache-names` 属性在启动时创建其他缓存.这些缓存在自动配置的 `Bucket` 上运行.您还可以使用自定义程序在另一个 `Bucket` 上创建其他缓存.假设在"main" `Bucket` 和一个（ `cache3` ）缓存上需要两个缓存（ `cache1` 和 `cache2` ），并且“另一个” `Bucket` 上的自定义时间为2秒.您可以通过配置创建前两个缓存，如下所示：

```java
spring.cache.cache-names=cache1,cache2
```

然后，您可以定义 `@Configuration` 类来配置额外的 `Bucket` 和 `cache3` 缓存，如下所示：

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

此示例配置重用通过自动配置创建的 `Cluster` .

### 32.1.7 Redis

如果[Redis](http://redis.io/)可用并已配置，则会自动配置 `RedisCacheManager` .通过设置 `spring.cache.cache-names` 属性可以在启动时创建其他缓存，并且可以使用 `spring.cache.redis.*` 属性配置缓存默认值.例如，以下配置创建 `cache1` 和 `cache2` 缓存，其生存时间为10分钟：

```java
spring.cache.cache-names=cache1,cache2
spring.cache.redis.time-to-live=600000
```

> By默认情况下，添加了一个键前缀，这样，如果两个单独的缓存使用相同的键，则Redis没有重叠键，也不能返回无效值.如果您创建自己的 `RedisCacheManager` ，我们强烈建议您启用此设置.

> 您可以通过添加自己的 `RedisCacheConfiguration`   `@Bean` 来完全控制配置.如果您正在寻找自定义序列化策略，这可能很有用.

### 32.1.8Caffeine

[Caffeine](https://github.com/ben-manes/caffeine)是Guava缓存的Java 8重写，取代了对Guava的支持.如果存在Caffeine，则会自动配置 `CaffeineCacheManager` （由 `spring-boot-starter-cache` “Starter”提供）.可以通过设置 `spring.cache.cache-names` 属性在启动时创建缓存，并且可以通过以下之一（按指示的顺序）自定义缓存：

由spring.cache.caffeine.spec定义的缓存规范定义了com.github.benmanes.caffeine.cache.CaffeineSpec bean定义了com.github.benmanes.caffeine.cache.Caffeine bean

例如，以下配置创建 `cache1` 和 `cache2` 缓存，最大大小为500，生存时间为10分钟

```java
spring.cache.cache-names=cache1,cache2
spring.cache.caffeine.spec=maximumSize=500,expireAfterAccess=600s
```

如果定义了 `com.github.benmanes.caffeine.cache.CacheLoader`  bean，它将自动关联到 `CaffeineCacheManager` .由于 `CacheLoader` 将与缓存管理器管理的所有缓存相关联，因此必须将其定义为 `CacheLoader<Object, Object>` .自动配置忽略任何其他泛型类型.

### 32.1.9简单

如果找不到其他提供程序，则配置使用 `ConcurrentHashMap` 作为缓存存储的简单实现.如果您的应用程序中没有缓存库，则这是默认值.默认情况下，会根据需要创建缓存，但您可以通过设置 `cache-names` 属性来限制可用缓存列表.例如，如果只需要 `cache1` 和 `cache2` 缓存，请将 `cache-names` 属性设置为如下：

```java
spring.cache.cache-names=cache1,cache2
```

如果这样做并且您的应用程序使用未列出的缓存，则在需要缓存时它会在运行时失败，但在启动时则不会.这类似于“真实”缓存提供程序在使用未声明的缓存时的行为方式.

### 32.1.10无

当配置中存在 `@EnableCaching` 时，也会出现合适的缓存配置.如果需要在某些环境中完全禁用缓存，请强制缓存类型为 `none` 以使用no-op实现，如以下示例所示：

```java
spring.cache.type=none
```

