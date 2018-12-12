## 33.测试

Spring Cloud Stream支持在不连接消息传递系统的情况下测试您的微服务应用程序.您可以使用 `spring-cloud-stream-test-support` 库提供的 `TestSupportBinder` 来执行此操作，该库可以作为测试依赖项添加到应用程序中，如以下示例所示：

```xml
<dependency>
<groupId>org.springframework.cloud</groupId>
<artifactId>spring-cloud-stream-test-support</artifactId>
<scope>test</scope>
</dependency>
```

>   `TestSupportBinder` 使用Spring Boot自动配置机制取代在类路径上找到的其他Binders.因此，在添加Binders作为依赖项时，必须确保正在使用 `test` 范围.

`TestSupportBinder` 允许您与绑定的通道进行交互，并检查应用程序发送和接收的任何消息.

对于出站消息通道， `TestSupportBinder` 注册单个订户并保留应用程序在 `MessageCollector` 中发出的消息.它们可以在测试期间检索并对它们进行断言.

您还可以将消息发送到入站消息通道，以便使用者应用程序可以使用消息.以下示例显示如何在处理器上测试输入和输出通道：

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

在前面的示例中，我们创建了一个具有输入通道和输出通道的应用程序，它们都通过 `Processor` 接口绑定.绑定接口被注入到测试中，以便我们可以访问两个通道.我们在输入通道上发送消息，我们使用Spring Cloud Stream的测试支持提供的 `MessageCollector` 来捕获消息已经发送到输出通道的结果.收到消息后，我们可以验证组件是否正常运行.

## 33.1禁用测试Binders自动配置

测试绑定程序取代类路径上所有其他绑定程序的目的是使测试应用程序变得容易，而无需更改生产环境依赖项.在某些情况下（例如，集成测试），使用实际的生产环境Binders是有用的，这需要禁用测试Binders自动配置.为此，您可以使用Spring Boot自动配置排除机制之一排除 `org.springframework.cloud.stream.test.binder.TestSupportBinderAutoConfiguration` 类，如以下示例所示：

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

禁用自动配置时，测试Binders在类路径上可用，其 `defaultCandidate` 属性设置为 `false` ，因此它不会干扰常规用户配置.它可以在名称 `test` 下引用，如以下示例所示：

`spring.cloud.stream.defaultBinder=test` 

