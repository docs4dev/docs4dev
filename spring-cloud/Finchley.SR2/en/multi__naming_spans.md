## 57. Naming spans

Picking a span name is not a trivial task. A span name should depict an operation name. The name should be low cardinality, so it should not include identifiers.

Since there is a lot of instrumentation going on, some span names are artificial:

-  `controller-method-name`  when received by a Controller with a method name of  `controllerMethodName` 

-  `async`  for asynchronous operations done with wrapped  `Callable`  and  `Runnable`  interfaces.

- Methods annotated with  `@Scheduled`  return the simple name of the class.

Fortunately, for asynchronous processing, you can provide explicit naming.

## 57.1 @SpanName Annotation

You can name the span explicitly by using the  `@SpanName`  annotation, as shown in the following example:

```java
@SpanName("calculateTax")
class TaxCountingRunnable implements Runnable {

	@Override public void run() {
		// perform logic
	}
}
```

In this case, when processed in the following manner, the span is named  `calculateTax` :

```java
Runnable runnable = new TraceRunnable(tracing, spanNamer,
		new TaxCountingRunnable());
Future<?> future = executorService.submit(runnable);
// ... some additional logic ...
future.get();
```

## 57.2 toString() method

It is pretty rare to create separate classes for  `Runnable`  or  `Callable` . Typically, one creates an anonymous instance of those classes. You cannot annotate such classes. To overcome that limitation, if there is no  `@SpanName`  annotation present, we check whether the class has a custom implementation of the  `toString()`  method.

Running such code leads to creating a span named  `calculateTax` , as shown in the following example:

```java
Runnable runnable = new TraceRunnable(tracing, spanNamer, new Runnable() {
	@Override public void run() {
		// perform logic
	}

	@Override public String toString() {
		return "calculateTax";
	}
});
Future<?> future = executorService.submit(runnable);
// ... some additional logic ...
future.get();
```

