## 50.功能

- 将跟踪和SpanID添加到Slf4J MDC，因此您可以从日志聚合器中的给定跟踪或Span中提取所有日志，如以下示例日志中所示：

```java
2016-02-02 15:30:57.902  INFO [bar,6bfd228dc00d216b,6bfd228dc00d216b,false] 23030 --- [nio-8081-exec-3] ...
2016-02-02 15:30:58.372 ERROR [bar,6bfd228dc00d216b,6bfd228dc00d216b,false] 23030 --- [nio-8081-exec-3] ...
2016-02-02 15:31:01.936  INFO [bar,46ab0d418373cbc9,46ab0d418373cbc9,false] 23030 --- [nio-8081-exec-4] ...
```

注意MDC的 `[appname,traceId,spanId,exportable]` 条目：

-  **spanId** ：发生的特定操作的ID.

-  **appname** ：记录范围的应用程序的名称.

-  **traceId** ：包含范围的延迟图的ID.

-  **exportable** ：是否应将日志导出到Zipkin.你想什么时候不能出口？当您想要在Span中包装某些操作并将其写入日志时.

- 对常见的分布式跟踪数据模型进行抽象：跟踪，跨距（形成DAG），注释和键值注释. Spring Cloud Sleuth基于HTrace，但与Zipkin（Dapper）兼容.

- Sleuth记录时序信息以帮助进行延迟分析.通过使用sleuth，您可以查明应用程序中的延迟原因.
写入
- Sleuth不会记录太多，也不会导致生产环境应用程序崩溃.为此，Sleuth：

- 在带内和其他带外传播有关您的呼叫图的结构数据.

- 包括HTTP等层的固定检测.

- 包括用于管理卷的采样策略.

- 可以向Zipkin系统报告查询和可视化.

- Instruments Spring应用程序的常见入口和出口点（servlet过滤器，异步endpoints，休息模板，预定操作，消息通道，Zuul过滤器和Feign客户端）.

- Sleuth包含用于跨HTTP或消息传递边界连接跟踪的默认逻辑.例如，HTTP传播适用于与Zipkin兼容的请求标头.

- Sleuth可以在进程之间传播上下文（也称为Baggage）.因此，如果在Span上设置Baggage元素，则会通过HTTP或消息传递将其下游发送到其他进程.

- Prov提供了一种创建或继续Span并通过注释添加标记和日志的方法.

- 如果 `spring-cloud-sleuth-zipkin` 在类路径上，则应用程序会生成并收集与Zipkin兼容的跟踪.默认情况下，它通过HTTP将它们发送到localhost（端口9411）上的Zipkin服务器.您可以通过设置 `spring.zipkin.baseUrl` 来配置服务的位置.

- 如果您依赖 `spring-rabbit` ，您的应用程序会将跟踪发送到RabbitMQ代理而不是HTTP.

- 如果您依赖 `spring-kafka` 并设置 `spring.zipkin.sender.type: kafka` ，您的应用程序会将跟踪发送到Kafka代理而不是HTTP.

>  `spring-cloud-sleuth-stream` 已弃用，不应再使用.

- Spring Cloud Sleuth与[OpenTracing](http://opentracing.io/)兼容.

|图片/ important.png |重要|
| ---- | ---- |
|如果使用Zipkin，请通过设置 `spring.sleuth.sampler.probability` 配置导出Span的概率（默认值：0.1，即10％）.否则，您可能会认为Sleuth无效，因为它忽略了一些Span. |

>  SLF4J MDC始终设置，并且回溯用户立即按照前面显示的示例在日志中查看跟踪和SpanID.其他日志记录系统必须配置自己的格式化程序才能获得相同的结果.默认值如下： `logging.pattern.level` 设置为 `%5p [${spring.zipkin.service.name:${spring.application.name:-}},%X{X-B3-TraceId:-},%X{X-B3-SpanId:-},%X{X-Span-Export:-}]` （这是logback用户的Spring Boot功能）.如果您不使用SLF4J，则不会自动应用此模式.

## 50.1勇敢的介绍

|图片/ important.png |重要|
| ---- | ---- |
|从版本 `2.0.0` 开始，Spring Cloud Sleuth使用[Brave](https://github.com/openzipkin/brave)作为跟踪库.为方便起见，我们在此处嵌入了Brave的部分文档. |

|图片/ important.png |重要|
| ---- | ---- |
|在绝大多数情况下，您只需使用Sleuth提供的Brave的 `Tracer` 或 `SpanCustomizer`  bean.下面的文档概述了Brave是什么以及它是如何工作的. |

Brave是一个用于捕获和报告Zipkin分布式操作的延迟信息的库.大多数用户不直接使用Brave.他们使用图书馆或框架，而不是代表他们雇用Brave.

此模块包含一个跟踪器，用于创建和连接Span，以模拟潜在的延迟分布式工作.它还包括通过网络边界传播跟踪上下文的库（例如，使用HTTP头）.

### 50.1.1追踪

最重要的是，您需要 `brave.Tracer` ，配置为[report to Zipkin](https://github.com/openzipkin/zipkin-reporter-java).

以下示例设置通过HTTP（而不是Kafka）将跟踪数据（spans）发送到Zipkin：

```java
class MyClass {

private final Tracer tracer;

// Tracer will be autowired
MyClass(Tracer tracer) {
this.tracer = tracer;
}

void doSth() {
Span span = tracer.newTrace().name("encode").start();
// ...
}
}
```

|图片/ important.png |重要|
| ---- | ---- |
|如果您的span包含的名称长度超过50个字符，则该名称将被截断为50个字符.你的名字必须明确而具体.大名称会导致延迟问题，有时甚至会引发异常. |

跟踪器创建并加入Span，模拟潜在分布式工作的延迟.它可以采用抽样来减少过程中的开销，减少发送到Zipkin的数据量，或两者兼而有之.

跟踪器返回的Span在完成后向Zipkin报告数据，如果未采样则不执行任何操作.启动范围后，您可以注释感兴趣的事件或添加包含详细信息或查找键的标记.

Span具有上下文，该上下文包括跟踪标识符，该标识符将Span放置在表示分布式操作的树中的正确位置.

### 50.1.2本地追踪

跟踪永不离开进程的代码时，在范围Span内运行它.

```java
@Autowired Tracer tracer;

// Start a new trace or a span within an existing trace representing an operation
ScopedSpan span = tracer.startScopedSpan("encode");
try {
// The span is in "scope" meaning downstream code such as loggers can see trace IDs
return encoder.encode();
} catch (RuntimeException | Error e) {
span.error(e); // Unless you handle exceptions, you might not know the operation failed!
throw e;
} finally {
span.finish(); // always finish the span
}
```

当您需要更多功能或更精细的控制时，请使用 `Span` 类型：

```java
@Autowired Tracer tracer;

// Start a new trace or a span within an existing trace representing an operation
Span span = tracer.nextSpan().name("encode").start();
// Put the span in "scope" so that downstream code such as loggers can see trace IDs
try (SpanInScope ws = tracer.withSpanInScope(span)) {
return encoder.encode();
} catch (RuntimeException | Error e) {
span.error(e); // Unless you handle exceptions, you might not know the operation failed!
throw e;
} finally {
span.finish(); // note the scope is independent of the span. Always finish a span.
}
```

以上两个示例都报告了完成时的完全相同的Span！

在上面的示例中，Span将是新的根Span或现有跟踪中的下一个子Span.

### 50.1.3自定义Span

拥有Span后，您可以为其添加标记.标签可用作查找键或详细信息.例如，您可以使用运行时版本添加标记，如以下示例所示：

```java
span.tag("clnt/finagle.version", "6.36.0");
```

当暴露向第三方定制Span的能力时，更喜欢 `brave.SpanCustomizer` 而不是 `brave.Span` .前者更易于理解和测试，并且不会使用span生命周期钩子诱惑用户.

```java
interface MyTraceCallback {
void request(Request request, SpanCustomizer customizer);
}
```

由于 `brave.Span` 实现 `brave.SpanCustomizer` ，您可以将其传递给用户，如以下示例所示：

```java
for (MyTraceCallback callback : userCallbacks) {
callback.request(request, span);
}
```

### 50.1.4隐含地查看当前Span

有时，您不知道跟踪是否正在进行，并且您不希望用户执行空检查.  `brave.CurrentSpanCustomizer` 通过向正在进行或删除的任何Span添加数据来处理此问题，如以下示例所示：

防爆.

```java
// The user code can then inject this without a chance of it being null.
@Autowired SpanCustomizer span;

void userCode() {
span.annotate("tx.started");
...
}
```

### 50.1.5 RPC跟踪

> 在滚动自己的RPC检测之前检查[instrumentation written here](https://github.com/openzipkin/brave/tree/master/instrumentation)和[Zipkin’s list](http://zipkin.io/pages/existing_instrumentations.html).

RPC跟踪通常由拦截器自动完成.在幕后，他们添加与他们在RPC操作中的角色相关的标签和事件.

以下示例显示如何添加客户端范围：

```java
@Autowired Tracing tracing;
@Autowired Tracer tracer;

// before you send a request, add metadata that describes the operation
span = tracer.nextSpan().name(service + "/" + method).kind(CLIENT);
span.tag("myrpc.version", "1.0.0");
span.remoteServiceName("backend");
span.remoteIpAndPort("172.3.4.1", 8108);

// Add the trace context to the request, so it can be propagated in-band
tracing.propagation().injector(Request::addHeader)
.inject(span.context(), request);

// when the request is scheduled, start the span
span.start();

// if there is an error, tag the span
span.tag("error", error.getCode());
// or if there is an exception
span.error(exception);

// when the response is complete, finish the span
span.finish();
```

#### One-Way追踪

有时，您需要在存在请求但没有响应的情况下建模异步操作.在正常的RPC跟踪中，使用 `span.finish()` 表示已收到响应.在单向跟踪中，您使用 `span.flush()` 代替，因为您不期望响应.

以下示例显示了客户端如何为单向操作建模：

```java
@Autowired Tracing tracing;
@Autowired Tracer tracer;

// start a new span representing a client request
oneWaySend = tracer.nextSpan().name(service + "/" + method).kind(CLIENT);

// Add the trace context to the request, so it can be propagated in-band
tracing.propagation().injector(Request::addHeader)
.inject(oneWaySend.context(), request);

// fire off the request asynchronously, totally dropping any response
request.execute();

// start the client side and flush instead of finish
oneWaySend.start().flush();
```

以下示例显示了服务器如何处理单向操作：

```java
@Autowired Tracing tracing;
@Autowired Tracer tracer;

// pull the context out of the incoming request
extractor = tracing.propagation().extractor(Request::getHeader);

// convert that context to a span which you can name and add tags to
oneWayReceive = nextSpan(tracer, extractor.extract(request))
.name("process-request")
.kind(SERVER)
... add tags etc.

// start the server side and flush instead of finish
oneWayReceive.start().flush();

// you should not modify this span anymore as it is complete. However,
// you can create children to represent follow-up work.
next = tracer.newSpan(oneWayReceive.context()).name("step2").start();
```

