## 56.Span生命周期

您可以通过 `brave.Tracer` 对Span执行以下操作：

- [start](multi__span_lifecycle.html#creating-and-finishing-spans)：启动范围时，将分配其名称并记录开始时间戳.

- [close](multi__span_lifecycle.html#creating-and-finishing-spans)：Span完成（记录Span的结束时间），如果Span被采样，则它有资格进行收集（例如，Zipkin）.

- [continue](multi__span_lifecycle.html#continuing-spans)：创建了一个新的span实例.它是继续它的副本.

- [detach](multi__span_lifecycle.html#continuing-spans)：范围未停止或关闭.它只会从当前线程中删除.

- [create with explicit parent](multi__span_lifecycle.html#creating-spans-with-explicit-parent)：您可以创建新的范围并为其设置显式父级.

> Spring Cloud Sleuth为您创建 `Tracer` 的实例.为了使用它，您可以自动装配它.

## 56.1创建和完成Span

您可以使用 `Tracer` 手动创建跨距，如以下示例所示：

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

在前面的示例中，我们可以看到如何创建span的新实例.如果此线程中已存在Span，则它将成为新Span的父级.

|图片/ important.png |重要|
| ---- | ---- |
|创建Span后始终清洁.此外，始终完成您要发送给Zipkin的任何范围. |

|图片/ important.png |重要|
| ---- | ---- |
|如果您的span包含的名称大于50个字符，则该名称将被截断为50个字符.你的名字必须明确而具体.大名称会导致延迟问题，有时甚至是异常. |

## 56.2持续Span

有时，您不想创建新的Span，但是您想要继续创建新的Span.这种情况的一个例子可能如下：

-  **AOP** ：如果在到达方面之前已经创建了Span，则可能不希望创建新Span.

-  **Hystrix** ：执行Hystrix命令很可能是当前处理的逻辑部分.事实上，这只是一个技术实现细节，您不一定要在跟踪中反映为单独的存在.

要继续Span，可以使用 `brave.Tracer` ，如以下示例所示：

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

## 56.3使用显式Parent创建Span

您可能希望启动新范围并提供该范围的显式父级.假设span的父级在一个线程中，并且您希望在另一个线程中启动新的span.在Brave中，无论何时调用 `nextSpan()` ，它都会创建一个范围，以引用当前在范围内的范围.您可以将范围放在范围内，然后调用 `nextSpan()` ，如图所示以下示例：

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

|图片/ important.png |重要|
| ---- | ---- |
|创建这样的Span后，您必须完成它.否则不报告（例如，Zipkin）. |

