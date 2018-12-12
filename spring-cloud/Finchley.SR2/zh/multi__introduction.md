## 48.简介

Spring Cloud Sleuth为[Spring Cloud](https://cloud.spring.io)实现了分布式跟踪解决方案.

## 48.1术语

Spring Cloud Sleuth借用了[Dapper’s](http://research.google.com/pubs/pub36356.html)术语.

**Span** ：基本工作单位.例如，发送RPC是一个新的Span，就像向RPC发送响应一样.Span由Span的唯一64位ID和Span为其一部分的跟踪的另一个64位ID标识. Spans还有其他数据，例如描述，带时间戳的事件，键值注释（标签），导致它们的Span的ID以及进程ID（通常是IP地址）.

可以启动和停止Span，并跟踪其时间信息.创建Span后，必须在将来的某个时刻停止它.

> 开始跟踪的初始范围称为 `root span` .该范围的ID值等于跟踪ID.

**Trace:** 一组跨越形成树状结构.例如，如果运行分布式大数据存储，则可能会由 `PUT` 请求形成跟踪.

**Annotation:** 用于及时记录事件的存在.使用[Brave](https://github.com/openzipkin/brave)检测，我们不再需要为[Zipkin](https://zipkin.io/)设置特殊事件，以了解客户端和服务器的位置，请求的开始位置以及结束位置.然而，出于学习目的，我们标记这些事件以突出发生了什么样的行动.

-  **cs** ：客户已发送.客户提出了请求.这个注释表示Span的开始.

-  **sr** ：收到服务器：服务器端收到请求并开始处理它.从此时间戳中减去 `cs` 时间戳会显示网络延迟.

-  **ss** ：服务器已发送.在完成请求处理时（当响应被发送回客户端时）注释.从此时间戳中减去 `sr` 时间戳会显示服务器端处理请求所需的时间.

-  **cr** ：收到客户.表示Span的结束.客户端已成功收到服务器端的响应.从此时间戳中减去 `cs` 时间戳会显示客户端从服务器接收响应所需的全部时间.

下图显示了 **Span** 和 **Trace** 在系统中的外观以及Zipkin注释：

![Trace Info propagation](https://www.docs4dev.com/images/9fe8d5f1-532b-4499-b3b0-dc9e651121c1.png)

注释的每种颜色表示Span（有7个Span - 从 **A** 到 **G** ）.请考虑以下注释：

```java
Trace Id = X
Span Id = D
Client Sent
```

此注释表示当前范围 **Trace Id** 设置为 **X** ， **Span Id** 设置为 **D** .此外， `Client Sent` 事件发生了.

下图显示了Span的父子关系：

![Parent child relationship](https://www.docs4dev.com/images/9b614306-4efc-40ee-9419-0599200b5c25.png)

## 48.2目的

以下部分参考上图中显示的示例.

### 48.2.1使用Zipkin进行分布式跟踪

这个例子有七个Span.如果你去Zipkin中的痕迹，你可以在第二个痕迹中看到这个数字，如下图所示：

![Traces](https://www.docs4dev.com/images/acee9cfd-b439-4101-a47d-c8bef5da61da.png)

但是，如果选择特定跟踪，则可以看到四个Span，如下图所示：

![Traces Info propagation](https://www.docs4dev.com/images/9260b923-9ed2-4f9d-8354-616ccf976580.png)

> 当您选择特定跟踪时，您会看到合并的Span.这意味着，如果通过Server Received和Server Sent或Client Received和Client Sent annotations向Zipkin发送了两个Span，则它们将显示为单个Span.

在这种情况下，为什么七个和四个Span之间存在差异？

- TwoSpan来自 `http:/start`  span.它具有服务器已接收（ `sr` ）和服务器已发送（ `ss` ）注释.

- TwoSpan来自从 `service1` 到 `service2` 到 `http:/foo` endpoints的RPC调用.客户端已发送（ `cs` ）和客户端已接收（ `cr` ）事件发生在 `service1` 端.服务器已接收（ `sr` ）和服务器已发送（ `ss` ）事件发生在 `service2` 端.这两个Span形成一个与RPC调用相关的逻辑Span.

- TwoSpan来自从 `service2` 到 `service3` 到 `http:/bar` endpoints的RPC调用.客户端已发送（ `cs` ）和客户端已接收（ `cr` ）事件发生在 `service2` 端.服务器已接收（ `sr` ）和服务器已发送（ `ss` ）事件发生在 `service3` 端.这两个Span形成一个与RPC调用相关的逻辑Span.

- TwoSpan来自从 `service2` 到 `service4` 到 `http:/baz` endpoints的RPC调用.客户端已发送（ `cs` ）和客户端已接收（ `cr` ）事件发生在 `service2` 端.服务器已接收（ `sr` ）和服务器已发送（ `ss` ）事件发生在 `service4` 端.这两个Span形成一个与RPC调用相关的逻辑Span.

因此，如果我们计算物理Span，我们有一个来自 `http:/start` ，两个来自 `service1` 调用 `service2` ，两个来自 `service2` 调用 `service3` ，两个来自 `service2` 调用 `service4` .总之，我们总共有七个Span.

从逻辑上讲，我们看到了四个总Spans的信息，因为我们有一个与 `service1` 的传入请求相关的Span和与RPC调用相关的三个Span.

### 48.2.2可视化错误

Zipkin允许您可视化跟踪中的错误.当抛出一个异常并且没有被捕获时，我们在Span上设置了适当的标签，然后Zipkin可以正确地着色.您可以在跟踪列表中看到一条红色的迹线.这似乎是因为抛出异常.

如果单击该跟踪，则会看到类似的图片，如下所示：

![Error Traces](https://www.docs4dev.com/images/96768b4a-06b5-4b44-8b9a-94aa4cf9d53f.png)

如果您随后单击其中一个跨距，则会看到以下内容

![Error Traces Info propagation](https://www.docs4dev.com/images/8732d34b-ea5f-4075-a5b7-6ed3c74f9c7d.png)

Span显示错误的原因以及与之相关的整个堆栈跟踪.

### 48.2.3勇敢的分布式追踪

从版本 `2.0.0` 开始，Spring Cloud Sleuth使用[Brave](https://github.com/openzipkin/brave)作为跟踪库.因此，Sleuth不再负责存储上下文，而是将该工作委托给Brave.

由于Sleuth与Brave有不同的命名和标记惯例，我们决定从现在开始遵循Brave的惯例.但是，如果要使用传统的侦听方法，可以将 `spring.sleuth.http.legacy.enabled` 属性设置为 `true` .

### 48.2.4实例

**Figure 48.1. Click the Pivotal Web Services icon to see it live!** 

![Zipkin deployed on Pivotal Web Services](https://www.docs4dev.com/images/57dd4528-e42d-40f5-99ee-262e5b83b3bf.png)

[Click here to see it live!](https://docssleuth-zipkin-server.cfapps.io/)

Zipkin中的依赖关系图应类似于以下图像：

![Dependencies](https://www.docs4dev.com/images/9011d169-07ad-47dd-a47a-0aeddfa066b6.png)

**Figure 48.2. Click the Pivotal Web Services icon to see it live!** 

![Zipkin deployed on Pivotal Web Services](https://www.docs4dev.com/images/bbbe973a-ff62-488d-a0cd-b7d48bc45220.png)

[Click here to see it live!](https://docssleuth-zipkin-server.cfapps.io/dependency)

### 48.2.5记录相关性

当使用grep通过扫描等于（例如） `2485ec27856c56f4` 的跟踪ID来读取这四个应用程序的日志时，您将获得类似于以下内容的输出：

```java
service1.log:2016-02-26 11:15:47.561  INFO [service1,2485ec27856c56f4,2485ec27856c56f4,true] 68058 --- [nio-8081-exec-1] i.s.c.sleuth.docs.service1.Application   : Hello from service1. Calling service2
service2.log:2016-02-26 11:15:47.710  INFO [service2,2485ec27856c56f4,9aa10ee6fbde75fa,true] 68059 --- [nio-8082-exec-1] i.s.c.sleuth.docs.service2.Application   : Hello from service2. Calling service3 and then service4
service3.log:2016-02-26 11:15:47.895  INFO [service3,2485ec27856c56f4,1210be13194bfe5,true] 68060 --- [nio-8083-exec-1] i.s.c.sleuth.docs.service3.Application   : Hello from service3
service2.log:2016-02-26 11:15:47.924  INFO [service2,2485ec27856c56f4,9aa10ee6fbde75fa,true] 68059 --- [nio-8082-exec-1] i.s.c.sleuth.docs.service2.Application   : Got response from service3 [Hello from service3]
service4.log:2016-02-26 11:15:48.134  INFO [service4,2485ec27856c56f4,1b1845262ffba49d,true] 68061 --- [nio-8084-exec-1] i.s.c.sleuth.docs.service4.Application   : Hello from service4
service2.log:2016-02-26 11:15:48.156  INFO [service2,2485ec27856c56f4,9aa10ee6fbde75fa,true] 68059 --- [nio-8082-exec-1] i.s.c.sleuth.docs.service2.Application   : Got response from service4 [Hello from service4]
service1.log:2016-02-26 11:15:48.182  INFO [service1,2485ec27856c56f4,2485ec27856c56f4,true] 68058 --- [nio-8081-exec-1] i.s.c.sleuth.docs.service1.Application   : Got response from service2 [Hello from service2, response from service3 [Hello from service3] and from service4 [Hello from service4]]
```

如果使用日志聚合工具（例如[Kibana](https://www.elastic.co/products/kibana)，[Splunk](http://www.splunk.com/)和其他），则可以对发生的事件进行排序. Kibana的一个例子类似于下图：

![Log correlation with Kibana](https://www.docs4dev.com/images/55234218-c51e-43ec-9be6-f8df9ec006d5.png)

如果你想使用[Logstash](https://www.elastic.co/guide/en/logstash/current/index.html)，那么以下列表显示了Logstash的Grok模式：

```java
filter {
# pattern matching logback pattern
grok {
match => { "message" => "%{TIMESTAMP_ISO8601:timestamp}\s+%{LOGLEVEL:severity}\s+\[%{DATA:service},%{DATA:trace},%{DATA:span},%{DATA:exportable}\]\s+%{DATA:pid}\s+---\s+\[%{DATA:thread}\]\s+%{DATA:class}\s+:\s+%{GREEDYDATA:rest}" }
}
}
```

> 如果要将Grok与Cloud Foundry中的日志一起使用，则必须使用以下模式：

```java
filter {
# pattern matching logback pattern
grok {
match => { "message" => "(?m)OUT\s+%{TIMESTAMP_ISO8601:timestamp}\s+%{LOGLEVEL:severity}\s+\[%{DATA:service},%{DATA:trace},%{DATA:span},%{DATA:exportable}\]\s+%{DATA:pid}\s+---\s+\[%{DATA:thread}\]\s+%{DATA:class}\s+:\s+%{GREEDYDATA:rest}" }
}
}
```

使用Logstash 
#### JSON Logback

通常，您不希望将日志存储在文本文件中，而是存储在Logstash可以立即选择的JSON文件中.为此，您必须执行以下操作（为了便于阅读，我们以 `groupId:artifactId:version` 表示法传递依赖项）.

**Dependencies Setup** 

确保Logback位于类路径上（ch.qos.logback：logback-core）.添加Logstash Logback编码.例如，要使用版本4.6，请添加net.logstash.logback：logstash-logback-encoder：4.6.

**Logback Setup** 

请考虑以下Logback配置文件（名为[logback-spring.xml](https://github.com/spring-cloud-samples/sleuth-documentation-apps/blob/master/service1/src/main/resources/logback-spring.xml)）的示例.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
	<include resource="org/springframework/boot/logging/logback/defaults.xml"/>
	
	<springProperty scope="context" name="springAppName" source="spring.application.name"/>
	<!-- Example for logging into the build folder of your project -->
	<property name="LOG_FILE" value="${BUILD_FOLDER:-build}/${springAppName}"/>

	<!-- You can override this to have a custom pattern -->
	<property name="CONSOLE_LOG_PATTERN"
			  value="%clr(%d{yyyy-MM-dd HH:mm:ss.SSS}){faint} %clr(${LOG_LEVEL_PATTERN:-%5p}) %clr(${PID:- }){magenta} %clr(---){faint} %clr([%15.15t]){faint} %clr(%-40.40logger{39}){cyan} %clr(:){faint} %m%n${LOG_EXCEPTION_CONVERSION_WORD:-%wEx}"/>

	<!-- Appender to log to console -->
	<appender name="console" class="ch.qos.logback.core.ConsoleAppender">
		<filter class="ch.qos.logback.classic.filter.ThresholdFilter">
			<!-- Minimum logging level to be presented in the console logs-->
			<level>DEBUG</level>
		</filter>
		<encoder>
			<pattern>${CONSOLE_LOG_PATTERN}</pattern>
			<charset>utf8</charset>
		</encoder>
	</appender>

	<!-- Appender to log to file -->
	<appender name="flatfile" class="ch.qos.logback.core.rolling.RollingFileAppender">
		<file>${LOG_FILE}</file>
		<rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
			<fileNamePattern>${LOG_FILE}.%d{yyyy-MM-dd}.gz</fileNamePattern>
			<maxHistory>7</maxHistory>
		</rollingPolicy>
		<encoder>
			<pattern>${CONSOLE_LOG_PATTERN}</pattern>
			<charset>utf8</charset>
		</encoder>
	</appender>
	
	<!-- Appender to log to file in a JSON format -->
	<appender name="logstash" class="ch.qos.logback.core.rolling.RollingFileAppender">
		<file>${LOG_FILE}.json</file>
		<rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
			<fileNamePattern>${LOG_FILE}.json.%d{yyyy-MM-dd}.gz</fileNamePattern>
			<maxHistory>7</maxHistory>
		</rollingPolicy>
		<encoder class="net.logstash.logback.encoder.LoggingEventCompositeJsonEncoder">
			<providers>
				<timestamp>
					<timeZone>UTC</timeZone>
				</timestamp>
				<pattern>
					<pattern>
						{
						"severity": "%level",
						"service": "${springAppName:-}",
						"trace": "%X{X-B3-TraceId:-}",
						"span": "%X{X-B3-SpanId:-}",
						"parent": "%X{X-B3-ParentSpanId:-}",
						"exportable": "%X{X-Span-Export:-}",
						"pid": "${PID:-}",
						"thread": "%thread",
						"class": "%logger{40}",
						"rest": "%message"
						}
					</pattern>
				</pattern>
			</providers>
		</encoder>
	</appender>
	
	<root level="INFO">
		<appender-ref ref="console"/>
		<!-- uncomment this to have also JSON logs -->
		<!--<appender-ref ref="logstash"/>-->
		<!--<appender-ref ref="flatfile"/>-->
	</root>
</configuration>
```

那个Logback配置文件：

- 将应用程序中的信息以JSON格式记录到 `build/${spring.application.name}.json` 文件中.

- Has注释掉了两个额外的appender：控制台和标准日志文件.

- 具有与上一节中介绍的相同的日志记录模式.

> 如果使用自定义 `logback-spring.xml` ，则必须传递 `bootstrap` 中的 `spring.application.name` 而不是 `application` 属性文件.否则，您的自定义logback文件无法正确读取该属性.

### 48.2.6传播Span上下文

Span上下文是必须传播到跨进程边界的任何子Span的状态.Span上下文的一部分是Baggage.跟踪和SpanID是Span上下文的必需部分.Baggage是可选部分.

Baggage是存储在范围上下文中的一组键：值对.Baggage与痕迹一起移动并附在每个Span上. Spring Cloud Sleuth了解如果HTTP标头以 `baggage-` 为前缀，则Headers与Baggage相关，对于消息传递，它以 `baggage_` 开头.

|图片/ important.png |重要|
| ---- | ---- |
|目前对Baggage物品的数量或大小没有限制.但是，请记住，太多可能会降低系统吞吐量或增加RPC延迟.在极端情况下，由于超出传输级别的消息或标头容量，过多的Baggage可能会使应用程序崩溃. |

以下示例显示了Span设置Baggage：

```java
Span initialSpan = this.tracer.nextSpan().name("span").start();
try (Tracer.SpanInScope ws = this.tracer.withSpanInScope(initialSpan)) {
	ExtraFieldPropagation.set("foo", "bar");
	ExtraFieldPropagation.set("UPPER_CASE", "someValue");
}
```

#### Baggage与Span标签

Baggage带有痕迹（每个子跨距包含其父母的Baggage）. Zipkin不了解Baggage，也不接收这些信息.

|图片/ important.png |重要|
| ---- | ---- |
|从Sleuth 2.0.0开始，您必须在项目配置中明确传递Baggage钥匙名称.阅读有关该设置的更多信息[here](multi__propagation.html#prefixed-fields) |

标签附加到特定范围.换句话说，它们仅针对该特定范围呈现.但是，您可以按标记搜索以查找跟踪，假设存在具有搜索标记值的范围.

如果您希望能够根据Baggage查找范围，则应在根Span中添加相应的条目作为标记.

|图片/ important.png |重要|
| ---- | ---- |
|范围必须在范围内. |

以下清单显示了使用Baggage的集成测试：

**The setup.** 

```java
spring.sleuth:
baggage-keys:
- baz
- bizarrecase
propagation-keys:
- foo
- upper_case
```

**The code.** 

```java
initialSpan.tag("foo",
		ExtraFieldPropagation.get(initialSpan.context(), "foo"));
initialSpan.tag("UPPER_CASE",
		ExtraFieldPropagation.get(initialSpan.context(), "UPPER_CASE"));
```

## 48.3为项目添加Sleuth

本节介绍如何使用Maven或Gradle将Sleuth添加到项目中.

|图片/ important.png |重要|
| ---- | ---- |
|要确保在Zipkin中正确显示应用程序名称，请在 `bootstrap.yml` 中设置 `spring.application.name` 属性. |

### 48.3.1仅Sleuth（对数关联）

如果您想在没有Zipkin集成的情况下仅使用Spring Cloud Sleuth，请将 `spring-cloud-starter-sleuth` 模块添加到项目中.

以下示例显示如何使用Maven添加Sleuth：

**Maven.** 

```xml
<dependencyManagement> 
<dependencies>
<dependency>
<groupId>org.springframework.cloud</groupId>
<artifactId>spring-cloud-dependencies</artifactId>
<version>${release.train.version}</version>
<type>pom</type>
<scope>import</scope>
</dependency>
</dependencies>
</dependencyManagement>

<dependency> 
<groupId>org.springframework.cloud</groupId>
<artifactId>spring-cloud-starter-sleuth</artifactId>
</dependency>
```

| |我们建议您通过Spring BOM添加依赖关系管理，这样您就无需自行管理版本. |
| ---- | ---- |
| |将依赖项添加到 `spring-cloud-starter-sleuth` . |

以下示例显示如何使用Gradle添加Sleuth：

**Gradle.** 

```java
dependencyManagement { 
imports {
mavenBom "org.springframework.cloud:spring-cloud-dependencies:${releaseTrainVersion}"
}
}

dependencies { 
compile "org.springframework.cloud:spring-cloud-starter-sleuth"
}
```

| |我们建议您通过Spring BOM添加依赖关系管理，这样您就无需自行管理版本. |
| ---- | ---- |
| |将依赖项添加到 `spring-cloud-starter-sleuth` . |

### 48.3.2带有Zipkin的Sleuth通过HTTP

如果你想要Sleuth和Zipkin，添加 `spring-cloud-starter-zipkin` 依赖项.

以下示例显示了如何为Maven执行此操作：

**Maven.** 

```xml
<dependencyManagement> 
<dependencies>
<dependency>
<groupId>org.springframework.cloud</groupId>
<artifactId>spring-cloud-dependencies</artifactId>
<version>${release.train.version}</version>
<type>pom</type>
<scope>import</scope>
</dependency>
</dependencies>
</dependencyManagement>

<dependency> 
<groupId>org.springframework.cloud</groupId>
<artifactId>spring-cloud-starter-zipkin</artifactId>
</dependency>
```

| |我们建议您通过Spring BOM添加依赖关系管理，这样您就无需自行管理版本. |
| ---- | ---- |
| |将依赖项添加到 `spring-cloud-starter-zipkin` . |

以下示例显示了如何为Gradle执行此操作：

**Gradle.** 

```java
dependencyManagement { 
imports {
mavenBom "org.springframework.cloud:spring-cloud-dependencies:${releaseTrainVersion}"
}
}

dependencies { 
compile "org.springframework.cloud:spring-cloud-starter-zipkin"
}
```

| |我们建议您通过Spring BOM添加依赖关系管理，这样您就无需自行管理版本. |
| ---- | ---- |
| |将依赖项添加到 `spring-cloud-starter-zipkin` . |

### 48.3.3在RabbitMQ或Kafka上使用Zipkin的Sleuth

如果要使用RabbitMQ或Kafka而不是HTTP，请添加 `spring-rabbit` 或 `spring-kafka` 依赖项.默认目标名称是 `zipkin` .

如果使用Kafka，则必须相应地设置属性 `spring.zipkin.sender.type` 属性：

```java
spring.zipkin.sender.type: kafka
```

>  `spring-cloud-sleuth-stream` 已弃用且与这些目的地不兼容.

如果你想要Sleuth结束RabbitMQ，添加 `spring-cloud-starter-zipkin` 和 `spring-rabbit` 依赖项.

以下示例显示了如何为Gradle执行此操作：

**Maven.** 

```xml
<dependencyManagement> 
<dependencies>
<dependency>
<groupId>org.springframework.cloud</groupId>
<artifactId>spring-cloud-dependencies</artifactId>
<version>${release.train.version}</version>
<type>pom</type>
<scope>import</scope>
</dependency>
</dependencies>
</dependencyManagement>

<dependency> 
<groupId>org.springframework.cloud</groupId>
<artifactId>spring-cloud-starter-zipkin</artifactId>
</dependency>
<dependency> 
<groupId>org.springframework.amqp</groupId>
<artifactId>spring-rabbit</artifactId>
</dependency>
```

| |我们建议您通过Spring BOM添加依赖关系管理，这样您就无需自行管理版本. |
| ---- | ---- |
| |将依赖项添加到 `spring-cloud-starter-zipkin` .这样，所有嵌套的依赖项都会被下载. |
| |要自动配置RabbitMQ，请添加 `spring-rabbit` 依赖项. |

**Gradle.** 

```java
dependencyManagement { 
imports {
mavenBom "org.springframework.cloud:spring-cloud-dependencies:${releaseTrainVersion}"
}
}

dependencies {
compile "org.springframework.cloud:spring-cloud-starter-zipkin" 
compile "org.springframework.amqp:spring-rabbit" 
}
```

| |我们建议您通过Spring BOM添加依赖关系管理，这样您就无需自行管理版本. |
| ---- | ---- |
| |将依赖项添加到 `spring-cloud-starter-zipkin` .这样，所有嵌套的依赖项都会被下载. |
| |要自动配置RabbitMQ，请添加 `spring-rabbit` 依赖项. |

