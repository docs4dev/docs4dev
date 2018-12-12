## 57. Metrics

Spring Boot Actuator provides dependency management and auto-configuration for [Micrometer](https://micrometer.io), an application metrics facade that supports numerous monitoring systems, including:

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

> To learn more about Micrometer’s capabilities, please refer to its [reference documentation](https://micrometer.io/docs), in particular the [concepts section](https://micrometer.io/docs/concepts).

## 57.1 Getting started

Spring Boot auto-configures a composite  `MeterRegistry`  and adds a registry to the composite for each of the supported implementations that it finds on the classpath. Having a dependency on  `micrometer-registry-{system}`  in your runtime classpath is enough for Spring Boot to configure the registry.

Most registries share common features. For instance, you can disable a particular registry even if the Micrometer registry implementation is on the classpath. For instance, to disable Datadog:

```java
management.metrics.export.datadog.enabled=false
```

Spring Boot will also add any auto-configured registries to the global static composite registry on the  `Metrics`  class unless you explicitly tell it not to:

```java
management.metrics.use-global-registry=false
```

You can register any number of  `MeterRegistryCustomizer`  beans to further configure the registry, such as applying common tags, before any meters are registered with the registry:

```java
@Bean
MeterRegistryCustomizer<MeterRegistry> metricsCommonTags() {
	return registry -> registry.config().commonTags("region", "us-east-1");
}
```

You can apply customizations to particular registry implementations by being more specific about the generic type:

```java
@Bean
MeterRegistryCustomizer<GraphiteMeterRegistry> graphiteMetricsNamingConvention() {
	return registry -> registry.config().namingConvention(MY_CUSTOM_CONVENTION);
}
```

With that setup in place you can inject  `MeterRegistry`  in your components and register metrics:

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

Spring Boot also [configures built-in instrumentation](production-ready-metrics.html#production-ready-metrics-meter) (i.e.  `MeterBinder`  implementations) that you can control via configuration or dedicated annotation markers.

## 57.2 Supported monitoring systems

### 57.2.1 AppOptics

By default, the AppOptics registry pushes metrics to [www.appoptics.com/](https://www.appoptics.com/) periodically. To export metrics to SaaS [AppOptics](http://micrometer.io/docs/registry/appoptics), your API token must be provided:

```java
management.metrics.export.appoptics.api-token=YOUR_TOKEN
```

### 57.2.2 Atlas

By default, metrics are exported to [Atlas](http://micrometer.io/docs/registry/atlas) running on your local machine. The location of the [Atlas server](https://github.com/Netflix/atlas) to use can be provided using:

```java
management.metrics.export.atlas.uri=http://atlas.example.com:7101/api/v1/publish
```

### 57.2.3 Datadog

Datadog registry pushes metrics to [datadoghq](https://www.datadoghq.com) periodically. To export metrics to [Datadog](http://micrometer.io/docs/registry/datadog), your API key must be provided:

```java
management.metrics.export.datadog.api-key=YOUR_KEY
```

You can also change the interval at which metrics are sent to Datadog:

```java
management.metrics.export.datadog.step=30s
```

### 57.2.4 Dynatrace

Dynatrace registry pushes metrics to the configured URI periodically. To export metrics to [Dynatrace](http://micrometer.io/docs/registry/dynatrace), your API token, device ID, and URI must be provided:

```java
management.metrics.export.dynatrace.api-token=YOUR_TOKEN
management.metrics.export.dynatrace.device-id=YOUR_DEVICE_ID
management.metrics.export.dynatrace.uri=YOUR_URI
```

You can also change the interval at which metrics are sent to Dynatrace:

```java
management.metrics.export.dynatrace.step=30s
```

### 57.2.5 Elastic

By default, metrics are exported to [Elastic](http://micrometer.io/docs/registry/elastic) running on your local machine. The location of the Elastic server to use can be provided using the following property:

```java
management.metrics.export.elastic.host=http://elastic.example.com:8086
```

### 57.2.6 Ganglia

By default, metrics are exported to [Ganglia](http://micrometer.io/docs/registry/ganglia) running on your local machine. The [Ganglia server](http://ganglia.sourceforge.net) host and port to use can be provided using:

```java
management.metrics.export.ganglia.host=ganglia.example.com
management.metrics.export.ganglia.port=9649
```

### 57.2.7 Graphite

By default, metrics are exported to [Graphite](http://micrometer.io/docs/registry/graphite) running on your local machine. The [Graphite server](https://graphiteapp.org) host and port to use can be provided using:

```java
management.metrics.export.graphite.host=graphite.example.com
management.metrics.export.graphite.port=9004
```

Micrometer provides a default  `HierarchicalNameMapper`  that governs how a dimensional meter id is [mapped to flat hierarchical names](http://micrometer.io/docs/registry/graphite#_hierarchical_name_mapping).

> To take control over this behaviour, define your  `GraphiteMeterRegistry`  and supply your own  `HierarchicalNameMapper` . An auto-configured  `GraphiteConfig`  and  `Clock`  beans are provided unless you define your own:

```java
@Bean
public GraphiteMeterRegistry graphiteMeterRegistry(GraphiteConfig config, Clock clock) {
	return new GraphiteMeterRegistry(config, clock, MY_HIERARCHICAL_MAPPER);
}
```

### 57.2.8 Humio

By default, the Humio registry pushes metrics to [cloud.humio.com](https://cloud.humio.com) periodically. To export metrics to SaaS [Humio](http://micrometer.io/docs/registry/humio), your API token must be provided:

```java
management.metrics.export.humio.api-token=YOUR_TOKEN
```

You should also configure one or more tags to identify the data source to which metrics will be pushed:

```java
management.metrics.export.humio.tags.alpha=a
management.metrics.export.humio.tags.bravo=b
```

### 57.2.9 Influx

By default, metrics are exported to [Influx](http://micrometer.io/docs/registry/influx) running on your local machine. The location of the [Influx server](https://www.influxdata.com) to use can be provided using:

```java
management.metrics.export.influx.uri=http://influx.example.com:8086
```

### 57.2.10 JMX

Micrometer provides a hierarchical mapping to [JMX](http://micrometer.io/docs/registry/jmx), primarily as a cheap and portable way to view metrics locally. By default, metrics are exported to the  `metrics`  JMX domain. The domain to use can be provided using:

```java
management.metrics.export.jmx.domain=com.example.app.metrics
```

Micrometer provides a default  `HierarchicalNameMapper`  that governs how a dimensional meter id is [mapped to flat hierarchical names](http://micrometer.io/docs/registry/jmx#_hierarchical_name_mapping).

> To take control over this behaviour, define your  `JmxMeterRegistry`  and supply your own  `HierarchicalNameMapper` . An auto-configured  `JmxConfig`  and  `Clock`  beans are provided unless you define your own:

```java
@Bean
public JmxMeterRegistry jmxMeterRegistry(JmxConfig config, Clock clock) {
	return new JmxMeterRegistry(config, clock, MY_HIERARCHICAL_MAPPER);
}
```

### 57.2.11 KairosDB

By default, metrics are exported to [KairosDB](http://micrometer.io/docs/registry/kairos) running on your local machine. The location of the [KairosDB server](https://kairosdb.github.io/) to use can be provided using:

```java
management.metrics.export.kairos.uri=http://kairosdb.example.com:8080/api/v1/datapoints
```

### 57.2.12 New Relic

New Relic registry pushes metrics to [New Relic](http://micrometer.io/docs/registry/new-relic) periodically. To export metrics to [New Relic](https://newrelic.com), your API key and account id must be provided:

```java
management.metrics.export.newrelic.api-key=YOUR_KEY
management.metrics.export.newrelic.account-id=YOUR_ACCOUNT_ID
```

You can also change the interval at which metrics are sent to New Relic:

```java
management.metrics.export.newrelic.step=30s
```

### 57.2.13 Prometheus

[Prometheus](http://micrometer.io/docs/registry/prometheus) expects to scrape or poll individual app instances for metrics. Spring Boot provides an actuator endpoint available at  `/actuator/prometheus`  to present a [Prometheus scrape](https://prometheus.io) with the appropriate format.

> The endpoint is not available by default and must be exposed, see [exposing endpoints](production-ready-endpoints.html#production-ready-endpoints-exposing-endpoints) for more details.

Here is an example  `scrape_config`  to add to  `prometheus.yml` :

```java
scrape_configs:
- job_name: 'spring'
	metrics_path: '/actuator/prometheus'
	static_configs:
	  - targets: ['HOST:PORT']
```

### 57.2.14 SignalFx

SignalFx registry pushes metrics to [SignalFx](http://micrometer.io/docs/registry/signalfx) periodically. To export metrics to [SignalFx](https://signalfx.com), your access token must be provided:

```java
management.metrics.export.signalfx.access-token=YOUR_ACCESS_TOKEN
```

You can also change the interval at which metrics are sent to SignalFx:

```java
management.metrics.export.signalfx.step=30s
```

### 57.2.15 Simple

Micrometer ships with a simple, in-memory backend that is automatically used as a fallback if no other registry is configured. This allows you to see what metrics are collected in the [metrics endpoint](production-ready-metrics.html#production-ready-metrics-endpoint).

The in-memory backend disables itself as soon as you’re using any of the other available backend. You can also disable it explicitly:

```java
management.metrics.export.simple.enabled=false
```

### 57.2.16 StatsD

The StatsD registry pushes metrics over UDP to a StatsD agent eagerly. By default, metrics are exported to a [StatsD](http://micrometer.io/docs/registry/statsd) agent running on your local machine. The StatsD agent host and port to use can be provided using:

```java
management.metrics.export.statsd.host=statsd.example.com
management.metrics.export.statsd.port=9125
```

You can also change the StatsD line protocol to use (default to Datadog):

```java
management.metrics.export.statsd.flavor=etsy
```

### 57.2.17 Wavefront

Wavefront registry pushes metrics to [Wavefront](http://micrometer.io/docs/registry/wavefront) periodically. If you are exporting metrics to [Wavefront](https://www.wavefront.com/) directly, your API token must be provided:

```java
management.metrics.export.wavefront.api-token=YOUR_API_TOKEN
```

Alternatively, you may use a Wavefront sidecar or an internal proxy set up in your environment that forwards metrics data to the Wavefront API host:

```java
management.metrics.export.wavefront.uri=proxy://localhost:2878
```

> If publishing metrics to a Wavefront proxy (as described in [the documentation](https://docs.wavefront.com/proxies_installing.html)), the host must be in the  `proxy://HOST:PORT`  format.

You can also change the interval at which metrics are sent to Wavefront:

```java
management.metrics.export.wavefront.step=30s
```

## 57.3 Supported Metrics

Spring Boot registers the following core metrics when applicable:

- JVM metrics, report utilization of:

- Various memory and buffer pools

- Statistics related to garbage collection

- Threads utilization

- Number of classes loaded/unloaded

- CPU metrics

- File descriptor metrics

- Kafka consumer metrics

- Log4j2 metrics: record the number of events logged to Log4j2 at each level

- Logback metrics: record the number of events logged to Logback at each level

- Uptime metrics: report a gauge for uptime and a fixed gauge representing the application’s absolute start time

- Tomcat metrics

- [Spring Integration](https://docs.spring.io/spring-integration/docs/current/reference/html/system-management-chapter.html#micrometer-integration) metrics

### 57.3.1 Spring MVC Metrics

Auto-configuration enables the instrumentation of requests handled by Spring MVC. When  `management.metrics.web.server.auto-time-requests`  is  `true` , this instrumentation occurs for all requests. Alternatively, when set to  `false` , you can enable instrumentation by adding  `@Timed`  to a request-handling method:

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

|  |A controller class to enable timings on every request handler in the controller. |
|----|----|
|  |A method to enable for an individual endpoint. This is not necessary if you have it on the class, but can be used to further customize the timer for this particular endpoint. |
|  |A method with  `longTask = true`  to enable a long task timer for the method. Long task timers require a separate metric name, and can be stacked with a short task timer. |

By default, metrics are generated with the name,  `http.server.requests` . The name can be customized by setting the  `management.metrics.web.server.requests-metric-name`  property.

By default, Spring MVC-related metrics are tagged with the following information:

|Tag|Description|
|----|----|
| `exception`  |Simple class name of any exception that was thrown while handling the request. |
| `method`  |Request’s method (for example,  `GET`  or  `POST` ) |
| `outcome`  |Request’s outcome based on the status code of the response. 1xx is  `INFORMATIONAL` , 2xx is  `SUCCESS` , 3xx is  `REDIRECTION` , 4xx  `CLIENT_ERROR` , and 5xx is  `SERVER_ERROR`  |
| `status`  |Response’s HTTP status code (for example,  `200`  or  `500` ) |
| `uri`  |Request’s URI template prior to variable substitution, if possible (for example,  `/api/person/{id}` ) |

To customize the tags, provide a  `@Bean`  that implements  `WebMvcTagsProvider` .

### 57.3.2 Spring WebFlux Metrics

Auto-configuration enables the instrumentation of all requests handled by WebFlux controllers and functional handlers.

By default, metrics are generated with the name  `http.server.requests` . You can customize the name by setting the  `management.metrics.web.server.requests-metric-name`  property.

By default, WebFlux-related metrics are tagged with the following information:

|Tag|Description|
|----|----|
| `exception`  |Simple class name of any exception that was thrown while handling the request. |
| `method`  |Request’s method (for example,  `GET`  or  `POST` ) |
| `outcome`  |Request’s outcome based on the status code of the response. 1xx is  `INFORMATIONAL` , 2xx is  `SUCCESS` , 3xx is  `REDIRECTION` , 4xx  `CLIENT_ERROR` , and 5xx is  `SERVER_ERROR`  |
| `status`  |Response’s HTTP status code (for example,  `200`  or  `500` ) |
| `uri`  |Request’s URI template prior to variable substitution, if possible (for example,  `/api/person/{id}` ) |

To customize the tags, provide a  `@Bean`  that implements  `WebFluxTagsProvider` .

### 57.3.3 Jersey Server Metrics

Auto-configuration enables the instrumentation of requests handled by the Jersey JAX-RS implementation. When  `management.metrics.web.server.auto-time-requests`  is  `true` , this instrumentation occurs for all requests. Alternatively, when set to  `false` , you can enable instrumentation by adding  `@Timed`  to a request-handling method:

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

|  |On a resource class to enable timings on every request handler in the resource. |
|----|----|
|  |On a method to enable for an individual endpoint. This is not necessary if you have it on the class, but can be used to further customize the timer for this particular endpoint. |
|  |On a method with  `longTask = true`  to enable a long task timer for the method. Long task timers require a separate metric name, and can be stacked with a short task timer. |

By default, metrics are generated with the name,  `http.server.requests` . The name can be customized by setting the  `management.metrics.web.server.requests-metric-name`  property.

By default, Jersey server metrics are tagged with the following information:

|Tag|Description|
|----|----|
| `exception`  |Simple class name of any exception that was thrown while handling the request. |
| `method`  |Request’s method (for example,  `GET`  or  `POST` ) |
| `outcome`  |Request’s outcome based on the status code of the response. 1xx is  `INFORMATIONAL` , 2xx is  `SUCCESS` , 3xx is  `REDIRECTION` , 4xx  `CLIENT_ERROR` , and 5xx is  `SERVER_ERROR`  |
| `status`  |Response’s HTTP status code (for example,  `200`  or  `500` ) |
| `uri`  |Request’s URI template prior to variable substitution, if possible (for example,  `/api/person/{id}` ) |

To customize the tags, provide a  `@Bean`  that implements  `JerseyTagsProvider` .

### 57.3.4 HTTP Client Metrics

Spring Boot Actuator manages the instrumentation of both  `RestTemplate`  and  `WebClient` . For that, you have to get injected with an auto-configured builder and use it to create instances:

-  `RestTemplateBuilder`  for  `RestTemplate` 

-  `WebClient.Builder`  for  `WebClient` 

It is also possible to apply manually the customizers responsible for this instrumentation, namely  `MetricsRestTemplateCustomizer`  and  `MetricsWebClientCustomizer` .

By default, metrics are generated with the name,  `http.client.requests` . The name can be customized by setting the  `management.metrics.web.client.requests-metric-name`  property.

By default, metrics generated by an instrumented client are tagged with the following information:

-  `method` , the request’s method (for example,  `GET`  or  `POST` ).

-  `uri` , the request’s URI template prior to variable substitution, if possible (for example,  `/api/person/{id}` ).

-  `status` , the response’s HTTP status code (for example,  `200`  or  `500` ).

-  `clientName` , the host portion of the URI.

To customize the tags, and depending on your choice of client, you can provide a  `@Bean`  that implements  `RestTemplateExchangeTagsProvider`  or  `WebClientExchangeTagsProvider` . There are convenience static functions in  `RestTemplateExchangeTags`  and  `WebClientExchangeTags` .

### 57.3.5 Cache Metrics

Auto-configuration enables the instrumentation of all available  `Cache` s on startup with metrics prefixed with  `cache` . Cache instrumentation is standardized for a basic set of metrics. Additional, cache-specific metrics are also available.

The following cache libraries are supported:

- Caffeine

- EhCache 2

- Hazelcast

- Any compliant JCache (JSR-107) implementation

Metrics are tagged by the name of the cache and by the name of the  `CacheManager`  that is derived from the bean name.

> Only caches that are available on startup are bound to the registry. For caches created on-the-fly or programmatically after the startup phase, an explicit registration is required. A  `CacheMetricsRegistrar`  bean is made available to make that process easier.

### 57.3.6 DataSource Metrics

Auto-configuration enables the instrumentation of all available  `DataSource`  objects with a metric named  `jdbc` . Data source instrumentation results in gauges representing the currently active, maximum allowed, and minimum allowed connections in the pool. Each of these gauges has a name that is prefixed by  `jdbc` .

Metrics are also tagged by the name of the  `DataSource`  computed based on the bean name.

> By default, Spring Boot provides metadata for all supported data sources; you can add additional  `DataSourcePoolMetadataProvider`  beans if your favorite data source isn’t supported out of the box. See  `DataSourcePoolMetadataProvidersConfiguration`  for examples.

Also, Hikari-specific metrics are exposed with a  `hikaricp`  prefix. Each metric is tagged by the name of the Pool (can be controlled with  `spring.datasource.name` ).

### 57.3.7 Hibernate Metrics

Auto-configuration enables the instrumentation of all available Hibernate  `EntityManagerFactory`  instances that have statistics enabled with a metric named  `hibernate` .

Metrics are also tagged by the name of the  `EntityManagerFactory`  that is derived from the bean name.

To enable statistics, the standard JPA property  `hibernate.generate_statistics`  must be set to  `true` . You can enable that on the auto-configured  `EntityManagerFactory`  as shown in the following example:

```java
spring.jpa.properties.hibernate.generate_statistics=true
```

### 57.3.8 RabbitMQ Metrics

Auto-configuration will enable the instrumentation of all available RabbitMQ connection factories with a metric named  `rabbitmq` .

## 57.4 Registering custom metrics

To register custom metrics, inject  `MeterRegistry`  into your component, as shown in the following example:

```java
class Dictionary {

	private final List<String> words = new CopyOnWriteArrayList<>();

	Dictionary(MeterRegistry registry) {
		registry.gaugeCollectionSize("dictionary.size", Tags.empty(), this.words);
	}

	// …

}
```

If you find that you repeatedly instrument a suite of metrics across components or applications, you may encapsulate this suite in a  `MeterBinder`  implementation. By default, metrics from all  `MeterBinder`  beans will be automatically bound to the Spring-managed  `MeterRegistry` .

## 57.5 Customizing individual metrics

If you need to apply customizations to specific  `Meter`  instances you can use the  `io.micrometer.core.instrument.config.MeterFilter`  interface. By default, all  `MeterFilter`  beans will be automatically applied to the micrometer  `MeterRegistry.Config` .

For example, if you want to rename the  `mytag.region`  tag to  `mytag.area`  for all meter IDs beginning with  `com.example` , you can do the following:

```java
@Bean
public MeterFilter renameRegionTagMeterFilter() {
	return MeterFilter.renameTag("com.example", "mytag.region", "mytag.area");
}
```

### 57.5.1 Common tags

Common tags are generally used for dimensional drill-down on the operating environment like host, instance, region, stack, etc. Commons tags are applied to all meters and can be configured as shown in the following example:

```java
management.metrics.tags.region=us-east-1
management.metrics.tags.stack=prod
```

The example above adds  `region`  and  `stack`  tags to all meters with a value of  `us-east-1`  and  `prod`  respectively.

> The order of common tags is important if you are using Graphite. As the order of common tags cannot be guaranteed using this approach, Graphite users are advised to define a custom  `MeterFilter`  instead.

### 57.5.2 Per-meter properties

In addition to  `MeterFilter`  beans, it’s also possible to apply a limited set of customization on a per-meter basis using properties. Per-meter customizations apply to any all meter IDs that start with the given name. For example, the following will disable any meters that have an ID starting with  `example.remote` 

```java
management.metrics.enable.example.remote=false
```

The following properties allow per-meter customization:

**Table 57.1. Per-meter customizations** 

|Property|Description|
|----|----|
| `management.metrics.enable`  |Whether to deny meters from emitting any metrics. |
| `management.metrics.distribution.percentiles-histogram`  |Whether to publish a histogram suitable for computing aggregable (across dimension) percentile approximations. |
| `management.metrics.distribution.minimum-expected-value` ,  `management.metrics.distribution.maximum-expected-value`  |Publish less histogram buckets by clamping the range of expected values. |
| `management.metrics.distribution.percentiles`  |Publish percentile values computed in your application |
| `management.metrics.distribution.sla`  |Publish a cumulative histogram with buckets defined by your SLAs. |

For more details on concepts behind  `percentiles-histogram` ,  `percentiles`  and  `sla`  refer to the ["Histograms and percentiles" section](https://micrometer.io/docs/concepts#_histograms_and_percentiles) of the micrometer documentation.

## 57.6 Metrics endpoint

Spring Boot provides a  `metrics`  endpoint that can be used diagnostically to examine the metrics collected by an application. The endpoint is not available by default and must be exposed, see [exposing endpoints](production-ready-endpoints.html#production-ready-endpoints-exposing-endpoints) for more details.

Navigating to  `/actuator/metrics`  displays a list of available meter names. You can drill down to view information about a particular meter by providing its name as a selector, e.g.  `/actuator/metrics/jvm.memory.max` .

> The name you use here should match the name used in the code, not the name after it has been naming-convention normalized for a monitoring system it is shipped to. In other words, if  `jvm.memory.max`  appears as  `jvm_memory_max`  in Prometheus because of its snake case naming convention, you should still use  `jvm.memory.max`  as the selector when inspecting the meter in the  `metrics`  endpoint.

You can also add any number of  `tag=KEY:VALUE`  query parameters to the end of the URL to dimensionally drill down on a meter, e.g.  `/actuator/metrics/jvm.memory.max?tag=area:nonheap` .

> The reported measurements are the sum of the statistics of all meters matching the meter name and any tags that have been applied. So in the example above, the returned "Value" statistic is the sum of the maximum memory footprints of "Code Cache", "Compressed Class Space", and "Metaspace" areas of the heap. If you just wanted to see the maximum size for the "Metaspace", you could add an additional  `tag=id:Metaspace` , i.e.  `/actuator/metrics/jvm.memory.max?tag=area:nonheap&tag=id:Metaspace` .

