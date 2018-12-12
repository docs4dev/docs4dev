## 57.命名范围

选择Span名称并非易事.范围名称应描述操作名称.名称应该是低基数，因此它不应包含标识符.

由于有很多仪器正在进行，因此一些Span名称是人为的：

当控制器收到方法名称为 `controllerMethodName` 的
-  `controller-method-name` 时

-  `async` 用于使用包装的 `Callable` 和 `Runnable` 接口完成的异步操作.

- Methods用 `@Scheduled` 注释返回类的简单名称.

幸运的是，对于异步处理，您可以提供显式命名.

## 57.1 @SpanName注释

您可以使用 `@SpanName` 注释显式命名范围，如以下示例所示：

```java
@SpanName("calculateTax")
class TaxCountingRunnable implements Runnable {

	@Override public void run() {
		// perform logic
	}
}
```

在这种情况下，当按以下方式处理时，Span名为 `calculateTax` ：

```java
Runnable runnable = new TraceRunnable(tracing, spanNamer,
		new TaxCountingRunnable());
Future<?> future = executorService.submit(runnable);
// ... some additional logic ...
future.get();
```

## 57.2 toString（）方法

为 `Runnable` 或 `Callable` 创建单独的类是非常罕见的.通常，会创建这些类的匿名实例.你不能注释这样的类.为了克服这个限制，如果没有 `@SpanName` 注释，我们检查该类是否具有 `toString()` 方法的自定义实现.

运行此类代码会导致创建名为 `calculateTax` 的span，如以下示例所示：

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

