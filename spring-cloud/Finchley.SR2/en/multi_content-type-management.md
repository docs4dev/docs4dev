## 30. Content Type Negotiation

Data transformation is one of the core features of any message-driven microservice architecture. Given that, in Spring Cloud Stream, such data is represented as a Spring  `Message` , a message may have to be transformed to a desired shape or size before reaching its destination. This is required for two reasons:

To convert the contents of the incoming message to match the signature of the application-provided handler. To convert the contents of the outgoing message to the wire format.

The wire format is typically  `byte[]`  (that is true for the Kafka and Rabbit binders), but it is governed by the binder implementation.

In Spring Cloud Stream, message transformation is accomplished with an  `org.springframework.messaging.converter.MessageConverter` .

> As a supplement to the details to follow, you may also want to read the following [blog post](https://spring.io/blog/2018/02/26/spring-cloud-stream-2-0-content-type-negotiation-and-transformation).

## 30.1 Mechanics

To better understand the mechanics and the necessity behind content-type negotiation, we take a look at a very simple use case by using the following message handler as an example:

```java
@StreamListener(Processor.INPUT)
@SendTo(Processor.OUTPUT)
public String handle(Person person) {..}
```

> For simplicity, we assume that this is the only handler in the application (we assume there is no internal pipeline).

The handler shown in the preceding example expects a  `Person`  object as an argument and produces a  `String`  type as an output. In order for the framework to succeed in passing the incoming  `Message`  as an argument to this handler, it has to somehow transform the payload of the  `Message`  type from the wire format to a  `Person`  type. In other words, the framework must locate and apply the appropriate  `MessageConverter` . To accomplish that, the framework needs some instructions from the user. One of these instructions is already provided by the signature of the handler method itself ( `Person`  type). Consequently, in theory, that should be (and, in some cases, is) enough. However, for the majority of use cases, in order to select the appropriate  `MessageConverter` , the framework needs an additional piece of information. That missing piece is  `contentType` .

Spring Cloud Stream provides three mechanisms to define  `contentType`  (in order of precedence):

HEADER: The contentType can be communicated through the Message itself. By providing a contentType header, you declare the content type to use to locate and apply the appropriate MessageConverter. BINDING: The contentType can be set per destination binding by setting the spring.cloud.stream.bindings.input.content-type property. Note The input segment in the property name corresponds to the actual name of the destination (which is “input” in our case). This approach lets you declare, on a per-binding basis, the content type to use to locate and apply the appropriate MessageConverter. DEFAULT: If contentType is not present in the Message header or the binding, the default application/json content type is used to locate and apply the appropriate MessageConverter.

As mentioned earlier, the preceding list also demonstrates the order of precedence in case of a tie. For example, a header-provided content type takes precedence over any other content type. The same applies for a content type set on a per-binding basis, which essentially lets you override the default content type. However, it also provides a sensible default (which was determined from community feedback).

Another reason for making  `application/json`  the default stems from the interoperability requirements driven by distributed microservices architectures, where producer and consumer not only run in different JVMs but can also run on different non-JVM platforms.

When the non-void handler method returns, if the the return value is already a  `Message` , that  `Message`  becomes the payload. However, when the return value is not a  `Message` , the new  `Message`  is constructed with the return value as the payload while inheriting headers from the input  `Message`  minus the headers defined or filtered by  `SpringIntegrationProperties.messageHandlerNotPropagatedHeaders` . By default, there is only one header set there:  `contentType` . This means that the new  `Message`  does not have  `contentType`  header set, thus ensuring that the  `contentType`  can evolve. You can always opt out of returning a  `Message`  from the handler method where you can inject any header you wish.

If there is an internal pipeline, the  `Message`  is sent to the next handler by going through the same process of conversion. However, if there is no internal pipeline or you have reached the end of it, the  `Message`  is sent back to the output destination.

### 30.1.1 Content Type versus Argument Type

As mentioned earlier, for the framework to select the appropriate  `MessageConverter` , it requires argument type and, optionally, content type information. The logic for selecting the appropriate  `MessageConverter`  resides with the argument resolvers ( `HandlerMethodArgumentResolvers` ), which trigger right before the invocation of the user-defined handler method (which is when the actual argument type is known to the framework). If the argument type does not match the type of the current payload, the framework delegates to the stack of the pre-configured  `MessageConverters`  to see if any one of them can convert the payload. As you can see, the  `Object fromMessage(Message<?> message, Class<?> targetClass);`  operation of the MessageConverter takes  `targetClass`  as one of its arguments. The framework also ensures that the provided  `Message`  always contains a  `contentType`  header. When no contentType header was already present, it injects either the per-binding  `contentType`  header or the default  `contentType`  header. The combination of  `contentType`  argument type is the mechanism by which framework determines if message can be converted to a target type. If no appropriate  `MessageConverter`  is found, an exception is thrown, which you can handle by adding a custom  `MessageConverter`  (see “[Section 30.3, “User-defined Message Converters”](multi_content-type-management.html#spring-cloud-stream-overview-user-defined-message-converters)”).

But what if the payload type matches the target type declared by the handler method? In this case, there is nothing to convert, and the payload is passed unmodified. While this sounds pretty straightforward and logical, keep in mind handler methods that take a  `Message<?>`  or  `Object`  as an argument. By declaring the target type to be  `Object`  (which is an  `instanceof`  everything in Java), you essentially forfeit the conversion process.

> Do not expect  `Message`  to be converted into some other type based only on the  `contentType` . Remember that the  `contentType`  is complementary to the target type. If you wish, you can provide a hint, which  `MessageConverter`  may or may not take into consideration.

### 30.1.2 Message Converters

`MessageConverters`  define two methods:

```java
Object fromMessage(Message<?> message, Class<?> targetClass);

Message<?> toMessage(Object payload, @Nullable MessageHeaders headers);
```

It is important to understand the contract of these methods and their usage, specifically in the context of Spring Cloud Stream.

The  `fromMessage`  method converts an incoming  `Message`  to an argument type. The payload of the  `Message`  could be any type, and it is up to the actual implementation of the  `MessageConverter`  to support multiple types. For example, some JSON converter may support the payload type as  `byte[]` ,  `String` , and others. This is important when the application contains an internal pipeline (that is, input → handler1 → handler2 →. . . → output) and the output of the upstream handler results in a  `Message`  which may not be in the initial wire format.

However, the  `toMessage`  method has a more strict contract and must always convert  `Message`  to the wire format:  `byte[]` .

So, for all intents and purposes (and especially when implementing your own converter) you regard the two methods as having the following signatures:

```xml
Object fromMessage(Message<?> message, Class<?> targetClass);

Message<byte[]> toMessage(Object payload, @Nullable MessageHeaders headers);
```

## 30.2 Provided MessageConverters

As mentioned earlier, the framework already provides a stack of  `MessageConverters`  to handle most common use cases. The following list describes the provided  `MessageConverters` , in order of precedence (the first  `MessageConverter`  that works is used):

ApplicationJsonMessageMarshallingConverter: Variation of the org.springframework.messaging.converter.MappingJackson2MessageConverter. Supports conversion of the payload of the Message to/from POJO for cases when contentType is application/json (DEFAULT). TupleJsonMessageConverter: DEPRECATED Supports conversion of the payload of the Message to/from org.springframework.tuple.Tuple. ByteArrayMessageConverter: Supports conversion of the payload of the Message from byte[] to byte[] for cases when contentType is application/octet-stream. It is essentially a pass through and exists primarily for backward compatibility. ObjectStringMessageConverter: Supports conversion of any type to a String when contentType is text/plain. It invokes Object’s toString() method or, if the payload is byte[], a new String(byte[]). JavaSerializationMessageConverter: DEPRECATED Supports conversion based on java serialization when contentType is application/x-java-serialized-object. KryoMessageConverter: DEPRECATED Supports conversion based on Kryo serialization when contentType is application/x-java-object. JsonUnmarshallingConverter: Similar to the ApplicationJsonMessageMarshallingConverter. It supports conversion of any type when contentType is application/x-java-object. It expects the actual type information to be embedded in the contentType as an attribute (for example, application/x-java-object;type=foo.bar.Cat).

When no appropriate converter is found, the framework throws an exception. When that happens, you should check your code and configuration and ensure you did not miss anything (that is, ensure that you provided a  `contentType`  by using a binding or a header). However, most likely, you found some uncommon case (such as a custom  `contentType`  perhaps) and the current stack of provided  `MessageConverters`  does not know how to convert. If that is the case, you can add custom  `MessageConverter` . See [Section 30.3, “User-defined Message Converters”](multi_content-type-management.html#spring-cloud-stream-overview-user-defined-message-converters).

## 30.3 User-defined Message Converters

Spring Cloud Stream exposes a mechanism to define and register additional  `MessageConverters` . To use it, implement  `org.springframework.messaging.converter.MessageConverter` , configure it as a  `@Bean` , and annotate it with  `@StreamMessageConverter` . It is then apended to the existing stack of `MessageConverter`s.

> It is important to understand that custom  `MessageConverter`  implementations are added to the head of the existing stack. Consequently, custom  `MessageConverter`  implementations take precedence over the existing ones, which lets you override as well as add to the existing converters.

The following example shows how to create a message converter bean to support a new content type called  `application/bar` :

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

Spring Cloud Stream also provides support for Avro-based converters and schema evolution. See “[Chapter 31, Schema Evolution Support](multi_schema-evolution.html)” for details.

