## 58.使用注释管理Span

您可以使用各种注释来管理Span.

## 58.1理由

使用注释管理Span有很多很好的理由，包括：

- API-agnostic意味着与Span协作.使用注释可以让用户添加Spanapi上没有库依赖的Span.这样做可以让Sleuth更改其核心API，从而减少对用户代码的影响.

- Reduced表面区域用于基本Span操作.如果没有此功能，则必须使用span api，它具有可能无法正确使用的生命周期命令.通过仅公开范围，标记和日志功能，您可以进行协作，而不会意外地破坏Span生命周期.

- 与运行时生成的代码协作.使用Spring Data和Feign等库，可以在运行时生成接口的实现.因此，对象的Span包装是繁琐的.现在，您可以通过接口和这些接口的参数提供注释.

## 58.2创建新Span

如果您不想手动创建本地Span，则可以使用 `@NewSpan` 注释.此外，我们提供 `@SpanTag` 注释以自动方式添加标签.

现在我们可以考虑一些使用示例.

```java
@NewSpan
void testMethod();
```

在没有任何参数的情况下对方法进行注释会导致创建一个新的span，其名称等于带注释的方法名称.

```java
@NewSpan("customNameOnTestMethod4")
void testMethod4();
```

如果在注释中提供值（直接或通过设置 `name` 参数），则创建的范围将提供值作为名称.

```java
// method declaration
@NewSpan(name = "customNameOnTestMethod5")
void testMethod5(@SpanTag("testTag") String param);

// and method execution
this.testBean.testMethod5("test");
```

您可以将名称和标签结合使用.让我们关注后者.在这种情况下，带注释的方法的参数运行时值的值将成为标记的值.在我们的示例中，标记键是 `testTag` ，标记值是 `test` .

```java
@NewSpan(name = "customNameOnTestMethod3")
@Override
public void testMethod3() {
}
```

您可以在类和接口上放置 `@NewSpan` 注释.如果覆盖接口的方法并为 `@NewSpan` 注释提供不同的值，则最具体的一个获胜（在这种情况下设置为 `customNameOnTestMethod3` ）.

## 58.3持续Span

如果要向现有范围添加标记和注释，可以使用 `@ContinueSpan` 注释，如以下示例所示：

```java
// method declaration
@ContinueSpan(log = "testMethod11")
void testMethod11(@SpanTag("testTag11") String param);

// method execution
this.testBean.testMethod11("test");
this.testBean.testMethod13();
```

（请注意，与 `@NewSpan` 注释相比，您还可以使用 `log` 参数添加日志.）

这样，Span继续下去：

创建名为 `testMethod11.before` 和 `testMethod11.after` 的
- Log条目.

- 如果抛出异常，还会创建名为 `testMethod11.afterFailure` 的日志条目.
创建_399_A标记，其键为 `testTag11` ，值为 `test` .

## 58.4高级标签设置

有三种不同的方法可以为Span添加标签.所有这些都由 `SpanTag` 注释控制.优先顺序如下：

尝试使用TagValueResolver类型的bean和提供的名称.如果尚未提供bean名称，请尝试计算表达式.我们搜索TagValueExpressionResolver bean.默认实现使用SPEL表达式解析.重要事项您只能引用SPEL表达式中的属性.由于安全性限制，不允许执行方法.如果我们找不到任何要计算的表达式，则返回参数的toString（）值.

### 58.4.1自定义提取器

以下方法的标记值由 `TagValueResolver` 接口的实现计算.其类名必须作为 `resolver` 属性的值传递.

请考虑以下带注释的方法：

```java
@NewSpan
public void getAnnotationForTagValueResolver(@SpanTag(key = "test", resolver = TagValueResolver.class) String test) {
}
```

现在进一步考虑以下 `TagValueResolver`  bean实现：

```java
@Bean(name = "myCustomTagValueResolver")
public TagValueResolver tagValueResolver() {
	return parameter -> "Value from myCustomTagValueResolver";
}
```

前两个例子导致将标记值设置为 `Value from myCustomTagValueResolver` .

### 58.4.2解析值的表达式

请考虑以下带注释的方法：

```java
@NewSpan
public void getAnnotationForTagValueExpression(@SpanTag(key = "test", expression = "'hello' + ' characters'") String test) {
}
```

没有 `TagValueExpressionResolver` 的自定义实现导致评估SPEL表达式，并且在Span上设置值为 `4 characters` 的标记.如果要使用其他表达式解析机制，可以创建自己的bean实现.

### 58.4.3使用toString（）方法

请考虑以下带注释的方法：

```java
@NewSpan
public void getAnnotationForArgumentToString(@SpanTag("test") Long param) {
}
```

使用值 `15` 运行上述方法会导致设置String值为 `"15"` 的标记.

