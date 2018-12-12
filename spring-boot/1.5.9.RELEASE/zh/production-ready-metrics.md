## 57.指标

Spring Boot Actuator为[Micrometer](https://micrometer.io)提供依赖关系管理和自动配置，[Micrometer](https://micrometer.io)是一个支持众多监控系统的应用程序指标外观，包括：

- [AppOptics](production-ready-metrics.html#production-ready-metrics-export-appoptics)

- [Atlas](production-ready-metrics.html#production-ready-metrics-export-atlas)

- [Datadog](production-ready-metrics.html#production-ready-metrics-export-datadog)

- [Dynatrace](production-ready-metrics.html#production-ready-metrics-export-dynatrace)

- [Elastic](production-ready-metrics.html#production-ready-metrics-export-dynatrace)

- [Ganglia](production-ready-metrics.html#production-ready-metrics-export-ganglia)

- [Graphite](production-ready-metrics.html#production-ready-metrics-export-graphite)

- [Humio](production-ready-metrics.html#production-ready-metrics-export-humio)

- [Influx](production-ready-metrics.html#production-ready-metrics-export-influx)

- [JMX](production-ready-metrics.html#production-ready-metrics-export-jmx)

- [KairosDB](production-ready-metrics.html#production-ready-metrics-export-kairos)

- [New Relic](production-ready-metrics.html#production-ready-metrics-export-newrelic)

- [Prometheus](production-ready-metrics.html#production-ready-metrics-export-prometheus)

- [SignalFx](production-ready-metrics.html#production-ready-metrics-export-signalfx)

- [Simple (in-memory)](production-ready-metrics.html#production-ready-metrics-export-simple)

- [StatsD](production-ready-metrics.html#production-ready-metrics-export-statsd)

- [Wavefront](production-ready-metrics.html#production-ready-metrics-export-wavefront)

> 要了解有关Micrometer功能的更多信息，请参阅[reference documentation](https://micrometer.io/docs)，特别是[concepts section](https://micrometer.io/docs/concepts).

## 57.1入门

Spring Boot自动配置复合 `MeterRegistry` ，并为组合路径中找到的每个受支持的实现添加一个注册表.在Spring运行时类路径中依赖 `micrometer-registry-{system}` 就足以让Spring Boot配置注册表.

大多数注册管理机构都有共同点例如，即使是千分尺，您也可以禁用特定的注册表注册表实现在类路径上.例如，要禁用Datadog：

```java
management.metrics.export.datadog.enabled=false
```

Spring Boot还会将所有自动配置的注册表添加到 `Metrics` 类的全局静态复合注册表中，除非您明确告诉它不要：

```java
management.metrics.use-global-registry=false
```

在向注册表注册任何仪表之前，您可以注册任意数量的 `MeterRegistryCustomizer`  bean以进一步配置注册表，例如应用通用标记：

```java
@Bean
MeterRegistryCustomizer<MeterRegistry> metricsCommonTags() {
	return registry -> registry.config().commonTags("region", "us-east-1");
}
```

您可以通过更具体地说明泛型类型，将自定义应用于特定的注册表实现：

```java
@Bean
MeterRegistryCustomizer<GraphiteMeterRegistry> graphiteMetricsNamingConvention() {
	return registry -> registry.config().namingConvention(MY_CUSTOM_CONVENTION);
}
```

使用该设置，您可以在组件中注入 `MeterRegistry` 并注册指标：

```java
@Component
public class SampleBean {

	private final Counter counter;

	public SampleBean(MeterRegistry registry) {
		this.counter = registry.counter("received.messages");
	}

	public void handleMessage(String message) {
		this.counter.increment();
		// handle message implementation
	}

}
```

Spring Boot也可以[configures built-in instrumentation](production-ready-metrics.html#production-ready-metrics-meter)（即 `MeterBinder` 实现），您可以通过配置或专用注释标记来控制.

## 57.2支持的监控系统

### 57.2.1 AppOptics

默认情况下，AppOptics注册表会定期将指标推送到[www.appoptics.com/](https://www.appoptics.com/).要将指标导出到SaaS [AppOptics](http://micrometer.io/docs/registry/appoptics)，必须提供您的API令牌：

```java
management.metrics.export.appoptics.api-token=YOUR_TOKEN
```

### 57.2.2阿特拉斯

默认情况下，度量标准将导出到本地计算机上运行的[Atlas](http://micrometer.io/docs/registry/atlas).可以使用以下方式提供[Atlas server](https://github.com/Netflix/atlas)的使用位置：

```java
management.metrics.export.atlas.uri=http://atlas.example.com:7101/api/v1/publish
```

### 57.2.3 Datadog

Datadog注册表会定期将指标推送到[datadoghq](https://www.datadoghq.com).要将指标导出到[Datadog](http://micrometer.io/docs/registry/datadog)，必须提供您的API密钥：

```java
management.metrics.export.datadog.api-key=YOUR_KEY
```

您还可以更改度量标准发送到Datadog的时间间隔：

```java
management.metrics.export.datadog.step=30s
```

### 57.2.4 Dynatrace

Dynatrace注册表定期将指标推送到配置的URI.要将指标导出到[Dynatrace](http://micrometer.io/docs/registry/dynatrace)，必须提供您的API令牌，设备ID和URI：

```java
management.metrics.export.dynatrace.api-token=YOUR_TOKEN
management.metrics.export.dynatrace.device-id=YOUR_DEVICE_ID
management.metrics.export.dynatrace.uri=YOUR_URI
```

您还可以更改度量标准发送到Dynatrace的时间间隔：

```java
management.metrics.export.dynatrace.step=30s
```

### 57.2.5弹性

默认情况下，度量标准将导出到本地计算机上运行的[Elastic](http://micrometer.io/docs/registry/elastic).可以使用以下属性提供要使用的弹性服务器的位置：

```java
management.metrics.export.elastic.host=http://elastic.example.com:8086
```

### 57.2.6 Ganglia

默认情况下，度量标准将导出到本地计算机上运行的[Ganglia](http://micrometer.io/docs/registry/ganglia).可以使用以下方式提供要使用的[Ganglia server](http://ganglia.sourceforge.net)主机和端口：

```java
management.metrics.export.ganglia.host=ganglia.example.com
management.metrics.export.ganglia.port=9649
```

### 57.2.7石墨

默认情况下，度量标准将导出到本地计算机上运行的[Graphite](http://micrometer.io/docs/registry/graphite).可以使用以下方式提供要使用的[Graphite server](https://graphiteapp.org)主机和端口：

```java
management.metrics.export.graphite.host=graphite.example.com
management.metrics.export.graphite.port=9004
```

千分尺提供了一个默认的 `HierarchicalNameMapper` ，用于控制尺寸表id是如何[mapped to flat hierarchical names](http://micrometer.io/docs/registry/graphite#_hierarchical_name_mapping).

> 要控制此行为，请定义 `GraphiteMeterRegistry` 并提供自己的 `HierarchicalNameMapper` .除非您定义自己的bean，否则会提供自动配置的 `GraphiteConfig` 和 `Clock`  bean：

```java
@Bean
public GraphiteMeterRegistry graphiteMeterRegistry(GraphiteConfig config, Clock clock) {
	return new GraphiteMeterRegistry(config, clock, MY_HIERARCHICAL_MAPPER);
}
```

### 57.2.8 Humio

默认情况下，Humio注册表会定期将指标推送到[cloud.humio.com](https://cloud.humio.com).要将指标导出到SaaS [Humio](http://micrometer.io/docs/registry/humio)，必须提供您的API令牌：

```java
management.metrics.export.humio.api-token=YOUR_TOKEN
```

您还应配置一个或多个标记，以标识要推送指标的数据源：

```java
management.metrics.export.humio.tags.alpha=a
management.metrics.export.humio.tags.bravo=b
```

### 57.2.9 Influx

默认情况下，度量标准将导出到本地计算机上运行的[Influx](http://micrometer.io/docs/registry/influx).可以使用以下方式提供[Influx server](https://www.influxdata.com)的使用位置：

```java
management.metrics.export.influx.uri=http://influx.example.com:8086
```

### 57.2.10 JMX

Micrometer提供了[JMX](http://micrometer.io/docs/registry/jmx)的分层映射，主要是一种在本地查看指标的便宜且可移植的方式.默认情况下，度量标准将导出到 `metrics`  JMX域.可以使用以下方式提供要使用的域：

```java
management.metrics.export.jmx.domain=com.example.app.metrics
```

千分尺提供了一个默认的 `HierarchicalNameMapper` ，用于控制尺寸计id的位置[mapped to flat hierarchical names](http://micrometer.io/docs/registry/jmx#_hierarchical_name_mapping).

> 要控制此行为，请定义 `JmxMeterRegistry` 并提供自己的 `HierarchicalNameMapper` .除非您定义自己的bean，否则会提供自动配置的 `JmxConfig` 和 `Clock`  bean：

```java
@Bean
public JmxMeterRegistry jmxMeterRegistry(JmxConfig config, Clock clock) {
	return new JmxMeterRegistry(config, clock, MY_HIERARCHICAL_MAPPER);
}
```

### 57.2.11 KairosDB

默认情况下，度量标准将导出到本地计算机上运行的[KairosDB](http://micrometer.io/docs/registry/kairos).可以使用以下方式提供[KairosDB server](https://kairosdb.github.io/)的使用位置：

```java
management.metrics.export.kairos.uri=http://kairosdb.example.com:8080/api/v1/datapoints
```

### 57.2.12新遗物

New Relic注册表定期将指标推送到[New Relic](http://micrometer.io/docs/registry/new-relic).要将指标导出到[New Relic](https://newrelic.com)，必须提供您的API密钥和帐户ID：

```java
management.metrics.export.newrelic.api-key=YOUR_KEY
management.metrics.export.newrelic.account-id=YOUR_ACCOUNT_ID
```

您还可以更改度量标准发送到New Relic的时间间隔：

```java
management.metrics.export.newrelic.step=30s
```

### 57.2.13普罗米修斯

[Prometheus](http://micrometer.io/docs/registry/prometheus)期望抓取或轮询各个应用实例以获取指标. Spring Boot提供 `/actuator/prometheus` 处的Actuatorendpoints，以提供具有适当格式的[Prometheus scrape](https://prometheus.io).

> 默认情况下endpoints不可用且必须公开，有关详细信息，请参阅[exposing endpoints](production-ready-endpoints.html#production-ready-endpoints-exposing-endpoints).

这是一个添加到 `prometheus.yml` 的示例 `scrape_config` ：

```java
scrape_configs:
- job_name: 'spring'
	metrics_path: '/actuator/prometheus'
	static_configs:
	  - targets: ['HOST:PORT']
```

### 57.2.14 SignalFx

SignalFx注册表定期将指标推送到[SignalFx](http://micrometer.io/docs/registry/signalfx).要将指标导出到[SignalFx](https://signalfx.com)，必须提供您的访问令牌：

```java
management.metrics.export.signalfx.access-token=YOUR_ACCESS_TOKEN
```

您还可以更改将指标发送到SignalFx的时间间隔：

```java
management.metrics.export.signalfx.step=30s
```

### 57.2.15简单

Micrometer附带一个简单的内存后端，如果没有配置其他注册表，它将自动用作后备.这使您可以查看[metrics endpoint](production-ready-metrics.html#production-ready-metrics-endpoint)中收集的指标.

只要您使用任何其他可用后端，内存后端就会自动禁用.您也可以显式禁用它：

```java
management.metrics.export.simple.enabled=false
```

### 57.2.16 StatsD

StatsD注册表急切地将UDP上的指标推送到StatsD代理.默认情况下，度量标准将导出到本地计算机上运行的[StatsD](http://micrometer.io/docs/registry/statsd)代理程序.可以使用以下方式提供要使用的StatsD代理主机和端口：

```java
management.metrics.export.statsd.host=statsd.example.com
management.metrics.export.statsd.port=9125
```

您还可以将StatsD线路协议更改为使用（默认为Datadog）：

```java
management.metrics.export.statsd.flavor=etsy
```

### 57.2.17 Wavefront

Wavefront注册表会定期将指标推送到[Wavefront](http://micrometer.io/docs/registry/wavefront).如果要直接将指标导出到[Wavefront](https://www.wavefront.com/)，则必须提供您的API令牌：

```java
management.metrics.export.wavefront.api-token=YOUR_API_TOKEN
```

或者，您可以在环境中使用Wavefront边线或内部代理，将指标数据转发到Wavefront API主机：

```java
management.metrics.export.wavefront.uri=proxy://localhost:2878
```

> 如果将指标发布到Wavefront代理（如[the documentation](https://docs.wavefront.com/proxies_installing.html)中所述），则主机必须采用 `proxy://HOST:PORT` 格式.

您还可以更改将指标发送到Wavefront的时间间隔：

```java
management.metrics.export.wavefront.step=30s
```

## 57.3支持的指标

Spring Boot在适用时注册以下核心指标：

- JVM指标，报告利用率：

- 各种内存和缓冲池

- 与垃圾收集相关的统计信息

- Threads利用率

- 已加载/卸载的类数

- CPU指标

- File描述符度量

- Kafka消费者指标

- Log4j2指标：记录每个级别记录到Log4j2的事件数

- Logback metrics：记录每个级别记录到Logback的事件数

- Uptime指标：报告正常运行时间的指标和表示应用程序绝对启动时间的固定指标

- Tomcat指标

- [Spring Integration](https://docs.spring.io/spring-integration/docs/current/reference/html/system-management-chapter.html#micrometer-integration)指标

### 57.3.1 Spring MVC指标

自动配置可以对Spring MVC处理的请求进行检测.当 `management.metrics.web.server.auto-time-requests` 为 `true` 时，将对所有请求进行此检测.或者，当设置为 `false` 时，您可以通过将 `@Timed` 添加到请求处理方法来启用检测：

```java
@RestController
@Timed 
public class MyController {

	@GetMapping("/api/people")
	@Timed(extraTags = { "region", "us-east-1" }) 
	@Timed(value = "all.people", longTask = true) 
	public List<Person> listPeople() { ... }

}
```

| |一个控制器类，用于在控制器中的每个请求处理程序上启用计时. |
| ---- | ---- |
| |启用单个endpoints的方法.如果您在类上拥有它，则不需要这样做，但可以用于进一步自定义此特定endpoints的计时器. |
| |使用 `longTask = true` 为该方法启用长任务计时器的方法.长任务计时器需要单独的度量标准名称，并且可以使用短任务计时器进行堆叠. |

默认情况下，使用名称 `http.server.requests` 生成度量标准.可以通过设置 `management.metrics.web.server.requests-metric-name` 属性来自定义名称.

默认情况下，Spring MVC相关指标标记有以下信息：

|标签|说明|
| ---- | ---- |
|  `exception`  |处理请求时抛出的任何异常的简单类名. |
|  `method`  |请求的方法（例如， `GET` 或 `POST` ）|
|  `outcome`  |请求的结果基于响应的状态代码. 1xx是 `INFORMATIONAL` ，2xx是 `SUCCESS` ，3xx是 `REDIRECTION` ，4xx  `CLIENT_ERROR` ，5xx是 `SERVER_ERROR`  |
|  `status`  |响应的HTTP状态代码（例如， `200` 或 `500` ）|
|  `uri`  |如果可能，请求变量替换之前的URI模板（例如， `/api/person/{id}` ）|

要自定义标记，请提供实现 `WebMvcTagsProvider` 的 `@Bean` .

### 57.3.2 Spring WebFlux指标

自动配置支持WebFlux控制器和功能处理程序处理的所有请求的检测.

默认情况下，会生成名称为 `http.server.requests` 的指标.您可以通过设置 `management.metrics.web.server.requests-metric-name` 属性来自定义名称.

默认情况下，与WebFlux相关的指标标记有以下信息：

|标签|说明|
| ---- | ---- |
|  `exception`  |处理请求时抛出的任何异常的简单类名. |
|  `method`  |请求的方法（例如， `GET` 或 `POST` ）|
|  `outcome`  |请求的结果基于响应的状态代码. 1xx是 `INFORMATIONAL` ，2xx是 `SUCCESS` ,3xx是 `REDIRECTION` ，4xx  `CLIENT_ERROR` ，5xx是 `SERVER_ERROR`  |
|  `status`  |响应的HTTP状态代码（例如， `200` 或 `500` ）|
|  `uri`  |请求变量替换之前的URI模板（如果可能）（例如， `/api/person/{id}` ）|

要自定义标记，请提供实现 `WebFluxTagsProvider` 的 `@Bean` .

### 57.3.3 Jersey服务器指标

自动配置可以对Jersey JAX-RS实现处理的请求进行检测.当 `management.metrics.web.server.auto-time-requests` 为 `true` 时，将对所有请求进行此检测.或者，当设置为 `false` 时，您可以通过将 `@Timed` 添加到请求处理方法来启用检测：

```java
@Component
@Path("/api/people")
@Timed 
public class Endpoint {
	@GET
	@Timed(extraTags = { "region", "us-east-1" }) 
	@Timed(value = "all.people", longTask = true) 
	public List<Person> listPeople() { ... }
}
```

| |在资源类上，为资源中的每个请求处理程序启用计时. |
| ---- | ---- |
| |在启用单个endpoints的方法上.如果您在类上拥有它，则不需要这样做，但可以用于进一步自定义此特定endpoints的计时器. |
| |在 `longTask = true` 的方法上为该方法启用长任务计时器.长任务计时器需要单独的度量标准名称，并且可以使用短任务计时器进行堆叠. |

默认情况下，使用名称 `http.server.requests` 生成度量标准.可以通过设置 `management.metrics.web.server.requests-metric-name` 属性来自定义名称.

默认情况下，Jersey服务器指标标记有以下信息：

|标签|说明|
| ---- | ---- |
|  `exception`  |处理请求时抛出的任何异常的简单类名. |
|  `method`  |请求的方法（例如， `GET` 或 `POST` ）|
|  `outcome`  |请求的结果基于响应的状态代码. 1xx是 `INFORMATIONAL` ，2xx是 `SUCCESS` ,3xx是 `REDIRECTION` ，4xx  `CLIENT_ERROR` ，5xx是 `SERVER_ERROR`  |
|  `status`  |回复HTTP状态代码（例如， `200` 或 `500` ）|
|  `uri`  |变量替换之前的请求URI模板（如果可能）（例如， `/api/person/{id}` ）|

要自定义标记，请提供实现 `JerseyTagsProvider` 的 `@Bean` .

### 57.3.4 HTTP客户端指标

Spring Boot Actuator管理 `RestTemplate` 和 `WebClient` 的检测.为此，您必须注入一个自动配置的构建器并使用它来创建实例：

-  `RestTemplateBuilder`  for  `RestTemplate` 

-  `WebClient.Builder`  for  `WebClient` 

也可以手动应用负责此检测的定制程序，即 `MetricsRestTemplateCustomizer` 和 `MetricsWebClientCustomizer` .

默认情况下，使用名称 `http.client.requests` 生成度量标准.可以通过设置 `management.metrics.web.client.requests-metric-name` 属性来自定义名称.

默认情况下，已检测客户端生成的度量标准使用以下信息进行标记：

-  `method` ，请求的方法（例如， `GET` 或 `POST` ）.

-  `uri` ，变量替换之前的请求URI模板（如果可能）（例如， `/api/person/{id}` ）.

-  `status` ，响应的HTTP状态代码（例如， `200` 或 `500` ）.

-  `clientName` ，URI的主机部分.

要自定义标记，并根据您选择的客户端，您可以提供实现 `RestTemplateExchangeTagsProvider` 或 `WebClientExchangeTagsProvider` 的 `@Bean` .  `RestTemplateExchangeTags` 和 `WebClientExchangeTags` 中有方便的静态函数.

### 57.3.5缓存指标

自动配置允许在启动时使用前缀为 `cache` 的度量标准检测所有可用的 `Cache` s.缓存检测针对一组基本指标进行了标准化.此外，还提供了特定于缓存的指标.

支持以下缓存库：

- Caffeine

- EhCache 2

- Hazelcast

- 任何兼容的JCache（JSR-107）实现

度量标准由缓存的名称和从bean名称派生的 `CacheManager` 的名称标记.

> 启动时可用的缓存只绑定到注册表.对于在启动阶段之后即时或以编程方式创建的缓存，需要显式注册.  `CacheMetricsRegistrar`  bean可用于简化该过程.

### 57.3.6数据源度量标准

自动配置允许使用名为 `jdbc` 的度量标准检测所有可用的 `DataSource` 对象.数据源检测会生成表示池中当前活动，最大允许和最小允许连接的计量器.这些仪表中的每一个都有一个以 `jdbc` 为前缀的名称.

度量标准也由基于bean名称计算的 `DataSource` 的名称标记.

> By默认情况下，Spring Boot为所有支持的数据源提供元数据;如果开箱即用不支持您喜欢的数据源，则可以添加其他 `DataSourcePoolMetadataProvider`  bean.有关示例，请参见 `DataSourcePoolMetadataProvidersConfiguration` .

此外，Hikari特定的指标以 `hikaricp` 前缀公开.每个度量标准都由池的名称标记（可以使用 `spring.datasource.name` 进行控制）.

### 57.3.7 Hibernate度量标准

自动配置启用所有可用的Hibernate  `EntityManagerFactory` 实例的检测，这些实例使用名为 `hibernate` 的度量标准启用统计信息.

度量标准也由 `EntityManagerFactory` 的名称标记，该名称源自bean名称.

要启用统计信息，标准JPA属性 `hibernate.generate_statistics` 必须设置为 `true` .您可以在自动配置的 `EntityManagerFactory` 上启用它，如以下示例所示：

```java
spring.jpa.properties.hibernate.generate_statistics=true
```

### 57.3.8 RabbitMQ指标

自动配置将使用名为 `rabbitmq` 的度量标准启用所有可用RabbitMQ连接工厂的检测.

## 57.4注册自定义指标

要注册自定义指标，请将 `MeterRegistry` 注入组件，如以下示例所示：

```java
class Dictionary {

	private final List<String> words = new CopyOnWriteArrayList<>();

	Dictionary(MeterRegistry registry) {
		registry.gaugeCollectionSize("dictionary.size", Tags.empty(), this.words);
	}

	// …

}
```

如果您发现跨组件或应用程序重复检测一套度量标准，则可以将此套件封装在 `MeterBinder` 实现中.默认情况下，所有 `MeterBinder`  beans的指标将自动绑定到Spring管理的 `MeterRegistry` .

## 57.5自定义各个指标

如果需要将自定义应用于特定的 `Meter` 实例，可以使用 `io.micrometer.core.instrument.config.MeterFilter` 接口.默认情况下，所有 `MeterFilter`  bean将自动应用于千分尺 `MeterRegistry.Config` .

例如，如果要将 `mytag.region` 标记重命名为 `mytag.area` ，对于以 `com.example` 开头的所有仪表ID，您可以执行以下操作：

```java
@Bean
public MeterFilter renameRegionTagMeterFilter() {
	return MeterFilter.renameTag("com.example", "mytag.region", "mytag.area");
}
```

### 57.5.1常用标签

通用标签通常用于操作环境中的维度向下钻取，如主机，实例，区域，堆栈等.共用标签应用于所有仪表，并且可以按以下示例所示进行配置：

```java
management.metrics.tags.region=us-east-1
management.metrics.tags.stack=prod
```

上面的示例将 `region` 和 `stack` 标记添加到所有仪表，其值分别为 `us-east-1` 和 `prod` .

> 如果您使用Graphite，常用标签的顺序很重要.由于使用此方法无法保证常用标记的顺序，因此建议Graphite用户定义自定义 `MeterFilter` .

### 57.5.2每米属性

除了 `MeterFilter`  beans之外，还可以使用属性在每米的基础上应用一组有限的自定义.每米自定义适用于任何所有仪表ID从给定的名称开始.例如，以下将禁用ID为 `example.remote` 的任何仪表

```java
management.metrics.enable.example.remote=false
```

以下属性允许每米定制：

**Table 57.1. Per-meter customizations** 

|地产|简介|
| ---- | ---- |
|  `management.metrics.enable`  |是否拒绝米发出任何指标. |
|  `management.metrics.distribution.percentiles-histogram`  |是否发布适合计算可聚合（跨维度）百分位近似值的直方图. |
|  `management.metrics.distribution.minimum-expected-value` ， `management.metrics.distribution.maximum-expected-value`  |通过限制预期值的范围来发布较少的直方图桶. |
|  `management.metrics.distribution.percentiles`  |发布在您的应用程序中计算的百分位数值
|  `management.metrics.distribution.sla`  |使用SLA定义的存储桶发布累积直方图. |

有关 `percentiles-histogram` ， `percentiles` 和 `sla` 背后概念的更多详细信息，请参阅千分尺文档的["Histograms and percentiles" section](https://micrometer.io/docs/concepts#_histograms_and_percentiles).

## 57.6度量标准终结点

Spring Boot提供了一个 `metrics` endpoints，可以在诊断上用于检查应用程序收集的指标.默认情况下endpoints不可用，必须公开，有关详细信息，请参阅[exposing endpoints](production-ready-endpoints.html#production-ready-endpoints-exposing-endpoints).

导航到 `/actuator/metrics` 会显示可用仪表名称列表.您可以向下钻取以查看有关特定仪表的信息，方法是将其名称作为选择器，例如，  `/actuator/metrics/jvm.memory.max` .

> 您在此处使用的名称应与代码中使用的名称相匹配，而不是命名后的名称 - 为其运送到的监控系统规范化的约定.换句话说，如果由于其蛇案例命名约定， `jvm.memory.max` 在普罗米修斯中显示为 `jvm_memory_max` ，则在检查 `metrics` endpoints中的仪表时，仍应使用 `jvm.memory.max` 作为选择器.

您还可以在URL的末尾添加任意数量的 `tag=KEY:VALUE` 查询参数，以便按比例向下钻取仪表，例如 `/actuator/metrics/jvm.memory.max?tag=area:nonheap` .

> 报告的测量值是与仪表名称和已应用的任何标记匹配的所有仪表的统计数据的总和.因此，在上面的示例中，返回的"Value"统计信息是堆的"Code Cache"，"Compressed Class Space"和"Metaspace"区域的最大内存占用量的总和.如果您只想查看"Metaspace"的最大大小，可以添加一个额外的 `tag=id:Metaspace` ，即 `/actuator/metrics/jvm.memory.max?tag=area:nonheap&tag=id:Metaspace` .

