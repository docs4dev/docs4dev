## 58. Managing Spans with Annotations

You can manage spans with a variety of annotations.

## 58.1 Rationale

There are a number of good reasons to manage spans with annotations, including:

- API-agnostic means to collaborate with a span. Use of annotations lets users add to a span with no library dependency on a span api. Doing so lets Sleuth change its core API to create less impact to user code.

- Reduced surface area for basic span operations. Without this feature, you must use the span api, which has lifecycle commands that could be used incorrectly. By only exposing scope, tag, and log functionality, you can collaborate without accidentally breaking span lifecycle.

- Collaboration with runtime generated code. With libraries such as Spring Data and Feign, the implementations of interfaces are generated at runtime. Consequently, span wrapping of objects was tedious. Now you can provide annotations over interfaces and the arguments of those interfaces.

## 58.2 Creating New Spans

If you do not want to create local spans manually, you can use the  `@NewSpan`  annotation. Also, we provide the  `@SpanTag`  annotation to add tags in an automated fashion.

Now we can consider some examples of usage.

```java
@NewSpan
void testMethod();
```

Annotating the method without any parameter leads to creating a new span whose name equals the annotated method name.

```java
@NewSpan("customNameOnTestMethod4")
void testMethod4();
```

If you provide the value in the annotation (either directly or by setting the  `name`  parameter), the created span has the provided value as the name.

```java
// method declaration
@NewSpan(name = "customNameOnTestMethod5")
void testMethod5(@SpanTag("testTag") String param);

// and method execution
this.testBean.testMethod5("test");
```

You can combine both the name and a tag. Let’s focus on the latter. In this case, the value of the annotated method’s parameter runtime value becomes the value of the tag. In our sample, the tag key is  `testTag` , and the tag value is  `test` .

```java
@NewSpan(name = "customNameOnTestMethod3")
@Override
public void testMethod3() {
}
```

You can place the  `@NewSpan`  annotation on both the class and an interface. If you override the interface’s method and provide a different value for the  `@NewSpan`  annotation, the most concrete one wins (in this case  `customNameOnTestMethod3`  is set).

## 58.3 Continuing Spans

If you want to add tags and annotations to an existing span, you can use the  `@ContinueSpan`  annotation, as shown in the following example:

```java
// method declaration
@ContinueSpan(log = "testMethod11")
void testMethod11(@SpanTag("testTag11") String param);

// method execution
this.testBean.testMethod11("test");
this.testBean.testMethod13();
```

(Note that, in contrast with the  `@NewSpan`  annotation ,you can also add logs with the  `log`  parameter.)

That way, the span gets continued and:

- Log entries named  `testMethod11.before`  and  `testMethod11.after`  are created.

- If an exception is thrown, a log entry named  `testMethod11.afterFailure`  is also created.

- A tag with a key of  `testTag11`  and a value of  `test`  is created.

## 58.4 Advanced Tag Setting

There are 3 different ways to add tags to a span. All of them are controlled by the  `SpanTag`  annotation. The precedence is as follows:

Try with a bean of TagValueResolver type and a provided name. If the bean name has not been provided, try to evaluate an expression. We search for a TagValueExpressionResolver bean. The default implementation uses SPEL expression resolution. IMPORTANT You can only reference properties from the SPEL expression. Method execution is not allowed due to security constraints. If we do not find any expression to evaluate, return the toString() value of the parameter.

### 58.4.1 Custom extractor

The value of the tag for the following method is computed by an implementation of  `TagValueResolver`  interface. Its class name has to be passed as the value of the  `resolver`  attribute.

Consider the following annotated method:

```java
@NewSpan
public void getAnnotationForTagValueResolver(@SpanTag(key = "test", resolver = TagValueResolver.class) String test) {
}
```

Now further consider the following  `TagValueResolver`  bean implementation:

```java
@Bean(name = "myCustomTagValueResolver")
public TagValueResolver tagValueResolver() {
	return parameter -> "Value from myCustomTagValueResolver";
}
```

The two preceding examples lead to setting a tag value equal to  `Value from myCustomTagValueResolver` .

### 58.4.2 Resolving Expressions for a Value

Consider the following annotated method:

```java
@NewSpan
public void getAnnotationForTagValueExpression(@SpanTag(key = "test", expression = "'hello' + ' characters'") String test) {
}
```

No custom implementation of a  `TagValueExpressionResolver`  leads to evaluation of the SPEL expression, and a tag with a value of  `4 characters`  is set on the span. If you want to use some other expression resolution mechanism, you can create your own implementation of the bean.

### 58.4.3 Using the toString() method

Consider the following annotated method:

```java
@NewSpan
public void getAnnotationForArgumentToString(@SpanTag("test") Long param) {
}
```

Running the preceding method with a value of  `15`  leads to setting a tag with a String value of  `"15"` .

