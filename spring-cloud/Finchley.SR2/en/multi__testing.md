## 33. Testing

Spring Cloud Stream provides support for testing your microservice applications without connecting to a messaging system. You can do that by using the  `TestSupportBinder`  provided by the  `spring-cloud-stream-test-support`  library, which can be added as a test dependency to the application, as shown in the following example:

```xml
<dependency>
<groupId>org.springframework.cloud</groupId>
<artifactId>spring-cloud-stream-test-support</artifactId>
<scope>test</scope>
</dependency>
```

> The  `TestSupportBinder`  uses the Spring Boot autoconfiguration mechanism to supersede the other binders found on the classpath. Therefore, when adding a binder as a dependency, you must make sure that the  `test`  scope is being used.

The  `TestSupportBinder`  lets you interact with the bound channels and inspect any messages sent and received by the application.

For outbound message channels, the  `TestSupportBinder`  registers a single subscriber and retains the messages emitted by the application in a  `MessageCollector` . They can be retrieved during tests and have assertions made against them.

You can also send messages to inbound message channels so that the consumer application can consume the messages. The following example shows how to test both input and output channels on a processor:

```java
@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment= SpringBootTest.WebEnvironment.RANDOM_PORT)
public class ExampleTest {

@Autowired
private Processor processor;

@Autowired
private MessageCollector messageCollector;

@Test
@SuppressWarnings("unchecked")
public void testWiring() {
Message<String> message = new GenericMessage<>("hello");
processor.input().send(message);
Message<String> received = (Message<String>) messageCollector.forChannel(processor.output()).poll();
assertThat(received.getPayload(), equalTo("hello world"));
}

@SpringBootApplication
@EnableBinding(Processor.class)
public static class MyProcessor {

@Autowired
private Processor channels;

@Transformer(inputChannel = Processor.INPUT, outputChannel = Processor.OUTPUT)
public String transform(String in) {
return in + " world";
}
}
}
```

In the preceding example, we create an application that has an input channel and an output channel, both bound through the  `Processor`  interface. The bound interface is injected into the test so that we can have access to both channels. We send a message on the input channel, and we use the  `MessageCollector`  provided by Spring Cloud Stream’s test support to capture that the message has been sent to the output channel as a result. Once we have received the message, we can validate that the component functions correctly.

## 33.1 Disabling the Test Binder Autoconfiguration

The intent behind the test binder superseding all the other binders on the classpath is to make it easy to test your applications without making changes to your production dependencies. In some cases (for example, integration tests) it is useful to use the actual production binders instead, and that requires disabling the test binder autoconfiguration. To do so, you can exclude the  `org.springframework.cloud.stream.test.binder.TestSupportBinderAutoConfiguration`  class by using one of the Spring Boot autoconfiguration exclusion mechanisms, as shown in the following example:

```java
@SpringBootApplication(exclude = TestSupportBinderAutoConfiguration.class)
@EnableBinding(Processor.class)
public static class MyProcessor {

@Transformer(inputChannel = Processor.INPUT, outputChannel = Processor.OUTPUT)
public String transform(String in) {
return in + " world";
}
}
```

When autoconfiguration is disabled, the test binder is available on the classpath, and its  `defaultCandidate`  property is set to  `false`  so that it does not interfere with the regular user configuration. It can be referenced under the name,  `test` , as shown in the following example:

`spring.cloud.stream.defaultBinder=test` 

