## 56. Span lifecycle

You can do the following operations on the Span by means of  `brave.Tracer` :

- [start](multi__span_lifecycle.html#creating-and-finishing-spans): When you start a span, its name is assigned and the start timestamp is recorded.

- [close](multi__span_lifecycle.html#creating-and-finishing-spans): The span gets finished (the end time of the span is recorded) and, if the span is sampled, it is eligible for collection (for example, to Zipkin).

- [continue](multi__span_lifecycle.html#continuing-spans): A new instance of span is created. It is a copy of the one that it continues.

- [detach](multi__span_lifecycle.html#continuing-spans): The span does not get stopped or closed. It only gets removed from the current thread.

- [create with explicit parent](multi__span_lifecycle.html#creating-spans-with-explicit-parent): You can create a new span and set an explicit parent for it.

> Spring Cloud Sleuth creates an instance of  `Tracer`  for you. In order to use it, you can autowire it.

## 56.1 Creating and finishing spans

You can manually create spans by using the  `Tracer` , as shown in the following example:

```java
// Start a span. If there was a span present in this thread it will become
// the `newSpan`'s parent.
Span newSpan = this.tracer.nextSpan().name("calculateTax");
try (Tracer.SpanInScope ws = this.tracer.withSpanInScope(newSpan.start())) {
	// ...
	// You can tag a span
	newSpan.tag("taxValue", taxValue);
	// ...
	// You can log an event on a span
	newSpan.annotate("taxCalculated");
} finally {
	// Once done remember to finish the span. This will allow collecting
	// the span to send it to Zipkin
	newSpan.finish();
}
```

In the preceding example, we could see how to create a new instance of the span. If there is already a span in this thread, it becomes the parent of the new span.

|images/important.png|Important|
|----|----|
|Always clean after you create a span. Also, always finish any span that you want to send to Zipkin. |

|images/important.png|Important|
|----|----|
|If your span contains a name greater than 50 chars, that name is truncated to 50 chars. Your names have to be explicit and concrete. Big names lead to latency issues and sometimes even exceptions. |

## 56.2 Continuing Spans

Sometimes, you do not want to create a new span but you want to continue one. An example of such a situation might be as follows:

-  **AOP** : If there was already a span created before an aspect was reached, you might not want to create a new span.

-  **Hystrix** : Executing a Hystrix command is most likely a logical part of the current processing. It is in fact merely a technical implementation detail that you would not necessarily want to reflect in tracing as a separate being.

To continue a span, you can use  `brave.Tracer` , as shown in the following example:

```java
// let's assume that we're in a thread Y and we've received
// the `initialSpan` from thread X
Span continuedSpan = this.tracer.toSpan(newSpan.context());
try {
	// ...
	// You can tag a span
	continuedSpan.tag("taxValue", taxValue);
	// ...
	// You can log an event on a span
	continuedSpan.annotate("taxCalculated");
} finally {
	// Once done remember to flush the span. That means that
	// it will get reported but the span itself is not yet finished
	continuedSpan.flush();
}
```

## 56.3 Creating a Span with an explicit Parent

You might want to start a new span and provide an explicit parent of that span. Assume that the parent of a span is in one thread and you want to start a new span in another thread. In Brave, whenever you call  `nextSpan()` , it creates a span in reference to the span that is currently in scope. You can put the span in scope and then call  `nextSpan()` , as shown in the following example:

```java
// let's assume that we're in a thread Y and we've received
// the `initialSpan` from thread X. `initialSpan` will be the parent
// of the `newSpan`
Span newSpan = null;
try (Tracer.SpanInScope ws = this.tracer.withSpanInScope(initialSpan)) {
	newSpan = this.tracer.nextSpan().name("calculateCommission");
	// ...
	// You can tag a span
	newSpan.tag("commissionValue", commissionValue);
	// ...
	// You can log an event on a span
	newSpan.annotate("commissionCalculated");
} finally {
	// Once done remember to finish the span. This will allow collecting
	// the span to send it to Zipkin. The tags and events set on the
	// newSpan will not be present on the parent
	if (newSpan != null) {
		newSpan.finish();
	}
}
```

|images/important.png|Important|
|----|----|
|After creating such a span, you must finish it. Otherwise it is not reported (for example, to Zipkin). |

