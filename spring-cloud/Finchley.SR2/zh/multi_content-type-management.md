## 30.内容类型谈判

数据转换是任何消息驱动的微服务架构的核心功能之一.鉴于此，在Spring Cloud Stream中，此类数据表示为Spring  `Message` ，在到达目标之前，可能必须将消息转换为所需的形状或大小.这有两个原因：

转换传入消息的内容以匹配应用程序提供的处理程序的签名.将传出消息的内容转换为有线格式.

有线格式通常为 `byte[]` （对于Kafka和RabbitBinders也是如此），但它受Binders实现的约束.

在Spring Cloud Stream中，使用 `org.springframework.messaging.converter.MessageConverter` 完成消息转换.

> 作为要遵循的细节的补充，您可能还想阅读以下[blog post](https://spring.io/blog/2018/02/26/spring-cloud-stream-2-0-content-type-negotiation-and-transformation).

## 30.1力学

为了更好地理解内容类型协商背后的机制和必要性，我们通过使用以下消息处理程序作为示例来查看一个非常简单的用例：

```java
@StreamListener(Processor.INPUT)
@SendTo(Processor.OUTPUT)
public String handle(Person person) {..}
```

> 为简单起见，我们假设这是应用程序中唯一的处理程序（我们假设没有内部管道）.

前面示例中显示的处理程序需要 `Person` 对象作为参数，并生成 `String` 类型作为输出.为了使框架成功将传入的 `Message` 作为参数传递给此处理程序，它必须以某种方式将 `Message` 类型的有效负载从有线格式转换为 `Person` 类型.换句话说，框架必须找到并应用适当的 `MessageConverter` .为此，框架需要用户的一些指示.其中一条指令已由处理程序方法本身的签名提供（ `Person` 类型）.因此，从理论上讲，这应该（并且在某些情况下）应该足够了.但是，对于大多数用例，为了选择合适的 `MessageConverter` ，框架需要一条额外的信息.缺少的那块是 `contentType` .

Spring Cloud Stream提供了三种机制来定义 `contentType` （按优先顺序排列）：

HEADER：contentType可以通过Message本身进行通信.通过提供contentType标头，您可以声明用于查找和应用相应MessageConverter的内容类型. BINDING：可以通过设置spring.cloud.stream.bindings.input.content-type属性为每个目标绑定设置contentType.注意属性名称中的输入段对应于目标的实际名称（在我们的示例中为“输入”）.此方法允许您在每个绑定的基础上声明用于查找和应用相应MessageConverter的内容类型. DEFAULT：如果MessageType或绑定中不存在contentType，则使用默认的application / json内容类型来定位和应用相应的MessageConverter.

如前所述，前面的列表还演示了绑定情况下的优先顺序.例如，标头提供的内容类型优先于任何其他内容类型.这同样适用于基于每个绑定设置的内容类型，它基本上允许您覆盖默认内容类型.但是，它也提供了合理的默认值（根据社区反馈确定）.

使 `application/json` 成为默认值的另一个原因源于分布式微服务架构驱动的互操作性要求，其中生产环境者和消费者不仅在不同的JVM中运行，而且还可以在不同的非JVM平台上运行.

当非void处理程序方法返回时，如果返回值已经是 `Message` ，则 `Message` 成为有效负载.但是，当返回值不是 `Message` 时，新的 `Message` 构造为返回值作为有效负载，同时继承输入 `Message` 的标头减去由 `SpringIntegrationProperties.messageHandlerNotPropagatedHeaders` 定义或过滤的标头.默认情况下，只有一个标头集： `contentType` .这意味着新 `Message` 没有设置 `contentType` 标头，从而确保 `contentType` 可以进化.您始终可以选择不从处理程序方法返回 `Message` ，您可以在其中注入任何所需的标头.

如果存在内部管道，则通过执行相同的转换过程将 `Message` 发送到下一个处理程序.但是，如果没有内部管道或者您已到达它的末尾，则 `Message` 将被发送回输出目标.

### 30.1.1内容类型与参数类型

如前所述，对于框架来选择合适的 `MessageConverter` ，它需要参数类型和（可选）内容类型信息.选择适当的 `MessageConverter` 的逻辑与参数解析器（ `HandlerMethodArgumentResolvers` ）一起存在，它在调用用户定义的处理程序方法之前触发（这是框架已知实际参数类型的时候）.如果参数类型与当前有效负载的类型不匹配，则框架委托给预先配置的 `MessageConverters` 的堆栈，以查看它们中的任何一个是否可以转换有效负载.如您所见，MessageConverter的 `Object fromMessage(Message<?> message, Class<?> targetClass);` 操作将 `targetClass` 作为其参数之一.该框架还确保提供的 `Message` 始终包含 `contentType` 标头.如果没有contentType标头，则它会注入每个绑定 `contentType` 标头或默认的 `contentType` 标头.  `contentType` 参数类型的组合是框架确定消息是否可以转换为目标类型的机制.如果找不到合适的 `MessageConverter` ，则抛出异常，您可以通过添加自定义 `MessageConverter` 来处理该异常（请参阅“[Section 30.3, “User-defined Message Converters”](multi_content-type-management.html#spring-cloud-stream-overview-user-defined-message-converters)”）.

但是如果有效负载类型与处理程序方法声明的目标类型匹配怎么办？在这种情况下，没有任何东西可以转换，并且有效载荷是未经修改的.虽然这听起来非常简单和合乎逻辑，但请记住以 `Message<?>` 或 `Object` 为参数的处理程序方法.通过将目标类型声明为 `Object` （这是Java中的 `instanceof` 所有内容），您基本上会丧失转换过程.

> 不要指望 `Message` 仅基于 `contentType` 转换为其他类型.请记住， `contentType` 是目标类型的补充.如果您愿意，可以提供一个提示，可能会或可能不会考虑.

### 30.1.2消息转换器

`MessageConverters` 定义两种方法：

```java
Object fromMessage(Message<?> message, Class<?> targetClass);

Message<?> toMessage(Object payload, @Nullable MessageHeaders headers);
```

了解这些方法及其用法的Contract非常重要，特别是在Spring Cloud Stream的背景下.

`fromMessage` 方法将传入的 `Message` 转换为参数类型.  `Message` 的有效负载可以是任何类型，并且由 `MessageConverter` 的实际实现来支持多种类型.例如，某些JSON转换器可能支持有效负载类型为 `byte[]` ， `String` 等.当应用程序包含内部管道（即输入→处理程序1→处理程序2→...→输出）并且上游处理程序的输出产生 `Message` （可能不是初始有线格式）时，这很重要.

但是， `toMessage` 方法具有更严格的约定，并且必须始终将 `Message` 转换为有线格式： `byte[]` .

因此，对于所有意图和目的（尤其是在实现您自己的转换器时），您认为这两种方法具有以下签名：

```xml
Object fromMessage(Message<?> message, Class<?> targetClass);

Message<byte[]> toMessage(Object payload, @Nullable MessageHeaders headers);
```

## 30.2提供了MessageConverters

如前所述，框架已经提供了一堆 `MessageConverters` 来处理大多数常见用例.以下列表按优先顺序描述了提供的 `MessageConverters` （使用的第一个 `MessageConverter` ）：

ApplicationJsonMessageMarshallingConverter：org.springframework.messaging.converter.MappingJackson2MessageConverter的变体.对于contentType为application / json（DEFAULT）的情况，支持将消息的有效负载转换为POJO或从POJO转换消息的有效负载. TupleJsonMessageConverter：DEPRECATED支持将消息的有效负载转换为org.springframework.tuple.Tuple. ByteArrayMessageConverter：当contentType为application / octet-stream时，支持将Message的有效负载从byte []转换为byte [].它本质上是一种传递，主要用于向后兼容. ObjectStringMessageConverter：当contentType为text / plain时，支持将任何类型转换为String.它调用Object的toString（）方法，或者如果有效负载是byte []，则调用一个新的String（byte []）. JavaSerializationMessageConverter：DEPRECATED当contentType为application / x-java-serialized-object时，支持基于java序列化的转换. KryoMessageConverter：DEPRECATED当contentType为application / x-java-object时，支持基于Kryo序列化的转换. JsonUnmarshallingConverter：类似于ApplicationJsonMessageMarshallingConverter.当contentType是application / x-java-object时，它支持任何类型的转换.它期望将实际类型信息作为属性嵌入到contentType中（例如，application / x-java-object; type = foo.bar.Cat）.

如果找不到合适的转换器，框架将抛出异常.当发生这种情况时，您应检查您的代码和配置，并确保您没有遗漏任何内容（即，确保您使用绑定或标头提供了 `contentType` ）.但是，最有可能的是，您发现了一些不常见的情况（例如自定义 `contentType` ），并且当前提供的 `MessageConverters` 堆栈不知道如何转换.如果是这种情况，您可以添加自定义 `MessageConverter` .见[Section 30.3, “User-defined Message Converters”](multi_content-type-management.html#spring-cloud-stream-overview-user-defined-message-converters).

## 30.3用户定义的消息转换器

Spring Cloud Stream公开了一种定义和注册的机制额外 `MessageConverters` .要使用它，请实现 `org.springframework.messaging.converter.MessageConverter` ，将其配置为 `@Bean` ，并使用 `@StreamMessageConverter` 进行注释.然后将它附加到`MessageConverter`s的现有堆栈上.

> 了解自定义 `MessageConverter` 实现被添加到现有堆栈的头部非常重要.因此，自定义 `MessageConverter` 实现优先于现有实现，这使您可以覆盖以及添加到现有转换器.

以下示例显示如何创建消息转换器bean以支持名为 `application/bar` 的新内容类型：

```java
@EnableBinding(Sink.class)
@SpringBootApplication
public static class SinkApplication {

...

@Bean
@StreamMessageConverter
public MessageConverter customMessageConverter() {
return new MyCustomMessageConverter();
}
}

public class MyCustomMessageConverter extends AbstractMessageConverter {

public MyCustomMessageConverter() {
super(new MimeType("application", "bar"));
}

@Override
protected boolean supports(Class<?> clazz) {
return (Bar.class.equals(clazz));
}

@Override
protected Object convertFromInternal(Message<?> message, Class<?> targetClass, Object conversionHint) {
Object payload = message.getPayload();
return (payload instanceof Bar ? payload : new Bar((byte[]) payload));
}
}
```

Spring Cloud Stream还为基于Avro的转换器和模式演变提供支持.有关详细信息，请参阅“[Chapter 31, Schema Evolution Support](multi_schema-evolution.html)”.

