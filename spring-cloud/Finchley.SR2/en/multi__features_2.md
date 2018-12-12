## 50. Features

- Adds trace and span IDs to the Slf4J MDC, so you can extract all the logs from a given trace or span in a log aggregator, as shown in the following example logs:

```java
2016-02-02 15:30:57.902  INFO [bar,6bfd228dc00d216b,6bfd228dc00d216b,false] 23030 --- [nio-8081-exec-3] ...
2016-02-02 15:30:58.372 ERROR [bar,6bfd228dc00d216b,6bfd228dc00d216b,false] 23030 --- [nio-8081-exec-3] ...
2016-02-02 15:31:01.936  INFO [bar,46ab0d418373cbc9,46ab0d418373cbc9,false] 23030 --- [nio-8081-exec-4] ...
```

Notice the  `[appname,traceId,spanId,exportable]`  entries from the MDC:

-  **spanId** : The ID of a specific operation that took place.

-  **appname** : The name of the application that logged the span.

-  **traceId** : The ID of the latency graph that contains the span.

-  **exportable** : Whether the log should be exported to Zipkin. When would you like the span not to be exportable? When you want to wrap some operation in a Span and have it written to the logs only.

- Provides an abstraction over common distributed tracing data models: traces, spans (forming a DAG), annotations, and key-value annotations. Spring Cloud Sleuth is loosely based on HTrace but is compatible with Zipkin (Dapper).

- Sleuth records timing information to aid in latency analysis. By using sleuth, you can pinpoint causes of latency in your applications.

- Sleuth is written to not log too much and to not cause your production application to crash. To that end, Sleuth:

- Propagates structural data about your call graph in-band and the rest out-of-band.

- Includes opinionated instrumentation of layers such as HTTP.

- Includes a sampling policy to manage volume.

- Can report to a Zipkin system for query and visualization.

- Instruments common ingress and egress points from Spring applications (servlet filter, async endpoints, rest template, scheduled actions, message channels, Zuul filters, and Feign client).

- Sleuth includes default logic to join a trace across HTTP or messaging boundaries. For example, HTTP propagation works over Zipkin-compatible request headers.

- Sleuth can propagate context (also known as baggage) between processes. Consequently, if you set a baggage element on a Span, it is sent downstream to other processes over either HTTP or messaging.

- Provides a way to create or continue spans and add tags and logs through annotations.

- If  `spring-cloud-sleuth-zipkin`  is on the classpath, the app generates and collects Zipkin-compatible traces. By default, it sends them over HTTP to a Zipkin server on localhost (port 9411). You can configure the location of the service by setting  `spring.zipkin.baseUrl` .

- If you depend on  `spring-rabbit` , your app sends traces to a RabbitMQ broker instead of HTTP.

- If you depend on  `spring-kafka` , and set  `spring.zipkin.sender.type: kafka` , your app sends traces to a Kafka broker instead of HTTP.

>  `spring-cloud-sleuth-stream`  is deprecated and should no longer be used.

- Spring Cloud Sleuth is [OpenTracing](http://opentracing.io/) compatible.

|images/important.png|Important|
|----|----|
|If you use Zipkin, configure the probability of spans exported by setting  `spring.sleuth.sampler.probability`  (default: 0.1, which is 10 percent). Otherwise, you might think that Sleuth is not working be cause it omits some spans. |

> The SLF4J MDC is always set and logback users immediately see the trace and span IDs in logs per the example shown earlier. Other logging systems have to configure their own formatter to get the same result. The default is as follows:  `logging.pattern.level`  set to  `%5p [${spring.zipkin.service.name:${spring.application.name:-}},%X{X-B3-TraceId:-},%X{X-B3-SpanId:-},%X{X-Span-Export:-}]`  (this is a Spring Boot feature for logback users). If you do not use SLF4J, this pattern is NOT automatically applied.

## 50.1 Introduction to Brave

|images/important.png|Important|
|----|----|
|Starting with version  `2.0.0` , Spring Cloud Sleuth uses [Brave](https://github.com/openzipkin/brave) as the tracing library. For your convenience, we embed part of the Brave’s docs here. |

|images/important.png|Important|
|----|----|
|In the vast majority of cases you need to just use the  `Tracer`  or  `SpanCustomizer`  beans from Brave that Sleuth provides. The documentation below contains a high overview of what Brave is and how it works. |

Brave is a library used to capture and report latency information about distributed operations to Zipkin. Most users do not use Brave directly. They use libraries or frameworks rather than employ Brave on their behalf.

This module includes a tracer that creates and joins spans that model the latency of potentially distributed work. It also includes libraries to propagate the trace context over network boundaries (for example, with HTTP headers).

### 50.1.1 Tracing

Most importantly, you need a  `brave.Tracer` , configured to [report to Zipkin](https://github.com/openzipkin/zipkin-reporter-java).

The following example setup sends trace data (spans) to Zipkin over HTTP (as opposed to Kafka):

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

|images/important.png|Important|
|----|----|
|If your span contains a name longer than 50 chars, then that name is truncated to 50 chars. Your names have to be explicit and concrete. Big names lead to latency issues and sometimes even thrown exceptions. |

The tracer creates and joins spans that model the latency of potentially distributed work. It can employ sampling to reduce overhead during the process, to reduce the amount of data sent to Zipkin, or both.

Spans returned by a tracer report data to Zipkin when finished or do nothing if unsampled. After starting a span, you can annotate events of interest or add tags containing details or lookup keys.

Spans have a context that includes trace identifiers that place the span at the correct spot in the tree representing the distributed operation.

### 50.1.2 Local Tracing

When tracing code that never leaves your process, run it inside a scoped span.

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

When you need more features, or finer control, use the  `Span`  type:

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

Both of the above examples report the exact same span on finish!

In the above example, the span will be either a new root span or the next child in an existing trace.

### 50.1.3 Customizing Spans

Once you have a span, you can add tags to it. The tags can be used as lookup keys or details. For example, you might add a tag with your runtime version, as shown in the following example:

```java
span.tag("clnt/finagle.version", "6.36.0");
```

When exposing the ability to customize spans to third parties, prefer  `brave.SpanCustomizer`  as opposed to  `brave.Span` . The former is simpler to understand and test and does not tempt users with span lifecycle hooks.

```java
interface MyTraceCallback {
void request(Request request, SpanCustomizer customizer);
}
```

Since  `brave.Span`  implements  `brave.SpanCustomizer` , you can pass it to users, as shown in the following example:

```java
for (MyTraceCallback callback : userCallbacks) {
callback.request(request, span);
}
```

### 50.1.4 Implicitly Looking up the Current Span

Sometimes, you do not know if a trace is in progress or not, and you do not want users to do null checks.  `brave.CurrentSpanCustomizer`  handles this problem by adding data to any span that’s in progress or drops, as shown in the following example:

Ex.

```java
// The user code can then inject this without a chance of it being null.
@Autowired SpanCustomizer span;

void userCode() {
span.annotate("tx.started");
...
}
```

### 50.1.5 RPC tracing

> Check for [instrumentation written here](https://github.com/openzipkin/brave/tree/master/instrumentation) and [Zipkin’s list](http://zipkin.io/pages/existing_instrumentations.html) before rolling your own RPC instrumentation.

RPC tracing is often done automatically by interceptors. Behind the scenes, they add tags and events that relate to their role in an RPC operation.

The following example shows how to add a client span:

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

#### One-Way tracing

Sometimes, you need to model an asynchronous operation where there is a request but no response. In normal RPC tracing, you use  `span.finish()`  to indicate that the response was received. In one-way tracing, you use  `span.flush()`  instead, as you do not expect a response.

The following example shows how a client might model a one-way operation:

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

The following example shows how a server might handle a one-way operation:

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

