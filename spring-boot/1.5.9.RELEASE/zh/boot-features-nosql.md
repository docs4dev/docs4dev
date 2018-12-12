## 31.使用NoSQL Technologies

Spring Data提供了其他项目，可帮助您访问各种NoSQL技术，包括：[MongoDB](https://projects.spring.io/spring-data-mongodb/)，_ [Neo4J](https://projects.spring.io/spring-data-neo4j/)，[Elasticsearch](https://github.com/spring-projects/spring-data-elasticsearch/)，[Solr](https://projects.spring.io/spring-data-solr/)，[Redis](https://projects.spring.io/spring-data-redis/)，[Gemfire](https://projects.spring.io/spring-data-gemfire/)，[Cassandra](https://projects.spring.io/spring-data-cassandra/)，[Couchbase](https://projects.spring.io/spring-data-couchbase/)和[LDAP](https://projects.spring.io/spring-data-ldap/). Spring Boot为Redis，MongoDB，Neo4j，Elasticsearch，Solr Cassandra，Couchbase和LDAP提供自动配置.您可以使用其他项目，但必须自己配置它们.请参阅[projects.spring.io/spring-data](https://projects.spring.io/spring-data)处的相应参考文档.

## 31.1 Redis

[Redis](http://redis.io/)是一个缓存，消息代理和功能丰富的键值存储. Spring Boot为[Lettuce](https://github.com/lettuce-io/lettuce-core/)和[Jedis](https://github.com/xetorthio/jedis/)客户端库提供了基本的自动配置，并为[Spring Data Redis](https://github.com/spring-projects/spring-data-redis)提供了它们之上的抽象.

有一个 `spring-boot-starter-data-redis` “Starter”用于以方便的方式收集依赖项.默认情况下，它使用[Lettuce](https://github.com/lettuce-io/lettuce-core/).该启动器处理传统和反应应用程序.

> we还提供 `spring-boot-starter-data-redis-reactive` “Starter”以与具有反应支持的其他商店保持一致.

### 31.1.1正在连接到Redis

您可以像任何其他Spring Bean一样注入自动配置的 `RedisConnectionFactory` ， `StringRedisTemplate` 或vanilla  `RedisTemplate` 实例.默认情况下，实例尝试连接到位于 `localhost:6379` 的Redis服务器.以下清单显示了这样一个bean的示例：

```java
@Component
public class MyBean {

	private StringRedisTemplate template;

	@Autowired
	public MyBean(StringRedisTemplate template) {
		this.template = template;
	}

	// ...

}
```

> 您还可以注册实现 `LettuceClientConfigurationBuilderCustomizer` 的任意数量的bean，以进行更高级的自定义.如果您使用Jedis，也可以使用 `JedisClientConfigurationBuilderCustomizer` .

如果您添加自己配置的任何类型的 `@Bean` ，它将替换默认值（除非在 `RedisTemplate` 的情况下，当排除基于bean名称时， `redisTemplate` ，而不是其类型）.默认情况下，如果 `commons-pool2` 在类路径上，则会获得池化连接工厂.

## 31.2 MongoDB

[MongoDB](https://www.mongodb.com/)是一个开源的NoSQL文档数据库，它使用类似JSON的模式而不是传统的基于表的关系数据. Spring Boot提供了一些使用MongoDB的便利，包括 `spring-boot-starter-data-mongodb` 和 `spring-boot-starter-data-mongodb-reactive` “Starters”.

### 31.2.1连接到MongoDB数据库

要访问Mongo数据库，可以注入自动配置的 `org.springframework.data.mongodb.MongoDbFactory` .默认情况下，实例尝试连接到 `mongodb://localhost/test` 处的MongoDB服务器.以下示例显示如何连接到MongoDB数据库：

```java
import org.springframework.data.mongodb.MongoDbFactory;
import com.mongodb.DB;

@Component
public class MyBean {

	private final MongoDbFactory mongo;

	@Autowired
	public MyBean(MongoDbFactory mongo) {
		this.mongo = mongo;
	}

	// ...

	public void example() {
		DB db = mongo.getDb();
		// ...
	}

}
```

您可以设置 `spring.data.mongodb.uri` 属性以更改URL并配置其他设置，例如副本集，如以下示例所示：

```java
spring.data.mongodb.uri=mongodb://user:[emailprotected]:12345,mongo2.example.com:23456/test
```

或者，只要您使用Mongo 2.x，就可以指定 `host`  /  `port` .例如，您可以在 `application.properties` 中声明以下设置：

```java
spring.data.mongodb.host=mongoserver
spring.data.mongodb.port=27017
```

如果您已经定义了自己的 `MongoClient` ，它将用于自动配置合适的 `MongoDbFactory` .支持 `com.mongodb.MongoClient` 和 `com.mongodb.client.MongoClient` .

> 如果您使用Mongo 3.0 Java驱动程序， `spring.data.mongodb.host` 和 `spring.data.mongodb.port` 不受支持.在这种情况下，应使用 `spring.data.mongodb.uri` 来提供所有配置.

> 如果未指定 `spring.data.mongodb.port` ，则使用默认值 `27017` .您可以从前面显示的示例中删除此行.

> 如果你不使用Spring Data Mongo，你可以注入 `com.mongodb.MongoClient`  beans而不是 `MongoDbFactory` .如果要完全控制BuildMongoDB连接，还可以声明自己的 `MongoDbFactory` 或 `MongoClient`  bean.

> 如果您使用的是被动驱动程序，则SSL需要Netty.如果Netty可用且自己的工厂尚未自定义，则自动配置会自动配置此工厂.

### 31.2.2 MongoTemplate

[Spring Data MongoDB](https://projects.spring.io/spring-data-mongodb/)提供了一个[MongoTemplate](https://docs.spring.io/spring-data/mongodb/docs/current/api/org/springframework/data/mongodb/core/MongoTemplate.html)类，其设计与Spring的 `JdbcTemplate` 非常相似.与 `JdbcTemplate` 一样，Spring Boot会自动配置一个bean来注入模板，如下所示：

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.mongodb.core.MongoTemplate;
import org.springframework.stereotype.Component;

@Component
public class MyBean {

	private final MongoTemplate mongoTemplate;

	@Autowired
	public MyBean(MongoTemplate mongoTemplate) {
		this.mongoTemplate = mongoTemplate;
	}

	// ...

}
```

有关完整的详细信息，请参见[MongoOperations Javadoc](https://docs.spring.io/spring-data/mongodb/docs/current/api/org/springframework/data/mongodb/core/MongoOperations.html).

### 31.2.3 Spring Data MongoDB存储库

Spring Data包括MongoDB的存储库支持.与前面讨论的JPA存储库一样，基本原则是基于方法名称自动构造查询.

事实上，Spring Data JPA和Spring Data MongoDB共享相同的通用基础架构.您可以从前面获取JPA示例，假设 `City` 现在是Mongo数据类而不是JPA  `@Entity` ，它的工作方式相同，如下例所示：

```java
package com.example.myapp.domain;

import org.springframework.data.domain.*;
import org.springframework.data.repository.*;

public interface CityRepository extends Repository<City, Long> {

	Page<City> findAll(Pageable pageable);

	City findByNameAndStateAllIgnoringCase(String name, String state);

}
```

> 您可以使用 `@EntityScan` 注释自定义文档扫描位置.

> 有关Spring Data MongoDB的完整详细信息，包括其丰富的对象映射技术，请参阅其[reference documentation](https://projects.spring.io/spring-data-mongodb/).

### 31.2.4嵌入式Mongo

Spring Boot为[Embedded Mongo](https://github.com/flapdoodle-oss/de.flapdoodle.embed.mongo)提供自动配置.要在Spring Boot应用程序中使用它，请在 `de.flapdoodle.embed:de.flapdoodle.embed.mongo` 上添加依赖项.

可以通过设置 `spring.data.mongodb.port` 属性来配置Mongo侦听的端口.要使用随机分配的空闲端口，请使用值0.  `MongoAutoConfiguration` 创建的 `MongoClient` 将自动配置为使用随机分配的端口.

> 如果未配置自定义端口，则默认情况下，嵌入式支持使用随机端口（而不是27017）.

如果类路径上有SLF4J，则Mongo生成的输出将自动路由到名为 `org.springframework.boot.autoconfigure.mongo.embedded.EmbeddedMongo` 的Logger.

您可以声明自己的 `IMongodConfig` 和 `IRuntimeConfig`  bean来控制Mongo实例的配置和日志记录路由.

## 31.3 Neo4j

[Neo4j](http://neo4j.com/)是一个开源的NoSQL图形数据库，它使用由一级关系连接的节点的丰富数据模型，这比传统的RDBMS方法更适合连接的大数据. Spring Boot为使用Neo4j提供了一些便利，包括 `spring-boot-starter-data-neo4j` “Starter”.

### 31.3.1连接到Neo4j数据库

要访问Neo4j服务器，您可以注入自动配置的 `org.neo4j.ogm.session.Session` .默认情况下，实例尝试使用Bolt协议连接到 `localhost:7687` 的Neo4j服务器.以下示例显示了如何注入Neo4j  `Session` ：

```java
@Component
public class MyBean {

	private final Session session;

	@Autowired
	public MyBean(Session session) {
		this.session = session;
	}

	// ...

}
```

您可以通过设置 `spring.data.neo4j.*` 属性来配置要使用的URI和凭据，如以下示例所示：

```java
spring.data.neo4j.uri=bolt://my-server:7687
spring.data.neo4j.username=neo4j
spring.data.neo4j.password=secret
```

您可以通过添加 `org.neo4j.ogm.config.Configuration`   `@Bean` 来完全控制会话创建.此外，添加 `@Bean` 类型 `SessionFactory` 会禁用自动配置并为您提供完全控制.

### 31.3.2使用嵌入模式

如果将 `org.neo4j:neo4j-ogm-embedded-driver` 添加到应用程序的依赖项中，Spring Boot会自动配置Neo4j的进程内嵌入式实例，该应用程序在应用程序关闭时不会保留任何数据.

> 由于嵌入式Neo4j OGM驱动程序本身不提供Neo4j内核，因此您必须自己声明 `org.neo4j:neo4j` 为依赖项.有关兼容版本的列表，请参阅[the Neo4j OGM documentation](https://neo4j.com/docs/ogm-manual/current/reference/#reference:getting-started).

当类路径上有多个驱动程序时，嵌入式驱动程序优先于其他驱动程序.您可以通过设置 `spring.data.neo4j.embedded.enabled=false` 显式禁用嵌入模式.

如果嵌入式驱动程序和Neo4j内核在类路径上，[Data Neo4j Tests](boot-features-testing.html#boot-features-testing-spring-boot-applications-testing-autoconfigured-neo4j-test)会自动使用嵌入式Neo4j实例，如上所述.

> 您可以通过提供配置中数据库文件的路径来为嵌入模式启用持久性，例如：  `spring.data.neo4j.uri=file://var/tmp/graph.db` .

### 31.3.3 Neo4jSession

默认情况下，如果您正在运行Web应用程序，则会话将绑定到线程以进行整个请求处理（即，它使用"Open Session in View"模式）.如果您不想要此行为，请将以下行添加到 `application.properties` 文件中：

```java
spring.data.neo4j.open-in-view=false
```

### 31.3.4 Spring Data Neo4j存储库

Spring Data包含对Neo4j的存储库支持.

Spring Data Neo4j与Spring Data JPA共享通用基础架构，就像许多其他Spring Data模块一样.您可以从之前的JPA示例中将 `City` 定义为Neo4j OGM  `@NodeEntity` 而不是JPA  `@Entity` ，并且存储库抽象以相同的方式工作，如以下示例所示：

```java
package com.example.myapp.domain;

import java.util.Optional;

import org.springframework.data.neo4j.repository.*;

public interface CityRepository extends Neo4jRepository<City, Long> {

	Optional<City> findOneByNameAndState(String name, String state);

}
```

`spring-boot-starter-data-neo4j` “Starter”启用存储库支持以及事务管理.您可以自定义位置以查找存储库和实体在 `@Configuration` -bean上分别使用 `@EnableNeo4jRepositories` 和 `@EntityScan` .

> 有关Spring Data Neo4j的完整详细信息，包括其对象映射技术，请参阅[reference documentation](https://projects.spring.io/spring-data-neo4j/).

## 31.4 Gemfire

[Spring Data Gemfire](https://github.com/spring-projects/spring-data-gemfire)为访问[Pivotal Gemfire](https://pivotal.io/big-data/pivotal-gemfire#details)数据管理平台提供了方便的Spring友好工具.有一个 `spring-boot-starter-data-gemfire` “Starter”用于以方便的方式收集依赖项.目前没有对Gemfire的自动配置支持，但您可以使用[single annotation: @EnableGemfireRepositories](https://github.com/spring-projects/spring-data-gemfire/blob/master/src/main/java/org/springframework/data/gemfire/repository/config/EnableGemfireRepositories.java)启用Spring Data Repositories.

## 31.5索尔

[Apache Solr](https://lucene.apache.org/solr/)是一个搜索引擎. Spring Boot为Solr 5客户端库提供了基本的自动配置，并为[Spring Data Solr](https://github.com/spring-projects/spring-data-solr)提供了它上面的抽象.有一个 `spring-boot-starter-data-solr` “Starter”用于以方便的方式收集依赖项.

### 31.5.1连接到Solr

您可以像任何其他Spring bean一样注入自动配置的 `SolrClient` 实例.默认情况下，实例尝试连接到 `localhost:8983/solr` 处的服务器.以下示例显示如何注入Solr bean：

```java
@Component
public class MyBean {

	private SolrClient solr;

	@Autowired
	public MyBean(SolrClient solr) {
		this.solr = solr;
	}

	// ...

}
```

如果您添加自己的 `@Bean` 类型 `SolrClient` ，它将替换默认值.

### 31.5.2 Spring Data Solr存储库

Spring Data包括Apache Solr的存储库支持.与前面讨论的JPA存储库一样，基本原则是根据方法名称自动构建查询.

事实上，Spring Data JPA和Spring Data Solr共享相同的通用基础架构.您可以从之前获取JPA示例，假设 `City` 现在是 `@SolrDocument` 类而不是JPA  `@Entity` ，它的工作方式相同.

> 有关Spring Data Solr的完整详细信息，请参阅[reference documentation](https://projects.spring.io/spring-data-solr/).

## 31.6 Elasticsearch

[Elasticsearch](https://www.elastic.co/products/elasticsearch)是一个开源，分布式，RESTful搜索和分析引擎. Spring Boot为Elasticsearch提供基本的自动配置.

Spring Boot支持多个HTTP客户端：

- 官方Java "Low Level"和"High Level" REST客户端

- [Jest](https://github.com/searchbox-io/Jest)

[Spring Data Elasticsearch](https://github.com/spring-projects/spring-data-elasticsearch)仍在使用传输客户端，您可以使用 `spring-boot-starter-data-elasticsearch` “Starter”开始使用它.

### 31.6.1 REST客户端连接到Elasticsearch

Elasticsearch发布了可用于查询集群的[two different REST clients](https://www.elastic.co/guide/en/elasticsearch/client/java-rest/current/index.html)："Low Level"客户端和"High Level"客户端.

如果你对类路径有 `org.elasticsearch.client:elasticsearch-rest-client` 依赖关系，Spring Boot将自动配置并注册 `RestClient`  bean，默认情况下为 `localhost:9200` .您可以进一步调整 `RestClient` 的配置方式，如以下示例所示：

```java
spring.elasticsearch.rest.uris=http://search.example.com:9200
spring.elasticsearch.rest.username=user
spring.elasticsearch.rest.password=secret
```

您还可以注册实现 `RestClientBuilderCustomizer` 的任意数量的bean以进行更高级的自定义.要完全控制注册，请定义 `RestClient`  bean.

如果你对类路径有 `org.elasticsearch.client:elasticsearch-rest-high-level-client` 依赖关系，Spring Boot将自动配置 `RestHighLevelClient` ，它包装任何现有的 `RestClient`  bean，重用其HTTP配置.

### 31.6.2使用Jest连接到Elasticsearch

如果类路径上有 `Jest` ，则可以注入一个自动配置的 `JestClient` ，默认情况下为 `localhost:9200` .您可以进一步调整客户端的配置方式，如以下示例所示：

```java
spring.elasticsearch.jest.uris=http://search.example.com:9200
spring.elasticsearch.jest.read-timeout=10000
spring.elasticsearch.jest.username=user
spring.elasticsearch.jest.password=secret
```

您还可以注册实现 `HttpClientConfigBuilderCustomizer` 的任意数量的bean以进行更高级的自定义.以下示例调整其他HTTP设置：

```java
static class HttpSettingsCustomizer implements HttpClientConfigBuilderCustomizer {

	@Override
	public void customize(HttpClientConfig.Builder builder) {
		builder.maxTotalConnection(100).defaultMaxTotalConnectionPerRoute(5);
	}

}
```

要完全控制注册，请定义 `JestClient`  bean.

### 31.6.3使用Spring Data连接到Elasticsearch

要连接到Elasticsearch，您必须提供一个或多个群集节点的地址.可以通过将 `spring.data.elasticsearch.cluster-nodes` 属性设置为以逗号分隔的 `host:port` 列表来指定地址.使用此配置，可以像任何其他Spring bean一样注入 `ElasticsearchTemplate` 或 `TransportClient` ，如以下示例所示：

```java
spring.data.elasticsearch.cluster-nodes=localhost:9300
```

```java
@Component
public class MyBean {

	private final ElasticsearchTemplate template;

	public MyBean(ElasticsearchTemplate template) {
		this.template = template;
	}

	// ...

}
```

如果添加自己的 `ElasticsearchTemplate` 或 `TransportClient`   `@Bean` ，则会替换默认值.

### 31.6.4 Spring Data Elasticsearch存储库

Spring Data包括对Elasticsearch的存储库支持.与前面讨论的JPA存储库一样，基本原则是根据方法名称自动为您构建查询.

事实上，Spring Data JPA和Spring Data Elasticsearch共享相同的通用基础架构.您可以从之前获取JPA示例，假设 `City` 现在是Elasticsearch  `@Document` 类而不是JPA  `@Entity` ，它的工作方式相同.

> 有关Spring Data Elasticsearch的完整详细信息，请参阅[reference documentation](https://docs.spring.io/spring-data/elasticsearch/docs/).

## 31.7卡珊德拉

[Cassandra](https://cassandra.apache.org/)是一个开源的分布式数据库管理系统，旨在处理许多商用服务器上的大量数据. Spring Boot为Cassandra提供了自动配置，并在[Spring Data Cassandra](https://github.com/spring-projects/spring-data-cassandra)提供了它上面的抽象.有一个 `spring-boot-starter-data-cassandra` “Starter”用于以方便的方式收集依赖项.

### 31.7.1连接到Cassandra

您可以像使用任何其他Spring Bean一样注入自动配置的 `CassandraTemplate` 或Cassandra  `Session` 实例.  `spring.data.cassandra.*` 属性可用于自定义连接.通常，您提供 `keyspace-name` 和 `contact-points` 属性，如以下示例所示：

```java
spring.data.cassandra.keyspace-name=mykeyspace
spring.data.cassandra.contact-points=cassandrahost1,cassandrahost2
```

您也可以注册任意数量的实现 `ClusterBuilderCustomizer` 的bean用于更高级的自定义.

以下代码清单显示了如何注入Cassandra bean：

```java
@Component
public class MyBean {

	private CassandraTemplate template;

	@Autowired
	public MyBean(CassandraTemplate template) {
		this.template = template;
	}

	// ...

}
```

如果您添加 `@Bean` 类型的 `@Bean` ，它将替换默认值.

### 31.7.2 Spring Data Cassandra存储库

Spring Data包含对Cassandra的基本存储库支持.目前，这比前面讨论的JPA存储库更有限，需要使用 `@Query` 注释finder方法.

> 有关Spring Data Cassandra的完整详细信息，请参阅[reference documentation](https://docs.spring.io/spring-data/cassandra/docs/).

## 31.8 Couchbase

[Couchbase](https://www.couchbase.com/)是一个开源的，分布式的，多模型NoSQL面向文档的数据库，针对交互式应用程序进行了优化. Spring Boot为Couchbase提供了自动配置，并在[Spring Data Couchbase](https://github.com/spring-projects/spring-data-couchbase)提供了它上面的抽象.有 `spring-boot-starter-data-couchbase` 和 `spring-boot-starter-data-couchbase-reactive` “Starters”用于以方便的方式收集依赖项.

### 31.8.1正在连接Couchbase

您可以通过添加Couchbase SDK和一些配置来获得 `Bucket` 和 `Cluster` .  `spring.couchbase.*` 属性可用于自定义连接.通常，您提供引导主机，存储桶名称和密码，如以下示例所示：

```java
spring.couchbase.bootstrap-hosts=my-host-1,192.168.1.123
spring.couchbase.bucket.name=my-bucket
spring.couchbase.bucket.password=secret
```

> 您需要至少提供引导主机，在这种情况下，存储桶名称为 `default` ，密码为空字符串.或者，您可以定义自己的 `org.springframework.data.couchbase.config.CouchbaseConfigurer`   `@Bean` 来控制整个配置.

也可以自定义某些 `CouchbaseEnvironment` 设置.例如，以下配置更改用于打开新 `Bucket` 的超时并启用SSL支持：

```java
spring.couchbase.env.timeouts.connect=3000
spring.couchbase.env.ssl.key-store=/location/of/keystore.jks
spring.couchbase.env.ssl.key-store-password=secret
```

检查 `spring.couchbase.env.*` 属性以获取更多详细信息.

### 31.8.2 Spring Data Couchbase存储库

Spring Data包括对Couchbase的存储库支持.有关Spring Data Couchbase的完整详细信息，请参阅[reference documentation](https://docs.spring.io/spring-data/couchbase/docs/current/reference/html/).

您可以像使用任何其他Spring Bean一样注入自动配置的 `CouchbaseTemplate` 实例，前提是默认 `CouchbaseConfigurer` 可用（当您启用Couchbase支持时会发生这种情况，如前所述）.

以下示例显示了如何注入Couchbase bean：

```java
@Component
public class MyBean {

	private final CouchbaseTemplate template;

	@Autowired
	public MyBean(CouchbaseTemplate template) {
		this.template = template;
	}

	// ...

}
```

您可以在自己的配置中定义一些bean来覆盖自动配置提供的bean：

- A  `CouchbaseTemplate`   `@Bean` ，名称为 `couchbaseTemplate` .

- An  `IndexManager`   `@Bean` ，名称为 `couchbaseIndexManager` .

- A  `CustomConversions`   `@Bean` ，名称为 `couchbaseCustomConversions` .

为避免在您自己的配置中对这些名称进行硬编码，您可以重用Spring Data Couchbase提供的 `BeanNames` .例如，您可以自定义要使用的转换器，如下所示：

```java
@Configuration
public class SomeConfiguration {

	@Bean(BeanNames.COUCHBASE_CUSTOM_CONVERSIONS)
	public CustomConversions myCustomConversions() {
		return new CustomConversions(...);
	}

	// ...

}
```

> 如果您想完全绕过Spring Data Couchbase的自动配置，请提供您自己的 `org.springframework.data.couchbase.config.AbstractCouchbaseDataConfiguration` 实现.

## 31.9 LDAP

[LDAP](https://en.wikipedia.org/wiki/Lightweight_Directory_Access_Protocol)（轻量级目录访问协议）是一种开放的，与供应商无关的行业标准应用程序协议，用于通过IP网络访问和维护分布式目录信息服务. Spring Boot为任何兼容的LDAP服务器提供自动配置，并支持来自[UnboundID](https://www.ldap.com/unboundid-ldap-sdk-for-java)的嵌入式内存中LDAP服务器.

LDAP抽象由[Spring Data LDAP](https://github.com/spring-projects/spring-data-ldap)提供.有一个 `spring-boot-starter-data-ldap` “Starter”用于以方便的方式收集依赖项.

### 31.9.1连接到LDAP服务器

要连接到LDAP服务器，请确保声明对 `spring-boot-starter-data-ldap` “Starter”或 `spring-ldap-core` 的依赖关系，然后在application.properties中声明服务器的URL，如以下示例所示：

```java
spring.ldap.urls=ldap://myserver:1235
spring.ldap.username=admin
spring.ldap.password=secret
```

如果需要自定义连接设置，可以使用 `spring.ldap.base` 和 `spring.ldap.base-environment` 属性.

根据这些设置自动配置 `LdapContextSource` .如果您需要自定义它，例如使用 `PooledContextSource` ，您仍然可以注入自动配置的 `LdapContextSource` .确保将自定义 `ContextSource` 标记为 `@Primary` ，以便自动配置的 `LdapTemplate` 使用它.

### 31.9.2 Spring Data LDAP存储库

Spring Data包括对LDAP的存储库支持.有关Spring Data LDAP的完整详细信息，请参阅[reference documentation](https://docs.spring.io/spring-data/ldap/docs/1.0.x/reference/html/).

您也可以像使用任何其他Spring Bean一样注入自动配置的 `LdapTemplate` 实例，如以下示例所示：

```java
@Component
public class MyBean {

	private final LdapTemplate template;

	@Autowired
	public MyBean(LdapTemplate template) {
		this.template = template;
	}

	// ...

}
```

### 31.9.3嵌入式内存LDAP服务器

出于测试目的，Spring Boot支持从[UnboundID](https://www.ldap.com/unboundid-ldap-sdk-for-java)自动配置内存中的LDAP服务器.要配置服务器，请将依赖项添加到 `com.unboundid:unboundid-ldapsdk` 并声明 `base-dn` 属性，如下所示：

```java
spring.ldap.embedded.base-dn=dc=spring,dc=io
```

> 可以定义多个base-dn值，但是，由于专有名称通常包含逗号，因此必须使用正确的符号来定义它们.

默认情况下，服务器在随机端口上启动并触发常规LDAP支持.无需指定 `spring.ldap.urls` 属性.

如果类路径上有 `schema.ldif` 文件，则用于初始化服务器.如果要从其他资源加载初始化脚本，还可以使用 `spring.ldap.embedded.ldif` 属性.

默认情况下，标准模式用于验证 `LDIF` 文件.您可以通过设置 `spring.ldap.embedded.validation.enabled` 属性完全关闭验证.如果您有自定义属性，则可以使用 `spring.ldap.embedded.validation.schema` 来定义自定义属性类型或对象类.

## 31.10 InfluxDB

[InfluxDB](https://www.influxdata.com/)是一个开源时间序列数据库，针对诸如运营监控，应用程序指标，物联网传感器数据和实时分析等领域中的时间序列数据的快速，高可用性存储和检索进行了优化.

### 31.10.1连接到InfluxDB

如果 `influxdb-java` 客户端在类路径上并且设置了数据库的URL，则Spring Boot会自动配置 `InfluxDB` 实例，如以下示例所示：

```java
spring.influx.url=http://172.0.0.1:8086
```

如果与InfluxDB的连接需要用户和密码，则可以相应地设置 `spring.influx.user` 和 `spring.influx.password` 属性.

InfluxDB依赖于OkHttp.如果你需要调整http客户端 `InfluxDB` 在幕后使用，你可以注册一个 `InfluxDbOkHttpClientBuilderProvider`  bean.

