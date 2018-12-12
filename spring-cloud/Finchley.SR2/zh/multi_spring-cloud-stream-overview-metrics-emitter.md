## 35.指标Launcher

Spring Boot Actuator为[Micrometer](https://micrometer.io/)提供依赖关系管理和自动配置，这是一个支持众多[monitoring systems](https://docs.spring.io/spring-boot/docs/2.0.0.RELEASE/reference/htmlsingle/#production-ready-metrics)的应用程序指标外观.

Spring Cloud Stream支持将任何可用的基于微米的度量标准发送到绑定目标，允许定期从流应用程序收集度量标准数据，而无需依赖轮询各个endpoints.

通过定义 `spring.cloud.stream.bindings.applicationMetrics.destination` 属性来激活度量标准Launcher，该属性指定当前Binders用于发布度量标准消息的绑定目标的名称.

例如：

```java
spring.cloud.stream.bindings.applicationMetrics.destination=myMetricDestination
```

前面的示例指示Binders绑定到 `myMetricDestination` （即Rabbit交换，Kafka主题和其他）.

以下属性可用于自定义指标的发布：

spring.cloud.stream.metrics.key要发出的度量标准的名称.每个应用程序应该是唯一值.默认值：$ {spring.application.name：$ {vcap.application.name:${spring.config.name:application}}} spring.cloud.stream.metrics.properties允许添加到指标的白名单应用程序属性payload默认值：null. spring.cloud.stream.metrics.meter-filter用于控制想要捕获的“米”的模式.例如，指定spring.integration.*可捕获名称以spring.integration开头的计量表的度量标准信息.默认值：捕获所有“米”.spring.cloud.stream.metrics.schedule-interval控制发布度量标准数据的速率的间隔.默认值：1分钟

考虑以下：

```java
java -jar time-source.jar \
--spring.cloud.stream.bindings.applicationMetrics.destination=someMetrics \
--spring.cloud.stream.metrics.properties=spring.application** \
--spring.cloud.stream.metrics.meter-filter=spring.integration.*
```

以下示例显示了作为上述命令的结果发布到绑定目标的数据的有效负载：

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

> 如果在迁移到Micrometer后，度量标准消息的格式略有变化，则已发布的消息也将 `STREAM_CLOUD_STREAM_VERSION` 标头设置为 `2.x` ，以帮助区分旧版Spring Cloud Stream的度量标准消息.

