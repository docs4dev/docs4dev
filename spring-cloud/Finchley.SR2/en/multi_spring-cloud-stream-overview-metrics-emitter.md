## 35. Metrics Emitter

Spring Boot Actuator provides dependency management and auto-configuration for [Micrometer](https://micrometer.io/), an application metrics facade that supports numerous [monitoring systems](https://docs.spring.io/spring-boot/docs/2.0.0.RELEASE/reference/htmlsingle/#production-ready-metrics).

Spring Cloud Stream provides support for emitting any available micrometer-based metrics to a binding destination, allowing for periodic collection of metric data from stream applications without relying on polling individual endpoints.

Metrics Emitter is activated by defining the  `spring.cloud.stream.bindings.applicationMetrics.destination`  property, which specifies the name of the binding destination used by the current binder to publish metric messages.

For example:

```java
spring.cloud.stream.bindings.applicationMetrics.destination=myMetricDestination
```

The preceding example instructs the binder to bind to  `myMetricDestination`  (that is, Rabbit exchange, Kafka topic, and others).

The following properties can be used for customizing the emission of metrics:

spring.cloud.stream.metrics.key The name of the metric being emitted. Should be a unique value per application. Default: ${spring.application.name:${vcap.application.name:${spring.config.name:application}}} spring.cloud.stream.metrics.properties Allows white listing application properties that are added to the metrics payload Default: null. spring.cloud.stream.metrics.meter-filter Pattern to control the 'meters' one wants to capture. For example, specifying spring.integration.* captures metric information for meters whose name starts with spring.integration. Default: all 'meters' are captured. spring.cloud.stream.metrics.schedule-interval Interval to control the rate of publishing metric data. Default: 1 min

Consider the following:

```java
java -jar time-source.jar \
--spring.cloud.stream.bindings.applicationMetrics.destination=someMetrics \
--spring.cloud.stream.metrics.properties=spring.application** \
--spring.cloud.stream.metrics.meter-filter=spring.integration.*
```

The following example shows the payload of the data published to the binding destination as a result of the preceding command:

```java
{
	"name": "application",
	"createdTime": "2018-03-23T14:48:12.700Z",
	"properties": {
	},
	"metrics": [
		{
			"id": {
				"name": "spring.integration.send",
				"tags": [
					{
						"key": "exception",
						"value": "none"
					},
					{
						"key": "name",
						"value": "input"
					},
					{
						"key": "result",
						"value": "success"
					},
					{
						"key": "type",
						"value": "channel"
					}
				],
				"type": "TIMER",
				"description": "Send processing time",
				"baseUnit": "milliseconds"
			},
			"timestamp": "2018-03-23T14:48:12.697Z",
			"sum": 130.340546,
			"count": 6,
			"mean": 21.72342433333333,
			"upper": 116.176299,
			"total": 130.340546
		}
	]
}
```

> Given that the format of the Metric message has slightly changed after migrating to Micrometer, the published message will also have a  `STREAM_CLOUD_STREAM_VERSION`  header set to  `2.x`  to help distinguish between Metric messages from the older versions of the Spring Cloud Stream.

