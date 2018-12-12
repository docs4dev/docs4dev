## 52. Propagation

Propagation is needed to ensure activities originating from the same root are collected together in the same trace. The most common propagation approach is to copy a trace context from a client by sending an RPC request to a server receiving it.

For example, when a downstream HTTP call is made, its trace context is encoded as request headers and sent along with it, as shown in the following image:

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

The names above are from [B3 Propagation](https://github.com/openzipkin/b3-propagation), which is built-in to Brave and has implementations in many languages and frameworks.

Most users use a framework interceptor to automate propagation. The next two examples show how that might work for a client and a server.

The following example shows how client-side propagation might work:

```java
@Autowired Tracing tracing;

// configure a function that injects a trace context into a request
injector = tracing.propagation().injector(Request.Builder::addHeader);

// before a request is sent, add the current span's context to it
injector.inject(span.context(), request);
```

The following example shows how server-side propagation might work:

```java
@Autowired Tracing tracing;
@Autowired Tracer tracer;

// configure a function that extracts the trace context from a request
extractor = tracing.propagation().extractor(Request::getHeader);

// when a server receives a request, it joins or starts a new trace
span = tracer.nextSpan(extractor.extract(request));
```

## 52.1 Propagating extra fields

Sometimes you need to propagate extra fields, such as a request ID or an alternate trace context. For example, if you are in a Cloud Foundry environment, you might want to pass the request ID, as shown in the following example:

```java
// when you initialize the builder, define the extra field you want to propagate
Tracing.newBuilder().propagationFactory(
ExtraFieldPropagation.newFactory(B3Propagation.FACTORY, "x-vcap-request-id")
);

// later, you can tag that request ID or use it in log correlation
requestId = ExtraFieldPropagation.get("x-vcap-request-id");
```

You may also need to propagate a trace context that you are not using. For example, you may be in an Amazon Web Services environment but not be reporting data to X-Ray. To ensure X-Ray can co-exist correctly, pass-through its tracing header, as shown in the following example:

```java
tracingBuilder.propagationFactory(
ExtraFieldPropagation.newFactory(B3Propagation.FACTORY, "x-amzn-trace-id")
);
```

> In Spring Cloud Sleuth all elements of the tracing builder  `Tracing.newBuilder()`  are defined as beans. So if you want to pass a custom  `PropagationFactory` , it’s enough for you to create a bean of that type and we will set it in the  `Tracing`  bean.

### 52.1.1 Prefixed fields

If they follow a common pattern, you can also prefix fields. The following example shows how to propagate  `x-vcap-request-id`  the field as-is but send the  `country-code`  and  `user-id`  fields on the wire as  `x-baggage-country-code`  and  `x-baggage-user-id` , respectively:

```java
Tracing.newBuilder().propagationFactory(
ExtraFieldPropagation.newFactoryBuilder(B3Propagation.FACTORY)
.addField("x-vcap-request-id")
.addPrefixedFields("x-baggage-", Arrays.asList("country-code", "user-id"))
.build()
);
```

Later, you can call the following code to affect the country code of the current trace context:

```java
ExtraFieldPropagation.set("x-country-code", "FO");
String countryCode = ExtraFieldPropagation.get("x-country-code");
```

Alternatively, if you have a reference to a trace context, you can use it explicitly, as shown in the following example:

```java
ExtraFieldPropagation.set(span.context(), "x-country-code", "FO");
String countryCode = ExtraFieldPropagation.get(span.context(), "x-country-code");
```

|images/important.png|Important|
|----|----|
|A difference from previous versions of Sleuth is that, with Brave, you must pass the list of baggage keys. There are two properties to achieve this. With the  `spring.sleuth.baggage-keys` , you set keys that get prefixed with  `baggage-`  for HTTP calls and  `baggage_`  for messaging. You can also use the  `spring.sleuth.propagation-keys`  property to pass a list of prefixed keys that are whitelisted without any prefix. Notice that there’s no  `x-`  in front of the header keys. |

### 52.1.2 Extracting a Propagated Context

The  `TraceContext.Extractor<C>`  reads trace identifiers and sampling status from an incoming request or message. The carrier is usually a request object or headers.

This utility is used in standard instrumentation (such as  `HttpServerHandler`` ) but can also be used for custom RPC or messaging code.

`TraceContextOrSamplingFlags`  is usually used only with  `Tracer.nextSpan(extracted)` , unless you are sharing span IDs between a client and a server.

### 52.1.3 Sharing span IDs between Client and Server

A normal instrumentation pattern is to create a span representing the server side of an RPC.  `Extractor.extract`  might return a complete trace context when applied to an incoming client request.  `Tracer.joinSpan`  attempts to continue this trace, using the same span ID if supported or creating a child span if not. When the span ID is shared, the reported data includes a flag saying so.

The following image shows an example of B3 propagation:

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

Some propagation systems forward only the parent span ID, detected when  `Propagation.Factory.supportsJoin() == false` . In this case, a new span ID is always provisioned, and the incoming context determines the parent ID.

The following image shows an example of AWS propagation:

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

Note: Some span reporters do not support sharing span IDs. For example, if you set  `Tracing.Builder.spanReporter(amazonXrayOrGoogleStackdrive)` , you should disable join by setting  `Tracing.Builder.supportsJoin(false)` . Doing so forces a new child span on  `Tracer.joinSpan()` .

### 52.1.4 Implementing Propagation

`TraceContext.Extractor<C>`  is implemented by a  `Propagation.Factory`  plugin. Internally, this code creates the union type,  `TraceContextOrSamplingFlags` , with one of the following: *  `TraceContext`  if trace and span IDs were present. *  `TraceIdContext`  if a trace ID was present but span IDs were not present. *  `SamplingFlags`  if no identifiers were present.

Some  `Propagation`  implementations carry extra data from the point of extraction (for example, reading incoming headers) to injection (for example, writing outgoing headers). For example, it might carry a request ID. When implementations have extra data, they handle it as follows: * If a  `TraceContext`  were extracted, add the extra data as  `TraceContext.extra()` . * Otherwise, add it as  `TraceContextOrSamplingFlags.extra()` , which  `Tracer.nextSpan`  handles.

