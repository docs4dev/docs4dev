## 52.传播

需要传播以确保源自同一根的活动在同一迹线中收集在一起.最常见的传播方法是通过向接收它的服务器发送RPC请求来从客户端复制跟踪上下文.

例如，当进行下游HTTP调用时，其跟踪上下文将被编码为请求标头并与其一起发送，如下图所示：

```java
Client Span                                                Server Span
┌──────────────────┐                                       ┌──────────────────┐
│                  │                                       │                  │
│   TraceContext   │           Http Request Headers        │   TraceContext   │
│ ┌──────────────┐ │          ┌───────────────────┐        │ ┌──────────────┐ │
│ │ TraceId      │ │          │ X─B3─TraceId      │        │ │ TraceId      │ │
│ │              │ │          │                   │        │ │              │ │
│ │ ParentSpanId │ │ Extract  │ X─B3─ParentSpanId │ Inject │ │ ParentSpanId │ │
│ │              ├─┼─────────>│                   ├────────┼>│              │ │
│ │ SpanId       │ │          │ X─B3─SpanId       │        │ │ SpanId       │ │
│ │              │ │          │                   │        │ │              │ │
│ │ Sampled      │ │          │ X─B3─Sampled      │        │ │ Sampled      │ │
│ └──────────────┘ │          └───────────────────┘        │ └──────────────┘ │
│                  │                                       │                  │
└──────────────────┘                                       └──────────────────┘
```

上面的名称来自[B3 Propagation](https://github.com/openzipkin/b3-propagation)，它内置于Brave，并且具有许多语言和框架的实现.

大多数用户使用框架拦截器来自动传播.接下来的两个示例显示了它如何对客户端和服务器起作用.

以下示例显示了客户端传播如何工作：

```java
@Autowired Tracing tracing;

// configure a function that injects a trace context into a request
injector = tracing.propagation().injector(Request.Builder::addHeader);

// before a request is sent, add the current span's context to it
injector.inject(span.context(), request);
```

以下示例显示了服务器端传播如何工作：

```java
@Autowired Tracing tracing;
@Autowired Tracer tracer;

// configure a function that extracts the trace context from a request
extractor = tracing.propagation().extractor(Request::getHeader);

// when a server receives a request, it joins or starts a new trace
span = tracer.nextSpan(extractor.extract(request));
```

## 52.1传播额外的字段

有时您需要传播额外的字段，例如请求ID或备用跟踪上下文.例如，如果您位于Cloud Foundry环境中，则可能需要传递请求ID，如以下示例所示：

```java
// when you initialize the builder, define the extra field you want to propagate
Tracing.newBuilder().propagationFactory(
ExtraFieldPropagation.newFactory(B3Propagation.FACTORY, "x-vcap-request-id")
);

// later, you can tag that request ID or use it in log correlation
requestId = ExtraFieldPropagation.get("x-vcap-request-id");
```

您可能还需要传播未使用的跟踪上下文.例如，您可能位于Amazon Web Services环境中，但未向X-Ray报告数据.要确保X-Ray能够正确共存，请传递其跟踪标头，如以下示例所示：

```java
tracingBuilder.propagationFactory(
ExtraFieldPropagation.newFactory(B3Propagation.FACTORY, "x-amzn-trace-id")
);
```

> 在Spring Cloud Sleuth中，跟踪构建器 `Tracing.newBuilder()` 的所有元素都被定义为bean.所以如果你想传递一个自定义 `PropagationFactory` ，那么你就可以创建一个这种类型的bean了，我们将它设置在 `Tracing`  bean中.

### 52.1.1前缀字段

如果它们遵循通用模式，您还可以为字段添加前缀.以下示例显示如何按原样传播 `x-vcap-request-id` 字段，但分别将 `country-code` 和 `user-id` 字段作为 `x-baggage-country-code` 和 `x-baggage-user-id` 发送到线路上：

```java
Tracing.newBuilder().propagationFactory(
ExtraFieldPropagation.newFactoryBuilder(B3Propagation.FACTORY)
.addField("x-vcap-request-id")
.addPrefixedFields("x-baggage-", Arrays.asList("country-code", "user-id"))
.build()
);
```

稍后，您可以调用以下代码来影响当前跟踪上下文的国家/地区代码：

```java
ExtraFieldPropagation.set("x-country-code", "FO");
String countryCode = ExtraFieldPropagation.get("x-country-code");
```

或者，如果您有对跟踪上下文的引用，则可以显式使用它，如以下示例所示：

```java
ExtraFieldPropagation.set(span.context(), "x-country-code", "FO");
String countryCode = ExtraFieldPropagation.get(span.context(), "x-country-code");
```

|图片/ important.png |重要|
| ---- | ---- |
|与之前版本的Sleuth不同的是，对于Brave，您必须通过Baggage键列表.有两个属性可以实现这一目标.使用 `spring.sleuth.baggage-keys` ，您可以设置以HTTP调用为前缀的 `baggage-` 和用于消息传递的 `baggage_` .您还可以使用 `spring.sleuth.propagation-keys` 属性来传递列入白名单但没有任何前缀的前缀键列表.请注意，Headers键前面没有 `x-` . |

### 52.1.2提取传播的上下文

`TraceContext.Extractor<C>` 从传入的请求或消息中读取跟踪标识符和采样状态.运营商通常是请求对象或标头.

此实用程序用于标准检测（例如 `HttpServerHandler`` ），但也可用于自定义RPC或消息传递代码.

`TraceContextOrSamplingFlags` 通常仅与 `Tracer.nextSpan(extracted)` 一起使用，除非您在客户端和服务器之间共享span ID.

### 52.1.3在客户端和服务器之间共享SpanID

正常的检测模式是创建表示RPC服务器端的span.  `Extractor.extract` 可能会在应用于传入的客户端请求时返回完整的跟踪上下文.  `Tracer.joinSpan` 尝试继续此跟踪，如果支持则使用相同的SpanID，否则创建子Span.当共享SpanID时，报告的数据包括一个标记.

下图显示了B3传播的示例：

```java
┌───────────────────┐      ┌───────────────────┐
Incoming Headers             │   TraceContext    │      │   TraceContext    │
┌───────────────────┐(extract)│ ┌───────────────┐ │(join)│ ┌───────────────┐ │
│ X─B3-TraceId      │─────────┼─┼> TraceId      │ │──────┼─┼> TraceId      │ │
│                   │         │ │               │ │      │ │               │ │
│ X─B3-ParentSpanId │─────────┼─┼> ParentSpanId │ │──────┼─┼> ParentSpanId │ │
│                   │         │ │               │ │      │ │               │ │
│ X─B3-SpanId       │─────────┼─┼> SpanId       │ │──────┼─┼> SpanId       │ │
└───────────────────┘         │ │               │ │      │ │               │ │
│ │               │ │      │ │  Shared: true │ │
│ └───────────────┘ │      │ └───────────────┘ │
└───────────────────┘      └───────────────────┘
```

某些传播系统仅转发父级SpanID，在 `Propagation.Factory.supportsJoin() == false` 时检测到.在这种情况下，始终配置新的跨区ID，并且传入上下文确定父ID.

下图显示了AWS传播的示例：

```java
┌───────────────────┐      ┌───────────────────┐
x-amzn-trace-id              │   TraceContext    │      │   TraceContext    │
┌───────────────────┐(extract)│ ┌───────────────┐ │(join)│ ┌───────────────┐ │
│ Root              │─────────┼─┼> TraceId      │ │──────┼─┼> TraceId      │ │
│                   │         │ │               │ │      │ │               │ │
│ Parent            │─────────┼─┼> SpanId       │ │──────┼─┼> ParentSpanId │ │
└───────────────────┘         │ └───────────────┘ │      │ │               │ │
└───────────────────┘      │ │  SpanId: New  │ │
│ └───────────────┘ │
└───────────────────┘
```

注意：某些Span报告者不支持共享SpanID.例如，如果设置 `Tracing.Builder.spanReporter(amazonXrayOrGoogleStackdrive)` ，则应通过设置 `Tracing.Builder.supportsJoin(false)` 来禁用连接.这样做会强制 `Tracer.joinSpan()` 上的新子Span.

### 52.1.4实施传播

`TraceContext.Extractor<C>` 由 `Propagation.Factory` 插件实现.在内部，此代码使用以下之一创建联合类型 `TraceContextOrSamplingFlags` ：*  `TraceContext` （如果存在跟踪和SpanID）. *  `TraceIdContext` 如果存在跟踪ID但没有跨区ID. *  `SamplingFlags` 如果没有标识符.

一些 `Propagation` 实现从提取点（例如，读取传入头）到注入（例如，编写传出头）时携带额外数据.例如，它可能带有请求ID.当实现具有额外数据时，它们按如下方式处理：*如果提取了 `TraceContext` ，则将额外数据添加为 `TraceContext.extra()` . *否则，将其添加为 `TraceContextOrSamplingFlags.extra()` ， `Tracer.nextSpan` 处理.

